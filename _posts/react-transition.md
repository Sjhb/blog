---
title: react-transition体验
date: 2018-06-18 21:32:47
tags: react transition
---

### 是什么

React中，视图都是数据驱动的，以单向数据流的形式实现。而我们在很多应用场景中，都需要实现动画，实现稍微复杂的动画都需要操作DOM，这显然与React中的数据驱动背道而驰，所以操作DOM也基本是一条死路了。那有什么好的办法实现动画吗？答案就是：ReactCSSTransitionGroup。

### 简单上手

#### 安装

```bash
$ yarn add react-transition-group
```

或者

```bash
$ npm install react-transition-group
```

#### 简单说明

`ReactCSSTransitionGroup`提供了三种不同类型的React组件供我们使用，这里使用`Transition`这个组件示例。

实现要声明的非常重要的一点就是：`Transition`并不会改变React组件本身默认的行为，只会将状态信息，例如`entering`、`entered`,传递给子内容，具体怎么根据状态信息实现动画则由开发者自己决定

`Transition`有几个重要的属性，先简单列出来：

- `children`: `Transition`组件子内容需要是函数而不是React组件，因为需要将当前的状态（'entering', 'entered', 'exiting', 'exited', 'unmounted'）传递给该函数，进而传递给子组件。例如：

```jsx
<Transition timeout={150}>
  {(status) => (
    <MyComponent className={`fade fade-${status}`} />
  )}
</Transition>
```
- `in`: 用来触发状态改变，例如：`false`-》`true`,`Transition`的状态就会由`exited`->`entering`->`entered`
- `mountOnEnter`: 默认情况下，`Transition`组件挂载时，其子组件也会挂载，可以设置`mountOnEnter`为`true`，使其子组件在`in`设置为`true`时才被挂载，类似于懒加载的效果
- `unmountOnExit`: 与`mountOnEnter`类似，当`Transition`组件状态为`exited`,子组件将会被卸载
- `appear`: 正常情况下，`Transition`组件被挂载时，是不会有动画效果的，如果需要一开始挂载时就需要动画效果，可在`Transition`组件上声明`appear`属性为`true`
- `timeout`: 状态变化之间所需要的时间
- `onEnter`、`onEntering`、`onEntered`、`onExit`、`onExiting`、`onExited`一些状态变化时的钩子

#### 编写小demo

依据上面的说明，我简单写了一个小demo：根据状态的变化，渲染出来的方块会呈现不同的颜色

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import { Transition } from 'react-transition-group';

class Example extends React.Component {
  state = {
    show: false
  };
  componentDidMounted() {
    setTimeout(() => {
      this.setState({
        show: true
      });
    }, 1000);
  }
  render() {
    const { show } = this.state;
    return (
      <Transition in={this.state.show} timeout={500}>
        {state => {
          let baseStyle = {
              transition: 'background .2s ease-out',
              width: '100px',
              height: '100px'
          };
          switch (state) {
              case 'exited':
              baseStyle.background = 'black';
              break;
              case 'entering':
              baseStyle.background = 'red';
              break;
              case 'entered':
              baseStyle.background = 'green';
              break;
              case 'exiting':
              baseStyle.background = 'blue';
              break;
          }
          return (
            <div style={baseStyle}></div>
            );
        }}
      </Transition>
    );
  }
}

ReactDOM.render(
  <Example />,
  document.getElementById('root')
);

```

### CSSTransition

`CSSTransition`是受到了`ng-animate`启发。`CSSTransition`提供了三个状态：`appear`, `enter`, 和`exit`。使用时非常简单：当三个状态之间切换时，`CSSTransition`会修改子组件的`class`,使用预定义的类代替，例如：当`classNames="fade"`时，依次会使用：`fade-enter`, `fade-enter-active`, `fade-enter-done`, `fade-exit`, `fade-exit-active`, `fade-exit-done`, `fade-appear`, 和`fade-appear-active`

### TransitionGroup

`TransitionGroup`就如同状态机一样可以管理一组`Transition`或者`CSSTransition`组件。其内部的`Transition`或者`CSSTransition`组件会自动加上`in`属性，常见例子就是数组渲染。