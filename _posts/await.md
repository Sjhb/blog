---
title: 异步加载：Async/Await
date: 2017-08-20 02:40:09
tags: web es7
---

## Promise

在Promise出现以前，Javascript实现异步编程的方式有两种：

- 回调函数
- 事件监听

由于上面两种方式都有很大的局限性，CommonJS提出了Promise规范，并被纳入了es6。

```javascript
// 简单的Promise串联实例
// p1完成后会返回一个完成状态的Promise对象p2,然后被添加到新的Promise中
let p1 = new Promise((resolve, reject) => {
  ···向后台请求中···
  resolve('p1')
})
let p2 = new Promise((resolve, reject) => {
  ···向后台请求中··
  resolve('p2')
})
p1.then((value) =>{
  console.log(value)
  return p2
}).then((value) => {
  console.log(value)
}).catch((err) => {
    console.log(err)
})

```

Promise的链式调用解决了回调地狱的现象，更加清晰明了，但是仍然不够简单易用，在处理多个连续请求的时候会显得比较复杂，为了捕捉拒绝处理，会加上很多catch。

##  Async/Await

很多人称 Async/Await为非同步的终极解决方案，可见其强大。

使用 Async/Await，我们可以像编写同步的代码来写异步代码！

```javascript
const getData = async () => {
  try {
    const data = await fetching()
    console.log(data)
  } catch(err) {
    console.log(err)
  }
}
getData()
```

当然也可以这么写

```javascript
async function getData() {
  try {
    const data = await fetching()
    console.log(data)
  } catch(err) {
    console.log(err)
  }
}
getData()
```
上面的代码意思是在声明getData这个方法的时候使用了async关键词，表名getData是一个异步的方法，同时在getData方法内部，通过await关键词表明，一定要等待fetching方法完成才能够进行后面的操作，这里是console.log(data)

> 可以说async 函数就是 Generator 函数的语法糖。就是将 Generator 函数的星号（*）替换成 async，将 yield 替换成 await，仅此而已。

## 实例

下面的例子很好的说明了async的好处

>以下代码在Node8.0中运行

```javascript
let fs = require('fs')

let readFile = (filename) => {
    return new Promise((resolve, reject) => {
      fs.readFile(filename, (error, data) => {
        if (error) {
          reject(error)
        }
        resolve(data)
      })
    })
}

// Promise 连续读取两个方式读取文件
readFile('test.txt').then((value) => {
  console.log(value.toString())
}).catch((err) => {
  console.log(err)
}).then(() => {
  return readFile('ted.txt')
}).then((value) => {
  console.log(value.toString())
}).catch((err) => {
  console.log(err)
})

//Async方式读取连续两个文件文件
async function readFileByAsync(){
  try {
    let _test = await readFile('test.txt')
    let _ted = await readFile('ted.txt')
    console.log(_test.toString())
    console.log(_ted.toString())
  } catch(err) {
    console.log(err)
  }
}
readFileByAsync()
```
