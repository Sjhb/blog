---
title: 兼容低版本微信的flexbox
date: 2017-07-07 21:53:00
tags: [web]
categories: 浏览器兼容
---

## 兼容低版本微信的flexbox

> 对低版本的微信，安卓和ios兼容性良好

旧flexbox: old flexbox
新flexbox: flexbox

Destop

| Feature     | IE           | FireFox(Gecko) | Opera | Chrome              | Safari(WebKit)      |
| ----------- | ------------ | -------------- | ----- | ------------------- | ------------------- |
| old flexbox | --           | 2-27(2-21:moz) | --    | 4-20(-webkit-)      | 3.1-6(-webkit-)     |
| flexbox     | 10+(10:-ms-) | 28+            | 12.1  | 20+(21-28:-webkit-) | 6.1+(6.1-8:webkit-) |

Mobile

| Feature     | Android           | Firefox Mobile(Gecko) | IE Phone | Opera Mobile | iOS Safari             |
| ----------- | ----------------- | --------------------- | -------- | ------------ | ---------------------- |
| old flexbox | 2.1-4.3(-webkit-) | --                    | --       | --           | 3.2-6.1(-webkit-)      |
| flexbox     | 4.4+              | 54+                   | 54+      | 12.1+        | 7.1+(7.1-8.4:-webkit-) |



## 兼容写法



> 使用构建工具的时候利用autofixture可以自动生成兼容的写法

>旧版语法需要将行内子元素变成块元素



### display

```
.father {

 	display: -webkit-box; /* 老版本语法: Safari, iOS, Android browser, older WebKit browsers. */
    display: -moz-box; /* 老版本语法: Firefox (buggy) */
    display: -ms-flexbox; /* 混合版本语法: IE 10 */
    display: -webkit-flex; /* 新版本语法: Chrome 21+ */
    display: flex; /* 新版本语法: Opera 12.1, Firefox 22+ */    
}
```

### 主轴对齐方式

>old-flexbox 中space-around 不可用

```
.father{
    box-pack: start | end | center | justify;
    /*主轴对齐：左对齐（默认） | 右对齐 | 居中对齐 | 左右对齐*/

    justify-content: flex-start | flex-end | center | space-between | space-around;
    /*主轴对齐方式：左对齐（默认） | 右对齐 | 居中对齐 | 两端对齐 | 平均分布*/
}
```

### 子元素交叉对齐方式

```
.father{
    box-align: start | end | center | baseline | stretch;
    /*交叉轴对齐：顶部对齐（默认） | 底部对齐 | 居中对齐 | 文本基线对齐 | 上下对齐并铺满*/

    align-items: flex-start | flex-end | center | baseline | stretch;
    /*交叉轴对齐方式：顶部对齐（默认） | 底部对齐 | 居中对齐 | 上下对齐并铺满 | 文本基线对齐*/
}
```
### 子元素从左到右排列

```
.father{
    -webkit-box-direction: normal;
    -webkit-box-orient: horizontal;
    -moz-flex-direction: row;
    -webkit-flex-direction: row;
    flex-direction: row;
}
```

### 子元素从右到左

```
.father{
    -webkit-box-pack: end;//子元素沿主轴末端排列
    -webkit-box-direction: reverse;//子元素倒序排列
    -webkit-box-orient: horizontal;
    -moz-flex-direction: row-reverse;
    -webkit-flex-direction: row-reverse;
    flex-direction: row-reverse;
}
```

### 子元素从上倒下排列

```
.father{
    -webkit-box-direction: normal;
    -webkit-box-orient: vertical;
    -moz-flex-direction: column;
    -webkit-flex-direction: column;
    flex-direction: column;
}
```

### 子元素从下到上排列

```
.father{
    -webkit-box-pack: end;
    -webkit-box-direction: reverse;
    -webkit-box-orient: vertical;
    -moz-flex-direction: column-reverse;
    -webkit-flex-direction: column-reverse;
    flex-direction: column-reverse;
}
```

### 子元素伸缩

放大：

```
.item{
    -webkit-box-flex: 1.0;
    -moz-flex-grow: 1;
    -webkit-flex-grow: 1;
    flex-grow: 1;
    /*放大一倍，默认是不放大的*/
}
```

缩小：

```
.item{
    -webkit-box-flex: 1.0;
    -moz-flex-shrink: 1;
    -webkit-flex-shrink: 1;
    flex-shrink: 1;
    /*缩小一倍，默认不缩小*/
}
```

### 子元素显示次序

```
.item{
    -webkit-box-ordinal-group: 1;
    -moz-order: 1;
    -webkit-order: 1;
    order: 1;
}
```