---
title: 正则表达式（三）
date: 2017-09-17 00:00:42
tags: web regexp javscript
---

## 字符串对象的正则方法

>字符串对象的正则方法共有4个：match()、replace()、search()和split()。在ES6中，这4个方法，在语言内部全部调用RegExp的实例方法，从而做到所有与正则相关的方法，全都定义在RegExp对象上。

### String.prototype.match

> es6中 String.prototype.match 调用的是 RegExp.prototype[Symbol.match]

match方法与exec方法非常类似：对字符传检索并把结果返回以数组返回，失败则为null。这个数组带有index和input属性：index表示匹配的第一个字符的位置，input表示检索的字符串。

```javascript
let s1 = 'abcd'
let s2 = 'efgh'
let r = /c(d)/
s1.match(r) // ["cd", "d", index: 2, input: "abcd"]
s2.match(r) // null
```

> 当匹配的正则对象带有g表示，match会一次返回所有的结果
> 这时设置lastIndex是无效的

```javascript
let s1 = 'abcd'
let r = /a|d/g
s1.match(r) // ["a", "d"]
r.lastIndex = 1
s1.match(r) // ["a", "d"]
```

> y标识在正则对象中使用的时候，macth可以多次执行，并从上次成功的位置开始，否则回到开头位置。即全局重复多次匹配
> lastIndex这时设置是有效的

```javascript
let s1 = 'abcd'
let r = /a|b/y
s1.match(r) // ["a", index: 0, input: "abcd"]
r.lastIndex = 1
s.match(r) // ["b", index: 1, input: "abcd"]
r.lastIndex // 2
s.match(r) // null
r.lastIndex // 0
```

### String.prototype.search

> es6中 String.prototype.search 调用的是RegExp.prototype[Symbol.search]

search方法会返回匹配对象在检索的字符串中首次出现的位置，如果没有匹配到则返回-1

```javascript
let reg1 = /t/
let reg2 = /v/
'test'.search(reg1) // 0
'test'.search(reg2) // -1
```

g和y标识符在这个方法中无效

### String.prototype.replace

> es6中 String.prototype.replace 调用的是RegExp.prototype[Symbol.replace]

replace方法会返回替换目标字符串指定内容之后的字符串，它会接收两个参数：第一个参数指定匹配的模式，即正则对象；第二个参数指定用于替换的内容（string.replace(reg, replacement)）。

```javascript
let str = 'test'
let reg3 = /t/g
let reg4 = /t/
str.replace(reg3, 'a') // 'aesa'
str.replace(reg4, 'a') // 'aest'
```

未加g标识repalce方法只会替换匹配到的第一个对象，加了g标识则会替换所有匹配到的内容

> replace 方法的第二个参数可以使用$符号来指定内容
|符号|说明|
|$&| 指代匹配的子字符串。
|$`| 指代匹配结果前面的文本。
|$'| 指代匹配结果后面的文本。
|$n| 指代匹配成功的第n组内容，n是从1开始的自然数。
|$$| 指代美元符号$。

```javascript
let str = 'abc'
let reg = /b/g
str.replace(reg, '$&t') // 'abtc'
str.replace(reg, '$`t') // 'aatc'
str.replace(reg, '$\'t') // 'actc'
str.replace(/(b)(c)/, '$2$1') //'acb'
str.replace(reg, '$$') //'a$c'
```

> replace 方法第二个参数可以是函数，函数的返回值将会作为替换的内容


```javascript
let str = 'abc'
let reg = /b/g
str.replace(reg, function (matched) {
    return matched.toUpperCase()
}) // 'aBc'
```

这个函数接收多个参数：第一个参数是匹配的内容，第二部分是多个参数：捕捉到的组匹配（有多少个组匹配，就有多少个对应的参数），倒数第二个参数是匹配的位置，最后一个是原字符串


```javascript
let str = 'abc.jpg'
let reg = /(\w+)(\.)(\w+)/g
str.replace(reg, function (matched, $1, $2, $3, position, input) {
    console.log($1) // 'abc'
    console.log($2) // '.'
    console.log($3) // 'jpg'
    console.log(position) // '0'
    console.log(input) // 'abc,jpg'
    return $1 + $2 + 'png'
}) // 'abc.png'
```

### String.prototype.split

Split 方法会按匹配规则返回分割后的字符串

```javascript
let str = 'abc'
str.split(/b/) // ["a", "c"]
```

Split方法接收两个参数：第一个是匹配规则，第二个是可选参数，指定要返回数组的最大成员数

```javascript
let str = 'abc'
str.split(/b/, 1) // ["a"]
```

如果有组匹配，则会把匹配到的内容也返回

```javascript
let str = 'abbbc'
str.split(/(b*)/) // ["a","bbb", "c"]
```

> 另外split匹配到的字符串位于被检索的字符串的开头或者尾部位置，则返回的数组第一个元素或者最后一个元素是空的字符串

```javascript
let str = 'bbb'
str.split(/b/) // ["", "", "", ""]
```