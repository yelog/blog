---
title: deepin系统使用记录
enlink: deepin-linux
date: 2017-03-30 18:58:50
categories:
- 工具
- 软件记录
tags:
- deepin
- 运维
---
## 传送门
- [官网](https://www.deepin.org/)
- [论坛](https://bbs.deepin.org/)
- [deepin'wiki](https://wiki.deepin.org/)
- [Deepin应用列表](https://wiki.deepin.org/index.php?title=Deepin%E5%BA%94%E7%94%A8%E7%AE%A1%E7%90%86)

## 输入法
安装输入法，除了商店下载（好多输入法没有被收录进deepin商店），可以使用fcitx安装。
如安装google拼音输入法：
```bash
$ sudo aptitude install fcitx fcitx-googlepinyin
```
如果当前在使用ibus，而不是fcitx的话，看下面
1）安装fcitx，并安装google拼音
```bash
$ sudo apt-get install fcitx fcitx-googlepinyin im-config
```
2）打开输入法配置
```bash
$ im-config
```
依次：`ok`->`yes`,选择fcitx为默认输入法框架,`ok`->`ok`

## 制作启动器图标
以创建 `atom` 这款编辑器的启动器图标为例。
1）进入 `/usr/share/applications/` 目录，创建 `atom.desktop` 文件
2）编辑 `atom.desktop` 文件
```xml
[Desktop Entry]
Name=Atom
Comment=A hackable text editor for the 21st century
Exec=/opt/atom/atom %F
Icon=/opt/atom/atom.png
Type=Application
StartupNotify=true
Categories=TextEditor;Development;Utility;
MimeType=text/plain;
```
>解释：
`Name`：创建的图标名称
`Comment`：备注，随便填
`Exec`：启动文件的位置
`Icon`：图标位置
`Type`：类型，启动程序就填Application
`StartupNotify`: 启动通知，填true就行了。详细可查 [Startup notification](https://developer.gnome.org/integration-guide/stable/startup-notification.html.en)
`Categories`： 分类，随便填，比如：Application;
`MimeType`： 打开文件类型

## 修改apt源
修改 `/etc/apt/sources.list`
默认的源
```bash
deb [by-hash=force] http://packages.deepin.com/deepin/ unstable main contrib non-free
```
阿里云的源
```bash
deb [by-hash=force] http://mirrors.aliyun.com/deepin unstable main contrib non-free
```


## 更换文件管理器
### Nautilus
1) 深度商店下载安装 Nautilus
2）卸载深度任务管理器
```bash
$ sudo apt remove dde-file-manager
```
Nautilus 常用的快捷键

| 快捷键 | 作用 |
| :- | :- |
| F2 | 重命名  |
| Ctrl + 1 | 图标视图 |
| Ctrl + 2 | 列表视图 |
| Ctrl + T | 新建标签页 |
| Ctrl + W | 关闭标签页 |
| Alt + 数字 | 切换到指定标签页 |
| Ctrl + D | 收藏到当前文件夹到书签 |
| Shift + F10 | 打开鼠标右键菜单 |
| Alt + 左方向键 | 后退 |
| Alt + 右方向键 | 前进 |
| Ctrl + Q | 退出 |
