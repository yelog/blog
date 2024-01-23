---
title: '前端页面开发规范'
enlink: 'font-develop-rule'
date: 2017-03-08 08:58:21
categories:
- 大前端
tags:
- translation
---

## 一、前言

随着开发人员的不断增加，在没有规范的情况下，就会导致开发的页面不统一，不像是一个系统。为了解决这个问题，就有了此规范的出现，当然为了不影响各个功能的灵活性，此规范要求不高， 请耐心阅读，并应用到日常开发中。

当然，如果你有更好的建议，可以通过邮件联系 yangyj13@lenovo.com，进行沟通来完善此篇规范。



## 二、编程规范

### 2.1 命名规范

#### 2.1.1 文件命名

全部采用小写方式，以横杠分割。

正例: `resource.vue`、`user-info.vue`

反例: `basic_data.vue`、`EventLog.vue`

#### 2.1.2 目录命名

全部采用小写方式，以横杠分割。

正例: `system`、`ship-support`

反例: `errorPage`、`Components`

#### 2.1.3 JS、CSS、SCSS、HTML、PNG文件命名

全部采用小写方式，以横杠分割。

正例: `btn.scss`、`element-ui.scss`、`lenovo-logo.png`

反例: `leftSearch.scss`、`LeGrid.js`

#### 2.1.4 命名规范性

代码中命名严禁使用拼音和英文混合的方式，更不允许直接使用中文的方式。说明: 正确的英文拼写和语法可以让阅读者易于理解，避免歧义。注意，即使纯拼音的命名方式也要避免采用。

正例: `loading`、`searchForm`、`tableHeight`、`dmsLoading`、`rmb` 专有名词缩写，视同英文
反例: `getLiaoPanName`、`DMSLoading`

### 2.2 插件使用

#### 2.2.1 eslint 代码规范

注意：前端的代码格式化已经在 `eslint` 中声明了，所以确保自己已经启用了 `eslint`，并使 `eslint` 进行代码格式化。

#### 2.2.2 i18n 国际化

所有展示的内容都要支持国际化。国际化内容写到 `/src/lang/` 下的对应模块，通过 `this.$t('xx.xx.xx')` 来使用。

英文国际化的列或标签，请使用开头字母大写的方式，如: `UserId`、`Status`、`UserName`。

![image-20211227175812563](https://cdn.jsdelivr.net/gh/yelog/assets/images/picgo_qiniuimage-20211227175812563.png)

### 2.3 组件使用

#### 2.3.1 table 表格

表格组件推荐使用 [vxe-table](https://xuliangzhan_admin.gitee.io/vxe-table/#/table/start/install)，功能更加全面，之后也会主力优化此表格。比如可编辑表格的样式经过优化：[可编辑表格](http://10.176.66.58/#/example/table/edit)


#### 2.3.2 dialog 弹窗

弹窗组件推荐使用 [vxe-modal](https://xuliangzhan_admin.gitee.io/vxe-table/#/table/module/modal)，代码设计更加合理，功能也更加全面。

#### 2.3.2 element-ui

除 `table` 和 `modal` 外，其他组件比如 `form`、`button`、`DateTimePicker` 优先使用 [element-ui](https://element.eleme.cn/#/zh-CN/component/radio) 。

##### 2.3.2.1 icon 图标

图标优先使用  [element-ui](https://element.eleme.cn/#/zh-CN/component/icon) 的图标。如果没有合适的，可以在 [iconfont](https://www.iconfont.cn) 上寻找到合适的图标后，找 yangyj13@lenovo.com 进行添加。

##### 2.3.2.2 button 按钮

按钮大小：除了在表格中的按钮要使用 `size="mini"` 外，其他情况使用默认大小即可。

按钮颜色：普通的 查询/修改/操作 等按钮使用蓝色 `type="primary"`，新增使用绿色 `type="success"`，删除等“危险”操作使用红色 `type="danger"`。推荐给按钮添加图标，可在 [element-ui-icon](https://element.eleme.cn/#/zh-CN/component/icon) 寻找合适的图标。

![image-20211227172602560](https://cdn.jsdelivr.net/gh/yelog/assets/images/picgo_qiniuimage-20211227172602560.png)



#### 2.3.3 其他组件

如果上述组件并不能满足业务需求，可以优先在网上找到合适的组件后，与 yangyj13@lenoov.com 联系后添加。

### 2.4 页面布局

#### 2.4.1 新增/修改表单

普通的表单，采用中间对其的方案，也就是整个表单的 `label-width` 设置为一样的。

注意：一般的，新增修改使用弹窗的方式，展示表单。新增/修改可以共用代码，具体可以参考 `common/system/va-config.vue`



```html
<el-form
        ref="dialogForm"
        v-loading="edit.loading"
        :model="edit.form"
        :rules="edit.formRules"
        label-width="150px"
        style="padding-right: 30px;"
      >
  ...
</el-form>
```

![image-20211227170528261](https://cdn.jsdelivr.net/gh/yelog/assets/images/picgo_qiniuimage-20211227170528261.png)

#### 2.4.2 查询表单+表格

这种应该是最长间的需求方案了，可以参考 `/common/system/user.vue`，在写的时候注意以下几点：

1. `label-width` 不要设置，保证标签文字开头和表格对齐。
2. `el-form` 使用 `:inline="true"` 设置表单内容行内显示。
3. 设置 `vxe-table` 的 `height` 属性，保证表格底部贴住网页底部，又不会有滚动条（表格内允许有滚动条）
4. 按钮也放到表单中，不要单独一行。

最终效果如下：

![image-20211227174221673](https://cdn.jsdelivr.net/gh/yelog/assets/images/picgo_qiniuimage-20211227174221673.png)


