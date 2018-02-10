---
title: React学习笔记（三）
date: 2018-01-28 23:15:30
tags:
---

#### 状态提升

在React中props属性是从父组件传递过来的，这个属性是只读的。当组件与组件共用同一个变量的时候，例如使用了props.A变量，这时候如果组件一想要改变变量A的值并让其他的组件也监听到该变化，就需要将这个变量变化提升到父组件上，称之为状态提升（Lifting State Up）

例如，在下面的例子中，实现了人民币、美元对黄金的的转换，它们之间就是用了状态提升来保持状态的一致

```jsx
// 输入数据的组件，事件的监听和对应的值都是来自于父组件，在这个组件中只是接收用户的输入并交由父组件处理，输入框展示的值也是由父组件传入
// 整个状态直接由本组件提升到了父组件内，当父组件状态发生改变直接重新渲染
class MoneyInput extends React.Component {
  constructor(props) {
    super(props)
    this.handleInput = this.handleInput.bind(this)
  }
  handleInput (e) {
    this.props.onMoneyChanged(e.target.value)
  }
  render () {
    let money = this.props.money
    let moneyType = this.props.type
    return (
      <fieldset>
        <legend>{moneyType === 'R' ? '请输入人民币金额' : '请输入美元金额'}</legend>
        <input value={money} onChange={this.handleInput}></input>
      </fieldset>
    )
  }
}
// 主要的组件用来组合输入框并对应显示黄金兑换比例
class Calculator extends React.Component {
  constructor (props) {
    super(props)
    this.state = {
      type: 'R',
      money: ''
    }
    this.handleRmbChanged = this.handleRmbChanged.bind(this)
    this.handleDollarChanged = this.handleDollarChanged.bind(this)
  }
  handleRmbChanged (rmb) {
    this.setState({type: 'R', money: rmb})
  }
  handleDollarChanged (dollar) {
    this.setState({type: 'D', money: dollar})
  }
  render () {
    let type = this.state.type
    let money = this.state.money
    let rmb = type === 'R' ? money : money * 6.7663
    let dollar = type === 'D' ? money : money / 6.7663
    let gold = dollar * 0.028
    return (
      <div>
        <MoneyInput money={rmb} type='R' onMoneyChanged={this.handleRmbChanged} />
        <MoneyInput money={dollar} type='D' onMoneyChanged={this.handleDollarChanged} />
        <label style={{marginTop: '10px'}}>对应黄金为{gold}g</label>
      </div>
    )
  }
}
ReactDOM.render(
  <Calculator />,
  document.getElementById('root')
)
```

### 进阶内容

#### 深入JSX

- JSX为`React.createElement(component, props, ...children)`方法提供了语法糖

例如

```jsx
<Mybutton onClick={handleClick}>
  Submit
</Mybutton>
```

将被解析为

```jsx
React.createElement(Mybutton, {onClick: handleClick}, 'Submit')
```

- 在使用React.createElement时，必须保证React对象在作用域內，在使用JSX提供的语法糖时，React对象往往容易被忽略。

- JSX支持点符号来指明组件类型

```jsx
const Mycomponents = {
  Mybutton: function Mybutton (props) {
    return <input type="button" value={props.buttonValue}/>
  }
}
ReactDOM.render(
  <Mycomponents.Mybutton buttonValue="submit"/>,
  document.getElementById('root')
)
```

- 自定义组件必须使用大写,以区分html标签
- 对组件声明属性时，可以使用结构赋值

```jsx
function Mybutton (props) {
  return (
    <Button type={props.type} size={props.size} onClick={props.click}>
      button
    </Button>
  )
}
let buttonTag = {
  type: 'primary',
  size: 'small',
  click: function (e) {
    console.log(e)
  }
}
ReactDOM.render(
  <Mybutton {...buttonTag}/>,
  document.getElementById('root')
)
```

- `props.children`:父组件在使用子组件传递时头部标签和尾部标签之间的内容，例如`<MyComponent>Hello world!</MyComponent>`，在子组件中，`props.children`就为`'test'`
`props.children`的值在以下几种情况中是不一样的

