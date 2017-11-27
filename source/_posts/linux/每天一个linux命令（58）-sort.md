---
title: '每天一个linux命令（58）: sort'
permalink: 'linux-command（57）-sort'
date: 2017-05-23 11:44:12
categories:
- 运维
tags:
- linux命令
---
sort是在Linux里非常常用的一个命令，管排序的。

## 1.工作原理
sort将文件的每一行作为一个单位，相互比较，比较原则是从首字符向后，依次按ASCII码值进行比较，最后将他们按升序输出。

```bash
$ cat seq.txt
apple
allow
photo
bowl
peal
check
cheese

$ sort seq.txt
allow
apple
bowl
check
cheese
peal
photo
```
## 2.sort的-u选项
它的作用很简单，就是在输出行中去除重复行。
## 3.sort的-r选项
sort默认的排序方式是升序，如果想改成降序，就加个-r就搞定了。
## 4.sort的-o选项
由于sort默认是把结果输出到标准输出，所以需要用重定向才能将结果写入文件，形如sort filename > newfile。

但是，如果你想把排序结果输出到原文件中，用重定向可就不行了。

```bash
[rocrocket@rocrocket programming]$ sort -r number.txt > number.txt
[rocrocket@rocrocket programming]$ cat number.txt
[rocrocket@rocrocket programming]$
```
看，竟然将number清空了。

就在这个时候，-o选项出现了，它成功的解决了这个问题，让你放心的将结果写入原文件。这或许也是-o比重定向的唯一优势所在。

## 5.sort的-n选项
你有没有遇到过10比2小的情况。我反正遇到过。出现这种情况是由于排序程序将这些数字按字符来排序了，排序程序会先比较1和2，显然1小，所以就将10放在2前面喽。这也是sort的一贯作风。

我们如果想改变这种现状，就要使用-n选项，来告诉sort，“要以数值来排序”！

## 6.sort的-t选项和-k选项
如果有一个文件的内容是这样：
```bash
[rocrocket@rocrocket programming]$ cat facebook.txt
banana:30:5.5
apple:10:2.5
pear:90:2.3
orange:20:3.4
```
这个文件有三列，列与列之间用冒号隔开了，第一列表示水果类型，第二列表示水果数量，第三列表示水果价格。

那么我想以水果数量来排序，也就是以第二列来排序，如何利用sort实现？

幸好，sort提供了-t选项，后面可以设定间隔符。（是不是想起了cut和paste的-d选项，共鸣～～）

指定了间隔符之后，就可以用-k来指定列数了。
```bash
[rocrocket@rocrocket programming]$ sort -n -k 2 -t : facebook.txt
apple:10:2.5
orange:20:3.4
banana:30:5.5
pear:90:2.3
```
我们使用冒号作为间隔符，并针对第二列来进行数值升序排序，结果很令人满意。
## 7.其他的sort常用选项
-f会将小写字母都转换为大写字母来进行比较，亦即忽略大小写

-c会检查文件是否已排好序，如果乱序，则输出第一个乱序的行的相关信息，最后返回1

-C会检查文件是否已排好序，如果乱序，不输出内容，仅返回1

-M会以月份来排序，比如JAN小于FEB等等

-b会忽略每一行前面的所有空白部分，从第一个可见字符开始比较。
