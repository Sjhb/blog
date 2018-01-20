---
title: 迭代器与生成器
date: 2017-11-04 23:23:52
tags: es6
---

#### 迭代器是什么

在ES6中，迭代器是一个特殊对象，常常是用在遍历行为当中，使用这个对象可以很方便地对集合对象（数组、Set以及Map集合）和字符串进行数据操作。

> 例如
> 普通for循环：

```javascript
let arr = [1, 2, 3]
for (let i = 0; i < arr.length; i++) {
  console.log(arr[i])
}
```

> 迭代器循环

```javascript
let arr = [1, 2, 3]
for (let item of arr) {
  console.log(item)
}
```

#### 具体

迭代器一个next()方法，每次调用这个 方法都会返回一个结果对象，这个对象包含了一下属性：

- value: 调用next()方法后具体需要使用到的值，例如在上面的例子中，`value`的值就是数字1、2、3
- done: 有更多数据时返回`false`，没有数据了则会返回`true`

再没有更多数据时继续调用next()方法，返回的结果对象的value会一直是undefined，done的值会一直是false
> 使用es5实现iterator

```javascript
function createIterator (arr) {
  let i = 0
  return {
    next () {
      let done = (i >= arr.length)
      let value = done ? undefined : arr[i++]
      return {
          done: done,
          value: value
      }
    }
  }
}
let iterator = createIterator([1, 2])
iterator.next() // { done: false, value: 1 }
iterator.next() // { done: false, value: 2 }
iterator.next() // { done: true, value: undefined }
iterator.next() // { done: true, value: undefined }
```

#### 生成器是什么

生成器就是返回迭代器的函数，使用关键字yield，并且function关键字后面会跟一个`*`，例如

```javascript
function * createIterator() {
  yield 1
  yield 2
}
let iterator =  createIterator()
console.log(iterator.next()) // { value: 1, done: false }
console.log(iterator.next()) // { value: 2, done: false }
console.log(iterator.next()) // { value: undefined, done: true }
```

函数每次执行到yield语句时就会暂停执行，执行next()方法才会继续下去，直到再次遇到yield或者函数运行结束。跟我们上面自定义的es5语法创建的迭代器很相似吧。

> yield关键字只有在带有*的方法体内使用

#### for-of 循环

- 具有Symbol.iterator属性的对象称之为可迭代对象