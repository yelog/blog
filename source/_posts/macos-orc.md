---
title: 在 macOS 上做 OCR：从截屏到可点词的实践笔记
enlink: macos-orc
date: 2026-01-24 15:25:16
categories:
- 大前端
tags:
- macos
- swift
- ocr
---

# 在 macOS 上做 OCR：从截屏到可点词的实践笔记

最近在写一个 macOS 平台的快速翻译软件 SnapTra Translator，核心体验是“按住快捷键，把鼠标悬停在文字上就能看到翻译气泡”。这背后离不开 OCR：先把屏幕上一小块区域截下来，再用 Vision 把文字识别出来，最后根据光标位置选中最接近的单词。

这篇文章把我在项目里的实践整理成一份“可落地”的 macOS OCR 指南：既讲思路，也给关键代码和避坑点。

## 你能得到什么

- 一个 macOS OCR 的完整链路：截屏 → 识别 → 选词 → 调试
- 可直接复用的关键代码段（Vision + ScreenCaptureKit）
- 真实项目里的设计取舍与坑位

## 为什么选择 Vision + ScreenCaptureKit

在 macOS 上做 OCR，本质是：

1. 拿到屏幕图像（需要屏幕录制权限）
2. 把图像喂给 Vision 的文本识别
3. 根据识别结果的 bounding box 做交互（比如“光标附近取词”）

Vision 是系统级 OCR，稳定、轻量、无需额外模型；ScreenCaptureKit 是更现代的截屏方式，能更好控制采样区域与性能。

## SnapTra 的 OCR 链路概览

我在 SnapTra Translator 里采用了“光标周围小范围截屏”的策略：

- 每次翻译只截取鼠标附近一块固定大小区域（默认 520x140）
- OCR 结果返回每行文本，再进一步切成更细的单词 token
- 将 token 的 bounding box 与鼠标位置比对，选中最接近的词

这套链路让它能做到“点哪翻哪”，而不是整屏 OCR。

## 关键实现：截屏

下面是项目里截取光标周围画面的核心逻辑，使用 `ScreenCaptureKit` 获取一张 `CGImage`：

```swift
final class ScreenCaptureService {
    let captureSize = CGSize(width: 520, height: 140)

    func captureAroundCursor() async -> (image: CGImage, region: CaptureRegion)? {
        let mouseLocation = NSEvent.mouseLocation
        guard let screen = NSScreen.screens.first(where: { NSMouseInRect(mouseLocation, $0.frame, false) }) else {
            return nil
        }
        let rectInScreen = captureRect(for: mouseLocation, in: screen.frame, size: captureSize)
        let cgRect = convertToDisplayLocalCoordinates(rectInScreen, screen: screen)

        let display = try await getDisplay(for: displayID)
        let filter = SCContentFilter(display: display, excludingWindows: [])
        let configuration = makeConfiguration(for: cgRect, scaleFactor: screen.backingScaleFactor)
        let image = try await SCScreenshotManager.captureImage(contentFilter: filter, configuration: configuration)
        return (image, CaptureRegion(rect: rectInScreen, screen: screen, displayID: displayID, scaleFactor: scaleFactor))
    }
}
```

这段代码的重点是：只抓一个小矩形区域，不做整屏截图，性能会好很多。

## 关键实现：OCR 识别

Vision 的识别部分非常直接：

```swift
final class OCRService {
    func recognizeWords(in image: CGImage, language: String) async throws -> [RecognizedWord] {
        try await Task.detached(priority: .userInitiated) {
            let request = VNRecognizeTextRequest()
            request.recognitionLevel = .accurate
            request.usesLanguageCorrection = true
            if #available(macOS 13.0, *) {
                request.revision = VNRecognizeTextRequestRevision3
                request.automaticallyDetectsLanguage = true
            } else {
                request.recognitionLanguages = [language]
            }

            let handler = VNImageRequestHandler(cgImage: image)
            try handler.perform([request])
            return OCRService.extractWords(from: request.results ?? [])
        }.value
    }
}
```

