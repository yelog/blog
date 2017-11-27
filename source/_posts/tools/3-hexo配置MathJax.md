---
title: 3-hexo配置MathJax数学公式渲染
permalink: 3-hexo-mathjax
date: 2017-07-05 15:09:42
mathjax: true
categories:
- 工具
tags:
- mathjax
---
<hr>
在用 markdown 写文档时，免不了碰到数学公式。

## 处理hexo的MarkDown渲染器与MathJax的冲突
由于hexo的MarkDown渲染器与MathJax有冲突，所以在使用之前需要修改两个地方。

编辑 `node_modules\marked\lib\marked.js` 脚本

1. 将451行 ，这一步取消了对 `\\,\{,\}` 的转义(escape)
```js
escape: /^\\([\\`*{}\[\]()# +\-.!_>])/,
改为
escape: /^\\([`*\[\]()# +\-.!_>])/,
```
2. 将459行，这一步取消了对斜体标记 `_` 的转义
```js
em: /^\b_((?:[^_]|__)+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
改为
em:/^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```

## 开启MathJax
修改 `3-hexo/_config.yml`
```xml
# MathJax 数学公式支持
mathjax:
  on: true #是否启用
  per_page: false # 若只渲染单个页面，此选项设为false，页面内加入 mathjax: true
```
考虑到页面的加载速度，支持渲染单个页面。

设置 `per_page: false` ,在需要渲染的页面内 加入 `mathjax: true`

这样，就可以在页面内写MathJax公式了。

## MathJax公式书写
公式书写依然按照MarkDown语法来，基本上也和LaTeX相同，单 `$` 符引住的是行内公式，双$符引住的是行间公式。
* MathJax公式书写参考
[MathJax basic tutorial and quick reference](http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference)

### 1.MathJax行内公式
含有下划线 `_` 的公式 `$x_mu$` ： $x_mu$

希腊字符 `$\sigma$` ： $\sigma$

双 `\\` 公式内换行
```js
$$
f(n) =
\begin{cases}
n/2,  & \text{if $n$ is even} \\
3n+1, & \text{if $n$ is odd}
\end{cases}
$$
```
$$
f(n) =
\begin{cases}
n/2,  & \text{if $n$ is even} \\
3n+1, & \text{if $n$ is odd}
\end{cases}
$$

行内公式 `$y=ax+b$`：$y=ax+b$

行内公式 `$\cos 2\theta = \cos^2 \theta - \sin^2 \theta = 2 \cos^2 \theta$`：$\cos 2\theta = \cos^2 \theta - \sin^2 \theta = 2 \cos^2 \theta$

行内公式 `$M(\beta^{\ast}(D),D) \subseteq C$` ： $M(\beta^{\ast}(D),D) \subseteq C$

### 2.MathJax行间公式

行间公式`$$ \sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6} $$`：
$$ \sum_{i=0}^n i^2 = \frac{(n^2+n)(2n+1)}{6} $$

行间公式`$$ x = \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$`：
$$ x = \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$

### 3.MathJax公式自动编号

书写时使用
```
$$
\begin{equation}
\end{equation}
$$
```
进行公式自动编号，同时会自动连续编号，例如：
```xml
$$
\begin{equation}
\sum_{i=0}^n F_i \cdot \phi (H, p_i) - \sum_{i=1}^n a_i \cdot ( \tilde{x_i}, \tilde{y_i}) + b_i \cdot ( \tilde{x_i}^2 , \tilde{y_i}^2 )
\end{equation}
$$
$$
\begin{equation}
\beta^*(D) = \mathop{argmin} \limits_{\beta} \lambda {||\beta||}^2 + \sum_{i=1}^n max(0, 1 - y_i f_{\beta}(x_i))
\end{equation}
$$
```
$$
\begin{equation}
\sum_{i=0}^n F_i \cdot \phi (H, p_i) - \sum_{i=1}^n a_i \cdot ( \tilde{x_i}, \tilde{y_i}) + b_i \cdot ( \tilde{x_i}^2 , \tilde{y_i}^2 )
\end{equation}
$$
$$
\begin{equation}
\beta^*(D) = \mathop{argmin} \limits_{\beta} \lambda {||\beta||}^2 + \sum_{i=1}^n max(0, 1 - y_i f_{\beta}(x_i))
\end{equation}
$$

## MathJax公式手动编号
可以在公式书写时使用 `\tag{手动编号}` 添加手动编号，例如：
```
$$
\begin{equation}
\sum_{i=0}^n F_i \cdot \phi (H, p_i) - \sum_{i=1}^n a_i \cdot ( \tilde{x_i}, \tilde{y_i}) + b_i \cdot ( \tilde{x_i}^2 , \tilde{y_i}^2 ) \tag{1.2.3}
\end{equation}
$$
```
$$
\begin{equation}
\sum_{i=0}^n F_i \cdot \phi (H, p_i) - \sum_{i=1}^n a_i \cdot ( \tilde{x_i}, \tilde{y_i}) + b_i \cdot ( \tilde{x_i}^2 , \tilde{y_i}^2 ) \tag{1.2.3}
\end{equation}
$$

不加 `\begin{equation} \end{equation}` 也可以，例如：
```
$$
\beta^*(D) = \mathop{argmin} \limits_{\beta} \lambda {||\beta||}^2 + \sum_{i=1}^n max(0, 1 - y_i f_{\beta}(x_i)) \tag{我的公式3}
$$
```
$$
\beta^*(D) = \mathop{argmin} \limits_{\beta} \lambda {||\beta||}^2 + \sum_{i=1}^n max(0, 1 - y_i f_{\beta}(x_i)) \tag{我的公式3}
$$

行内公式加\tag{}后会自动成为行间公式，例如： `$z = (p_0, ..... , p_n) \tag{公式21} $`
$z = (p_0, ..... , p_n) \tag{公式21} $

### 4.其他公式书写技巧
**如何将下标放到正下方？**
① 如果是数学符号，那么直接用 `\limits` 命令放在正下方，如Max函数下面的取值范围，需要放在Max的正下方。可以如下实现：
`$ \max \limits_{a<x<b}\{f(x)\} $`
$ \max \limits_{a<x<b}\{f(x)\} $

② 若是普通符号，那么要用 `\mathop` 先转成数学符号再用 `\limits`，如
`$ \mathop{a}\limits_{i=1} $`
$ \mathop{a}\limits_{i=1} $

**MathJax矩阵输入**
无括号矩阵：
```
$$
\begin{matrix}
1 & x & x^2 \\
1 & y & y^2 \\
1 & z & z^2 \\
\end{matrix}
$$
```
$$
\begin{matrix}
1 & x & x^2 \\
1 & y & y^2 \\
1 & z & z^2 \\
\end{matrix}
$$

有括号有竖线矩阵：
```
$$
\left[
    \begin{array}{cc|c}
      1&2&3\\
      4&5&6
    \end{array}
\right]
$$
```
$$
\left[
    \begin{array}{cc|c}
      1&2&3\\
      4&5&6
    \end{array}
\right]
$$

行内小矩阵：
`$\bigl( \begin{smallmatrix} a & b \\ c & d \end{smallmatrix} \bigr)$`
$\bigl( \begin{smallmatrix} a & b \\ c & d \end{smallmatrix} \bigr)$

这里有个问题，上面的写法在矩阵内没有换行，我看了下源码，双反斜杠\\又被MarkDown渲染引擎转义为单个反斜杠了，解决方法是写三个反斜杠\\\或在双反斜杠后换行即可：

`$\bigl( \begin{smallmatrix} a & b \\\ c & d \end{smallmatrix} \bigr)$`
$\bigl( \begin{smallmatrix} a & b \\\ c & d \end{smallmatrix} \bigr)$

## 参考
[Hexo博客(13)添加MathJax数学公式渲染](http://masikkk.com/article/hexo-13-MathJax/)
[在Hexo中渲染MathJax数学公式](http://www.jianshu.com/p/7ab21c7f0674)
[MathJax basic tutorial and quick reference](https://math.meta.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference)
