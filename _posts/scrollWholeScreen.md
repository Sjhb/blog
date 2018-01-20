---
title: 整屏滚动（初稿）
date: 2017-06-22 19:01:06
tags: 
---

## 方案

整屏滚动常常用在产品展示页、主页，例如Apple，锤子，等等。应用到移动端效果也是不错的。

在未实践之前以为很难，事实上整个实现过程是比较简单的，我的思路如下：

1. 设置一个div作为容器，div内存在多个section，每个section都能单独撑满窗口，div上下位置变化可以切换显示在窗口的section，形成滚动的效果。
2. 为多个滚屏（如section）设置为屏幕的宽度和高度，高度需要动态识别屏幕的高度。
3. 给容器div设置滚动动画，每次滚动就会触发动画。
4. 监听鼠标滚轮事件，滚动事件发生时触发容器div的滚动。
5. 设置变量记录每次滚动后的状态，实现滚动监听。

### 滚屏原理

![scrollScreen](https://cdn.rawgit.com/Sjhb/image.server/63fcff8f/markdown/scrollScreen.png)

如图所示，窗口中div.mian，div内有三个section，每个section刚好能铺满浏览器窗口。滚屏只是div.main的上下滑动而已，div.main上下滑动的距离是定长的，通过设定div.main的top属性就可以实现上下的位置变化。

## 实现

### 实现过程

>  这里以chrome浏览器为例
>
>  这里只是简要地列出一些代码，全部代码可见的github ： [scrollScreen](https://github.com/Sjhb/scrollScreen )

HTML :

```&amp;amp;amp;amp;amp;amp;amp;lt;html&amp;amp;amp;amp;amp;amp;amp;gt;
<div class="nav padding-big-top padding-big-bottom" id="nav">
    ...
</div>
<div class="main">
    <section class="section blue" id="blue"></section>
    <section class="section gay" id="gay"></section>
    <section class="section red" id="red"></section>
    <section class="section green flex flex-center" id="green">
        <img id="imgBall" src="img/demo.jpg" class="img-ball">
    </section>
</div>
```

HTML部分主要是头部的nav导航栏和多个单屏

CSS : 

```&amp;amp;amp;amp;amp;amp;lt;css&amp;amp;amp;amp;amp;amp;gt;
.section{position: relative;}
.blue{background-color:blue;}
.gay{background-color: gray;}
.red{background-color: red;}
.green{background-color: green;}
.main{
    width: 100%;
    position: absolute;
    top: 0px;
    left: 0px;
    transition: 1s top  cubic-bezier(0.86, 0, 0.07, 1);
}
.nav{
    width: 100%;
    background-color: white;
    position: fixed;
    z-index: 100;
    box-shadow: 0 1px 5px rgba(0,0,0,.1);
        transition: padding .3s ease-in-out 0.5s;
}
.padding-big-top{padding-top: 25px;}
.padding-big-bottom{padding-bottom: 25px;}
.padding-middle-top{padding-top: 8px;}
.padding-middle-bottom{padding-bottom: 8px;}
```

这里使用transition来实现动画效果，滚屏采用自定义函数，自定义函数生成自这个网站  [函数生成网站](http://cubic-bezier.com/) 。

- 首先是单屏高度的自适应

  可以使用Jquery的resize()方法，浏览器窗口发生变化的时候会发生resize事件，可以指定在resize事件发生，触发自定义事件。

| 语法                           | 参数       | 描述                          |
| ---------------------------- | -------- | --------------------------- |
| $(selector).resize(function) | function | 传入function，指定resize事件运行时的函数 |

  所以，通过resize事件动态设定单屏高度就好了。

```&amp;amp;amp;amp;amp;amp;lt;script&amp;amp;amp;amp;amp;amp;gt;
var sHeight = 0;
function resizeSec() {
    var height = window.innerHeight
        || document.documentElement.clientHeight
        || document.body.clientHeight;
    for (var i = 0; i < $(".section").length; i++) {
        $(".section").height(height);
    }
    sHeight = height;
}
$(window).resize(resizeSec)
```

- 单屏的动画

  使用css3的transition，可以很方便的实现动画效果。

  ```
  transition: 1s top  cubic-bezier(0.86, 0, 0.07, 1)
  ```

- 监听滚轮事件

  onmusewheel可以监听鼠标的滚轮事件，并能指定事件发生后执行的函数。所以使用onmousewheel事件来实现滚轮监听，滚动事件发生后，div.main就向上或者向下滑动，通过设置top的方式。

  申明一个变量sIndex来存储每次滚动后的状态，可以用来实现滚动监听。

  > onmousewheel 不兼容firfox

  ​

  ```
  var main = document.querySelector('.main');
  var sIndex = 0,sScroll = false;//滚动时就不再触发事件，防止误点;
  function scrollTo(i) {
      //操作动画的函数
      sScroll = true;
      main.style.top = -i * sHeight + 'px';
      setTimeout(function () {
          sScroll = false
      }, 700);

  }
  main.onmousewheel = function (e) {
      //PC端的滚轮事件，不兼容火狐
      if (!sScroll) {
          if (e.wheelDelta > 0) {
              //上一页
              if (sIndex == 0) {return;}
              sIndex--;
              scrollTo(sIndex);
          } else {
              //下一页
              if (sIndex == sLength - 1) { return;}
              sIndex++;
              scrollTo(sIndex);
          }
      }
  }
  ```




### 存在的问题

- 当用户使用触控板是，滑动速度很快，滚屏会滚动很多次





## 全部代码(正在开发中的)

### HTML

```
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>transiton demo</title>
    <link type="text/css" rel="stylesheet" href="style/demo.css">
    <script src="assets/js/jquery-3.2.1.js"></script>
</head>

<body id="body">
<!--整屏滚动：-->
<!--单个滚屏（一个div）设置为屏幕的宽度和高度；-->
<!--滚屏之间的滚动添加动画效果；-->
<!--右侧sroll栏监听滚屏事件，显示滚屏状态；-->
<!--滚屏-->
<div class="nav padding-big-top padding-big-bottom" id="nav">
    <div class="logo">logo</div>
    <div class="tags">
        <ul class="list">
            <li>tag</li>
            <li>tag</li>
            <li>tag</li>
            <li>tag</li>
        </ul>
    </div>
</div>
<div class="main">
    <section class="section blue" id="blue">
    </section>
    <section class="section gay" id="gay">
    </section>
    <section class="section red" id="red">
    </section>
    <section class="section green flex flex-center" id="green">
        <img id="imgBall" src="img/demo.jpg" class="img-ball">
    </section>
</div>

<script src="js/demo.js"></script>
</body>
</html>
```

### CSS

```
html,body{
margin: 0;
margin: 0;
}
body{
    overflow: hidden;
}
.hide{
    display: none;
}
.section{
    position: relative;
}
.flex{
    display: flex;
}
.flex-center{
    justify-content: center;
}
.blue{
    background-color:blue;
}
.gay{
    background-color: gray;
}
.red{
    background-color: red;
}
.green{
    background-color: green;
}
.main{
    width: 100%;
    position: absolute;
    top: 0px;
    left: 0px;
    transition: 1s top cubic-bezier(0.86, 0, 0.07, 1);
}
.nav{
    width: 100%;
    background-color: white;
    position: fixed;
    z-index: 100;
    box-shadow: 0 1px 5px rgba(0,0,0,.1);
        transition: padding .3s ease-in-out 0.5s;
}
.padding-big-top{
    padding-top: 25px;
}
.padding-big-bottom{
    padding-bottom: 25px;
}
.padding-middle-top{
    padding-top: 8px;
}
.padding-middle-bottom{
    padding-bottom: 8px;
}
.logo{
    margin-left: 50px;
    font-size: 30px;
    padding: 5px;
    float: left;
}
.list{
    float: right;
    display: flex;
    justify-content: center;
    padding-right: 200px;
}
.list>li{
    list-style: none;
    padding: 10px 20px;
}
@keyframes up-img{
    from{
        margin-top: 350px;
        max-width: 50px;
        max-height: 50px;
    }
    to{
        margin-top: 125px;
        max-width: 300px;
        max-height: 150px;
    }
}
.img-ball{
    animation: 2s 0.5s up-img forwards ;
    margin-top: 350px;
    max-width: 50px;
    max-height: 50px;
}


```

### JavaScript

```
/**
 * Created by manlin on 2017/6/22.
 */
var sHeight = 0;//滚屏高度
var sLength = document.querySelectorAll('.section').length;
var sIndex = 0;
var sScroll = false;//滚动时就不再触发事件，防止误点
var main = document.querySelector('.main'); //需要移动的DOM
var nav = document.querySelector('.nav')//头部导航
function resizeSec() {
//得到窗口高度
    var height = window.innerHeight
        || document.documentElement.clientHeight
        || document.body.clientHeight;
    for (var i = 0; i < $(".section").length; i++) {
        $(".section").height(height);
    }
    sHeight = height;
}
$(window).resize(resizeSec)

function shiftClass(el, tClass) {
    el.className = tClass;
}
//测试图片，显示隐藏，保证每次的动画都生效
function hideImg() {
    //缓一秒再隐藏
    setTimeout(function () {
        shiftClass(document.getElementById("imgBall"), 'hide')
    }, 1000)
}
function showImg() {
    shiftClass(document.getElementById("imgBall"), 'img-ball')
}

function scrollTo(i) {
    //操作动画的函数
    sScroll = true;
    main.style.top = -i * sHeight + 'px';
    setTimeout(function () {
        sScroll = false
    }, 700);

}

//nav缩放
function setNav() {
    document.getElementById("nav").className = "nav";
}
function enlargeNav() {
    setNav();
    $("#nav").addClass("padding-big-top padding-big-bottom");
}
function narrowNav() {
    setNav();
    $("#nav").addClass("padding-middle-top padding-middle-bottom");
}


//一下这样做的目的是让动画重复播放

//各滚屏需要显示的元素
function showBlue() {}
function showGray() {}
function showRed() {}
function showGreen() {
    showImg();
}

//各滚屏需要隐藏的元素
function hideBlue() {
}
function hideGray() {
}
function hideRed() {
}
function hideGreen() {
    hideImg();
}

//1、 隐藏除第一屏的之外的所有的未显示的动画元素 hideAll();
function hideAll() {
    hideGray();
    hideRed();
    hideGreen();
}
main.onmousewheel = function (e) {
    //PC端的滚轮事件，嗯不兼容火狐
    if (!sScroll) {
        if (e.wheelDelta > 0) {
            //上一页
            if (sIndex == 0) {return;}
            sIndex--;
            scrollTo(sIndex);
            switch (sIndex){
                case 2:{hideGreen();showRed();break}
                case 1:{hideRed();showGray();break}
                case 0:{hideGray();showBlue();enlargeNav();break}
            }
        } else {
            //下一页
            if (sIndex == sLength - 1) { return;}
            sIndex++;
            scrollTo(sIndex);
            switch (sIndex){
                case 1:{narrowNav();showGray();hideBlue();break;}
                case 2:{hideGray();showRed();break;}
                case 3:{hideRed();showGreen();break;}
            }
        }
    }
}
// 初始化
function init() {
    hideAll();
}
init();
```

