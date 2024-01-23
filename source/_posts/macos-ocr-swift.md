---
title: swift 离线图片识别文字(ocr)
enlink: macos-ocr-swift
date: 2024-01-02 10:09:19
categories:
- 开发
- swift
tags:
- swift
- macos
- ocr
---

## 背景

最近打算写一个 macos 翻译软件, 需要用到 ocr 图像识别, 并且因为速度问题, 一开始就考虑使用系统的自带能力来实现.

经过翻阅文档和 chatgpt 拉扯了一下午, 最终成功实现.


## 代码

代码逻辑为, 接受参数: 图片路径, 然后获取图片, 通过 `VNImageRequestHandler` 对图片进行文字识别

如下代码可以直接放进一个 `ocr.swift`, 然后执行 `swiftc -o ocr ocr.swift` , 在执行 `./ocr /Users/yelog/Desktop/3.png` 后面为你实际的有文字的图片路经

```swift
//
//  ocr.swift
//  Fast Translation
//
//  Created by 杨玉杰 on 2023/12/31.
//

import SwiftUI
import Vision


func handleDetectedText(request: VNRequest?, error: Error?) {
    if let error = error {
        print("ERROR: \(error)")
        return
    }
    guard let results = request?.results, results.count > 0 else {
        print("No text found")
        return
    }

    for result in results {
        if let observation = result as? VNRecognizedTextObservation {
            for text in observation.topCandidates(1) {
                let string = text.string
                print("识别: \(string)")
            }
        }
    }
}

func ocrImage(path: String) {
    let cgImage = NSImage(byReferencingFile: path)?.ciImage()?.cgImage

    let requestHandler = VNImageRequestHandler(cgImage: cgImage!)
    let request = VNRecognizeTextRequest(completionHandler: handleDetectedText)

    // 设置文本识别的语言为英文
    request.recognitionLanguages = ["en"]
    request.recognitionLevel = .accurate

    do {
        try requestHandler.perform([request])
    } catch {
        print("Unable to perform the requests: \(error).")
    }
}

extension NSImage {
    func ciImage() -> CIImage? {
        guard let data = self.tiffRepresentation,
              let bitmap = NSBitmapImageRep(data: data) else {
            return nil
        }
        let ci = CIImage(bitmapImageRep: bitmap)
        return ci
    }
}

// 执行函数，从命令行参数中获取图片的地址
ocrImage(path: CommandLine.arguments[1])

```

然后准备待识别的有文字的图片

![待识别的图片](https://cdn.jsdelivr.net/gh/yelog/assets/images/202401021353013.png) 

```bash
# 编译 swift 文件
swiftc -o ocr ocr.swift
# 执行并且传递图片路径参数
./ocr /Users/yelog/Desktop/3.png
```

![执行识别效果](https://cdn.jsdelivr.net/gh/yelog/assets/images/202401021354591.png)


## 最后

最近打算着手写一些 macos 的小工具, 如果对 `swift` 或者 `macos` 感兴趣的可以关注或评论.
