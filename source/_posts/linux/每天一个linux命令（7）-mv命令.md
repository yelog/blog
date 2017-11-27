---
title: '每天一个linux命令（7）: mv'
permalink: 'linux-command（7）-mv'
date: 2016-12-07 15:55:20
categories:
- 运维
tags:
- linux命令
---
　　mv命令是move的缩写，可以用来移动文件或者将文件改名（move (rename) files），是Linux系统下常用的命令，经常用来备份文件或者目录
<!--more -->
### 命令格式
```bash
$ mv [选项] 源文件或目录 目标文件或目录
```
### 命令功能
　　视mv命令中第二个参数类型的不同（是目标文件还是目标目录），mv命令将文件重命名或将其移至一个新的目录中。当第二个参数类型是文件时，mv命令完成文件重命名，此时，源文件只能有一个（也可以是源目录名），它将所给的源文件或目录重命名为给定的目标文件名。当第二个参数是已存在的目录名称时，源文件或目录参数可以有多个，mv命令将各参数指定的源文件均移至目标目录中。在跨文件系统移动文件时，mv先拷贝，再将原有文件删除，而链至该文件的链接也将丢失。
### 命令参数
| 参数 | 描述     |
| :------------- | :------------- |
| -b      | 若需覆盖文件，则覆盖前先行备份。     |
| -f     | force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖     |
| -i      | 若目标文件 (destination) 已经存在时，就会询问是否覆盖！     |
| -u      | 若目标文件已经存在，且 source 比较新，才会更新(update)     |
|  -t       | --target-directory=DIRECTORY move all SOURCE arguments into DIRECTORY，即指定mv的目标目录，该选项适用于移动多个源文件到一个目录的情况，此时目标目录在前，源文件在后。     |
### 命令实例
**`例一`：文件改名**
```bash
$ mv test.txt test1.txt
```
**`例二`：移动文件**
```bash
#将文件test.txt 移动到/usr/doc目录下
$ mv test.txt /usr/doc
```
**`例三`：将文件log1.txt,log2.txt,log3.txt移动到目录/usr/doc中**
```bash
$ mv log1.txt log2.txt log3.txt /usr/doc
$ mv -t /usr/doc log1.txt log2.txt log3.txt
```
**`例四`：将文件file1改名为file2，如果file2已经存在，则询问是否覆盖**
```bash
$ mv -i log1.txt log2.txt
```
**`例五`：将文件file1改名为file2，即使file2存在，也是直接覆盖掉**
```bash
$ mv -f log3.txt log2.txt
```
**`例六`：目录的移动**
```bash
#将doc下的product目录移动到/usr/doc目录下
$ mv doc/product /usr/doc
```
**`例七`：移动当前文件夹下的所有文件到上一级目录**
```bash
$ mv * ../
```
**`例八`：文件被覆盖前做简单备份，前面加参数-b**
```bash
$ mv log1.txt -b log2.txt
```
> -b不接受参数，mv会去读取环境变量VERSION_CONTROL来作为备份策略。
--backup该选项指定如果目标文件存在时的动作，共有四种备份策略：
1. CONTROL=none或off : 不备份。
2. CONTROL=numbered或t：数字编号的备份
3. CONTROL=existing或nil：如果存在以数字编号的备份，则继续编号备份m+1...n：执行mv操作前已存在以数字编号的文件log2.txt.~1~，那么再次执行将产生log2.txt~2~，以次类推。如果之前没有以数字编号的文件，则使用下面讲到的简单备份。
4. CONTROL=simple或never：使用简单备份：在被覆盖前进行了简单备份，简单备份只能有一份，再次被覆盖时，简单备份也会被覆盖。
