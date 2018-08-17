---
title: JavaScript类型转换
date: 2018-08-11 22:39:35
tags: type
---

开始讨论JavasSript类型转换之前，先要明确JavasSript的几大基本数据类型：数字类型（number）、字符串（string）、布尔（boolean）、null、undefined、Symbole(ES6)，以及Object 对象。类型转换即由上面的其中一种数据类型转换为另一种。通常类型转换分显式类型转换和隐式转换。

#### 显式类型转换

显式类型转换即由我们控制，主动地进行数据的类型转换。

简单的，我们可以使用Boolean()、String()、Number()、Object()这些类型转换函数，进行类型转换

```javascript
let a = '3';
let b = 4
Number('3'); // 3
String(4); // '4'
Boolean(null); // false
Object(3); // new Number(3)
```

值得注意的是，除了null和undefined外，任何值都有toString()方法，它的作用和String()函数一样。Number类型的toString()方法还可以传递基数，来指定数字要转换的进制，默认是十进制

```javascript
(0.2).toString(2); // "0.001100110011001100110011001100110011001100110011001101"
(17).toString(16); // "11"
(17).toString(8); // "21"
```

在Number类型中，与toString()方法类似的还有toFixed()、toExponential()、toPrecision()。它们对于多出精度的部分都是四舍五入的。

- toFixed() 可把 Number 四舍五入为指定小数位数的数字（处理精度丢失的时候很好用）。
- toExponential() 对象的值转换成指数计数法。
- toPrecision() 可在对象的值超出指定位数时将其转换为指数计数法。

```javascript
(1001).toExponential(); // "1.001e+3"
(1001).toPrecision(2); // "1.0e+3" 只保留两位，其它的被四舍五入掉了
```

Number()方法进行转换的时候是基于十进制的，JavaScript还提供了其它两个方法:parseInt()、parseFloat()。parseInt()只解析整数，并且可以指定转换的基数，parseFloat()则可以解析整数和浮点数，但是不能指定基数。它们都会忽略数字前后的非数字内容。

```javascript
parseInt('0x11'); // 17
parseInt('a'); // NaN
parseFloat('1.1.21'); // 1.11
```

#### 隐式类型转换

隐式类型转换通常是程序替我们做的，而不是我们手动指定的。通常发生在运算符前后。

一些简单的规则：

- 运算符运算时，多数情况要将运算的值转换为原始值再运算。当对象类型值出现在运算符周围的时候也会发生类型转换，具体看下一节。
- `===`恒等前后不发生类型转换。
- `+`用作两个数相加时，其中一个是字符转，另外一个值如果不是数字，则会发生类型转换，转换为String类型，类似于使用了String()方法。另外，`+`作为单元运算符的时候，表示转换为数字，类似于使用了Number()方法。

#### 对象转换为原始值

- 对象转换为布尔类型：均为true。 new Boolean(false)转换为布尔类型也是true，因为它时一个对象.

- 转换为字符串：

调用对象的toString()方法，需要注意的是，宿主对象（运行环境提供的全局对象）有各自的算法转换为字符串和数字。

```javascript
[1,2,3].toString(); // "1,2,3"
(function(){console.log(1)}).toString(); // "function(){console.log(1)}"
/^123$/.toString(); // "/^123$/"
new Date(1997,01,01).toString(); // "Sat Feb 01 1997 00:00:00 GMT+0800 (中国标准时间)"
({}).toString(); // "[object Object]"

({}) == "[object Object]" // true,前面发生了类型转换
```

如果对象没有toString()方法，或者toString()方法返回的不是一个原始值，JavaScript会调用对象的valueOf()方法（如果存在的话）。valueOf()方法的返回值如果是原始值，JavaScript就将这个值转化为字符串（如果本身不是字符串的话），并返回字符串结果。

>> valueOf()方法: 如果存在任意的原始值，它就默认将对象转换为表示它的原始值。大多数的对象没有这个原始值，这时valueOf()只是简单的返回这些对象本身。例如：`(new Date(1997,01,01)).valueOf()`返回854726400000，表示1970年1月1日以来的毫秒数（不是时间戳）

- 转换为数字：

如果对象存在valueOf()方法，并且这个方法如果返回原始值，则JavaScript将这个原始值转换为数类型（如果需要的话），返回这个数字.

否则，如果对象具有一个toString()方法,并且这个方法如果返回原始值,则JavaScript将这个原始值转换为数字（如果需要的话），返回这个数字.

否则，抛出异常。

且看下面的例子
```javascript
Number([]); // 0
```
因为数组具有valueOf()方法，但是返回的值是对象，所以在执行转换操作的时候执行toString()方法。toString()方法返回空字符串，转换为数字类型就是0

- 未指明要转换到的数据类型，只是要转换为原始值：任何对象（除去日期对象）都会先调用valueOf()方法，然后是toString()方法，不管得到的原始值是否可直接使用，它们都不会被进一步转换为字符串或者是数字。

### 最后

说出下面的运行结果

```javascript
let foo = {a:2}
foo + 1

foo.valueOf = function(){return 1}
foo + 1
foo == 1
foo ==== 1

foo.valueOf = function(){return '1'}
foo + 1

foo.valueOf = function(){return {}}
foo.toString = function(){return {}}
foo + 1
```






