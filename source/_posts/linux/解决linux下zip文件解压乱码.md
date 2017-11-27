---
title: 解决linux下zip文件解压乱码
permalink: 解决linux下zip文件解压乱码
date: 2017-04-25 09:10:40
categories:
- 运维
tags:
- linux命令
---
## 原因
由于zip格式并没有指定编码格式，Windows下生成的zip文件中的编码是GBK/GB2312等，因此，导致这些zip文件在Linux下解压时出现乱码问题，因为Linux下的默认编码是UTF8。

## 解决方案
使用7z解压。

1. 安装p7zip和convmv
```bash
# fedora
$ su -c 'yum install p7zip convmv'
# ubuntu
$ sudo apt-get install p7zip convmv
```

2. 执行一下命令解压缩
```bash
# 使用7z解压缩
$ LANG=C 7za x your-zip-file.zip
# 递归转码
$ convmv -f GBK -t utf8 --notest -r .
```
