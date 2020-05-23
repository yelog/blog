---
title: 3-hexo快捷键说明
date: 2017-03-24 16:45:31
permalink: 3-hexo-shortcuts
categories:
- 工具
- hexo
tags:
- hexo
- 3-hexo
---
今日公司断网了半个小时，就利用这段时间给主题添加了快捷键操作，方便使用。

快捷键为vim风格的。按键可能与vimium（chrome插件）的快捷键有冲突，插件设置屏蔽掉此站的快捷键即可

如果有比较好的建议，欢迎骚扰。
## 说明

### 全局

| Key       | Descption         |
| :-------- | :---------------- |
| s/S       | 全屏/取消全屏     |
| w/W       | 打开/关闭文章目录 |
| i/I       | 获取搜索框焦点    |
| j/J       | 向下滑动          |
| k/K       | 向上滑动          |
| gg/GG     | 到最顶端          |
| shift+G/g | 到最下端          |



### 搜索框

| Key | Descption |
| :- | :- |
| ESC | 1.如果输入框有内容，清除内容<br>2.如果输入框无内容，失去焦点 |
| 下 | 向下选择文章 |
| 上 | 向上选择文章 |
| 回车 | 打开当前选中的文章，若没有，则默认打开第一个 |

### 关闭快捷键

在主题下 `_config.yml` 中 找到 `shortcutKey` 设为 `false`

```yaml
shortcutKey: false
```

