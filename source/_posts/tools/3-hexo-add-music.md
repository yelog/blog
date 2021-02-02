---
title: 3-hexo 添加音乐插件
enlink: 3-hexo-add-music
date: 2019-10-08 10:44:30
categories:
- 工具
- hexo
tags:
- hexo
- 3-hexo
---

## 网易云音乐
### 1. 复制网易云音乐插件代码

前往[网易云音乐官网](https://music.163.com/)，搜索一个作为背景音乐的歌曲，并进入播放页面，点击 **生成外链播放器**
![生成外链播放器](https://i.loli.net/2019/10/08/RgSUj1i8vXNk5IP.png)

设置好想要显示的样式后，复制 html 代码

![](https://i.loli.net/2019/10/08/rbHRZEoB4mzip75.png)

最好外层在加一个 `div`，如下，可直接将上一步复制的 `iframe` 替换下方里面的 `iframe`
```html
<div id="musicMouseDrag" style="position:fixed; z-index: 9999; bottom: 0; right: 0;">
    <div id="musicDragArea" style="position: absolute; top: 0; left: 0; width: 100%;height: 10px;cursor: move; z-index: 10;"></div>
    <iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=38592976&auto=1&height=66"></iframe>
</div>
```
### 2. 将插件引入到主题中

将上一步加过 `div` 的代码粘贴到主题下 `layout/_partial/footer.ejs` 的最后面
![效果](https://i.loli.net/2019/10/08/FRJOKxLECcvinmf.png)
### 3. 调整位置

默认给的样式是显示在右下角，可以通过调整上一步粘贴的 `div` 的 `style` 中 `bottom` 和 `right` 来调整位置。

### 4. 自由拖动

如果需要自由拖动，在刚才添加的代码后面，再添加下面代码即可，鼠标就可以在音乐控件的 **上边沿** 点击拖动

```html
<!--以下代码是为了支持随时拖动音乐控件的位置，如没有需求，可去掉下面代码-->
<script>
    var $DOC = $(document)
    $('#musicMouseDrag').on('mousedown', function (e) {
      // 阻止文本选中
      $DOC.bind("selectstart", function () {
        return false;
      });
      $('#musicDragArea').css('height', '100%');
      var $moveTarget = $('#musicMouseDrag');
      $moveTarget.css('border', '1px dashed grey')
      var div_x = e.pageX - $moveTarget.offset().left;
      var div_y = e.pageY - $moveTarget.offset().top;
      $DOC.on('mousemove', function (e) {
        var targetX = e.pageX - div_x;
        var targetY = e.pageY - div_y;
        targetX = targetX < 0 ? 0 : (targetX + $moveTarget.outerWidth() >= window.innerWidth) ? window.innerWidth - $moveTarget.outerWidth() : targetX;
        targetY = targetY < 0 ? 0 : (targetY + $moveTarget.outerHeight() >= window.innerHeight) ? window.innerHeight - $moveTarget.outerHeight() : targetY;
        $moveTarget.css({'left': targetX + 'px', 'top': targetY + 'px', 'bottom': 'inherit', 'right': 'inherit'})
      }).on('mouseup', function () {
        $DOC.unbind("selectstart");
        $DOC.off('mousemove')
        $DOC.off('mouseup')
        $moveTarget.css('border', 'none')
        $('#musicDragArea').css('height', '10px');
      })
    })
</script>
```

