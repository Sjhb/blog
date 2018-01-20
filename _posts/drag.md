---
title: 网页内拖拽：drag
date: 2017-08-11 23:56:49
tags: web
---

###  API

> `drag`:当页面内的元素或者文本被拖动或者选择的时候会被触发，可冒泡，目标：被拖拽的元素
>
> `dragend`:当拖拽操作结束后会触发，可冒泡，目标：被拖拽的元素
>
> `dragenter`:拖拽元素或者文本进入另外元素时会在目标元素触发，可冒泡，目标：拖拽进入的元素
>
> `dragexit`: 拖拽元素或者文本离开目标节点，可冒泡，目标：拖拽离开的元素// chrome不支持，firfox支持
>
> `dragleave`:拖拽元素或者文本离开一个合法的节点，可冒泡，目标：被拖拽的元素
>
> `dragover`:当一个元素被拖动到一个合法的节点时触发，可冒泡，目标：拖拽元素下的元素
>
> `dragstart`：开始拖动元素或者文本时触发，可冒泡，目标：被拖拽的元素
>
> `drop`:放下元素时触发，可冒泡，目标：放置的目标元素

>拖动的对象必须要声明`draggabled`为` true`，否则拖动无效
>
>相关信息可以从事件参数`event`中取

>`DataTransfer` 对象:
>
>在进行拖放操作时，`DataTransfer` 对象用来保存被拖动的数据。它可以保存一项或多项数据、一种或者多种数据类型,这个对象在所有的拖动事件属性`dataTransfer`  都是可用的，但是不能单独创建。`DataReansfer`可以用来实现从电脑桌面向网页内拖动数据，非常好用。

### 实践

调整色块的顺序（vue为例，这里未使用Datatransfer）：

> 在本例中，document.querySlectorAll 返回的是一个静态的NodeList列表，配合翻页时要重新获取对象列表，遍历时也不能使用Iterator，因为它并不是从Array继承来的 详情：[MDN:NodeList][url]
>
> 拖拽元素能同时触发多个底层元素的dragenter事件

html: 在一个区域内铺上12个色块，色块颜色由css指定

```html
<template>
<div class="drag">
  <div class="container">
    <div class="pictures">
      <template v-for="picture in pictures">
        <div class="picture"
             :key="picture.value"
             :style="{'background': picture.label}"
             draggable="true" :value="picture.value">
          {{picture.value}}
        </div>
      </template>
    </div>
  </div>
</div>
</template>
```



css:

```css
.drag {
  width: 100%;
  .container {
    width: 80%;
    margin: 0 auto;
    .pictures {
      margin: 0 auto;
      width: 500px;
      display: flex;
      justify-content: space-around;
      flex-wrap: wrap;
      .picture {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 156px;
        height: 200px;
        font-size: 40px;
        color: white;
        text-align: center;
        margin-top: 10px;
      }
    }
  }
```

js 

```javascript
<script>
  export default {
    name: 'drag',
    data () {
      return {
        // 模拟数据
        pictures: [
          {
            value: 1,
            label: '#FAD7A0  '
          },
          ......
          {
            value: 12,
            label: '#BA4A00  '
          }
        ],
        // 临时保存拖拽元素的信息
        drage: [],
        // 临时保存颜色
        tempColor: ''
      }
    },
    methods: {
      // 为所有颜色块设置drag事件
      setEvent () {
        let _that = this
        let _pictures = document.querySelectorAll('.picture')
        let _total = _pictures.length
        // 不能使用for...in语句
        for (let item = 0; item < _total; item++) {
          let _el = _pictures[item]
          _el.addEventListener('drag', function (event) {
          }, false)
          _el.addEventListener('dragstart', function (event) {
            _that.drage = [{el: event.target}]
            _that.drage[0].value = event.target.innerHTML
          })
          _el.addEventListener('dragenter', function (event) {
//            过滤掉多次触发的dragenter事件
            if (_that.drage.length < 2) {
              _that.drage[1] = {el: event.target}
              _that.drage[1] = {value: event.target.innerHTML}
              _that.tempColor = event.target.style.background
              event.target.style.background = '#5c5c5c'
            } else {
              return false
            }
          }, false)
          _el.addEventListener('dragleave', function (event) {
            if (_that.drage.length === 1) {
              return false
            } else {
              _that.drage.pop()
              event.target.style.background = _that.tempColor
              _that.tempColor = ''
            }
          }, false)
          _el.addEventListener('dragend', function (event) {
          }, false)
          _el.addEventListener('dragover', function (event) {
            event.preventDefault()
          }, false)
          _el.addEventListener('drop', function (event) {
            // 阻止事件冒泡
            event.preventDefault()
            let _tempB = _that.drage[0].el.style.background
            let _tempI = _that.drage[0].value
            _that.drage[0].el.style.background = _that.tempColor
            _that.drage[0].el.innerHTML = event.target.innerHTML
            event.target.style.background = _tempB
            event.target.innerHTML = _tempI
          }, false)
        }
      }
    },
    mounted () {
      this.setEvent()
    }
  }
</script>
```

效果如下：

选取目标元素：

![stepone](/images/step1.png)

拖拽目标元素到目标区域：

![stepone](/images/step2.png)

在目标区域放下目标元素：

![stepone](/images/step3.png)

[url]:https://developer.mozilla.org/zh-CN/docs/Web/API/NodeList