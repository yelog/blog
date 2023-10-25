---
title: jQuery之checkbox|radio|select操作
enlink: jQuery-checkbox-radio-select
date: 2016-11-23 16:19:34
categories:
- 大前端
tags:
- jQuery
---
jQuery1.6中添加了prop方法,看起来和用起来都和attr方法一样,但是在一些特殊情况下,attribute和properties的区别非常大,在jQuery1.6之前，.attr()方法在获取一些attributes的时候使用了property值，这样会导致一些不一致的行为。在jQuery1.6中，.prop()方法提供了一中明确的获取property值得方式，这样.attr()方法仅返回attributes。 --摘自jQuery API文档
<!--more -->
## checkbox
```html
<input type='checkbox' value='1'/>
<input type='checkbox' value='2'/>
<input type='checkbox' value='3'/>
```
```javascript
$("input[type=checkbox]")                    //获取所有的checkbox
$("input[type=checkbox]:checked")            //获取所有的被选中的checkbox
$("input[type=checkbox]:not(:checked)")      //获取所有未被选中的checkbox
$("input[type=checkbox]").not(":checked")    //获取所有未被选中的checkbox

$("input[type=checkbox]:first")              //获取第一个checkbox的value值

$("input[type=checkbox]:checked").length     //获取被选中checkbox的数量

$("input[type=checkbox]:first").prop("checked")               //判断第一个checkbox是否被选中
$("input[type=checkbox]:first").prop("checkbox",true)         //选中第一个checkbox

$("input[type=checkbox]:not(:checked)").prop("checked",true) //全选
$("input[type=checkbox]:checkbox").prop("checked",false)     //都不选中

//反选
$("input[type=checkbox]").each(function(){
    if($(this).prop("checked")){
        $(this).prop("checked",false);
    }else{
        $(this).prop("checked",true);
    }
})
```

## radio
```html
<input type='radio' name='rank' value='1' />
<input type='radio' name='rank' value='2' />
<input type='radio' name='rank' value='3' />
```
```javascript
$("input[type=radio]")                 //获取所有的radio
$("input[type=radio]:checked")         //获取被选中的radio
$("input[type=radio]:not(:checkbox)")  //获取所有没有被选中的radio

$("input[type=radio]:checked").val()   //获取被选中的radio的value值

$("input[type=radio]:first").prop("checked")       //判断第一个radio是否被选中
$("input[type=radio]:first").prop("checked",true)  //选中第一个radio
```
## select
```html
<select>
    <option value='1'>1</option>
    <option value='2'>2</option>
    <option value='4'>3</option>
</select>

```
```javascript
$("select option:selected")        //获取被选中的option
$("select").val()                  //获取选中option的value值
$("select option:selected").text() //获取被选中的option的text值

$("select option:first").prop("selected")           //判断第一个option是否被选中
$("select option:first").prop("selected",true)      //选中第一个option
$("select option:selected").prop("selected",false)  //取消当前选中,然后默认选中第一个

```
