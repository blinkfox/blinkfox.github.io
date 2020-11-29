---
title: 轻量级滚动动画JavaScript库aos.js
date: 2018-10-08 22:30:00
author: blinkfox
img: https://statics.sh1a.qingstor.com/2018/10/08/1.jpg
categories: 前端
tags:
  - JavaScript
---

## 一、简介

[aos.js][1]是一款效果超赞的页面滚动的 JavaScript 动画库插件。该动画库可以在页面滚动时提供28种不同的元素动画效果，以及多种`easing`效果。在页面往回滚动时，元素会恢复到原来的状态。

![AOS][2]

> 注：从`2.0.0`版本之后,只支持使用`data-aos`属性，不再支持使用`aos`属性。

## 二、安装

### 1. Bower 安装

你可以使用 [Bower][3] 包管理工具安装`aos`：

```bash
bower install aos --save
```

### 2. npm

你也能在 [npm][4] 上找到 `aos`：

```bash
npm install aos --save
```

### 3. Github 下载

Github 下载点击[此处][5]

## 三、使用示例

### 1. 使用方法

引入`CSS`样式文件：

```html
<link rel="stylesheet" href="bower_components/aos/dist/aos.css" />
```

添加`JavaScript`脚本文件：

```html
<script src="bower_components/aos/dist/aos.js"></script>
```

初始化载入`AOS`：

```html
<script>
    AOS.init();
</script>
```

### 2. 简单示例

```css
body {
    font-family: Helvetica,Tahoma;
}

*,
*:before,
*:after {
    box-sizing: border-box;
}

.aos-all {
    width: 1000px;
    max-width: 98%;
    margin: 10vh auto 0 auto;
}

.aos-item {
    display: inline-block;
    float: left;
    width: 33.3333%;
    height: 300px;
    padding: 20px;
}

.aos-item__inner {
    position: relative;
    width: 100%;
    height: 100%;
    background: #1da4e2;
    line-height: 260px;
    text-align: center;
    color: #fff;
}

@media screen and (max-width: 800px) {
    .aos-item {
        width: 50%;
    }
}
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>AOS的简单示例</title>
    <meta name="viewport" content="width=device-width">
    <link type="text/css" rel="stylesheet" href="aos/aos.css" />
    <link type="text/css" rel="stylesheet" href="aos_test.css" />
</head>
<body onload="initLoad();">

    <div id="transcroller" class="aos-all">
        <div class="aos-item" data-aos="fade-up">
            <div class="aos-item__inner"><h3>1</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-down">
            <div class="aos-item__inner"><h3>2</h3></div>
        </div>
        <div class="aos-item" data-aos="zoom-out-down">
            <div class="aos-item__inner"><h3>3</h3></div>
        </div>
        <div class="aos-item" data-aos="flip-down">
            <div class="aos-item__inner"><h3>4</h3></div>
        </div>
        <div class="aos-item" data-aos="flip-up">
            <div class="aos-item__inner"><h3>5</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-down">
            <div class="aos-item__inner"><h3>6</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>7</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-down">
            <div class="aos-item__inner"><h3>8</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>9</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-down">
            <div class="aos-item__inner"><h3>10</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-up">
            <div class="aos-item__inner"><h3>11</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-down">
            <div class="aos-item__inner"><h3>12</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>13</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-up">
            <div class="aos-item__inner"><h3>14</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>15</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-up">
            <div class="aos-item__inner"><h3>16</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-down">
            <div class="aos-item__inner"><h3>17</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-up">
            <div class="aos-item__inner"><h3>18</h3></div>
        </div>
        <div class="aos-item" data-aos="zoom-out">
            <div class="aos-item__inner"><h3>19</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-up">
            <div class="aos-item__inner"><h3>20</h3></div>
        </div>
        <div class="aos-item" data-aos="zoom-out">
            <div class="aos-item__inner"><h3>21</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>22</h3></div>
        </div>
        <div class="aos-item" data-aos="zoom-out-up">
            <div class="aos-item__inner"><h3>23</h3></div>
        </div>
        <div class="aos-item" data-aos="zoom-out-down">
            <div class="aos-item__inner"><h3>24</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>25</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>26</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>27</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>28</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>29</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>30</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>31</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>32</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>33</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>34</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>35</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>36</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>37</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>38</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>39</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>40</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>41</h3></div>
        </div>
        <div class="aos-item" data-aos="fade-in">
            <div class="aos-item__inner"><h3>42</h3></div>
        </div>
    </div>

<script type="text/javascript" src="aos/aos.js"></script>
<script type="text/javascript">
    function initLoad() {
        AOS.init();
    }
</script>
</body>
</html>
```

### 3. 异步示例

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>AOS 异步使用的示例</title>
    <meta name="viewport" content="width=device-width">
    <link type="text/css" rel="stylesheet" href="aos.css" />
    <link type="text/css" rel="stylesheet" href="aos_test.css" />
</head>
<body onload="initLoad();">

    <div id="aos_async" class="aos-all"></div>

<script type="text/javascript" src="aos.js"></script>
<script type="text/javascript">
    function initLoad() {
        AOS.init();
    }

    // 0.5秒执行一次
    setInterval(addItem, 500);

    var itemsCounter = 1;
    var container = document.getElementById('aos_async');

    /**
     * 动态生成的div元素
     */
    function addItem () {
        if (itemsCounter > 42) return;
        var item = document.createElement('div');
        item.classList.add('aos-item');
        item.setAttribute('data-aos', 'fade-up');
        item.innerHTML = '<div class="aos-item__inner"><h3>' + itemsCounter + '</h3></div>';
        container.appendChild(item);
        itemsCounter++;
    }
</script>
</body>
</html>
```

## 四、动画样式

以下是`AOS`已经提供了的多种动画：

### 1. Fade animations

- fade-up
- fade-down
- fade-left
- fade-right
- fade-up-right
- fade-up-left
- fade-down-right
- fade-down-left

### 2. Flip animations

- flip-up
- flip-down
- flip-left
- flip-right

### 3. Slide animations

- slide-up
- slide-down
- slide-left
- slide-right

### 4. Zoom animations

- zoom-in
- zoom-in-up
- zoom-in-down
- zoom-in-left
- zoom-in-right
- zoom-out
- zoom-out-up
- zoom-out-down
- zoom-out-left
- zoom-out-right

### 5. Anchor placement

- top-bottom
- top-center
- top-top
- center-bottom
- center-center
- center-top
- bottom-bottom
- bottom-center
- bottom-top

## 五、Easing 函数

你可以选择以下任意一个时间函数来做出很好的做动画元素：

- linear
- ease
- ease-in
- ease-out
- ease-in-out
- ease-in-back
- ease-out-back
- ease-in-out-back
- ease-in-sine
- ease-out-sine
- ease-in-out-sine
- ease-in-quad
- ease-out-quad
- ease-in-out-quad
- ease-in-cubic
- ease-out-cubic
- ease-in-out-cubic
- ease-in-quart
- ease-out-quart
- ease-in-out-quart

  [1]: http://michalsnik.github.io/aos/
  [2]: https://statics.sh1a.qingstor.com/2018/10/08/aos.png
  [3]: https://bower.io/
  [4]: https://www.npmjs.com/
  [5]: https://github.com/michalsnik/aos/archive/master.zip