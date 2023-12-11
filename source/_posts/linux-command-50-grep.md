---
title: '每天一个linux命令（50）: grep'
enlink: 'linux-command-50-grep'
date: 2017-01-18 10:12:46
categories:
- 运维
tags:
- linux命令
---
　　Linux系统中grep命令是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹 配的行打印出来。grep全称是Global Regular Expression Print，表示全局正则表达式版本，它的使用权限是所有用户。
<!-- more -->
　　grep的工作方式是这样的，它在一个或多个文件中搜索字符串模板。如果模板包括空格，则必须被引用，模板后的所有字符串被看作文件名。搜索的结果被送到标准输出，不影响原文件内容。

　　grep可用于shell脚本，因为grep通过返回一个状态值来说明搜索的状态，如果模板搜索成功，则返回0，如果搜索不成功，则返回1，如果搜索的文件不存在，则返回2。我们利用这些返回值就可进行一些自动化的文本处理工作。

### 命令格式
```bash
$ grep [option] pattern file
```
### 命令功能
　　用于过滤/搜索的特定字符。可使用正则表达式能多种命令配合使用，使用上十分灵活。
### 命令参数
| 参数 | 描述 |
| :- | :- |
| -a <br>--text | 不要忽略二进制的数据 |
| -A<显示行数>  <br>--after-context=<显示行数> | 除了显示符合范本样式的那一列之外，并显示该行之后的内容 |
| -b <br>--byte-offset | 在显示符合样式的那一行之前，标示出该行第一个字符的编号 |
| -B<显示行数>  <br>--before-context=<显示行数> | 除了显示符合样式的那一行之外，并显示该行之前的内容 |
| -c    <br>--count | 计算符合样式的列数 |
| -C<显示行数>    <br>--context=<显示行数>或-<显示行数> | 显示上下文n行 |
| -d <动作>      <br>--directories=<动作> | 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep指令将回报信息并停止动作 |
| -e<范本样式>  <br>--regexp=<范本样式> | 指定字符串做为查找文件内容的样式 |
| -E      <br>--extended-regexp | 将样式为延伸的普通表示法来使用 |
| -f<规则文件>  <br>--file=<规则文件> | 指定规则文件，其内容含有一个或多个规则样式，让grep查找符合规则条件的文件内容，格式为每行一个规则样式 |
| -F   <br>--fixed-regexp | 将样式视为固定字符串的列表 |
| -G   <br>--basic-regexp | 样式视为普通的表示法来使用 |
| -h   <br>--no-filename | 在显示符合样式的那一行之前，不标示该行所属的文件名称 |
| -H   <br>--with-filename | 在显示符合样式的那一行之前，表示该行所属的文件名称 |
| -i    <br>--ignore-case | 忽略字符的大小写 |
| -l    <br>--file-with-matches | 只列出匹配的文件名 |
| -L   <br>--files-without-match | 列出不匹配的文件名 |
| -n   <br>--line-number | 显示行号 |
| -q   <br>--quiet或--silent | 不显示任何信息 |
| -r   <br>--recursive | 递归查询， 此参数的效果和指定“-d recurse”参数相同 |
| -s   <br>--no-messages | 不显示错误信息 |
| -v   <br>--revert-match | 显示不包含匹配文本的所有行 |
| -V   <br>--version | 显示版本信息 |
| -w   <br>--word-regexp | 只显示全字符合的列 |
| -x    <br>--line-regexp | 只显示全列符合的列 |
| -y | 此参数的效果和指定“-i”参数相同 |