几个小点：

- `recognitionLevel = .accurate` 适合需要高精度的翻译场景
- 开启 `usesLanguageCorrection` 能提升英文识别的可读性
- 在 macOS 13+ 用 `automaticallyDetectsLanguage`，可以更自然地识别混合文本

## 关键实现：从行到“词”

Vision 返回的通常是“文本行”，但翻译场景需要更细粒度。

SnapTra 里我做了两步：

1. 先按英文字符拆分 token
2. 对 CamelCase 做进一步切分（比如 `ApplePay` → `Apple` + `Pay`）

同时，我没有直接使用 Vision 的 `boundingBox(for:)`，因为它对自定义分词并不稳定，而是用“字符比例”计算 box，保证稳定性。

```swift
// 始终使用字符比例计算边界框，确保稳定性
// Vision 的 boundingBox(for:) 对自定义分词（CamelCase）支持不稳定
private static func boundingBoxByCharacterRatio(_ textBox: CGRect, text: String, for range: Range<String.Index>) -> CGRect? {
    let totalCount = text.count
    let startOffset = text.distance(from: text.startIndex, to: range.lowerBound)
    let endOffset = text.distance(from: text.startIndex, to: range.upperBound)
    let startFraction = CGFloat(startOffset) / CGFloat(totalCount)
    let endFraction = CGFloat(endOffset) / CGFloat(totalCount)
    let x = textBox.minX + textBox.width * startFraction
    let width = textBox.width * (endFraction - startFraction)
    return CGRect(x: x, y: textBox.minY, width: width, height: textBox.height)
}
```

## 如何“点词”：用鼠标位置选中最相关的词

OCR 的输出是多个单词加 bounding box。为了实现“鼠标指哪翻哪”，我做了两件事：

- 只保留 bounding box 包含鼠标位置的词
- 如果有多个候选，取中心点离鼠标最近的

这样可以在密集文本里也保持稳定体验。

## 权限与系统设置：屏幕录制是前置条件

只要涉及屏幕截图，就需要 Screen Recording 权限。项目里通过 `CGPreflightScreenCaptureAccess()` 判断当前权限，并提供快捷跳转系统设置：

```swift
func requestAndOpenScreenRecording() {
    CGRequestScreenCaptureAccess()
    openPrivacyPane(anchor: "Privacy_ScreenCapture")
}
```

体验上，我会在权限状态变化后自动刷新，并在未授权时提示用户。

## 调试手段：把 OCR 区域画出来

“看不到”是 OCR 调试最大的阻力。SnapTra 里做了一个 Debug OCR Region 开关：

- 红框显示当前截屏区域
- 绿框显示每个识别出来的单词边界

这可以直观观察“截屏区域是否对齐”、“识别结果是否偏移”，非常推荐在早期就加上。

## 常见坑位与建议

1. 不要整屏 OCR：性能会很差，体验会卡
2. 识别结果的坐标系是归一化坐标，转到屏幕坐标要注意 y 轴翻转
3. 语言设置不要死板：混合语言场景很常见
4. 分词策略比你想象的重要：翻译工具对词粒度很敏感

## 落地小结

如果你要做一个“屏幕取词 + 翻译”的 macOS 工具，推荐的最小链路是：

1. 用 ScreenCaptureKit 截取鼠标附近小区域
2. 用 Vision 识别文本
3. 自己做分词与 box 细化
4. 用鼠标位置选词
5. 加一个 Debug OCR Region 视图，调试效率直接翻倍

## 最后

我在 SnapTra Translator 里踩过的坑、做过的取舍，都尽量写在上面了。OCR 看似简单，但真正落地到“手感很好”的产品，细节真的很多。

如果你也在做 macOS OCR 或翻译工具，欢迎交流想法。

