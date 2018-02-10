---
title: react学习笔记(四)
date: 2018-02-03 01:08:57
tags:
---

### Refs

在典型的React数据流中，父组件向子组件通信只能使用`Props`属性，当要修改子元素或者子组件时需要重新渲染它们，但是有些时候，我们确实需要获得子元素或者子组件的引用，来做一些事情，也就是`ref`。

1. 什么时候该使用`refs`呢

- 监听`focus`、文本选中、媒体播放等事件
- 控制动画
- 管理第三方库

2. 不要过度使用`refs`，虽然使用`refs`确实能带了很多便利，但是我们构建应用时还是应当以数据驱动的方式来构建项目。
3. 添加`ref`到Dom元素的方法，且看下面的例子，当点击focus按钮就会聚焦到输入框

```jsx
class MyInput extends React.Component {
  constructor (props) {
    super(props)
  }
  handleFocus () {
    this.textInput.focus()
  }
  render () {
    return (
      <div>
        <input type="text" ref={input => {this.textInput = input}}></input>
        <input type="button" onClick={this.handleFocus.bind(this)} value="focus"></input>
      </div>
    )
  }
}
ReactDOM.render(
  <MyInput />,
  document.getElementById('root')
)
```

4. 添加`ref`到React组件的方法，与添加到DOM元素的方法一致，对上面的例子稍作修改

```jsx
class MyInput extends React.Component {
  constructor (props) {
    super(props)
  }
  handleFocus () {
    this.textInput.focus()
  }
  render () {
    return (
      <div>
        <input type="text" ref={input => {this.textInput = input}}></input>
        <input type="button" onClick={this.handleFocus.bind(this)} value="focus"></input>
      </div>
    )
  }
}
class AutoFocus extends React.Component {
  constructor (props) {
    super(props)
  }
  componentDidMount () {
    this.textInputComponent.handleFocus()
  }
  render () {
    return (
      <div>
        <MyInput ref={input => {this.textInputComponent = input}}/>
      </div>
    )
  }
}
ReactDOM.render(
  <AutoFocus />,
  document.getElementById('root')
)
```

5. 向父组件暴露子组件的引用，这里需要使用props来传递，在真正的dom元素上绑定来自父组件的props上即可。

### 性能优化

#### shouldComponentUpdate

`shouldComponentUpdate`是React组件的一个生命周期函数，在组件渲染之前触发，默认返回TRUE，返回FALSE时不会更新组件。

利用这个生命周期函数可以控制组件的渲染，避免不必要的资源开销，优化我们的组件见。

下面例子中，只有userIputA值变化后才会更新组件

```jsx
class InputPreview extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      userInputA: '',
      userInputB: ''
    }
    this.handleInputA = this.handleInputA.bind(this)
    this.handleInputB = this.handleInputB.bind(this)
  }
  handleInput (e) {
    this.setState({
      userInputA: e.target.value
    })
  }
  handleInputB (e) {
    this.setState({
      userInputB: e.target.value
    })
  }
  shouldComponentUpdate (nextProps, nextState) {
    if (nextState.userInputA === this.state.userInputA) {
      return false
    }
    return true
  }
  render () {
    return (
      <div>
        <input type="text" onChange={this.handleInputA} value={this.state.userInputA}></input>
        <input type="text" onChange={this.handleInputB} value={this.state.userInputB}></input>
        <h3>{this.state.userInputA}</h3>
        <h3>{this.state.userInputB}</h3>
      </div>
    )
  }
}
ReactDOM.render(
  <InputPreview />,
  document.getElementById('root')
)
```

### Portals

Portals 提供了一种很好的将子节点渲染到父组件以外的DOM节点的方式，基本用法：

- ReactDom.createPortal(child, container)

其中child是任何可渲染的React子元素，container则是一个DOM元素，

简单例子：Modal虽然渲染在了其他元素下面，但是仍然会将事件冒泡到组件Parent

```html
<div id="root">
  <div id="app"></div>
  <div id="modal"></div>
</div>
```

```jsx
class Modal extends React.Component {
  constructor (props) {
    super(props)
  }
  render () {
    return ReactDOM.createPortal(this.props.children, document.getElementById("modal"))
  }
}
class Parent extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      door: 'open'
    }
  }
  handleButton (e) {
    let x = this.state.door === 'open' ? 'close' : 'open'
    this.setState ({
      door: x
    })
  }
  render () {
    return (
      <div>
        <label>door`s state: {this.state.door}</label>
        <Modal>
          <input type="button" onClick={this.handleButton.bind(this)} value="shift"/>
        </Modal>
      </div>
    )
  }
}
ReactDOM.render(
  <Parent />,
  document.getElementById('app')
)
```
