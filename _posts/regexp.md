---
title: 正则表达式（一）
date: 2017-09-10 00:01:34
tags: JavaScript,regular-expression
---

## 声明方式

1. 字面量，使用反斜杠（/）表示开始和结束

```javascript
let regexp = /test/
```

2. 使用RegExp的构造函数

```javascript
let regexp = new RegExp('test')
```

> 以上两种方式的区别在于：字面量写法在js代码编译时新建正则表达式，构造函数方式在运行时新建正则表达式
> 什么意思呢？就是说字面量写法一开始就新建正则表达式，当正则表达式保持不变的时候，字面量的写法可以获得更好的性能，而当正则表达式模式是可变的时候最好使用构造函函数方式

### 标志

申明正则表达式的时候可以带上参数，这些参数可以单独或者一起使用。

| 标志     | 说明                                |
| ------ | --------------------------------- |
| g      | 全局搜索                              |
| i      | 不区分大小写搜索                          |
| m      | 多行搜索                              |
| y(es6) | 执行“粘性”搜索,匹配从目标字符串的当前位置开始，可以使用y标志。 |
| u(es6) | 将模式视为Unicode序列点的序列                |

> 字符串对象的match方法对字符串进行正则匹配，返回匹配结果。
- g:

```javascript
let regSimple = /ab/
let regGlobal = /ab/g
let resultSimple ='ac ababab abc'.match(regSimple)
let resultGlobal ='ac ababab abc'.match(regGlobal)
console.log(resultSimple) // Array(1) ["ab"]
console.log(resultGlobal) // Array(4) ["ab", "ab", "ab", "ab"]
```

- i:

```javascript
let regSimple = new RegExp('ab')
let regGlobal = new RegExp('ab', 'i')
let resultSimple ='aB'.match(regSimple)
let resultGlobal ='ab'.match(regGlobal)
console.log(resultSimple) // null
console.log(resultGlobal) // Array(1) ["ab"]
```

- m:

```javascript
let regSimple = new RegExp('^ab') // ^表示ab必须位于字符串的开头
let regGlobal = new RegExp('^ab', 'm')
let resultSimple ='aB\nab'.match(regSimple)
let resultGlobal ='aab\nab'.match(regGlobal) // 多行模式，\n表示换行
console.log(resultSimple) // null
console.log(resultGlobal) // Array(1) ["ab"]
```

- y:
- u:

> y和u标识比较复杂，后续专开文章解释

## 正则表达式对象的属性和方法

![reg](/images/regexp.png)


### 属性

- 与标识相关的属性

| 属性           | 说明                        |
| ------------ | ------------------------- |
| multiline    | 返回一个布尔值，表示是否设置了m修饰符，该属性只读 |
| ignoreCase   | 返回一个布尔值，表示是否设置了i修饰符，该属性只读 |
| global       | 返回一个布尔值，表示是否设置了g修饰符，该属性只读 |
| unicode(es6) | 返回一个布尔值，表示是否设置了u修饰符，该属性只读 |
| sticky(es6)  | 返回一个布尔值，表示是否设置了y修饰符，该属性只读 |
| flags(es6)   | 返回一个字符串，表示设置的标识，该属性只读     |

```javascript
let test= new RegExp('TESET','imugy')
console.log(test.flags) // gimuy
console.log(test.multiline) // true
console.log(test.ignoreCase) // true
console.log(test.global) // true
console.log(test.sticky) // true
console.log(test.unicode) // true
```

- 与标识相关的属性

| 属性        | 说明                                    |
| --------- | ------------------------------------- |
| lastIndex | 返回下一次开始搜索的位置。该属性可读写，但是只在设置了g和y修饰符时有意义 |
| source    | 返回正则表达式的字符串形式（不包括反斜杠），该属性只读           |

```javascript
let test= new RegExp('test','gy')
console.log(test.lastIndex) // 0
test.lastIndex = 1
console.log(test.lastIndex) // 1
console.log(test.source) // test
```

### 方法

- test():返回一个布尔值，表示当前模式是否能匹配参数字符串。

```javascript
let reg = /ab/
console.log(reg.test('abc')) // true
console.log(reg.test('acd')) // false
```

> 正则对象的lastIndex会在设置了g和y标识的时候记录下次搜索的起始位置，并且可以修改，具体可看下面的例子

```javascript
let reg = /ab/g
console.log(reg.lastIndex) // 0
console.log(reg.test('abcab')) // true
console.log(reg.lastIndex) // 2
console.log(reg.test('abcab')) // true
console.log(reg.lastIndex) // 5
console.log(reg.test('abcab')) // false
console.log(reg.lastIndex) // 0
console.log(reg.test('abcab')) // true
reg.lastIndex = 5
console.log(reg.lastIndex) // 5
console.log(reg.test('abcab')) // false
```

- exec():返回匹配结果。如果发现匹配，就返回一个数组，成员是每一个匹配成功的子字符串，否则返回null。

```javascript
let reg = /ab/
console.log(reg.exec('abc')) // array(1) ["ab"]
console.log(reg.exec('acd')) // null
```
