---
title: shell速查表
permalink: shell-command
date: 2018-09-08 11:48:24
categories: 运维
tags:
- shell
- linux
---
## 1. 变量
```bash
#!/bin/bash
msg="hello world"
echo $msg
```
>**变量名的命名须遵循如下规则：**
>- 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
>- 中间不能有空格，可以使用下划线（\_）。
>- 不能使用标点符号。
>- 不能使用bash里的关键字（可用help命令查看保留关键字）。

## 2. 传参
```bash
#!/bin/bash
echo "执行的文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
echo "第三个参数为：$3";
```
>**脚本内获取参数的格式为：**
>$n。n 代表一个数字，1 为执行脚本的第一个参数，2 为执行脚本的第二个参数，以此类推……
>**另外，还有几个特殊字符用来处理参数：**
>
>| 参数 | 说明 |
>| --- | --- |
>| `$#` | 传递到脚本的参数个数 |
>| `$*` | 以一个单字符串显示所有向脚本传递的参数。<br>如`"$*"`用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。 |
>| `$$` | 脚本运行的当前进程ID号 |
>| `$!` | 后台运行的最后一个进程的ID号 |
>| `$@` | 与`$*`相同，但是使用时加引号，并在引号中返回每个参数。<br>如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。 |
>| `$-` | 显示Shell使用的当前选项，与set命令功能相同。 |
>| `$?` | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。 |

## 3. 数组
```bash
#!/bin/bash
my_array=(A B "C" D)
echo "第一个元素为: ${my_array[0]}"
echo "第二个元素为: ${my_array[1]}"
echo "第三个元素为: ${my_array[2]}"
echo "第四个元素为: ${my_array[3]}"

echo "数组的元素为: ${my_array[*]}"
echo "数组的元素为: ${my_array[@]}"

echo "数组元素个数为: ${#my_array[*]}"
echo "数组元素个数为: ${#my_array[@]}"
```
执行结果如下：
```yml
第一个元素为: A
第二个元素为: B
第三个元素为: C
第四个元素为: D
数组的元素为: A B C D
数组的元素为: A B C D
数组元素个数为: 4
数组元素个数为: 4
```
## 4. 基本运算符
>原生 bash 不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用。

expr 是一款表达式计算工具，使用它能完成表达式的求值操作。
### ① 算数运算符
```bash
#!/bin/bash
echo "2加2等于"`expr 2 + 2`
echo "2减2等于"`expr 2 - 2`
echo "2乘2等于"`expr 2 \* 2`
echo "2除2等于"`expr 2 / 2`
echo "2除2取余"`expr 2 % 2`
```
### ② 关系运算符
```bash
#!/bin/bash
a=10
b=20
if [ $a -eq $b ] # 检测两个数是否相等，相等返回 true。
if [ $a -ne $b ] # 检测两个数是否不相等，不相等返回 true。
if [ $a -gt $b ] # 检测左边的数是否大于右边的，如果是，则返回 true。
if [ $a -lt $b ] # 检测左边的数是否小于右边的，如果是，则返回 true。
if [ $a -ge $b ] # 检测左边的数是否大于等于右边的，如果是，则返回 true。
if [ $a -le $b ] # 检测左边的数是否小于等于右边的，如果是，则返回 true。
```
### ③ 布尔运算符
```bash
#!/bin/bash
if [ ! false ]       # 非运算，返回 true
if [ true -o false ] # 或运算，返回 true
if [ true -a false ] # 与运算，返回 false
```

### ④ 逻辑运算符
```bash
#!/bin/bash
a=10
b=20
if [[ $a -lt $b && $a -gt $b ]]   # 逻辑的 AND, 返回 false
if [ $a -lt $b ] && [ $a -gt $b ] # 逻辑的 AND, 返回 false
if [[ $a -lt $b || $a -gt $b ]]   # 逻辑的 OR, 返回 true
if [ $a -lt $b ] || [ $a -gt $b ] # 逻辑的 OR, 返回 true
```

### ⑤ 字符串运算符
```bash
#!/bin/bash
a="abc"
b="efg"
if [ $a = $b ]   # 检测两个字符串是否相等，相等返回 true。
if [ $a != $b ]  # 检测两个字符串是否相等，不相等返回 true。
if [ -z $a ]     # 检测字符串长度是否为0，为0返回 true。
if [ -n "$a" ]   # 检测字符串长度是否为0，不为0返回 true。
if [ $a ]        # 检测字符串是否为空，不为空返回 true。
```

### ⑥ 文件测试运算符
文件测试运算符用于检测 Unix 文件的各种属性。

| 操作符 | 说明 |
| --- | --- |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。 |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。 |
| -r file | 检测文件是否可读，如果是，则返回 true。 |
| -w file | 检测文件是否可写，如果是，则返回 true。 |
| -x file | 检测文件是否可执行，如果是，则返回 true。 |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。 |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。 |

