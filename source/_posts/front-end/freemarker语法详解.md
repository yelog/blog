---
title: FreeMarker语法详解
permalink: FreeMarker
date: 2016-10-05 11:58:50
categories:
- 大前端
tags:
- java
- freemarker
---
FreeMarker是一款 **模板引擎** :即一种基于模板和要改变的数据，并用来生成输出文本（HTML网页、电子邮件、配置文件、源代码等）的通用工具。
FreeMarker模板文件主要有4部分组成
1. **文本**，直接输出的部分
2. **注释**，即<#--...-->格式不会输出
3. **插值**（Interpolation）：即${..}或者#{..}格式的部分,将使用数据模型中的部分替代输出
4. **FTL指令**：FreeMarker指令，和HTML标记类似，名字前加#予以区分，不会输出。

<!--more -->
## 一些规则
###  FTL指令规则
FreeMarker有三种FTL标签，这和HTML的标签是完全类似的
     开始标签：<#directivename parameters>
     结束标签：</#directivename>
     空标签： <#directivename parameters />
     实际上，使用标签时前面的#符号也可能变成@，如果该指令是一个用户指令而不是系统内建指令时，应将#符号改为@符号
### 插值规则
FreeMarker的插值有如下两种类型
    1、通用插值：${expr}
    2、数字格式化插值：#{expr}或者#{expr;format}
通用插值，有可以分为四种情况
    a、插值结果为字符串值：直接输出表达式结果
    b、插值结果为数字值：根据默认格式(#setting 指令设置)将表达式结果转换成文本输出。可以使用内建的字符串函数格式单个插值，例如
```c
<#setting number_format = "currency" />
<#assign str = 42 />
${str}
${str?string}
${str?string.number}
${str?string.currency}
${str?string.percent}
${str?string.computer}

日期处理
${openingTime?string.short}
${openingTime?string.medium}
${openingTime?string.long}
${openingTime?string.full}
${nextDiscountDay?string.short}
${nextDiscountDay?string.medium}
${nextDiscountDay?string.long}
${nextDiscountDay?string.full}
${lastUpdated?string.short}
${lastUpdated?string.medium}
${lastUpdated?string.long}
${lastUpdated?string.full}
${lastUpdated?string("yyyy-MM-dd HH:mm:ss zzzz")}
${lastUpdated?string("EEE, MMM d, ''yy")}
${lastUpdated?string("EEEE, MMMM dd, yyyy, hh:mm:ss a '('zzz')'")}
```
### if,elseif,elseif
```c
<#if condition>
……
<#elseif condition2>
……
<#else>
……
</#if>
```
### switch,case
```c
<#switch value>  
  <#case refValue1>  
         ...  
         <#break>  
  <#case refValue2>  
         ...  
         <#break>  
  ...  
  <#case refValueN>  
         ...  
         <#break>  
  <#default>
         ...  
</#switch>  
<#t> 去掉左右空白和回车换行  

<#lt>去掉左边空白和回车换行  

<#rt>去掉右边空白和回车换行  

<#nt>取消上面的效果  
```
### list

```c
<#list sequence as item>  
...  
<#if item = "spring">
  <#break>
</#if>  
...  
</#list>  
```
>iterm_index:当前值得下标，从0开始
item_has_next:判断list是否还有值

### include
```c
<#include filename [options]>
```
>options 包含两个属性
encoding="GBK"
parse="true" 是否作为ftl语法解析，默认是true
示例：<#include "/common/copyright.ftl" encoding="GBK" parse="true">

### import
```c
<#import path as hash>
```
> 类似于java里的import,它导入文件，然后就可以在当前文件里使用被导入文件里的宏组件

### compress
```c
<#compress>  
  ...  
</#compress>
```
### escape, noescape
```c
<#escape identifier as expression>  
  ...  
  <#noescape>...</#noescape>  
  ...  
</#escape>  
```
>主要使用在相似的字符串变量输出，比如某一个模块的所有字符串输出都必须是html安全的，这个时候就可以使用该表达式
示例：

```c
<#escape x as x?html>  
  First name: ${firstName}  
  <#noescape>Last name: ${lastName}</#noescape>  
  Maiden name: ${maidenName}  
</#escape>  
```
相同表达式
```c
First name: ${firstName?html}  
Last name: ${lastName }  
Maiden name: ${maidenName?html}  
```
### assign
```c
<#assign name=value>  

<#-- 或则 -->  

<#assign name1=value1 name2=value2 ... nameN=valueN>  

<#-- 或则 -->  

<#assign same as above... in namespacehash>  

<#-- 或则 -->  

<#assign name>  
  capture this  
</#assign>  

<#-- 或则 -->  

<#assign name in namespacehash>  
  capture this  
</#assign>  
```
>生成变量,并且给变量赋值

### global
```c
<#global name=value>  
<#--或则-->  
<#global name1=value1 name2=value2 ... nameN=valueN>  
<#--或则-->  
<#global name>  
  capture this  
</#global>  
```
>全局赋值语法，利用这个语法给变量赋值，那么这个变量在所有的namespace [A1] 中是可见的, 如果这个变量被当前的assign 语法覆盖 如<#global x=2> <#assign x=1> 在当前页面里x=2 将被隐藏，或者通过${.global.x} 来访问

### setting
```c
<#setting name=value>  
```
>用来设置整个系统的一个环境
locale
number_format
boolean_format
date_format , time_format , datetime_format
time_zone
classic_compatible

### macro, nested, return
```c
<#macro name param1 param2 ... paramN>  
  ...  
  <#nested loopvar1, loopvar2, ..., loopvarN>  
  ...  
  <#return>  
  ...  
</#macro>  
```
### t, lt, rt
```c
<#t> 去掉左右空白和回车换行  
<#lt>去掉左边空白和回车换行  
<#rt>去掉右边空白和回车换行  
<#nt>取消上面的效果  
```
