---
title: '每天一个linux命令（10）: cat'
enlink: 'linux-command-10-cat'
date: 2016-12-10 10:52:50
categories:
- 运维
tags:
- linux命令
---
　　cat命令的用途是连接文件或标准输入并打印。这个命令常用来显示文件内容，或者将几个文件连接起来显示，或者从标准输入读取内容并显示，它常与重定向符号配合使用。
<!--more -->
### 命令格式
```bash
$ cat [选项] [文件]...
```
### 命令功能
**cat主要有三大功能**
1. 一次显示整个文件:`cat filename`
2. 从键盘创建一个文件:`cat > filename` 只能创建新文件,不能编辑已有文件.
3. 将几个文件合并为一个文件:`cat file1 file2 > file`

### 命令参数
| 参数 | 描述     |
| :------------- | :------------- |
| -A, --show-all | 等价于 -vET |
| -b, --number-nonblank | 对非空输出行编号 |
| -e | 等价于 -vE |
| -E, --show-ends | 在每行结束处显示 $ |
| -n, --number | 对输出的所有行编号,由1开始对所有输出的行数编号 |
| -s, --squeeze-blank | 有连续两行以上的空白行，就代换为一行的空白行 |
| -t | 与 -vT 等价 |
| -T, --show-tabs | 将跳格字符显示为 ^I |
| -u | (被忽略) |
| -v, --show-nonprinting | 使用 ^ 和 M- 引用，除了 LFD 和 TAB 之外 |

### 命令实例
**`例一`：把 log2012.log 的文件内容加上行号后输入 log2013.log 这个文件里**
```bash
$ cat -n log2012.log log2013.log
```
**`例二`：把 log2012.log 和 log2013.log 的文件内容加上行号（空白行不加）之后将内容附加到 log.log 里**
```bash
$ cat -b log2012.log log2013.log log.log
```
**`例三`：把 log2012.log 的文件内容加上行号后输入 log.log 这个文件里**
```bash
$ cat -n log2012.log > log.log
```
**`例四`：使用here doc来生成文件**
```bash
$ cat >log.txt <<EOF
> Hello
> World
> Linux
> PWD=$(pwd)
> EOF
```
**`例五`：tac (反向列示)**
```bash
$ tac log.txt
PWD=/opt/soft/test
Linux
World
Hello
```
> tac 是将 cat 反写过来，所以他的功能就跟 cat 相反， cat 是由第一行到最后一行连续显示在萤幕上，而 tac 则是由最后一行到第一行反向在萤幕上显示出来！
