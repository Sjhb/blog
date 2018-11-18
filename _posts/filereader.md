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

### 一些应用场景以及解决方案

`FileReader`对象常常都是配和其它的Api来使用，下面列举一些常见的应用场景，感受一下它的便利性：

#### 
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <input type="file" id="fileSelect" name="testFile" multiple></input>
  <script>
    function ab2str(buf) {
      return String.fromCharCode.apply(null, new Uint16Array(buf));
    }
    let fileInput = document.getElementById('fileSelect')
    let file = null
    let fileReader = new FileReader()
    function getFile (file) {
      let url = window.URL.createObjectURL(file)
      // fileReader.readAsDataURL(file)
      fileReader.readAsArrayBuffer(file)
      // fileReader.readAsText(file)
      fileReader.onloadend = (e) => {
        let ab = fileReader.result
        console.log(ab2str(ab))

        // let img = document.createElement('img')
        // img.file = file
        // img.src = fileReader.result
        // document.body.appendChild(img)
      }
    }
    fileInput.addEventListener('change', e => getFile(e.target.files[0]))
    fileInput.click()
  </script>
</body>
</html>