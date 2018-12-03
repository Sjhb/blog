---
title: Virtual Dom 上手体验
date: 2018-11-25 23:25:15
tags:
---

#### 什么是Virtual Dom

虚拟DOM（Virtual Dom 以下简称VD）其实就是Javascript对象，居于真实DOM与原始代码之间，类似桥梁的作用。MVVM框架中，我们编写应用代码，管理数据，框架根据我们的代码生成VD树，进而根据VD树生成真实的dom树渲染到浏览器中。当应用数据更新时，VD树也会对应更新，然后框架会调用Diff算法比较真实dom树和VD树之间的差异，更新一部分真实DOM

#### 为什需要VD

更新一个网页会经历三个过程：

1. js计算
2. 生成渲染树
3. 绘制页面

如果没有VD，每一次我们需要更新页面的时候，框架不知道我们的想要的页面是怎样的，每次都需要重新计算渲染，更新整个DOM树。DOM操作性能开销是很大的。而使用VD，根据Diff算法算出哪些dom需要更新，将多个更新操作合并为一个，则能有效地减少性能消耗。

另一方面，我们希望关心核心代码逻辑，对于dom操作，如果有一种合理的方法代替我们操作，那将会我们减少很多工作。

#### VD上手体验

首先，我们先根据jsx或者html代码生成中间代码，然后我们编写转化函数，得到VD。

- 创建项目并安装依赖，这里我们使用babel来进行转化：

```bash
$ npm init vd test
$ yarn global add babel-cli
$ yarn add -dev babel babel-plugin-transform-react-jsx
```

增加配置文件.babelrc

```javascript
{
  "plugins": [
    ["transform-react-jsx", {
      // pragma[string]，默认为 React.createElement。 替换在编译 JSX 表达式时使用到的函数。
      "pragma": "h"
    }]
  ]
}
```

- 准备原始代码: `test.js`

```javascript
module.exports = function render() {
  return (
      <div>
          Hello World
          <ul>
              <li id="1" class="li-1">
                  第1
              </li>
          </ul>
          <footer>
              footer
          </footer>
      </div>
  )
}
```

- 运行命令：`babel test.js --out-file test-m.js`，将会得到以下内容：

```javascript
module.exports = function render() {
    return h(
        "div",
        null,
        "Hello World",
        h(
            "ul",
            null,
            h(
                "li",
                { id: "1", "class": "li-1" },
                "\u7B2C1"
            )
        ),
        h(
            "footer",
            null,
            "footer"
        )
    )
}
```

- 现在我们需要编写`h`函数，就可以将上面代码转化为VD，首先，VD是对每个dom元素的描述，主要分为三个部分：标签名、属性、子元素，基于此：

```javascript
function h (tag, props, ...childs) {
  return {
    tag,
    props: props || {},
    children: childs.slice()
  }
}
```

改写开始的`test.js`， 增加`h`函数: `const h = require('./h.js')`

得到如下结果`test-m.js`：
```javascript
const h = require('./h.js')
module.exports = function render() {
    return h(
        "div",
        null,
        "Hello World",
        h(
            "ul",
            null,
            h(
                "li",
                { id: "1", "class": "li-1" },
                "\u7B2C1"
            )
        ),
        h(
            "footer",
            null,
            "footer"
        )
    )
}
```

运行将得到下面的VD
```javascript
{
    tag: "div",
    props: {},
    children: [
        "Hello World", 
        {
            tag: "ul",
            props: {},
            children: [{
                tag: "li",
                props: {
                    id: 1,
                    class: "li-1"
                },
                children: ["第", 1]
            }]
        },
        {
          tag: 'footer',
          props: null,
          children: ["footer"]
        }
    ]
}
```

#### 将VD转化为真实DOM

这一步思路就是：遍历VD，构建出真实的DOM树

我们准备`createElement`函数从VD中构建真实DOM

```javascript
function createElement (vd) {
  // 如果vdom是字符串或者数字类型，则创建文本节点，比如“Hello World”
  if (typeof vd === 'string' || typeof vd === 'number') {
    return document.createTextNode(vd)
  }
  const { tag, props, children } = vd
  const el = document.createElement(tag)
  for (let name in props) {
    el.setAttribute(name, props[name])
  }
  children.map(createElement).forEach(dom => el.appendChild(dom))
  return el
}
```

将上面的DOM树挂载到body元素下：`document.body.appendChild(createElement(render()))`

整理代码，形成HTML文件并在浏览器中测试：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <script>
    function h (tag, props, ...childs) {
      return {
        tag,
        props,
        children: childs.slice()
      }
    }
  </script>
  <script>
    function render() {
        return h(
            "div",
            null,
            "Hello World",
            h(
                "ul",
                null,
                h(
                    "li",
                    { id: "1", "class": "li-1" },
                    "\u7B2C1"
                )
            ),
            h(
                "footer",
                null,
                "footer"
            )
        );
    };
  </script>
  <script>
    function createElement (vd) {
      // 如果vdom是字符串或者数字类型，则创建文本节点，比如“Hello World”
      if (typeof vd === 'string' || typeof vd === 'number') {
        return document.createTextNode(vd)
      }
      const { tag, props, children } = vd
      const el = document.createElement(tag)
      for (let name in props) {
        el.setAttribute(name, props[name])
      }
      children.map(createElement).forEach(dom => el.appendChild(dom))
      return el
    }
    document.body.appendChild(createElement(render()))
  </script>
</body>
</html>
```

运行结果如下：

![](/images/VD.png)

#### 总结

在本篇文章中，我简单的将静态的jsx代码生成了真实的DOM树，这是最简单的体验。实际框架中要比上面的写法复杂得多。下面一篇文章我将会总结怎么更新VD。
