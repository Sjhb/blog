---
title: 正则表达式（二）
date: 2017-09-16 21:45:33
tags: regexp web javascript
---

### 深入exec()方法

使用正则对象的exec()方法去检索字符串会返回一个数组，如果匹配成功则这个数组的元素是与正则表达式匹配的文本，并且这个数组会携带两个属性：index和input，index是匹配到文本的第一个字符的位置，input则是被检索的字符串。匹配失败exec()方法会返回null

> 使用组匹配的时候，返回的数组有两个元素，第一个是整个匹配的字符串，第二个是与组内容相匹配的内容。

```javascript
let demo = /t(e+)/.exec('estest')
demo.length // 2
demo[0] // 'te'
demo[1] // 'e'
demo.index // 2
demo.input // 'estest'
```

> 当使用g和y标识的时候，如同test()方法一样，正则对象可以对字符串进行多次检索，检索的位置从上一次结束的位置开始。如果到达字符串的结尾，会返回null并重新从字符传的头部开始

```javasctipt
var reg = /t/g
reg.exec('tet') // ['t', index: 0, input: 'tet']
reg.lastIdnex // 1
reg.exec('tet') // ['t', index: 2, input: 'tet']
reg.lastIdnex // 3
reg.exec('tet') // null
reg.lastIdnex // 0
reg.exec('tet') // ['t', index: 0, input: 'tet']
reg.lastIdnex // 1
```

### /y标识：“粘连”（sticky）。

y标识与g标识类似，但是功能不同，y表示粘性匹配，什么意思呢。当使用y标识的时候，test()和exec()匹配的位置会从lastIndex指定的位置匹配，如果失败会重置lastIndex的值为0，而不会跳过这些匹配失败的字符;成功则会返回相应的值并把lastIndex的值指向匹配成功的字符串的下一个字符的位置。

```javascript
let y =  /q/y
let g = /q/g

y.test('wqa') // false
y.lastIndex // 0
y.lastIndex = 1
y.test('wqa') // true
y.lastIndex // 2
y.test('wqa') // false
y.lastIndex // 0

g.test('wqa') // true
g.lastIndex // 2
```
> y标识表示下一次匹配要从上一次匹配成功的下一个位置开始，不会漏掉中间的字符，如不合法字符，而g表示则会忽略掉这些字符。

### /u标识

u标识表示‘Unicode模式’，用来处理那些unicode码大于\uFFFF的字符，比如四个字节的UTF-16编码

```javascript
/^\uD83D/u.test('\uD83D\uDC2A') // false
/^\uD83D/.test('\uD83D\uDC2A') // true
```

'\uD83D\uDC2A'表示的是一个字符，加上u表示后就会正确识别字符编码，否则没加u标识就会认为是两个字符。
