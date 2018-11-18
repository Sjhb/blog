---
title: 前端页面中的文件操作
date: 2018-11-17 21:24:01
tags: file
---

前端经常要跟文件打交道，下载上传这样的只是简单的文件读取和写入操作，可能有时候我们还需要进行文件內的信息读取。所以，浏览器环境下，我们还是有必要学习文件的相关操作。

### FileReader

 - `FileReader`是浏览器提供的文件操作对象，利用这个对象我们能进行文件读取相关的读取操作。这个文件可以来自用户在一个`<input>`元素上选择文件后返回的FileList对象,也可以来自拖放操作生成的 DataTransfer对象,还可以是来自在一个HTMLCanvasElement上执行mozGetAsFile()方法后返回结果
 
 这个对象有自己的属性、方法、事件，另外它继承自`EventTarget`,也就是说`FileReader`上的事件可以通过`addEventListener`方法使用。下面列举一下操作的属性、方法、事件：

属性：
 
 - `FileReader.error`: 只读, 一个DOMException，表示在读取文件时发生的错误 。
 
 - `FileReader.readyState`: 只读, 表示FileReader状态的数字。取值如下：0: EMPTY,还没有加载任何数据、1:LOADING,数据正在被加载、2：DONE，已完成全部的读取请。

 - `FileReader.result`: 只读, 文件的内容。该属性仅在读取操作完成后才有效，数据的格式取决于使用哪个方法来启动读取操作。

事件:

 - `FileReader.onabort`: 处理abort事件。该事件在读取操作被中断时触发。
 - `FileReader.onerror `: 处理error事件。该事件在读取操作发生错误时触发。
 - `FileReader.onload `: 处理load事件。该事件在读取操作完成时触发。
 - `FileReader.onloadstart `: 处理loadstart事件。该事件在读取操作开始时触发。
 - `FileReader.onloadend `: 处理loadend事件。该事件在读取操作结束时（要么成功，要么失败）触发。
 - `FileReader.onprogress `: 处理progress事件。该事件在读取Blob时触发。

方法:

 - `FileReader.abort() `: 中止读取操作。在返回时，readyState属性为DONE。
 - `FileReader.readAsArrayBuffer() `: 开始读取指定的 Blob中的内容, 一旦完成, result 属性中保存的将是被读取文件的 ArrayBuffer 数据对象.
 - `FileReader.readAsBinaryString() `: 开始读取指定的Blob中的内容。一旦完成，result属性中将包含所读取文件的原始二进制数据。
 - `FileReader.readAsDataURL() `: 开始读取指定的Blob中的内容。一旦完成，result属性中将包含一个data: URL格式的字符串以表示所读取文件的内容。
 - `FileReader.readAsText() `: 开始读取指定的Blob中的内容。一旦完成，result属性中将包含一个字符串以表示所读取的文件内容。

### Blob

`Blob`对象表示一个不可变、原始数据的类文件对象。`Blob`表示的不一定是JavaScript原生格式的数据。`File`接口基于Blob，继承了blob的功能并将其扩展使其支持用户系统上的文件，我们在`input`标签上，声明`type=file`,获取的文件对象的类型就为`File`

创建一个新的`Blob`对象，我们只需要使用`Blob`构造函数:

```javascript
let aFileParts = ['<a id="a"><b id="b">hey!</b></a>'] // 一个包含DOMString的数组
let oMyBlob = new Blob(aFileParts, {type : 'text/html'}) // 得到 blob
```

另外， `URL.createObjectURL()`静态方法会创建一个`DOMString`(UTF-16字符串)，利用这个方法我们可以为我们的`File`或者`Blob`对象创建一个地址（URL），这个`URL`可以直接使用，例如`img`标签的`src`属性

### 二进制数组

`ArrayBuffer`对象、`TypedArray`对象、`DataView`对象是JavaScript操作二进制数据的一个接口。这些对象早就存在，属于独立的规格，ES6将它们纳入了ECMAScript规格，并且增加了新的方法。