### 规则表达式
**grep的规则表达式**
　　`^`  #锚定行的开始 如：`'^grep'`匹配所有以grep开头的行。    
　　`$`  #锚定行的结束 如：`'grep$'`匹配所有以grep结尾的行。    
　　`.`  #匹配一个非换行符的字符 如：`'gr.p'`匹配gr后接一个任意字符，然后是p。    
　　`*`  #匹配零个或多个先前字符 如：`'*grep'`匹配所有一个或多个空格后紧跟grep的行。    
　　`.*`   #一起用代表任意字符。   
　　`[]`   #匹配一个指定范围内的字符，如`'[Gg]rep'`匹配Grep和grep。    
　　`[^]`  #匹配一个不在指定范围内的字符，如：`'[^A-FH-Z]rep'`匹配不包含A-R和T-Z的一个字母开头，紧跟rep的行。    
　　`\(..\)`  #标记匹配字符，如`'\(love\)'`，love被标记为1。    
　　`\<`      #锚定单词的开始，如:`'\<grep'`匹配包含以grep开头的单词的行。    
　　`\>`      #锚定单词的结束，如`'grep\>'`匹配包含以grep结尾的单词的行。    
　　`x\{m\}`  #重复字符x，m次，如：`'o\{5\}'`匹配包含5个o的行。    
　　`x\{m,\}`  #重复字符x,至少m次，如：`'o\{5,\}'`匹配至少有5个o的行。    
　　`x\{m,n\}`  #重复字符x，至少m次，不多于n次，如：`'o\{5,10\}'`匹配5--10个o的行。   
　　`\w`    #匹配文字和数字字符，也就是[A-Za-z0-9]，如：`'G\w*p'`匹配以G后跟零个或多个文字或数字字符，然后是p。   
　　`\W`   #`\w`的反置形式，匹配一个或多个非单词字符，如点号句号等。   
　　`\b`   #单词锁定符，如: `'\bgrep\b'`只匹配grep。  
**POSIX字符**
　　为了在不同国家的字符编码中保持一至，POSIX(The Portable Operating System Interface)增加了特殊的字符类，如[:alnum:]是[A-Za-z0-9]的另一个写法。要把它们放到[]号内才能成为正则表达式，如`[A-Za-z0-9]`或`[[:alnum:]]`。在linux下的grep除fgrep外，都支持POSIX的字符类。
　　[:alnum:]    #文字数字字符   
　　[:alpha:]    #文字字符   
　　[:digit:]    #数字字符   
　　[:graph:]    #非空字符（非空格、控制字符）   
　　[:lower:]    #小写字符   
　　[:cntrl:]    #控制字符   
　　[:print:]    #非空字符（包括空格）   
　　[:punct:]    #标点符号   
　　[:space:]    #所有空白字符（新行，空格，制表符）   
　　[:upper:]    #大写字符   
　　[:xdigit:]   #十六进制数字（0-9，a-f，A-F）
### 使用实例
**`例一`：查找指定进程**
```bash
$ ps -ef|grep hexo
faker    13401 19030  0 09:51 pts/2    00:00:15 hexo
faker    15465 15449  0 10:34 pts/3    00:00:00 grep hexo
```
>**说明：**
第一条记录是查找出的进程；第二条结果是grep进程本身，并非真正要找的进程。

**`例二`：查找指定进程数**
```bash
$ ps -ef|grep hexo -c
$ ps -ef|grep -c hexo
2
```
**`例三`：从2.txt中读取关键词在1.txt中进行搜索**
```bash
# -n显示行号
$ cat 1.txt | grep -nf 2.txt
1:If you please draw me a sheep!
2:What!
```
**`例四`：从文件中查找关键词**
```bash
$ grep 'jump' 1.txt
I jumped to my feet,completely thunderstruck.
```
**`例五`：从多个文件中查找关键词**
```bash
$ grep 'jump' 1.txt 2.txt
1.txt:I jumped to my feet,completely thunderstruck.
2.txt:I jump
```
>**说明：**
多文件时，输出查询到的信息内容行时，会把文件的命名在行最前面输出并且加上":"作为标示符

**`例六`：grep不显示本身进程**
```bash
$ ps aux|grep \[s]sh
$ ps aux | grep ssh | grep -v "grep"
```
**`例七`：找出以u开头的行内容**
```bash
$ cat 1.txt |grep ^u
If you please draw me a sheep!
I jumped to my feet,completely thunderstruck.
```
**`例八`：输出非u开头的行内容**
```bash
$ cat 1.txt | grep ^[^I]
What!
Draw me a sheep!
```
**`例九`：输出以!结尾的行内容**
```bash
$ cat 1.txt |grep \!$
If you please draw me a sheep!
What!
Draw me a sheep!
```
**`例十`：显示包含sh或者at字符的内容行**
```bash
$ cat 1.txt |grep -E "sh|at"
If you please draw me a sheep!
What!
Draw me a sheep!
```
**`例十一`：显示当前目录下面以.txt 结尾的文件中的所有包含每个字符串至少有7个连续小写字符的字符串的行**
```bash
$ grep '[a-z]\{7\}' *.txt
1.txt:I jumped to my feet,completely thunderstruck.
3.txt:kdfkksjdf112123
4.txt:kisdfsf
```