## 5. echo
### ① 命令格式
```bash
#!/bin/bash
echo "It is a test"
echo It is a test
echo "\"It is a test\""      # 转义
name=Chris
echo "$name is handsome"
echo -e "OK! \n"             # 显示换行 -e 开启转义
echo "It is a test" > myfile # 显示结果定向至文件
echo '$name\"'               # 原样输入字符串，不进行转义或取变量（使用单引号）
echo `date`                  # 显示命令执行结构
```
### ② 颜色显示
```bash
echo -e "\033[字背景颜色；文字颜色m字符串\033[0m"

echo -e “\033[30m 黑色字 \033[0m”
echo -e “\033[31m 红色字 \033[0m”
echo -e “\033[32m 绿色字 \033[0m”
echo -e “\033[33m 黄色字 \033[0m”
echo -e “\033[34m 蓝色字 \033[0m”
echo -e “\033[35m 紫色字 \033[0m”
echo -e “\033[36m 天蓝字 \033[0m”
echo -e “\033[37m 白色字 \033[0m”

echo -e “\033[40;37m 黑底白字 \033[0m”
echo -e “\033[41;37m 红底白字 \033[0m”
echo -e “\033[42;37m 绿底白字 \033[0m”
echo -e “\033[43;37m 黄底白字 \033[0m”
echo -e “\033[44;37m 蓝底白字 \033[0m”
echo -e “\033[45;37m 紫底白字 \033[0m”
echo -e “\033[46;37m 天蓝底白字 \033[0m”
echo -e “\033[47;30m 白底黑字 \033[0m”

\33[0m 关闭所有属性
\33[1m 设置高亮度
\33[4m 下划线
\33[5m 闪烁
\33[7m 反显
\33[8m 消隐
\33[30m — \33[37m 设置前景色
\33[40m — \33[47m 设置背景色
\33[nA 光标上移n行
\33[nB 光标下移n行
\33[nC 光标右移n行
\33[nD 光标左移n行
\33[y;xH设置光标位置
\33[2J 清屏
\33[K 清除从光标到行尾的内容
\33[s 保存光标位置
\33[u 恢复光标位置
\33[?25l 隐藏光标
\33[?25h 显示光标
```

## 6. sprintf
```bash
#!/bin/bash
printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234
printf "%-10s %-8s %-4.2f\n" 杨过 男 48.6543
printf "%-10s %-8s %-4.2f\n" 郭芙 女 47.9876
```
结果：
```
姓名     性别   体重kg
郭靖     男      66.12
杨过     男      48.65
郭芙     女      47.99
```
>`%s %c %d %f` 都是格式替代符
>`d`: Decimal 十进制整数 -- 对应位置参数必须是十进制整数，否则报错！
`s`: String 字符串 -- 对应位置参数必须是字符串或者字符型，否则报错！
`c`: Char 字符 -- 对应位置参数必须是字符串或者字符型，否则报错！
`f`: Float 浮点 -- 对应位置参数必须是数字型，否则报错！
>`%-10s` 指一个宽度为10个字符（-表示左对齐，没有则表示右对齐）,任何字符都会被显示在10个字符宽的字符内，如果不足则自动以空格填充，超过也会将内容全部显示出来。
>`%-4.2f` 指格式化为小数，其中.2指保留2位小数。

## 7. test
Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。
```bash
#!/bin/bash
num1=100
num2=100
if test $[num1] -eq $[num2]
```
## 8. 流程控制
### ① if-else
```bash
#!/bin/bash
a=10
b=20
if [ $a == $b ]
then
   echo "a 等于 b"
elif [ $a -gt $b ]
then
   echo "a 大于 b"
elif [ $a -lt $b ]
then
   echo "a 小于 b"
else
   echo "没有符合的条件"
fi
```
### ② for
```bash
#!/bin/bash
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```
### ③ while
```bash
#!/bin/bash
int=1
while(( $int<=5 ))
do
    echo $int
    let "int++"
done
```
### ④ case
```bash
#!/bin/bash
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac
```
### ⑤ break
break命令允许跳出所有循环（终止执行后面的所有循环）。
```bash
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的! 游戏结束"
            break
        ;;
    esac
done
```
### ⑥ continue
跳出当前循环。
```bash
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字: "
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的!"
            continue
            echo "游戏结束"
        ;;
    esac
done
```

### ⑦ until
```bash
#!/bin/bash

a=0

until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```

## 9. 函数
```bash
#!/bin/bash

funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```
结果：
```
第一个参数为 1 !
第二个参数为 2 !
第十个参数为 10 !
第十个参数为 34 !
第十一个参数为 73 !
参数总数有 11 个!
作为一个字符串输出所有参数 1 2 3 4 5 6 7 8 9 34 73 !
```

## 10. 输入输出
```bash
#!/bin/bash
who > today.log # 执行结果覆盖到文件 today.log
echo "菜鸟教程：www.runoob.com" >> today.log # 执行结果追加到文件 today.log
wc -l < today.log # 统计 today.log 行数
wc -l << EOF
    李白
    苏轼
    王勃
EOF
```

## 11. 文件包含
test1.sh
```bash
#!/bin/bash
name="Chris"
```
test2.sh
```bash
#!/bin/bash
#使用 . 号来引用test1.sh 文件
. ./test1.sh

# 或者使用以下包含文件代码
# source ./test1.sh

echo $name
```
>注：被包含的文件 test1.sh 不需要可执行权限。

## reference:
[1] http://www.runoob.com/linux/linux-shell.html