这些对象原始的设计目的，与WebGL项目有关。所谓WebGL，就是指浏览器与显卡之间的通信接口，为了满足JavaScript与显卡之间大量的、实时的数据交换，它们之间的数据通信必须是二进制的，而不能是传统的文本格式。文本格式传递一个32位整数，两端的JavaScript脚本与显卡都要进行格式转化，将非常耗时。这时要是存在一种机制，可以像C语言那样，直接操作字节，将4个字节的32位整数，以二进制形式原封不动地送入显卡，脚本的性能就会大幅提升。

二进制数组就是在这种背景下诞生的。它很像C语言的数组，允许开发者以数组下标的形式，直接操作内存，大大增强了JavaScript处理二进制数据的能力，使得开发者有可能通过JavaScript与操作系统的原生接口进行二进制通信。

二进制数组由三个对象组成。

（1）`ArrayBuffer`对象：代表内存之中的一段二进制数据，可以通过“视图”进行操作。“视图”部署了数组接口，这意味着，可以用数组的方法操作内存。

（2) `TypedArray`对象：用来生成内存的视图，通过9个构造函数，可以生成9种数据格式的视图，比如Uint8Array（无符号8位整数）数组视图, Int16Array（16位整数）数组视图, Float32Array（32位浮点数）数组视图等等。

（3）`DataView`对象：用来生成内存的视图，可以自定义格式和字节序，比如第一个字节是Uint8（无符号8位整数）、第二个字节是Int16（16位整数）、第三个字节是Float32（32位浮点数）等等。

简单说，`ArrayBuffer`对象代表原始的二进制数据，`TypedArray`对象代表确定类型的二进制数据，`DataView`对象代表不确定类型的二进制数据

> 上面内容摘抄自阮一峰的Javascript标准参考教程：http://javascript.ruanyifeng.com/stdlib/arraybuffer.html#toc0

这些对象拓展了`Javascipt`，让我们能够做到一些以前不会想到的事情，例如直接读取excel文件内容，又如直接读取文件的元信息。

### 一些应用场景以及解决方案

以上Api结合使用能够极大拓展前端的能力，下面列举一些常见的应用场景，感受一下它的便利性：

#### 图片选择预览与上传

设想我们现在需要这样一个需求：为用户提供上传图片的功能，要求能够预览并查看上传进度。我们知道`img`标签必须要图片资源地址或者具体`Data URL`格式图片数据，那怎么实现即时预览呢？

下面就简要描述一下步骤：

1. 提供一个标签，用户选取图片数据：`<input type="file" id="file-button" name="files" accept="imga"/>`

2. 监听图片选取，并对每个图片生成img标签实现预览：

```javascript
let fileInputDom = document.getElementById('file-button')
fileInputDom.addEventListener('change', e => previewImg(e.target.files))
function previewImg (files) {
  let fileArr = Array.from(files)
  fileArr.forEach(file => {
    let img = document.createElement('img')
    let url = window.URL.createObjectURL(file)
    img.src = url
    img.onload = () => window.URL.revokeObjectURL(url)
    document.body.appendChild(img)
  })
}
```

可以看到`URL.revokeObjectURL()`方法使我们实现预览的关键

- 实现上传功能，并监听进度，这里简单用原生示例：

添加一个上传按钮： `<input type="button" id="upload" value="上传"/>`

增加上传逻辑

```javascript
let uploadButton = document.getElementById('upload')
uploadButton.onclick = function () {
  let img = fileInputDom.files[0]
  let fileReader = new FileReader()
  let xhr = new XHRHttpRequest()
  xhr.upload.addEventListener('progress', function (e) {
    if (e.lengthComputable) {
      let percentage = Math.round((e.loaded * 100) / e.total)
      console.log(percentage)
    }
  }, false)
  xhr.upload.addEventListener('load', function(e) {
    console.log('done')
  }, false)
  xhr.open("POST", "http://localhost:8080")
  xhr.overrideMimeType('text/plain charset=x-user-defined-binary')
  fileReader.load = function (evt) {
    xhr.send(evt.target.result)
  }
  fileReader.readAsArrayBuffer(img)
}
```

