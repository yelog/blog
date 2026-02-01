---
title: 从模板到实战：写一个 IntelliJ 平台插件（以 I18n Toolkit 为例）
enlink: create-intellij-plugin
date: 2026-02-01 18:24:21
categories:
- 后端
tags:
- java
- IntellijIDEA
- i18n
---

# 从模板到实战：写一个 IntelliJ 平台插件（以 I18n Toolkit 为例）

如果你想为 JetBrains IDE（IntelliJ IDEA、WebStorm、Rider 等）写一个插件，最省心的起点就是官方模板 `intellij-platform-plugin-template`。本文以开源插件 **I18n Toolkit** 为例，结合真实代码，讲清楚从克隆模板、跑起来，到实现核心功能的完整路径。

> 示例仓库：I18n Toolkit（开源）
> https://github.com/yelog/i18n-toolkit
>
> 模板仓库：IntelliJ Platform Plugin Template
> https://github.com/JetBrains/intellij-platform-plugin-template

---

## 一、准备环境

在插件开发中，IDE 版本、JDK 版本和 Gradle 版本强相关。I18n Toolkit 的约束来自项目配置：

- **JDK**：21
- **Gradle**：9.2.1（必须使用 `./gradlew`）
- **IntelliJ Platform**：2025.2.5

建议：**始终使用 Gradle Wrapper**，避免本机 Gradle 版本与项目不一致。

---

## 二、从模板开始：克隆 + 初始化

### 1. 克隆模板

```bash
git clone https://github.com/JetBrains/intellij-platform-plugin-template
cd intellij-platform-plugin-template
```

### 2. 初始化项目信息

你需要修改几个关键文件：

- `settings.gradle.kts`：项目名称
- `gradle.properties`：插件名、版本、ID、平台版本等
- `src/main/resources/META-INF/plugin.xml`：插件 ID、名称、描述、扩展点
- `README.md`：保留 `<!-- Plugin description -->` 标记区域

模板自带脚本也能一键替换变量，但手工改更直观。

### 3. 启动开发 IDE

```bash
./gradlew runIde
```

此命令会启动一个 **IDE 沙箱环境**，插件会自动加载在这个测试 IDE 中。

---

## 三、项目结构速览

插件主要结构是固定的：

```
src/
├── main/
│   ├── kotlin/…                 # Kotlin 源码
│   └── resources/
│       └── META-INF/plugin.xml  # 插件描述文件
└── test/                         # 测试
```

I18n Toolkit 的代码划分更细，按功能分包：

- `service/`：缓存与核心数据
- `scanner/`：翻译文件扫描
- `parser/`：多格式解析
- `completion/` / `annotator/` / `reference/`：补全、诊断、导航
- `searcheverywhere/` / `statusbar/`：搜索与状态栏

这样的结构非常适合插件类项目：入口清晰、扩展点映射明确。

---

## 四、核心功能如何实现：I18n Toolkit 代码拆解

下面从核心功能视角，拆解这款插件的实现方式，并补充关键类与关键流程的实现细节，方便你对 IntelliJ 平台 API 的落点有更清晰的映射。

### 0. 启动与更新链路：ProjectActivity + VFS 监听

插件运行期的“数据生命周期”由两条链路保证稳定：

- **启动初始化**：`I18nProjectActivity` 实现 `ProjectActivity`，在项目打开时初始化缓存；同时替换 `QuickJavaDoc` Action，确保 i18n key 悬停/快捷文档体验一致。
- **动态加载**：`I18nDynamicPluginListener` 支持插件动态加载/卸载，无需重启 IDE；加载时对所有已打开项目初始化缓存并刷新 UI。
- **VFS 变更监听**：`I18nFileListener` 基于 `AsyncFileListener` 监听创建/修改/删除/移动/复制事件，仅处理 i18n 翻译文件，触发 `I18nCacheService.invalidateFile()` → `refresh()` → `I18nUiRefresher.refresh()` 的刷新链路。

示例代码（启动与监听）：

