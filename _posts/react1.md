---
title: React学习笔记（一）
date: 2018-01-20 22:52:24
tags:
---

本系列博客将记录我学习React的一些心得。

### 是什么

React是一个Javascript前端框架\
之所以要学这个框架，主要出于以下几点
- 大大节省开发时间，缩短项目开发周期
- 这个框架的横向价值巨大，衍生项目React Native可做到三端通
- 良好的生态圈

### 基础

#### 安装

直接去官网下载源文件即可，下载地址：https://react-cn.github.io/react/downloads.html
下载的包解压出来即可得到目标文件

> 学习时暂时先不结合webpack使用。

#### hello word

在不使用Webpack直接使用React功能时需要准备好以下几个文件：
- `react.js`:提供react核心功能
- `react-dom.js`:提供与dom相关的操作
- `browser.min.js`: 将react中es6语法转成es5语法

第一、二个文件可在上面下载的文件中提取，第三个则可以使用cdn地址：https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.24/browser.min.js

代码示例

```html
<body>
    <div id="root"></div>
    <script src="./react.js"></script>
    <script src="./react-dom.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.24/browser.min.js"></script>
    <script type="text/babel">
        ReactDOM.render(
            <h1>Hello, world!</h1>,
            document.getElementById('root')
        );
    </script>
</body>
```

> 上面`script`标签需要注明类型为`text/babel`，这是因为react使用的jsx语法与javascript语法相矛盾。

上面代码会向根节点（`div.root`)渲染一个`h1`标签，内容为`Hello word!`

对比Vue和React，这两个框架在渲染上都做了优化：只更新需要更新的节点，例如上面的例子，当再次调用`ReactDOM.render`方法时更新的只是`h1`标签，根节点并不会更新。

#### jsx

jsx语法就是将Javascript和html两种语言混合起来的一种新的语法

例如

- 声明一个元素：`const element = <h1 class="title">helo word</h1>`
- 在元素中插入变量：`const element = <h1>{val}</h1>`，花括号表示内部是js的内容
- 声明嵌套的标签：
```jsx
function getAvatar (user) {
    return (
        <div>
            <img src={user.path} />
            <span>{user.name}</span>
        </div>
    )
}
```

这些写法都会被jsx编译成为js能够识别的写法。

#### 组件和属性

组件就是就像是积木，有自己独立的作用域，即拿即用

声明一个简单的组件

```jsx
function SayWord (props){
    return <h1>{prps.word}</h1>
}
ReactDOM.render(
    <SayWord word="hello word" />,
    document.getElementById('root')
)
```

使用es6中的Class声明组件

```jsx
class SayWord extends React.Component {
    render () {
      return <h1>{this.props.word}</h1>
    }
}
ReactDOM.render(
  <SayWord word='haha' />,
  document.getElementById('root')
)
```

组件之间可以相互调用，我们可以根据业务抽象组件

```jsx
// 简单的输入框组
class Desc extends React.Component {
    render () {
      return <label>{this.props.word}</label>
    }
}
class InputArea extends React.Component {
    render () {
      return <input type="text"/>
    }
}
function InputGroup (props) {
  return (
    <div>
      <Desc word={props.label}/>
      <InputArea />
    </div>
  )
}
const input = {
  label: '用户名'
}
ReactDOM.render(
  InputGroup(input),
  document.getElementById('root')
)
```

从上面的例子可以看出，React中是单向将变量绑定到组件上的，我们在使用这些变量时，切不可改变原有变量的值，这样，这个组件或者方法可以称之为`纯净的`(pure)

#### 虚拟dom

React的一大创新就是虚拟dom，虚拟dom顾名思义就是不存在的dom节点。React在更新节点的时候会先生成数据结构，称之为虚拟dom，等这些虚拟dom都准备好了之后才会反应到实际的页面当中，在中间有一个DOM diff算法的概念。
