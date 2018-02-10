---
title: React学习笔记(二)
date: 2018-01-21 23:10:59
tags: react
---

### React 相关知识点

#### 状态state

每一个React组件都应当有自己独立的作用域和独立的运行状态，而且尽量少的依赖外部环境。
React组件中提供了一个变量：state，可以将之视为状态机，保存组件的各个状态

```jsx
class Clock extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      date: new Date()
    }
  }
  componentDidMount () {
    this.timer = setInterval(() => {
      this.setState({date: new Date()})
    }, 1000)
  }
  componentWillUnmount () {
    clearInterval(this.timer)
  }
  render () {
    return <label>{this.state.date.toString()}</label>
  }
}
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
)
```

上面例子中，state保存了date时间变量，通过setState方法更新state里的date变量。

> 不能直接更改state里面的值，例如`this.state.name = 'John'`,因为通过setState方法才可以触发视图更新

#### 组件生命周期钩子

上面例子中可以主要到Clock组件有componentDidMount、componentWillUnmount两个方法，这是组件的生命周期钩子，在组件进行状态变化时会触发。

组件的所有生命周期钩子有
- `componentWillMount()`：组件由虚拟dom渲染为真实dom前触发
- `componentDidMount()`： 组件由虚拟dom渲染为真实dom后触发
- `componentWillUpdate(object nextProps, object nextState)`：组件状态更新前触发
- `componentDidUpdate(object prevProps, object prevState)`：组件状态更新后触发
- `componentWillUnmount()`：组件状态销毁前触发

#### 事件监听

React事件监听跟原生dom事件差别不大，需要注意的几点
- React 事件名称使用小驼峰，例如`onclick`=>`onClick`
- 注意this的指向

下面的例子中采用了三种方式绑定事件,第一二种使用的是bind函数绑定上下文，第三种使用的是箭头函数

```jsx
class Clock extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      isLogin: false
    }
    this.changeStateA = this.changeState.bind(this)
  }
  changeState (...args) {
    this.setState((preState) => {
      return {
        isLogin: !preState.isLogin
      }
    })
  }
  render () {
    return (
      <div>
        <label>{this.state.isLogin ? 'Login' : 'Logout'}</label>
        <button onClick={this.changeState.bind(this)}>Shift State</button>
        <button onClick={this.changeStateA}>Shift StateA</button>
        <button onClick={(e) => {this.changeState(e)}}>Shift StateB</button>
      </div>
    )
  }
}
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
)
```

#### 条件渲染

React中因为XML与Javascript混写，有很多种方式可以实现条件渲染，我这里举几个常用的例子.
无非就是通过控制js来返回要输出渲染的HTML代码

```jsx
function ExampleA (props) {
  if (props.login) {
    return <label>A:Login</label>
  } else {
    return <label>A:Logout</label>
  }
}
function ExampleB (props) {
  return (
    <div>
        {props.login &&<label>B:Login</label>}
        {!props.login &&<label>B:Logout</label>}
    </div>
  )
}
function ExampleC (props) {
  return (
    <div>
      {props.login ? 
        (<label>C:Login</label>) :
        (<label>C:Logout</label>)
      }
    </div>
  )
}
function Examples () {
  return (
    <div>
    <ExampleA login={true}/>
    <ExampleB login={false}/>
    <ExampleC login={true}/>
  </div>
  )
}
ReactDOM.render(
  <Examples />,
  document.getElementById('root')
)
```

#### 列表渲染

循环渲染很简单，只需要看下面的例子就懂了，需要注意的是进行列表渲染时要指定具体的key

```jsx

function Listrender (props) {
  let cells = props.numberList.map((temp, index) => <li key={index}>{temp}</li>)
  return (
    <ul>
      {cells}
    </ul>
  )
}
ReactDOM.render(
  <Listrender numberList={[1,2,3,4,5]}/>,
  document.getElementById('root')
)

```

#### form表单

对于form表单，浏览器有自己的默认行为，我们可以使用React取代浏览器自带的行为，取代默认行为组件称为`Controlled Component`。下面具体举例具体的

- `textarea`: 默认的带有值得textarea标签因当如下:

```html

<textarea>
  Hello there, this is some text in a text area
</textarea>

```

在React组件中则使用value来属性来指明textarea里面的值，具体如下

```jsx
class Fromtextarea extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      textareaValue: 'haha'
    }
    this.handleChange = this.handleChange.bind(this)
  }
  handleChange (e) {
    this.setState({textareaValue: e.target.value})
  }
  render () {
    return (
      <form>
        <textarea value={this.state.textareaValue} onChange={this.handleChange}></textarea>
      </form>
    )
  }
}
ReactDOM.render(
  <Fromtextarea />,
  document.getElementById('root')
)
```

- `select`: 下拉选择框

```jsx
class Select extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      value: 'b'
    }
    this.handleChange = this.handleChange.bind(this)
  }
  handleChange (e) {
    this.setState({
      value: e.target.value
    })
  }
  render () {
    return (
      <form>
        <select value={this.state.value} onChange={this.handleChange}>
          <option value="a">a</option>
          <option value="b">b</option>
        </select>
      </form>
    )
  }
}

ReactDOM.render(
  <Select />,
  document.getElementById('root')
)

```

>如果要多选，只需在select标签加上`multiple={true}`，但此时select的value属性只接收数组，所以onChange事件对应的方法和传入的value都需要做更改。

- `fileInput`: 文件选择器

```jsx
class FileInput extends React.Component {
  constructor(props) {
    super(props)
    this.handleSubmit = this.handleSubmit.bind(this)
    this.fileInput = null
  }
  handleSubmit (e) {
    e.preventDefault()
    console.log(this.fileInput.files)
  }
  render () {
    return (
      <form onSubmit={this.handleSubmit}>
       <input type="file" ref={input => {this.fileInput = input}}></input>
       <button type="submit"> Submit </button>
      </form>
    )
  }
}
ReactDOM.render(
  <FileInput />,
  document.getElementById('root')
)
```

> ref设置回调函数，表示当这个dom元素渲染完成或者发生变化时，将当前的dom实例，在这里是`input`标签，赋值给fileIput变量。跟多React相关知识可参考`https://segmentfault.com/a/1190000008665915`