```kotlin
class I18nProjectActivity : ProjectActivity {
    override suspend fun execute(project: Project) {
        I18nCacheService.getInstance(project).initialize()
        installQuickDocOverride()
    }
}

class I18nFileListener : AsyncFileListener {
    override fun prepareChange(events: MutableList<out VFileEvent>): AsyncFileListener.ChangeApplier? {
        val relevantEvents = events.filter { e ->
            val file = e.file ?: return@filter false
            e is VFileContentChangeEvent || e is VFileCreateEvent ||
                e is VFileDeleteEvent || e is VFileMoveEvent || e is VFileCopyEvent
        }.filter { e -> e.file?.let(I18nDirectoryScanner::isTranslationFile) == true }
        if (relevantEvents.isEmpty()) return null
        return object : AsyncFileListener.ChangeApplier {
            override fun afterVfsChange() {
                ProjectManager.getInstance().openProjects.forEach { project ->
                    relevantEvents.forEach { it.file?.let { f ->
                        I18nCacheService.getInstance(project).invalidateFile(f)
                    } }
                }
            }
        }
    }
}
```

### 1. 统一缓存中心：I18nCacheService

插件需要频繁获取翻译数据，**缓存服务**是第一优先级：

- 项目启动时初始化
- 扫描目录 → 解析文件 → 生成 TranslationData
- 提供 API：按 key 查找、按语言过滤、查找所有翻译
- 文件变更时刷新缓存并刷新 UI

这让补全、导航、搜索等功能都能基于同一份数据源。

实现细节补充：

- `initialize()` 用 `initialized` 标记避免重复初始化，实际核心逻辑在 `refresh()`。
- `refresh()` 包裹在 `ReadAction.compute` 中，确保 PSI 读取安全；同时维护 `keyToFiles` 方便后续 quick fix 快速定位。
- `TranslationData` 内部结构是 `key -> locale -> TranslationEntry`，并提供 `getTranslation()` 默认回退策略（`zh_CN` → `zh` → `en` → 首个可用值）。
- `getTranslationStrict()` 基于 `I18nLocaleUtils.buildLocaleCandidates()` 进行严格 locale 匹配，不做全局回退。

示例代码（缓存刷新核心流程）：

```kotlin
fun refresh() {
    val translationFiles = I18nDirectoryScanner.scanForTranslationFiles(project)
    keyToFiles.clear()
    val data = ReadAction.compute<TranslationData, RuntimeException> {
        val result = TranslationData(I18nFrameworkDetector.detect(project))
        translationFiles.forEach { file ->
            val pathInfo = I18nKeyGenerator.parseFilePath(file, project.basePath ?: "")
            val entries = TranslationFileParser.parse(project, file, pathInfo.keyPrefix, pathInfo.locale)
            entries.forEach { (key, entry) ->
                result.addEntry(entry)
                keyToFiles.getOrPut(key) { mutableSetOf() }.add(entry)
            }
        }
        result
    }
    translationData = data
}
```

### 2. 目录扫描：I18nDirectoryScanner

扫描逻辑有两个重点：

- 只扫描标准 i18n 目录：`locales` / `i18n` / `messages` / `lang` 等
- 排除目录：`node_modules`、`dist`、`build`、隐藏目录

这样可以有效避免无关文件的解析成本。

实现细节补充：

- `I18nDirectories.STANDARD_DIRS` 维护标准目录白名单；扫描使用 `VfsUtil.iterateChildrenRecursively`。
- 目录过滤逻辑同时跳过隐藏目录（以 `.` 开头）以及 `node_modules`、`dist`、`build`。
- 识别文件类型来自 `TranslationFileType`，支持 `json/yaml/yml/toml/js/mjs/cjs/ts/mts/cts/properties`。

示例代码（扫描与类型识别）：

```kotlin
object I18nDirectoryScanner {
    private val excludedDirNames = setOf("node_modules", "dist", "build")
    fun scanForTranslationFiles(project: Project): List<VirtualFile> {
        val baseDir = project.guessProjectDir() ?: return emptyList()
        val translationFiles = mutableListOf<VirtualFile>()
        findI18nDirectories(baseDir).forEach { dir ->
            VfsUtil.iterateChildrenRecursively(dir, ::shouldTraverse) { file ->
                val ext = file.extension?.lowercase()
                if (!file.isDirectory && ext in TranslationFileType.allExtensions()) {
                    translationFiles.add(file)
                }
                true
            }
        }
        return translationFiles
    }
}
```

