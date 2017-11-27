---
title: jQuery选择器与节点操作
permalink: jQuery-selector
date: 2016-11-22 17:11:48
categories:
- 大前端
tags:
- jQuery
---
jQuery 是一个 JavaScript 函数库。jQuery的语法设计使得许多操作变得容易，如操作文档对象（document）、选择文档对象模型（DOM）元素、创建动画效果、处理事件、以及开发Ajax程序。jQuery也提供了给开发人员在其上创建插件的能力。这使开发人员可以对底层交互与动画、高级效果和高级主题化的组件进行抽象化。
<!--more-->
## jQuery获取元素
### 元素选择器
```JavaScript
//元素选择器 <div >
$("div")
```
### id选择器
```JavaScript
//id选择器 <div id='id'>
$("#id") $("div#id")
```
### class选择器
```javascript
//class选择器 <div class='class'>
$(".class") $("div.class")
```
### 属性过滤选择器
```html
<li class='check' type='li_01'></li>
<li type='li_02'></li>
<li type='li_03'></li>
```
```javascript
//通过属性获取 如果属性值为有特殊字符，一定要加引号
$("[type]")                  //获取有type属性的元素
$("[type='li_01']")          //获取type值等于'li_01'的元素
$("[type!='li_01']")         //获取type值不等于'li_01'的元素
$("[type*='li']")            //模糊匹配 获取type值包含'li'的元素
$("[type^='li']")            //模糊匹配 获取type值以'li'开始的元素
$("[type$='01']")            //模糊匹配 获取type值以'01'结尾的元素
$("li[class='check'][type]") //获取多个条件同时满足的元素

```
### * 选择器
```javascript
//遍历form下的所有元素,将其margin设置0
$('form *').css('margin','0px')
```
### 并列选择器
```javascript
$('p, div').css('color','red'); //将p元素和div元素的字体颜色设置为red
```
### 层叠选择器
```html
<div class='a'>                  <!-- 父级div -->
    <div class='a1'>             <!-- 子级div1 -->
        <div class='a11'></div>  <!-- 孙级div1 -->
    </div>   
    <div class='a2'></div>       <!-- 子级div2 -->
    <div class='a3'></div>       <!-- 子级div3 -->
    <span></span>                <!-- 子级span1 -->
</div>
```
```javascript
$(".a div")       //选择class=a的元素下所有的div 即选择到子级div1,2,3 孙级div1
$(".a > div")     //选择class=a的元素的所有子div元素, 即选择到子级div1,2,3;
$("div + span")   //选择所有的div元素的下一个input元素节点,即选择到:子级div3
$(".a1 ~ div")    //同胞选择器,返回class为a2的标签元素的所有属于同一个父元素的div标签,即div1,2,3
```
### 基本过滤选择器
```html
<ul>
    <li></li>
    <li></li>
    <li></li>
    <li></li>
    <li></li>
</ul>
```
```javascript
$("li:first")     //选择所有li元素的第一个
$("li:last")      //选择所有li元素的最后一个
$("li:even")      //选择所有li元素的第0,2,4... ...个元素(序号从0开始)
$("li:odd")       //选择所有li元素的第1,3,5... ...个元素
$("li:eq(2)")     //选择所有li元素中的第三个(即序号为2)
$("li:gt(3)")     //选择所有li元素中序号大于3的li元素
$("li:ll(2)")     //选择所有li元素中序号小于2的li元素
```
```html
<input type="checkbox" />
<input type="checkbox" />
```
```javascript
$("input[type='checkbox']:checked")        //获取所有已被选中的type等于checkbox的input元素
$("input[type='checkbox']:not(:checked)")  //获取所有未被选中的type等于checkbox的input元素
```
### 内容过滤器
```javascript
$("div:contains('Faker')")     //选择所有div中含有Faker文本的元素
$("div:empty")                 //选择所有div中为空(不包含任何元素/文本)的元素
$("div:has('p')")              //选择所有div中包含p元素的元素
$("div:parent")                //选择所有的含有子元素或文本的div
```
### 可视化过滤器
```javascript
$("div:hidden")            //选择所有被hidden的div元素
$("div:not(:hidden)")      //选择所有没有被hidden的div元素
$("div:visible")           //所有可视化的div元素
$("div:not(:visible)")     //所有非可视化的div元素
```
### 子元素过滤器
```html
<body>
    <div class='d1'>
        <div class='d11'>
            <div class='d111'>
            </div>
        </div>
    </div>
</body>
```
```javascript
$("body div:first-child")  //返回所有的body元素下 所有div 为父元素的第一个元素 的元素.
//:first 与 :first-child 的区别用法
//$("body div:first")只匹配到第一个合适的元素 即只匹配到 d1
//$("body div:first-child") 匹配所有合适的元素:d1是body的第一个元素,d11是d1的第一个元素..
//所以匹配到d1,d11,d111
$("div span:last-child")   //返回所有的body元素下 所有div 为父元素的最后一个元素 的元素.
//:last 与 :last-child 的区别参考first
$("div button:only-child") //如果button是它父级元素的唯一子元素,此button将会被匹配
```
### 表单元素选择器
```javascript
$(":input")    //选择所有的表单输入元素，包括input, textarea, select 和 button
$(":text")     //选择所有的text input元素
$(":password") //选择所有的password input元素
$(":radio")    //选择所有的radio input元素
$(":checkbox") //选择所有的checkbox input元素
$(":submit")   //选择所有的submit input元素
$(":image")    //选择所有的image input元素
$(":reset")    //选择所有的reset input元素
$(":button")   //选择所有的button input元素
$(":file")     //选择所有的file input元素
$(":hidden")   //选择所有类型为hidden的input元素或表单的隐藏域
```
### 表单元素过滤器
```javascript
$(":enabled")               //选择所有的可操作的表单元素
$(":disabled")              //选择所有的不可操作的表单元素
$(":checked")               //选择所有的被checked的表单元素
$("select option:selected") //选择所有的select 的子元素中被selected的元素
```
## 节点操作
### 获取和操作节点属性
```html
<a href='index.html' data-type='a' style="color:red;">index</a>
<input value='user' />
```
```javascript
$("a").attr("href");               //获取href属性值
$("a").attr("href","about.html");  //设置href属性值

$("a").data("type");               //获取data-type属性值

$("a").css("color");               //通过key(color/display/....)获取css值
$("a").css("color","black");       //通过key/value 设置css属性

$("a").text();                     //获取a的文本节点值
$("a").text("Index.html");         //设置a的文本节点值

$("input").val();                  //获取input的value值
$("input").val("username");        //设置input的value值
```
### 插入节点的方法
```html
<div class="head">
    <span>Faker<span>
</div>
```
```javascript
$(".head").append("<span>hello</span>")  //在.head中的最后插入一段html
//结果: <div class="head"><span>Faker</span><span>hello</span></div>

$("<span>hello</span>").appendTo(".head") //在.head中的最后插入一段html,
//结果: <div class="head"><span>Faker</span><span>hello</span></div>

$(".head").prepend("<span>hello</span>")  //在.head中的开始插入一段html,
//结果: <div class="head"><span>hello</span><span>Faker</span></div>

$("<span>hello</span>").prependTo(".head")  //在.head中的开始插入一段html,
//结果: <div class="head"><span>hello</span><span>Faker</span></div>

$(".head *:first").after("<span>hello</span>")  //在.head中的第一个元素后插入一段html,
//结果: <div class="head"><span>Faker</span><span>hello</span></div>

$("<span>hello</span>").insertAfter(".head *:first")  //在.head中的第一个元素后插入一段html,
//结果: <div class="head"><span>Faker</span><span>hello</span></div>

$(".head *:first").before("<span>hello</span>")  //在.head中的第一个元素前插入一段html,
//结果: <div class="head"><span>hello</span><span>Faker</span></div>

$("<span>hello</span>").insertBefore(".head *:first")  //在.head中的第一个元素后插入一段html,
//结果: <div class="head"><span>hello</span><span>Faker</span></div>
```
### $.load()方法
>在指定位置加载请求回来的html页面

```html
<div class="head">

</div>
```
```javascript
$(".head").load(url[,data][,callback])
url:           请求HTML页面的URL地址
data(可选):     请求的key/value参数
callback(可选)  请求完成的回调函数
```