1. 字符串，这时候会忽略空行，换行会编程一个空格，这与浏览器行为是一致的，下面几种写法都是一样的：

```html
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
```

2. jsx组件

```jsx
function Mybutton (props) {
  return <button>{props.children}</button>
}
function Myspan (props) {
  return <span>{props.spanValue}</span>
}
ReactDOM.render(
  (
    <Mybutton>
      <Myspan spanValue="button"/>
    </Mybutton>
  ),
  document.getElementById('root')
)
```

3. JavaScript表达式

```jsx
function Myul (props) {
  return <ul>{props.children}</ul>
}
function Myli (props) {
  return <li>{props.spanValue}</li>
}
let arr = ['firstName', 'secondName']
ReactDOM.render(
  (
    <Myul>
      {arr.map((temp, index) => <Myli spanValue={temp} key={index}/>)}
    </Myul>
  ),
  document.getElementById('root')
)
```

4. 函数

```jsx
let arr = ['firstName', 'secondName']
function Myul (props) {
  return <ul>
    {arr.map(temp => props.children(temp))}
  </ul>
}
function Myli (props) {
  return <li>{props.spanValue}</li>
}
ReactDOM.render(
  (
    <Myul>
      {(name) => <Myli key={name} spanValue={name}/>}
    </Myul>
  ),
  document.getElementById('root')
)
```

### 使用PropTypes进行类型检测

我们在开发React应用程序的时候，可以使用TypeScript这样的代码审查工具来帮我们检查代码，其实，React也为我们提供了检查工具：PropsTypes

在不使用npm包时可引用cdn地址：`https://cdnjs.cloudflare.com/ajax/libs/prop-types/15.6.0/prop-types.js`
开源项目地址：`https://github.com/facebook/prop-types/`

简单用法如下：

```jsx
class MyComponent extends React.Component {
  constructor(props) {
    super(props)
  }
  render () {
    return <span>{this.props.content}</span>
  }
}
let myPropTypes = {
  content: PropTypes.string
}
const props = {
  content: 'Hello it`s me'
}
PropTypes.checkPropTypes(myPropTypes, props, 'prop', 'MyComponent')
ReactDOM.render(
  <MyComponent content={props.content}/>,
  document.getElementById('root')
)
```

当我把props.content 赋值为1时，就会报错**prop-types.js:841 Warning: Failed prop type: Invalid prop `content` of type `number` supplied to `MyComponent`, expected `string`.**

#### PropTypes提供的各种检测类型

```jsx
import React from 'react';
import PropTypes from 'prop-types';

class MyComponent extends React.Component {
  render() {
    // ... do things with the props
  }
}

MyComponent.propTypes = {
  // You can declare that a prop is a specific JS primitive. By default, these
  // are all optional.
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // Anything that can be rendered: numbers, strings, elements or an array
  // (or fragment) containing these types.
  optionalNode: PropTypes.node,

  // A React element.
  optionalElement: PropTypes.element,

  // You can also declare that a prop is an instance of a class. This uses
  // JS's instanceof operator.
  optionalMessage: PropTypes.instanceOf(Message),

  // You can ensure that your prop is limited to specific values by treating
  // it as an enum.
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // An object that could be one of many types
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message)
  ]),

  // An array of a certain type
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // An object with property values of a certain type
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // An object taking on a particular shape
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number
  }),

  // You can chain any of the above with `isRequired` to make sure a warning
  // is shown if the prop isn't provided.
  requiredFunc: PropTypes.func.isRequired,

  // A value of any data type
  requiredAny: PropTypes.any.isRequired,

  // You can also specify a custom validator. It should return an Error
  // object if the validation fails. Don't `console.warn` or throw, as this
  // won't work inside `oneOfType`.
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // You can also supply a custom validator to `arrayOf` and `objectOf`.
  // It should return an Error object if the validation fails. The validator
  // will be called for each key in the array or object. The first two
  // arguments of the validator are the array or object itself, and the
  // current item's key.
  customArrayProp: PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```