### 3. 多格式解析：TranslationFileParser

插件支持：

- JSON / YAML / TOML
- Properties
- JS / TS（对象字面量）

解析方式是“**尽可能利用 PSI**”：

- JSON / JS / TS：走 PSI 结构，能拿到精确 offset
- YAML / TOML：用第三方 parser，offset 为估算值

这解释了为什么 YAML / TOML 的定位可能稍有偏差，但实际效果可接受。

实现细节补充：

- **JSON**：用 `JsonFile` / `JsonObject` 递归解析，`TranslationEntry.offset` 精确定位到 key 的 `textOffset`。
- **JS/TS**：解析 `export default`、变量声明与表达式语句，提取对象字面量中的字符串值。
- **YAML/TOML**：使用 SnakeYAML / toml4j 解析结构，offset 通过累加长度估算。
- **Properties**：按行扫描 `key=value`，过滤注释行并记录行内 offset。

示例代码（JSON / JS/TS 解析片段）：

```kotlin
private fun parseJsonObject(obj: JsonObject, prefix: String, locale: String, file: VirtualFile, out: MutableMap<String, TranslationEntry>) {
    obj.propertyList.forEach { prop ->
        val key = prop.name
        val fullKey = if (prefix.isEmpty()) key else "$prefix$key"
        when (val value = prop.value) {
            is JsonStringLiteral -> out[fullKey] = TranslationEntry(fullKey, value.value, locale, file, prop.nameElement.textOffset, prop.nameElement.textLength)
            is JsonObject -> parseJsonObject(value, "$fullKey.", locale, file, out)
        }
    }
}

private fun parseJsExpression(expr: JSExpression?, prefix: String, locale: String, file: VirtualFile, out: MutableMap<String, TranslationEntry>) {
    if (expr is JSObjectLiteralExpression) {
        expr.properties.forEach { prop ->
            val key = prop.name ?: return@forEach
            val fullKey = if (prefix.isEmpty()) key else "$prefix$key"
            val value = prop.value as? JSLiteralExpression
            value?.stringValue?.let { out[fullKey] = TranslationEntry(fullKey, it, locale, file, prop.textOffset, key.length) }
        }
    }
}
```

### 4. Key 前缀生成：I18nKeyGenerator

i18n 目录结构通常体现模块或业务层级，例如：

```
src/views/mes/locales/lang/zh_CN/order.ts
```

插件通过路径自动推导：

- locale：`zh_CN`
- keyPrefix：`mes.order.`

这样在代码里写 `t('create')`，也能正确定位到 `mes.order.create`。

实现细节补充：

- `parseFilePath()` 区分 **views 模式** 与 **标准模式**：views 模式会把业务单元与模块组合为前缀。
- 对 `message/messages` 目录进行特殊处理：避免把它作为 module 前缀（常见于 Spring Message）。
- 当路径里找不到 locale 时，会回退到文件名判断 locale。

示例代码（路径解析与前缀生成）：

```kotlin
fun parseFilePath(file: VirtualFile, projectBasePath: String): PathInfo {
    val relativePath = file.path.removePrefix(projectBasePath).removePrefix("/")
    val parts = relativePath.split("/")
    val fileName = file.nameWithoutExtension
    return when {
        isViewsLocalePattern(parts) -> parseViewsLocalePattern(parts, fileName)
        isStandardLocalePattern(parts) -> parseStandardLocalePattern(parts, fileName)
        else -> PathInfo(locale = extractLocale(parts, fileName), module = null, businessUnit = null, keyPrefix = "")
    }
}
```

### 5. 框架检测：I18nFrameworkDetector

插件会自动识别 i18n 框架：

- vue-i18n
- react-i18next
- next-intl
- @nuxtjs/i18n
- react-intl
- Spring Message（检测 `pom.xml` 或 `build.gradle`）

如果识别成功，可自动决定语义规则和函数习惯，减少配置。

实现细节补充：

