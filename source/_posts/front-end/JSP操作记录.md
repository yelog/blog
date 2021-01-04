---
title: JSP操作记录
enlink: jsp-use-record
date: 2017-04-21 11:22:12
categories:
- 大前端
tags:
- jsp
- jstl
---
## 问题
### EL表达式失效
```html
<!-- jsp渲染器不识别el表达式，结果页面展示效果如下 -->
{person.id} {person.name}
```
**解决方法：**
在页面内加入下面代码即可
```html
<%@ page isELIgnored="false" %>
```
## Map
### 遍历
```html
<c:forEach items="${map}" var="entry">  
   <c:out value="${entry.key}" />  
   <c:out value="${entry.value}" />  
</c:forEach>  
```
### 取值
```html
<c:out value="${map[key]}" />
```
