---
title: 开箱即用：Parcel
date: 2017-12-17 00:59:59
tags: Parcel
---

### 安装

```

​$ npm i -g parcel-bundler

​$ yarn global add parcel-bundler

```

老规矩，`Hello Parcel`

准备原料

```

$ sudo mkdir parcel-learn

$ npm init parcel-learn

```

新建入口文件

```html

// index.html

<!DOCTYPE html>

<html lang="ch">

<head>

​    <meta charset="UTF-8">

​    <title>parcel</title>

​    <style>

​        body{

​            text-align: center

​        }

​    </style>

</head>

<body class="main">

​    <script src="./index.js"></script>

</body>

</html>

```

```javascript

// index.js

let taga = document.createElement('h1')

let tagb = document.createElement('h1')

taga.innerHTML = '🚀'

tagb.innerHTML = 'Hello Parcel'

document.body.appendChild(taga)

document.body.appendChild(tagb)

```

运行命令: `parcel index.html -p 8000`

![hello](/images/parcel-hello.png)

哎哟，不错喔。真的是开箱即用。

### 资源

`Parceljs` 默认支持了`CommonJS`和`ES6`导入语法

```javascript

​    // CommonJs

​    import create from './create'

​    // ES6

​    const append = require('./append')

```

> `Parceljs`不会通过內联的形式导出文件

比如

```javascript

import demo from './demo.png'

img.src = demo

```

内联方式（其实就是自己写值）

```javascript

import fs from 'fs'

let demo = 'data:image/png;base64,' + fs.readFileSync(__dirname + '/demo.png', 'base64')

img.src = demo

```

css:支持`@import`,url里的资源必须相对于当前文件，且支持预编译语言：less、sass...

```css

/* other.css */

body {

​    background: url('./demo.png')

}

/* main.css */

@import './other.css';

body {

​    text-align: center;

​    background-repeat: no-repeat;

}

```

### 使用`babel`

要使用`Babel`转换ES6语法，只需要安装`babel`并配置好`.babelrc`文件，其他的交给`Parcel`来做就好了

```javascript

$ npm install -save babel-preset-env

```

```json

// .babelrc

{

  "presets": ["env"]

}

```

```javascript

// index.js
let p = new Promise((resolve, reject) => {

​    setTimeout(() => {

​        resolve(new Date())

​    }, 2000)

})

let test = async () => {

​    let res = await p

​    alert(res)

}

test()

```

![babel](/images/babel.png)

### js模块拆分和按需加载

通过使用动态`import()`函数可以实现代码的拆分，而`import()`函数是异步加载的，返回一个 Promise 对象。这样，当用到新的代码，可通过`import()`从远端获取，动态加载。

简单示例

```javascript

// index.js

import "babel-polyfill"

// 这些代码用到时才会从远端拉取

let pages = {

​    pagea: import('./renderb'),

​    pageb: import('./rendera')

}

let el = document.getElementById('ceshi')

let state = 1

el.onclick = async () => {

​    if (state === 1) {

​        let res = await pages.pagea

​        res.createH('hello')

​        state = 2

​    } else {

​        let res = await pages.pageb

​        res.createH('word')

​    }

}

// rendera.js

export function createH(str) {

​    let el = document.createElement('h1')

​    el.innerHTML = str

​    document.body.appendChild(el)

}

// renderb.js

export function createH(str) {

​    let el = document.createElement('h2')

​    el.innerHTML = str

​    document.body.appendChild(el)

}

```

```html

<!DOCTYPE html>

<html lang="ch">

<head>

​    <meta charset="UTF-8">

​    <title>parcel</title>

​    <style>

​        body{

​            text-align: center

​        }

​    </style>

</head>

<body class="main">

​    <button id="ceshi">测试</button>

​    <script src="./index.js"></script>

</body>

</html>

```

结果: 未点击 =》 点击一次 =》 点击两次

![](/images/state1.png)

![](/images/state2.png)

![](/images/state3.png)

> 很酷，对吧