- **Spring 检测**：直接读取 `pom.xml` / `build.gradle(.kts)` 文本，判断关键依赖字符串。
- **JS/TS 检测**：通过 PSI 解析 `package.json`，遍历 `dependencies / devDependencies / peerDependencies`。

示例代码（依赖检测）：

```kotlin
private fun parsePackageJson(project: Project, file: VirtualFile): I18nFramework {
    val psiFile = PsiManager.getInstance(project).findFile(file) as? JsonFile ?: return I18nFramework.UNKNOWN
    val rootObject = psiFile.topLevelValue as? JsonObject ?: return I18nFramework.UNKNOWN
    val deps = mutableSetOf<String>()
    listOf("dependencies", "devDependencies", "peerDependencies").forEach { depType ->
        (rootObject.findProperty(depType)?.value as? JsonObject)?.propertyList?.forEach { deps.add(it.name) }
    }
    return I18N_PACKAGES.firstOrNull { deps.contains(it) }?.let(I18nFramework::fromPackageName) ?: I18nFramework.UNKNOWN
}
```

### 6. Inlay 提示：I18nInlayHintsProvider

这类体验是插件“可感知度”最高的部分：

- 在 `t('key')` 后面显示翻译内容
- 支持 Vue template 的注入代码
- 缓存已处理位置，避免重复插入
- 可设置为“仅显示翻译”或“key + 翻译”模式

这套逻辑结合了 PSI + Inlay API + InjectedLanguageManager。

实现细节补充：

- `globalProcessedHints` 以 `filePath:modStamp:offset` 为 key 去重，避免多语言实例重复插入提示。
- Vue 模板插值中的 `{{ t('key') }}` 使用 `InjectedLanguageManager` 处理 injected PSI。
- 先用 `I18nNamespaceResolver.getFullKey()` 拼接命名空间，再做翻译匹配。
- 若显示语言存在但缺失该 key，会显示 `Missing translation for 'locale'` 的提示文案。

示例代码（Inlay 去重与渲染）：

```kotlin
val hintKey = "$filePath:$modStamp:$offset"
if (globalProcessedHints.putIfAbsent(hintKey, true) != null) return

val presentation = factory.roundWithBackground(
    factory.smallText(" → $translationText")
)
sink.addInlineElement(offset, true, presentation, false)
```

### 7. 缺失 Key 诊断 + 快速修复

缺失 key 会被标红，并提供一键创建：

- `I18nKeyAnnotator` 负责提示错误
- `CreateI18nKeyQuickFix` 根据 key 自动选择目标文件
- 支持 JSON / JS / TS / Properties 直接写入

尤其是“根据 key 前缀和兄弟 key 自动选择文件”的逻辑，极大提升了体验。

实现细节补充：

- `I18nKeyAnnotator` 使用 `I18nFunctionResolver` 获取可配置的 i18n 函数名（默认 `t/$t/i18n/translate/...`）。
- 只高亮字符串内容本身（排除引号），并挂载 `CreateI18nKeyQuickFix`。
- `CreateI18nKeyQuickFix` 先尝试 **最长前缀匹配**，失败后再用 **兄弟 key** 反推文件。
- 实际写入通过 `WriteCommandAction` + PSI 操作完成，写入后用 `OpenFileDescriptor` 定位并把光标放在引号之间。

示例代码（缺失 Key 高亮范围）：

```kotlin
val elementRange = literalExpr.textRange
val keyStartOffset = elementRange.startOffset + 1
val keyEndOffset = keyStartOffset + partialKey.length
val highlightRange = TextRange(keyStartOffset, keyEndOffset)
holder.newAnnotation(HighlightSeverity.ERROR, "Unresolved i18n key: '$fullKey'")
    .range(highlightRange)
    .textAttributes(DefaultLanguageHighlighterColors.INVALID_STRING_ESCAPE)
    .withFix(CreateI18nKeyQuickFix(fullKey))
    .create()
```

### 8. Search Everywhere 集成

插件在 Search Everywhere 中新增 **I18n** 标签：

- key 和翻译都支持模糊搜索
- Enter：复制 key
- Ctrl+Enter：跳转到翻译文件
- 结果排序有评分策略（前缀匹配 > 包含匹配）

