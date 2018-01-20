---
title: es6:模块封装代码
date: 2017-08-26 16:57:12
tags: web es6
---

> 模块（module）指的是自动运行在严格模式下没有办法进行退出的代码块。模块概念的引进是为了解决作用域的问题。
>
> 与脚本的区别：模块只是导出和导入所需要的绑定，而不需要将所有的东西都放到一个文件当中。
>
> 1. 在一个模块内，顶层的this的值为undefined
> 2. 不支持HTML代码注释

>es6的Module 还没有被js引擎实现，Node的Module遵循CommonJs规范，使用module.exports导出，require导入（可参照这篇文章：[Node中没搞明白require和import，你会被坑的很惨](http://imweb.io/topic/582293894067ce9726778be9)）
>
>本文使用babel转化es6未被实现的语法（参考至[找回 Node.js 里面那些遗失的 ES6 特性](http://taobaofed.org/blog/2016/01/07/find-back-the-lost-es6-features-in-nodejs/)）

### Node环境

 基本用法：

- 导出:使用export关键字 
- staff.js

```javascript
export let red = '#ff0000'
export const green = '#00ff00'
export function getBule() {
    return '#0000ff'
}
let Person = class {
    constructor (name, sex) {
        this.name = name
        this.sex = sex
    }
}
export {Person}

// 错误示例,会报错，不能够进行解构
export 1

let a = 'a'
export a
```

- 导入:使用import关键词
- index.js

```javascript
import {red, green, getbule, Person} from './staff.js'
console.log('red:\t' + red + '\n')
console.log('green:\t' + green + '\n')
console.log('bule:\t' + getbule() + '\n')
console.log(new Person('ZhangSan', 'male'))
// 输出
// red:    #ff0000
// green:  #00ff00
// bule:   #0000ff
// Person { name: 'ZhangSan', sex: 'male' }
```

> import 绑定的值无法进行改变，也无法再定义同名的变量,
>
> 但是可也像闭包一样修改其内的值

```javascript
// fun.js
export let age = 20
export function setAge (inAge) {
    r age = inAge
}
// index.js
import {age, setAge} from './fun.js'

let age = 21 // 报错
age = 21 // 报错
console.log(age)// 20
console.log(setAge(21))// 21
```

导入整个模块（也被称为命名空间导入）：

```javascript
import * as staff from './module/staff.js'

console.log('red:\t' + staff.red + '\n')
console.log('green:\t' + staff.green + '\n')
console.log('bule:\t' + staff.getbule() + '\n')
console.log(new staff.Person('ZhangSan', 'male'))
// 输出
// red:    #ff0000
// green:  #00ff00
// bule:   #0000ff
// Person { name: 'ZhangSan', sex: 'male' }
```

> 导入时模块会被实例化，然后保存在内存当中，再次使用import只是在使用import实例化的对象

count.js

```javascript
let count = 1
export function up () {
    return ++count
}
export function upa () {
    return ++count
}
export function upb () {
    return ++count
}
```

index.js

```javascript
import {up} from './module/count'
console.log(up())
import {upa} from './module/count'
console.log(upa())
import {upb} from './module/count'
console.log(upb())
// 输出
// 2
// 3
// 4
```

使用as进行重命名:

```javascript
// person.js
let Person = class {
    constructor (name, sex) {
        this.name = name
        this.sex = sex
    }
}
export {Person as Human}

// index.js
import {Human} from "./module/color";
import {Human as Person} from "./module/color";
console.log(new Human('human', '10'))
// Person { name: 'human', sex: '10' }
console.log(new Person('person', '10'))
// Person { name: 'person', sex: '10' }
```

导入和导出默认值：

```javascript
// 导出：staff.js
// 写法一
export default function () {
    return 'staff'
}

// 写法二
function staff() {
    return 'staff'
}
export default staff

// 写法三
function staff() {
    return 'staff'
}
export {staff as default}

// 导入：index.js
import staff from './staff'
console.log(staff()) // staff
```

重新导出绑定

```javascript
// 写法一
import {staff} from './staff'
export {staff}
// 写法二
export {staff} from './staff'
export {staff as some} from './staff'
export {*} from './staff'
```

无绑定导入(修改一些全局变量或者实现浏览器并不支持的原生API的代码)：

```javascript
// fun.js
String.prototype.sayHey = function () {
    return 'Hey'
}
// index.js
import './staff'
console.log('test'.sayHey())// Hey
```

### 浏览器环境加载模块

使用script元素，需指定type=module，且默认defer属性

```html
// demoA.js
export let a = 'demoA'
// demoB.js
export let b = 'demoB'
// index.js
<!DOCTYPE html>
<script type="module" src="./demoA.js"></script>
<script>
    import {b} from './demoB.js'
    console.log(b)
</script>
```

> 目前浏览器都不支持，所以没法实践测试

加载顺序：

> 1. 下载解析demoA.js
> 2. 递归下载并解析demoA.js中导入的模块
> 3. 解析内联模块
> 4. 递归下载并解析内联模块中导入的资源
> 5. 下载解析demoB.js
> 6. 递归下载并解析demoB.js中导入的资源

文档解析完成之后执行的操作

> 1. 递归执行demoA.js中导入的资源
> 2. 执行demoA.js
> 3. 递归执行内联模块中导入的资源
> 4. 执行内联模块
> 5. 递归执行demoB.js中导入的资源
> 6. 执行demoA.js

模块说明符解析，必须依次开头，否则资源加载报错

>/ ：根目录
>
>./ ：当前目录
>
>../ ：父目录