---
title: 使用Mocha测试(一)
date: 2018-04-02 23:00:00
tags: 测试
---

###  Mocha

Mocha是一个能够运行在Node和浏览器中的多功能的JavaScript测试框架，它让异步测试简单且有趣。Mocha连续地运行测试，并给出灵活而精确的报告，同时能够将错误精确地映射到测试用例上。

Mocha不提供断言工具库，需要自己选择使用。

官方文档地址：[mochajs](https://mochajs.org/)

#### 安装

```bash
$ npm i -g mocha
```

#### 一个很简单的例子

下面的例子中，引入Node中`assert`断言模块，测试内容为判断`[1,2,3].indexOf(4)`结果与`-1`是否相等，预期结果为相等，不相等时测试将失败。

`describe`块称为"测试套件"（test suite），表示一组相关的测试。它是一个函数，第一个参数是测试套件的名称，第二个参数是一个实际执行的函数。

`it`块称为"测试用例"（test case），表示一个单独的测试，是测试的最小单位。它也是一个函数，第一个参数是测试用例的名称，第二个参数是一个实际执行的函数。

```javascript
// test.js
var assert = require('assert')
describe('Array', function () {
    describe('#indexOf()', function() {
        it('should return -1 when the value is not present', function () {
            assert.equal([1,2,3].indexOf(4), -1)
        })
    })
})
```

运行`mocha test.js`进行测试，结果如图:
![](/images/test-1.jpg)

#### 注册命令

`Mocha`默认会运行执行目录下的test子目录的测试脚本

当然，可以在`package.json`中注册命令:

```json
"scripts": {
    "test": "mocha"
}
```

将之前的test.js测试脚本放在test文件夹下面，这样就可直接运行`npm run test`命令了。

#### 在异步回调中测试

在异步回调的代码中测试很简单，在异步代码中，只需执行Mocha提供的done函数，即可返回测试结果，例如下面的例子

```javascript
describe('async test', function () {
    it('timeout in 1000ms', function (done) {
        setTimeout(function () {
            done(new Error('error'))
        }, 1000)
    })
})
```
如果done函数没有调用，测试套件会一直等待下去，直到超时，然后测试失败。

Mocha默认回调超时时间是2s，可以通过设置改变超时时间，可以查看一下例子，测试会通过
```javascript
describe('async test', function () {
    it('timeout changed', function (done) {
        this.timeout(6000)
        setTimeout(function () {
            done()
        }, 3000)
    })
})
```

#### Promise

这一些测试中，异步代码返回的是Promise，与其在promise中调用done()函数，不如直接返回Promise对象，Mocha会自动识别

```javascript
describe('promise test', function () {
    it('return promise', function () {
        return new Promise((resolve, reject) => {
            // resolve(1)
            reject(1)
        })
    })
})
```
执行结果如下：
![](/images/test-2.png)

#### 箭头函数

不建议使用箭头函数，因为这样会使得测试代码难以重构。


#### 同步代码

在同步代码中，如果省略掉回调函数，Mocha会自动执行后面的测试，例如在下面的例子中，如果测试程序不出错，Mocha会一直执行直到测试执行完毕。

```javascript
var assert = require('assert')
describe('sync test', function () {
    it('sync test', function () {
        assert.equal(1, 1)
        assert.equal(2, 2)
        assert.equal(3, 4)
    })
})
```
如果有一处报错，测试就会失败

#### 生命周期钩子

Mocha提供了四个钩子函数（按执行顺序排列）：
- `before`：当前作用域内所有测试用例之前执行
- `beforeEach`：当前作用域内每个测试用例之前执行
- `afterEach`： 当前作用域内每个测试用例结束之后执行
- `after`： 当前作用域内所有测试用例结束之后执行
同类钩子按书写的先后顺序执行

```javascript
const assert = require('assert')
describe('hook test', function () {
    let user = ''
    before(function () {
        console.log('hook before')
        user = ''
    })
    beforeEach(function () {
        console.log('hook beforeEach')
        user = 'wh'
    })
    it(function () {
        assert.equal(user, 'wh')
    })
    afterEach(function () {
        console.log('hook afterEach')
        user = ''
    })
    after(function () {
        console.log('hook after')
        user = ''
    })
})
```
执行结果如图：
![](/images/test-3.png)

生命钩子的描述信息：生命钩子调用时可能发生错误，申明一些钩子的信息是有必要的，上面的例子可修改为下面的方式：

```javascript
const assert = require('assert')
describe('hook test', function () {
    let user = ''
    before('before hook', function () {
        console.log('hook before')
        user = ''
    })
    beforeEach('beforeEach hook', function () {
        console.log('hook beforeEach')
        user = 'wh'
    })
    it('user test', function () {
        assert.equal(user, 'wh')
    })
    afterEach('afterEach hook', function () {
        console.log('hook afterEach')
        user = ''
    })
    after('after hook', function () {
        console.log('hook after')
        user = ''
    })
})
```

> 如果要在钩子中执行异步操作，只需要像在普通测试用例中执行done函数就ok了

#### 全局钩子

在任何一个测试文件中，只要在跟describe()作用域外声明的生命周期钩子，其作用域为全局的，它的回调函数会在所有的测试用例运行之前运行，无论（这个测试用例）处在哪个文件（这是因为Mocha有一个隐藏的describe()作用域，称为“根测试套件 root suite”）。

#### 延迟的跟测试套件

如果你需要在所有测试套件运行之前进行一些异步操作，你可以延迟根测试套件。以--delay参数运行，这会在全局注入一个特殊的回调函数run()：
```javascript
const assert = require('assert')
setTimeout(function () {
  console.log('before test')
  run()
}, 1000)
describe('delay test', function(){
  it('container', function(){
     assert.equal(1, 1)
  })
})
```
运行命令：`mocha --delay`

