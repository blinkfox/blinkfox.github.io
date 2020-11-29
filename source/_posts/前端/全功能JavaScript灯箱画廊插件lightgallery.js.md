---
title: 全功能JavaScript灯箱画廊插件lightgallery.js
date: 2018-10-06 23:30:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/10/06/gallery.jpg
categories: 前端
tags:
  - JavaScript
---

### 一、简介

> [lightgallery.js][1] 是一个全功能、轻量级、无依赖的灯箱画廊显示库。

![lightgallery][2]

### 二、主要特性

- 全响应式兼容
- 模块化的架构和内置插件
- 移动设备和触摸支持
- 桌面设备拖拽支持
- 双击查看图像的实际大小
- 动画缩略图
- 社交媒体分享
- YouTube，Vimeo，DailyMotion，VK和 HTML5 视频支持
- 20+ 硬件加速CSS3过渡
- 全屏支持
- 支持缩放
- 浏览器历史记录
- 响应式图像
- HTML iframe 支持
- 支持iFrame框架
- 单页多实例
- 可能过CSS(SCSS)定制样式
- 智能图像预加载与代码优化
- 桌面键盘导航
- 字体图标支持
- 还有更多

### 三、浏览器支持

lightgallery 支持所有主要的浏览器包括IE 9及以上。

### 四、安装下载

#### 1. Bower 安装

你可以使用 [Bower][3] 包管理工具安装`lightgallery`：

```bash
bower install lightgallery.js --save
```

#### 2. npm

你也能在 [npm][4] 上找到 `lightgallery`：

```bash
npm install lightgallery.js
```

#### 3. Github 下载

你也可以直接从 GitHub 下载[lightgallery][5]

### 五、基础示例

#### 1. 使用方法

首先，在 html 头文件`<head>`中引入`lightgallery.css`：

```html
<head>
    <link rel="stylesheet" href="css/lightgallery.css">
</head>
```

然后，在`<body>`标签结尾引入`lightgallery.min.js`，如果你想引入其他 lightgallery 的功能插件，你可以将这些插件引入到`lightgallery.min.js`之后，如下：

```html
<body>
    ...

    <script src="js/lightgallery.min.js"></script>

    <!-- lightgallery plugins -->
    <script src="js/lg-thumbnail.min.js"></script>
    <script src="js/lg-fullscreen.min.js"></script>
</body>
```

以下是页面标记的图片示例：

```html
<div id="lightgallery">
    <a href="img/img1.jpg">
        <img src="img/thumb1.jpg">
    </a>
    <a href="img/img2.jpg">
        <img src="img/thumb2.jpg">
    </a>
    ...
</div>
```

最后，是 JavaScript 调用插件的方式：

```javascript
<script>
    lightGallery(document.getElementById('lightgallery'));
</script>
```

#### 2. 完整示例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>lightgallery.js的使用示例</title>
    <link type="text/css" rel="stylesheet" href="/lightGallery/css/lightgallery.min.css" />
    <style type="text/css" rel="stylesheet">
        ul {
            float: left;
            list-style-type: none;
        }
        ul li {
            float: left;
            display: inline-block;
            margin: 5px;
        }
        .lgallery {
            width: 213px;
            height: 137px;
            cursor: pointer;
        }
    </style>
</head>
<body onload="initLoad();">

    <ul id="lightGallery">
        <li data-src="/assets/images/a.jpg">
            <img class="lgallery" src="/assets/images/a.jpg">
        </li>
        <li data-src="/assets/images/b.jpg">
            <img class="lgallery" src="/assets/images/b.jpg">
        </li>
        <li data-src="/assets/images/c.jpg">
            <img class="lgallery" src="/assets/images/c.jpg">
        </li>
        <li data-src="/assets/images/d.jpg">
            <img class="lgallery" src="/assets/images/d.jpg">
        </li>
        <li data-src="/assets/images/e.jpg">
            <img class="lgallery" src="/assets/images/e.jpg">
        </li>
        <li data-src="/assets/images/f.jpg">
            <img class="lgallery" src="/assets/images/f.jpg">
        </li>
        <li data-src="/assets/images/g.jpg">
            <img class="lgallery" src="/assets/images/g.jpg">
        </li>
    </ul>

<script type="text/javascript" src="/lightGallery/js/lightgallery.min.js"></script>
<script type="text/javascript" src="/lightGallery/js/plugins/lg-fullscreen.min.js"></script>
<script type="text/javascript" src="/lightGallery/js/plugins/lg-thumbnail.min.js"></script>
<script type="text/javascript" src="/lightGallery/js/plugins/lg-autoplay.min.js"></script>
<script type="text/javascript" src="/lightGallery/js/plugins/lg-hash.min.js"></script>
<script type="text/javascript" src="/lightGallery/js/plugins/lg-pager.min.js"></script>
<script type="text/javascript" src="/lightGallery/js/plugins/lg-share.min.js"></script>
<script type="text/javascript" src="/lightGallery/js/plugins/lg-zoom.min.js"></script>
<script type="text/javascript">
function initLoad() {
    var lg = document.getElementById('lightGallery');
    lightGallery(lg, {
        mode: 'lg-slide',
        cssEasing: 'ease',
        speed: 500
    });
}
</script>
</body>
</html>
```

### 六、学习和参考资源

- [API Reference][6]
- [Events][7]
- [Methods][8]
- [Data Attributes][9]
- [Dynamic variables][10]
- [Sass variables][11]
- [Module API][12]

  [1]: https://sachinchoolur.github.io/lightgallery.js/
  [2]: https://statics.sh1a.qingstor.com/2018/10/06/lg.png
  [3]: https://bower.io/
  [4]: https://www.npmjs.com/
  [5]: https://github.com/sachinchoolur/lightgallery.js
  [6]: https://sachinchoolur.github.io/lightgallery.js/docs/api.html
  [7]: https://sachinchoolur.github.io/lightgallery.js/docs/api.html#events
  [8]: https://sachinchoolur.github.io/lightgallery.js/docs/api.html#methods
  [9]: https://sachinchoolur.github.io/lightgallery.js/docs/api.html#attributes
  [10]: https://sachinchoolur.github.io/lightgallery.js/docs/api.html#dynamic
  [11]: https://sachinchoolur.github.io/lightgallery.js/docs/api.html#sass
  [12]: https://sachinchoolur.github.io/lightgallery.js/docs/plugin-api.html