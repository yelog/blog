---
title: npm使用介绍
enlink: npm
date: 2017-03-16 20:09:48
categories:
- 大前端
tags:
- node
---
## What
npm（全称Node Package Manager，即node包管理器）是Node.js默认的、以JavaScript编写的软件包管理系统。
作者：艾萨克·施吕特（Isaac Z. Schlueter）

## 安装
npm 是随同node.js一起安装的，所以安装node.js即可。

## 使用
```bash
# 查看版本
$ npm -v

# 升级
$ sudo npm install npm -g

# 安装模块
$ npm install <Module Name>    #本地安装
# 本地安装：安装到./node_modules(命令运行目录)
$ npm install <Module Name> -g #全局安装
# 全局安装：放在 /usr/local 下或者你 node 的安装目录。

# 卸载模块
$ npm uninstall <Module Name>

# 更新模块
$ npm update <Module Name>

# 查看所有安装的模块
$ npm ls     #所有本地模块
$ npm ls -g  #所有全局模块

# 搜索模块
$ npm search <Module Name>

```
