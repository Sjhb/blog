---
title: 使用Mocha测试(二)
date: 2018-04-05 00:31:51
tags: 测试
---

#### 挂起测试

如果测试用例没有传入回调函数，那么这个测试用例运行时将显示为挂起状态（pedding）
```javascript
describe('Array', function () {
  describe('#indexOf()', function () {
    // 挂起的测试用例
    it('未找到时应当返回-1' ,function (done) {
        done()
    })
} )
})
```

#### 独占测试

通过使用`.only`后缀，可以指定哪些测试套件或者测试用例可以运行，哪些不会运行

```javascript
const assert = require('assert')
describe('only test', function () {
    describe.only('descirbe run', function(){
        it('not run', function () {
            assert(true)
        })
        it.only('run', function () {
            assert(true)
        })
    })
    describe('describe not run', function () {
        it('not run', function () {
            assert(true)
        })
    })
})
```
![](/images/test-4.jpg)

#### 跳过测试

与独占测试相对的，可以使用`.skip`指定要跳过的测试套件或者测试用例,请看下面这个例子。

```javascript
const assert = require('assert')
describe('skip test', function () {
    describe('descirbe not skip', function(){
        it.skip('not run', function () {
            assert(true)
        })
        it('run', function () {
            assert(true)
        })
    })
    describe.skip('describe skip', function () {
        it('not run', function () {
            assert(true)
        })
    })
})
```
输出结果如图
![](/images/test-5.png)

被标记`.skip`的测试用例或者被标记`.skip`的测试套件中的测试用例都会在运行结果中显示为`pedding状态`。

可根据实际运行情况编写判断语句来决定是否执行测试，也可以在hook钩子中使用`skip`
```javascript
const assert = require('assert')
describe('skip test', function () {
    describe('descirbe not skip', function(){
        it('condition skip', function(){
            if (false) {
                assert(false)
            } else {
                this.skip()
            }
        })
    })
    before ('hook before', function () {
        if (true) {
            console.log('hook before pass')
        } else {
            this.skip()
        }
    })
})
```

#### 重试测试

存在这样的情况：失败的测试用例我们希望重新执行一次或者好几次，Mocha提供了这样的操作，可以为失败的测试用例指明要重试的次数

```javascript
const assert = require('assert')
describe('retry test', function () {
    let a = 1
    this.retries(4)
    beforeEach(function(){
        console.log(`retry count:${a}`)
        a++
    })
    it('retry it', function () {
        if (a==4) {
            assert(true)
        } else {
            assert(false)
        }
        
    })
})
```
最终测试结果为测试通过

#### 动态生成测试

根据需要，我们可以动态的生成测试用例，达到类似与参数化的测试功能。
```javascript
var assert = require('assert')
describe('generate test', function() {
  var tests = [
    {args: [1, 2],       expected: 3},
    {args: [1, 2, 3],    expected: 6},
    {args: [1, 2, 3, 4], expected: 10}
  ]
  tests.forEach(function(test) {
    it('correctly adds ' + test.args.length + ' args', function() {
      var res = test.args.reduce(function (pre, cur) {
        return pre + cur
      })
      assert.equal(res, test.expected)
    })
  })
})
```
上面的例子中，生成了三个测试用例。

#### 测试耗时

每次运行测试都会显示测试结果，我们可以通过`slow()`指明多长时间才算是耗时较长

```javascript
const assert = require('assert')
describe('retry test', function () {
    this.slow(100)
        it('retry it', function (done) {
            setTimeout(function () {
                done()
            }, 1000)
        })
})
```
执行结果如图，这个测试用了一秒多时间，被标记为红色（耗时长）
![](/images/test-6.png)