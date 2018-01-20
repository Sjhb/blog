---
title: 正则表达式（四）
date: 2017-09-21 09:29:47
tags: web javascript
---

## 匹配规则

### 元字符

在正则表达式中有些字符就代表了它本身的意义,比如a、b、c,而有些具有特殊含义，要代表本身字面含义就需要转义。

\: 转义符，将下一个字符标记为一个特殊字符、或一个原义字符、或一个向后引用、或一个八进制转义符,\字符要表示它本身就需要使用‘\\\’

```javascript
/\n/.test('n') //false 
/\n/.test('\\n') //false 
/\n/.test('\n') //true
```

^、$:位置字符，表示匹配的子表达式位于开头和结尾的位置，设置m标识时也表示位于换行符（\n）和回车符（\r）的前后,^位于'[]'中时表示非，如‘[^a]’表示除a以外的字符

```javascript
/^a/.test('abc') // true
/^a/m.test('bc\na') // true
/[^a]/.test('b') // true
/a$/.test('cba') // true
/a$/m.test('cbabc') // false
/a$/m.test('cba\nbc') // true
```

*: 表示前面的子表达式匹配零次或者多次

```javascript
'ttccttt'.match(/t*/g) // ["tt", "", "", "ttt", ""]
```

+: 表示前面的子表达式匹配一次或者多次

```javascript
'ttccttt'.match(/t+/g) // ["tt", "ttt"]
```

?: 表示前面的子表达式匹配零次或一次,这个字符可以用来限制*和+这两个字符，使之满足条件就停止匹配

```javascript
'ttccttt'.match(/t?/g) // ["t", "t", "", "", "t", "t", "t", ""]
'tttt'.match(/t*/) // ["tttt", index: 0, input: "tttt"]
'tttt'.match(/t*?/) // ["", index: 0, input: "tttt"]
'tttt'.match(/t+/) // ["tttt", index: 0, input: "tttt"]
'tttt'.match(/t+?/) // ["t", index: 0, input: "tttt"]
```

x{n}: n是非负整数，表示x代表的表达式确定匹配的次数

```javascript
'ttccttt'.match(/t{3}/g) // ["ttt"]
```

x{n,}: n是非负整数，表示x代表的表达式至少匹配的次数

```javascript
'ttccttt'.match(/t{3,}/g) // ["ttt"]
```

x{n,m}: n,m是非负整数，表示x代表的表达式匹配的次数大于等于n，小于等于m

```javascript
'ttcctttccttttt'.match(/t{3,4}/g) // ["ttt", "tttt"]
```

.: 匹配除回车（\r）、换行(\n) 、行分隔符（\u2028）和段分隔符（\u2029）以外的所有字符。

```javascript
/a.c/.test('abc') // true
/a.c/.test('adc') // true
```

|：或字符，表示或者关系

```javascript
'abc'.match(/a|c/g) // ["a", "c"]
```

(x): 圆括号表示组匹配，即按组的内容来匹配

```javascript
'abc'.match(/(ab)\w/) // ["abc", "ab", index: 0, input: "abc"]
'adc'.match(/(ab)\w/) // null
```

(?:xx): 这种格式表示匹配xx所代表的表达式并且不返回，这是比较有用的，特别是与或（|）组合使用的时候

```javascript
'word'.match(/wor(?:d|r)/) // ["word", index: 0, input: "word"]
'word'.match(/wor(d|r)/) // ["word", "d", index: 0, input: "word"]
```

/reg(?=xx)/: 正向预查，这种格式表示reg匹配的字符串周围的字符需要匹配xx代表的表达式，例如/(?=tt)ab/表示ab的前面必须有tt两个字符否则失败。且这是一种非获取匹配，xx代表的值不返回

```javascript
'word'.match(/wo(?=rd)/) // ["wo", index: 0, input: "word"]
'word'.match(/wo(?=d)/) // null
```

/reg(?!xx)/: 反向预查，与正向预查相反，reg匹配的周围的字符串需要不和xx所代表的表达式匹配，这也是一种非获取匹配

```javascript
'word'.match(/wo(?!rd)/) // null
'word'.match(/wo(?!d)/) // ["wo", index: 0, input: "word"]
```

[]: 方括号表示字符的集合，只要满足其中任何一个即可

```javascript
'abc'.match(/[bc]/g) // ["b", "c"]
```

> 方括号中脱字符（^）、连字符（-）这两个字符都具有特殊含义，脱字符表示取反，连字符表示取一定范围内的字符,连字符必须前后都有字符，否则就只代表它本身的含义

```javascript
/[^w]/.test('word') // true
/[^w]/.test('w') // false
'45s45wg5'.match(/[0-9]/g) // ["4", "5", "4", "5", "5"]
```

### 预定义模式

预定义模式就是指一些常见模式的简写

|模式|说明|
|------|-----|
|\d| 匹配0-9之间的任一数字，相当于[0-9]。
|\D| 匹配所有0-9以外的字符，相当于[^0-9]。
|\w| 匹配任意的字母、数字和下划线，相当于[A-Za-z0-9_]。
|\W| 除所有字母、数字和下划线以外的字符，相当于[^A-Za-z0-9_]。
|\s| 匹配空格（包括制表符、空格符、断行符等），相等于[\t\r\n\v\f]。
|\S| 匹配非空格的字符，相当于[^\t\r\n\v\f]。
|\b| 匹配词的边界。
|\B| 匹配非词边界，即在词的内部。

一些例子

```javascript
// 判断手机号格式是否合法
/^\d{11}$/.test('12345678912') // true
/^\d{11}$/.test('1234567891s') // false
/^\d{11}$/.test('1234567891') // false

// 去除字符串两端空格
'   word    '.replace(/^\s+|\s+$/g , '') // "word"

// 过滤字符串，将字符串全变成大写
'AsFsAeWErFAG'.replace(/\w+/, function (matched) {
    return matched.toUpperCase()
}) // "ASFSAEWERFAG"

/\bword/.test('hello word') // true
/\bord/.test('hello word') // false
```

### 组匹配

括号()表示组匹配，括号中的模式表示分组的内容

```javascript
/(t+)/.exec('rrrrtttt') // ["tttt", "tttt", index: 4, input: "rrrrtttt"]
```

可以使用\n来引用括号内匹配的内容，n的值代表第几个括号的内容，括号嵌套的时候n的值代表括号的层数，如\1代表最外层括号

```javascript
/(.)b(.)\2b\1/.test('abccba') // true
/a((..)\2)\1/.test('abcbcbcbc') // true
```

### 特殊字符

对一些不能打印的特殊字符，需要使用特殊的表达方式

|表达式|说明|
|-----|-----|
|\cX| 表示Ctrl-[X]，其中的X是A-Z之中任一个英文字母，用来匹配控制字符。
|[\b]| 匹配退格键(U+0008)，不要与\b混淆。
|\n| 匹配换行键。
|\r| 匹配回车键。
|\t| 匹配制表符tab（U+0009）。
|\v| 匹配垂直制表符（U+000B）。
|\f| 匹配换页符（U+000C）。
|\0| 匹配null字符（U+0000）。
|\xhh| 匹配一个以两位十六进制数（\x00-\xFF）表示的字符。
|\uhhhh| 匹配一个以四位十六进制数（\u0000-\uFFFF）表示的unicode字符。