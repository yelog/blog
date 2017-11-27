---
title: 解决粘贴到vim缩进错乱问题
permalink: vim-paste
date: 2017-06-01 19:45:24
categories:
- 运维
tags:
- vim
---
## 遇见
当我使用vim，想要粘贴下面这段脚本到 xx.sh 文件中
```bash
#!/bin/bash
if [ $1 ]
then
    if [ $1 == "help" ]; then
        echo -e "\033[37m pay 参数1 [参数2] \033[0m"
    else
        if [ $2 ]; then
            filename = $2
        fi
    fi
else
    echo -e "\033[37m 缺少关键词，通过'pay help'查看帮助信息 \033[0m"
fi
```
却出现了错乱,如下图所示
![缩进乱掉](http://oncj6b2vl.bkt.clouddn.com/FoIefs-sorNDjfPD69gH05is3tc5.png)

## 分析
vim 没有相应的程序来处理这个从其他应用复制粘贴的过程，所以Vim通过插入键盘输入的buffer来模拟这个粘贴的过程，这个时候Vim会以为这是用户输入的。

所以问题是：当上一行结束，光标进入下一行时Vim会自动以上一行的的缩进为初始位置。这样就会破坏原始文件的缩进。

## 解决问题
经过一番google，发现vim提供了 `paste` 选项，进入 paste 模式后，就可以正常缩进了。
```bash
# 进入 paste 模式
:set paste

# 退出 paste 模式
:set nopaste
```
![缩进正常](http://oncj6b2vl.bkt.clouddn.com/FpHOECFUPpgkJBKhKE_wcRYvPJ81.png)

如果不想每次都执行这个命令，可以在 `～/.vimrc` 中添加一行配置 `set pastetoggle=<F12>` ，这样就可以通过F12快速在paste模式中切换。
