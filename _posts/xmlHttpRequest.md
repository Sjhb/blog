---
title: XMLHttpRequest
date: 2017-09-10 23:11:07
tags: web
---

## 是什么

XMLHttpRequest是一个JavaScript DOM对象，提供了对HTTP协议的完全访问，能在不刷新页面的情况下与后台交换数据。例如POST、GET请求。

## 属性和方法

### 属性

属性:

- readyState:xhr的状态码，取值为0、1、2、3、4，分别表示:

|状态|名称|说明|
|-----|-----|-----|
|0|Uninitialize|初始化状态。XMLHttpRequest 对象已创建或已被 abort() 方法重置|
|1|Open|open()|方法已调用，但是 send() 方法未调用。请求还没有被发送|
|2|Sent|Send()|方法已调用，HTTP 请求已发送到 Web 服务器。未接收到响应|
|3|Receiving|所有响应头部都已经接收到。响应体开始接收但未完成|
|4|Loaded|HTTP 响应已经完全接收|

readyState的值只会逐级增加，不会减少，除非XMLHttpRequest对象的状态被重置了。
readyState改变时会触发XMLHttpRequest的onreadystatechange事件

- responseText

> 只读属性。保存接受到的来自服务器的响应体，即response的body，不包括头部。在readyState状态为3之前，这个属性是一个空字符串；为3时保存了已经接收到的部分；为4时保存了完整的响应体。

- responseXML

> 只读属性。它返回一个包含请求检索的HTML或XML的Document，如果请求未成功，尚未发送，或者检索的数据无法正确解析为 XML 或 HTML，则为 null。该响应被解析，就像它是一个 “text / xml” 流。当 responseType 设置为 “document” 并且请求已异步执行时，响应将解析为 “text / html” 流。responseXML 对于任何其他类型的数据以及 data: URLs 为 null。

> 如果服务器没有明确指出 Content-Type 头是 "text/xml" 还是 "application/xml", 可以使用XMLHttpRequest.overrideMimeType() 强制 XMLHttpRequest 解析为 XML.

- status

> 只读属性。服务器返回的HTPP状态码。在readyState=3之前访问这个属性会报异常

- statusText

> 只读属性。对HTTP状态码的字符串描述，如status为200时，statusText为'OK'。在readyState=3之前访问这个属性会报异常

- withCredentials

> booleanl类型的属性,默认为false，在同源中会无效，在跨域请求中会使用到：在跨域请求中，withCredentials设置为true时，会允许远程获取本地的cookie以及向本地写入cookie。这个时候远程的Access-Control-Allow-Origin不能为*。

- upload

> 有关上传的一些属性，XMLHttpRequest.upload是一个XMLHttpRequestEventTarget对象，可以对其绑定事件，监听upload对象的进度

![XMLHttpRequest.upload](/images/XMLHttpRequest.upload.png)


|事件|相应属性的信息类型|
|-----|-----|
|onloadstart|获取开始|
|onprogress|数据传输进行中|
|onabort|获取操作终止|
|onerror|获取失败|
|onload|获取成功|
|ontimeout|获取操作在用户指定的延迟时间内未完成|
|onloadend|获取完成（不管成功与否）|

> 使用这些属性和事件句柄，可以实现上传事件的监听，定制自己的组件

- timeout

- oprogress

> 事件句柄，

- onreadystatechange

> 事件句柄，当readyState发生改变的时候会触发该事件

### 方法

- abort()

> 取消当前响应，关闭连接并且结束任何未决的网络活动。这个方法把 XMLHttpRequest 对象重置为 readyState 为 0 的状态，并且取消所有未决的网络活动。

- getAllResponseHeaders()

> 把 HTTP 响应头部作为未解析的字符串返回。如果 readyState 小于 3，这个方法返回 null。否则，它返回服务器发送的所有 HTTP 响应的头部。头部作为单个的字符串返回，一行一个头部。每行用换行符 "\r\n" 隔开。

- getResponseHeader()

>返回指定的 HTTP 响应头部的值。其参数是要返回的 HTTP 响应头部的名称。可以使用任何大小写来制定这个头部名字，和响应头部的比较是不区分大小写的。该方法的返回值是指定的 HTTP 响应头部的值，如果没有接收到这个头部或者 readyState 小于 3 则为空字符串。如果接收到多个有指定名称的头部，这个头部的值被连接起来并返回，使用逗号和空格分隔开各个头部的值。

- open()

> 初始化 HTTP 请求参数，例如 URL 和 HTTP 方法，但是并不发送请求。

- send()

> 发送 HTTP 请求，使用传递给 open() 方法的参数，以及传递给该方法的可选请求体。

- setRequestHeader()

>向一个打开但未发送的请求设置或添加一个 HTTP 请求。