#### 根据动态的数据，生成文件下载或者上传

要实现纯前端生成文件并下载，就不得不使用`Blob`对象，利用它来根据已有的数据生成文件类型数据，并使用`a`标签下载，下面举几个例子

- 生成json文件并实现下载：

  1. HTML文件内容：`<a id="download" download="demo.json">download</a>`

  2. 准备数据：`let data = { "a": "a", "b": [1, , 2] }`

  3. 实现下载功能：

      ```javascript
      let dom = document.getElementById('download')
      let blob = new Blob([JSON.stringify(data)], {
        type: "application/json"
      })
      let url = window.URL.createObjectURL(blob)
      dom.setAttribute('href', url)
      ```

      这样，只要点击a标签就可以下载

- 生成html文件并实现上传

  1. 生成数据内容:

  ```javascript
   let data = `
      <!DOCTYPE html>
      <html lang="en">
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
      </head>
      <body>
        hello word
      <body>
      </html>
    `
    let blob = new Blob([data], {
      type: "text/html"
    })
  ```

  2. 利用XHRHttpReaquest对象和FileReader对象实现上传:

  ```javascript
    function upload () {
      let reader = new FileReader()  
      let xhr = new XMLHttpRequest()

      xhr.upload.addEventListener("load", function(e){
        console.log('done')
      }, false)
      
      xhr.open("POST", "http://localhost:8080")
      xhr.overrideMimeType('text/html charset=utf-8')
      reader.onload = function(evt) {
        xhr.send(evt.target.result)
      }
      reader.readAsText(blob)
		}
  ```

更常见的需求是：读取文件内容或者下载excel文件这样的需求，一般简单的文件可以直接读取内容，但是复杂的文件需要借助工具库才能完成，例如`sheet.js`

#### 预览pdf

这里预览pdf和上面预览图片有一些相似，都是要生成链接。不同的是，预览pdf需要通过`iframe`标签来完成，或者新建页面

```javascript
  // data[Blob|File]: 获取到的文件，可能本地获取，也可能远程下载,
  let url = window.URL.createObjectURL(data)
  let iframe = ducoment.createElement('iframe')
  iframe.src = url
  document.body.appendChild(iframe)
```

#### WebSocket

`WebSocket`中，我们可以发送和接收二进制数据：

```javascript
// ArrayBuffer转为字符串，参数为ArrayBuffer对象
function ab2str(buf) {
  return String.fromCharCode.apply(null, new Uint16Array(buf));
}

// 字符串转为ArrayBuffer对象，参数为字符串
function str2ab(str) {
   // 每个字符占用2个字节,javascript使用UTF-16编码
  let buf = new ArrayBuffer(str.length * 2)
  let bufView = new Uint16Array(buf)
  for (let i = 0, strLen = str.length; i < strLen; i++) {
    bufView[i] = str.charCodeAt(i)
  }
  return buf
}
let socket = new WebSocket('ws://127.0.0.1:8081');
// 使用二进制的数据类型连接
socket.binaryType = 'arraybuffer'

// 连接
socket.addEventListener('open', function (event) {
  // 发送二进制数据
  let typedArray = str2ab('hello word')
  socket.send(typedArray.buffer)
});

// 接受二进制数据
socket.addEventListener('message', function (event) {
  let arrayBuffer = event.data
  console.log(ab2str(arrayBuffer))
})
```

### 总结

`FileReader`对象和`Blob`对象使用起来比较简单，二进制数组使用起来更加复杂，和涉及到很多计算机比较底层基础的知识，我这里只是简单的了解学习。在浏览器的沙盒环境下，前端能做的还是比较有限。