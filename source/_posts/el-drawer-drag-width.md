---
title: el-drawer 实现鼠标拖拽宽度[ElementUI]
enlink: el-drawer-drag-width
date: 2022-06-24 19:38:00
categories:
- 大前端
tags:
- ElementUI
- Vue
---

### 实现效果

![el-drawer-drag-width](https://cdn.jsdelivr.net/gh/yelog/assets/images/picgo_qiniu2022-06-25%2010.12.32.gif)

### 实现思路

通过指令的方式, 在 `drawer` 的左侧边缘, 添加一个触发拖拽的长条形区域, 监听鼠标左键按下时启动 `document.onmousemove` 的监听, 监听鼠标距离浏览器右边的距离, 设置为 `drawer` 的宽度, 并添加约束: 不能小于浏览器宽度的 20%, 不能大于浏览器宽度的 80%.

### 指令代码
创建文件 `src/directive/elment-ui/drawer-drag-width.js`, 内容如下

```js
import Vue from 'vue'

/**
 * el-drawer 拖拽指令
 */
Vue.directive('el-drawer-drag-width', {
  bind(el, binding, vnode, oldVnode) {
    const drawerEle = el.querySelector('.el-drawer')
    console.log(drawerEle)
    // 创建触发拖拽的元素
    const dragItem = document.createElement('div')
    // 将元素放置到抽屉的左边边缘
    dragItem.style.cssText = 'height: 100%;width: 5px;cursor: w-resize;position: absolute;left: 0;'
    drawerEle.append(dragItem)

    dragItem.onmousedown = (downEvent) => {
      // 拖拽时禁用文本选中
      document.body.style.userSelect = 'none'
      document.onmousemove = function(moveEvent) {
        // 获取鼠标距离浏览器右边缘的距离
        let realWidth = document.body.clientWidth - moveEvent.pageX
        const width30 = document.body.clientWidth * 0.2
        const width80 = document.body.clientWidth * 0.8
        // 宽度不能大于浏览器宽度 80%，不能小于宽度的 20%
        realWidth = realWidth > width80 ? width80 : realWidth < width30 ? width30 : realWidth
        drawerEle.style.width = realWidth + 'px'
      }
      document.onmouseup = function(e) {
        // 拖拽时结束时，取消禁用文本选中
        document.body.style.userSelect = 'initial'
        document.onmousemove = null
        document.onmouseup = null
      }
    }
  }
})

```

然后在 `main.js` 中将其导入
```js
import './directive/element-ui/drawer-drag-width'
```

### 指令使用
在 `el-drawer`  上添加指令 `v-el-drawer-drag-width` 即可, 如下
```html
<el-drawer
  v-el-drawer-drag-width
  :visible.sync="helpDrawer.show"
  direction="rtl"
  class="my-drawer"
>
  <template #title>
    <div class="draw-title">{{ helpDrawer.title }}</div>
  </template>
  <Editor
    v-model="helpDrawer.html"
    v-loading="helpDrawer.loading"
    class="my-wang-editor"
    style="overflow-y: auto;"
    :default-config="helpDrawer.editorConfig"
    :mode="helpDrawer.mode"
    @onCreated="onCreatedHelp"
  />
</el-drawer>
```

