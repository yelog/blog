---
title: '字符串规范化(NFC/NFD)问题'
enlink: string-normalize-nfc-nfd
date: 2024-09-30 11:08:31
categories:
- 大前端
tags:
- javascript
- java
- encoding
---

# 前言

最近同事在接入西班牙语的数据时, 发现了一个问题, 涉及到西班牙语中包含重音符号的数据比对出了问题, 同事已经找出可能是字符串规范化(`Normalize`)的问题, 但是现象很奇怪, 今天就做了一些测试, 将测试结果记录下来. 以供大家参考.

常用规范化:
- NFC: 规范化组合型, 如 `é` 是一个字符
- NFD: 规范化分解型, 如 `é` 是两个字符 `e` 和 `´`
- NFKC: 兼容性规范化组合型
- NFKD: 兼容性规范化分解型

# 先说结论

在 HTML/JavasCript/Java/PostGreSQL 中, 不会自动对字符串的规范话进行转换, 也就是说, 从前端(html/js)传递到后端(java), 再传递到数据库(PostGreSQL)的过程中, 字符串的规范化是不变的, 所以只是文字传递, 不会出现问题.

出问题的地方是文件系统:
- Windows 文件系统（如 NTFS）通常使用 `NFC` 形式存储文件名。
- macOS 文件系统（如 HFS+ 或 APFS）通常使用 `NFD` 形式存储文件名。
- Linux 文件系统（如 ext4）通常使用 `NFC`，但这也可能因环境和设置而异。

导致上传文件时, 文件名的规范化不一致, 会导致文件名比对是不一致的.

# 问题现象

同名的文件名上传时, 会进入数据库比对一下文件名是否存在, 但是由于文件名的规范化不一致, 会导致文件名比对不一致. 从而重复上传文件.

比如我们做一个简单的测试, 如下代码, 页面上有两个元素, 一个文本输入框, 一个文件上传框, 当文件上传框选择文件后, 会比对文件名和文本输入框的值是否一致, 如果不一致, 则提示文件名不一致.

我们在本地新建一个文件, 文件名为 `é.txt`, 这个文件名的文本是 `NFC` 的形式, 当 `MacOS` 去页面上传文件, 会提示文件名不一致, windows 则提示文件名一致.

这是因为 `MacOS` 文件系统使用 `NFD` 形式存储文件名, 和文本输入框的值(`NFC`)不一致, 导致比对不一致.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title></title>
</head>
<body>
<input type="text" name="filename" value="é.txt" id='filename'>
<input type="file" multiple accept="*/*" onchange="previewFiles()" id="fileInput">
</body>
<script>
    // 监听事件,
    const previewFiles = (e) => {
        const files = document.querySelector('#fileInput').files;
        const filename = document.querySelector('#filename').value;
        console.log('filename normalize:', detectNormalizationForm(filename));
        console.log('file.name normalize:', detectNormalizationForm(files[0].name));
        // 比对 id 为 filename 的值 和 上传的文件名是否一致
        if (files.length === 0 || files[0].name !== filename) {
            alert('文件名不一致');
            return;
        } else {
            alert('文件名一致');
        }
    }

    function detectNormalizationForm(str) {
        if (str === str.normalize('NFC')) {
            return 'NFC';
        } else if (str === str.normalize('NFD')) {
            return 'NFD';
        } else if (str === str.normalize('NFKC')) {
            return 'NFKC';
        } else if (str === str.normalize('NFKD')) {
            return 'NFKD';
        } else {
            return 'Unknown';  // 如果没有匹配的规范化形式
        }
    }
</script>
</html>
```

# 问题解决

## 前端

前端可以在 `axios`/`ajax` 等统一请求的地方, 对请求的参数进行规范化, 保证传递的参数是 `NFC` 形式.

在前端处理是有局限性的, 比如上传文件时, 无法对文件内容进行处理

## 后端

后端也有几个地方需要处理
1. 拦截 `Controller`, `HandlerInterceptor` 等请求处理的地方, 对请求参数进行规范化
2. `Excel` 工具类, 解析成对象的地方, 对 `Excel` 中的字符串进行规范化(要求所有上传的 `Excel` 都使用这个方法进行解析)

## 数据库

可以在 Mybatis 拦截器中, 对所有的 `SQL` 进行规范化, 保证数据库中的数据都是 `NFC` 形式.

# 最后

> 如无必要, 勿增实体.

字符串作为系统中最常用的类型, 全部添加规范化会对系统的性能产生一定的影响. 系统如果涉及到重音符号的地方不多, 可以只在必要的地方进行规范化.