这让翻译搜索真正成为“IDE 级别”的能力。

实现细节补充：

- 搜索结果以 “key + 多语言翻译” 合并为一个条目。
- 评分策略在 `calculateMatchScore()` 中实现：前缀匹配最高，其次是包含匹配与 value 匹配。
- Enter 复制 key，Ctrl+Enter 直接打开翻译文件，行为明确且稳定。

示例代码（搜索评分片段）：

```kotlin
private fun calculateMatchScore(key: String, entries: Collection<TranslationEntry>, tokens: List<String>, compactQuery: String): Int {
    val keyLower = key.lowercase()
    val keyMatchesAll = tokens.isNotEmpty() && tokens.all { keyLower.contains(it) }
    var score = 0
    if (keyMatchesAll) score += 100
    if (tokens.isNotEmpty() && keyLower.startsWith(tokens.first())) score += 1000
    return score
}
```

### 9. 状态栏语言切换 + 翻译编辑

- 状态栏小部件支持显示与切换当前语言
- 翻译弹窗支持多语言编辑，并可实时写入文件

这些 UI 功能用到 `StatusBarWidget` 和 `JBPopupFactory`，是典型插件 UI 技术。

实现细节补充：

- `I18nStatusBarWidget` 通过 `ListPopup` 提供语言列表与“Go to Settings”入口，切换语言后触发 UI refresh。
- `I18nTranslationEditPopup` 使用 `Alarm` 做 300ms 防抖，编辑后实时写回文件。
- `I18nTranslationWriter` 根据文件类型分别替换 JSON/JS/Properties 内容，并处理引号与转义。

示例代码（状态栏切换与写回）：

```kotlin
val step = object : BaseListPopupStep<PopupItem>("I18n Toolkit", allItems) {
    override fun onChosen(selectedValue: PopupItem?, finalChoice: Boolean): PopupStep<*>? {
        if (selectedValue is PopupItem.LocaleItem) {
            settings.state.displayLocale = selectedValue.locale
            I18nUiRefresher.refresh(project)
        }
        return super.onChosen(selectedValue, finalChoice)
    }
}

WriteCommandAction.runWriteCommandAction(project, "Update i18n Translation", null, Runnable {
    val document = FileDocumentManager.getInstance().getDocument(entry.file) ?: return@Runnable
    document.replaceString(valueStart, lineEnd, newValue)
})
```

---

## 五、插件功能如何挂载：plugin.xml 扩展点

所有功能都需要通过 `plugin.xml` 注册：

- `projectService`：缓存服务
- `inlayProvider`：内联提示
- `annotator`：错误提示
- `completion.contributor`：补全
- `psi.referenceContributor`：导航
- `searchEverywhereContributor`：搜索
- `statusBarWidgetFactory`：状态栏

这里是 IntelliJ 平台插件开发的核心：

> **一切功能都是“扩展点 + 实现类”的组合。**

---

## 六、构建、测试与发布

常用命令：

```bash
./gradlew runIde          # 本地运行
./gradlew buildPlugin     # 构建分发包
./gradlew test            # 运行测试
./gradlew check           # 测试 + 覆盖率
./gradlew verifyPlugin    # 插件兼容性验证
```

如果要发布到 JetBrains Marketplace，补齐签名配置即可。

---

## 七、总结：从模板到可用插件的关键路径

**模板给你骨架，真正的价值来自你的“功能设计”与“用户体验”。**

I18n Toolkit 的实现告诉我们：

- 缓存与解析是插件性能的核心
- IntelliJ 扩展点体系决定功能边界
- 体验要足够“IDE 级”，才能真正提升开发效率

如果你也想写一个生产力插件，不妨从这个开源项目入手，读一遍核心类，跑一遍 `runIde`，就能快速进入实战状态。

---

如果你对该插件感兴趣或想参与贡献：

- 模板项目：https://github.com/JetBrains/intellij-platform-plugin-template
- 插件仓库：https://github.com/yelog/i18n-toolkit

祝你玩得开心，写出属于自己的 JetBrains 插件！

