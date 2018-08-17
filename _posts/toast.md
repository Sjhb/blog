---
title: Vue中怎么写一个Toast组件
date: 2018-06-03 22:27:47
tags: vue toast
---

### 使用VUE写一个Toast组件

#### Toast是什么

Toast组件即轻提示组件一般在移动端使用比较多，例如加载、警告、确认操作等等，所以一般都是弹出式且全屏遮罩的，有开发经验的人可能会想到滑动穿透的问题。

#### 要做什么

现在要实现一个Toast组件，要求如下：
- 使用命令方式进行使用，而不是采用Template模版的方法。
- 能够进行显示时间长短配置。
- 可监听完成情况，可注册回调方法或者是使用Promise监听。

#### 解决第一个问题

可能一开始觉得无处下手，但是仔细阅读Vue的官方文档我们可以发现，当一个Vue实例新建出来的时候，如果我们不挂载到具体真实的Dom上，Vue实例就会一直存在于内存中，我感觉可以从这里入手。

首先，我们来试验一下，项目中资源目录结构如下

|\
|--src\
|----assets\
|----components\
|------toast\
|--------index.js\
|----App.vue\
|----main.js\
|----index.html\

其中Toast文件中，index.js作为入口文件，内容如下：

```javascript
import Vue from 'vue'

var MyComponent = Vue.extend({
  template: '<div>This is toast!</div>'
})

let create = () => {
  let el = document.createElement('div')
  document.body.appendChild(el)
  // 创建并挂载到el  (会替换el)
  new MyComponent().$mount(el)
}
export default create

```

我们简单声明一个Vue子类`MyComponent`，默认输出一个方法，该方法就是创建一个DOM节点并装入body节点中，再根据之前的`MyComponent`创建一个Vue实例挂载到之前创建的DOM节点上。

接下来我们在App.vue中使用这个方法：

```javascript
import toast from './components/toast/index.js'
export default {
  name: 'App',
  template: `
    <div id="app">
      <button @click="createToast">toast</button>
    </div>`,
  methods: {
    createToast () {
      toast()
    }
  }
}
</script>
```

当我们在实际页面中点击Toast按钮，会发现body标签下面确实会挂载一个内容为：`This is toast!`的div元素。

到这里我们已经知道怎么做了：通过动态添加DOM节点并挂载Vue实例来实现命令式组件，这里贴一下我写出来的Toast：

```javascript
import Vue from 'vue'

var ToastComponent = Vue.extend({
  template: `
    <div id="toast" class="toast-contianer" @touchmove="touchDefault">
      <div class="mask"></div>
      <div class="content">拼命加载中...</div>
    </div>
  `,
  methods: {
    // 防止滑动穿透
    touchDefault (e) {
      e.preventDefault()
      e.stopPropagation()
    }
  }
})

let Toast = null
// 显示组件
let create = () => {
  let el = document.createElement('div')
  document.body.appendChild(el)
  // 创建并挂载到 el (会替换 el)
  Toast = new ToastComponent()
  Toast.$mount(el)
}
// 隐藏组件
let hideToast = () => {
  Toast.$destroy()
  document.body.removeChild(document.getElementById('toast'))
  // 注意清除对原来Vue实例的引用，避免内存泄露
  Toast = null
}
export default {
  loading: create,
  hide: hideToast
}
```

```css
.toast-contianer {
position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 100;
}
.toast-contianer .mask {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: #000;
  opacity: .4;
}
.toast-contianer .content {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: transparent;
  color: #fff;
  text-align: center;
  line-height: 100vh;
  z-index: 101;
}
```

使用示例：

```javascript
import toast from './components/toast/index.js'
// 省略代码...
  methods: {
    test () {
      toast.loading()
      setTimeout(() => {
        toast.hide()
      }, 1000)
    }
  }
// 省略代码...
```

#### 解决第二个问题

要对组件进行时长配置，就意味着我们的组件是依据输入显示的，调整我们的index.js,如下

```javascript
// 省略代码...
let Toast = null
// 显示组件
let create = () => {
  let el = document.createElement('div')
  document.body.appendChild(el)
  // 创建并挂载到 #app (会替换 #app)
  Toast = new ToastComponent()
  Toast.$mount(el)
}
// 隐藏组件
let hideToast = () => {
  Toast.$destroy()
  document.body.removeChild(document.getElementById('toast'))
  // 注意清除对原来Vue实例的引用，避免内存泄露
  Toast = null
}
// 判断输入参数，调整显示组件的形式
let showToastByArg = (duration) => {
  if (typeof duration === 'number') {
    create()
    setTimeout(() => {
      hideToast()
    }, duration)
    return
  }
  create()
}
export default {
  loading: showToastByArg,
  hide: hideToast
}
```

这样我们就可以通过参数，进行灵活的调用：如果传入时间，那就是自动在多少时间后隐藏；没有传入时间就一直显示，直到调用方法隐藏掉。

#### 解决第三个问题

如果要注册回调事件，则必然是传入了显示时间，所以可以改造前面的`showToastByArg`和`hideToast`方法：

```javascript
let hideToast = (cb) => {
  Toast.$destroy()
  document.body.removeChild(document.getElementById('toast'))
  // 注意清除对原来Vue实例的引用，避免内存泄露
  Toast = null
  cb && cb()
}
let showToastByArg = (duration, cb) => {
  if (typeof duration === 'number') {
    create()
    setTimeout(() => {
      hideToast(cb)
    }, duration)
    return
  }
  create()
}
```

那如果是想要使用Promise呢，同样非常简单，继续修改我们的`showToastByArg`方法：

```javascript
let showToastByArg = (duration, cb) => {
  if (typeof duration === 'number') {
    create()
    // 回调式
    if (cb) {
      setTimeout(() => {
        hideToast(cb)
      }, duration)
      return
    }
    // Promise式，注意使用ployfill
    return new Promise((resolve, reject) => {
      try {
        setTimeout(() => {
          hideToast()
          resolve()
        }, duration)
      } catch (err) {
        reject(err)
      }
    })
  }
  create()
}
```

这样使用的使用我们就有四种不同的使用方式：

1. 不穿入参数调用，需要手动调用关闭方法。
2. 传入显示的时间长度。
3. 传入显示的时间长度，并注册关闭Toast回调时间。
4. 传入显示的时间长度，根据返回的Promise做具体操作，例如

```javascript
import toast from './components/toast/index.js'
toast.loading(1000).then(() => alert('haha'))
```

#### 总结

上面写的Toast组件非常简单，在实际的应用中肯定会比这个复杂很多。像轻提示这种组件，必须要操作DOM节点才能实现在根节点挂载组件。还有一些可拓展的地方，例如将Toast方法注册到主Vue应用中、单例模式等等。
