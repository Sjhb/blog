---
title: Webhook实践
date: 2017-12-24 01:24:56
tags: web
---

### Webhook

`Webhook`简单的来说就是钩子。GitHub或者GitLab等仓库通过配置好`Webhook`钩子，当开发者对代码仓库操作时，管理者可以很好的对这些操作进行检测。
利用`Webhook`，我们可以实现自动化部署代码、重启服务、更新线上缓存等一系列操作，免去人肉部署代码。

### Webhook实践

这里我要实现监听`GitHub`仓库的push操作
准备工作：

- 一台对外开放的服务器，能接受请求
- 配置好WebHook的代码仓库，这里以GitHub为例

把大象关进冰箱里：

1. 编写服务器代码，部署到外网可访问的那台主机上

这里我以`Node`为例，接收`GitHub`发送过来的`post`请求，打印出`Github`改变的分支

```javascript
const HTTP = require('http')
const server = HTTP.createServer()
// 接收post请求，打印Github改变的分支,以及有的属性名
let dispatcher = (req, res) => {
  let data = ''
  if (req.method === 'POST') {
    req.setEncoding('utf8')
    req.on('data', (chunk) => {
      data += chunk
    })
    req.on('end', () => {
      data = JSON.parse(data)
      console.log(data.ref)
      console.log(Object.keys(data))
    })
    res.end()
  }
}
server.on('request', dispatcher)
// 部署到80端口
server.listen(80, () => {
  console.log('server-webhook is runing  on port 80')
})
```

将这段代码保存到`server.js`文件中，并部署到服务器。

2. 准备仓库，配置Webhook

`Webhook`钩子有很多种，例如`release`分支、`Pull request`事件，我们这里选择监听`push`事件。

如下图，配置好要监听的事件以及事件发生后`GitHub`应该请求的地址，这个地址就是上面服务器脚本应该部署的主机地址。
![](/images/webhhok.png)

3. 推送代码并观察服务器日志

我向配置好`Webhook`钩子的仓库`master`分支推送一次

![](/images/push.png)

过一会儿就可以看到服务器接收到请求，以及改变的分支
![](/images/server-hook.png)

> 利用这些参数我们可以执行脚本来更新线上数据等操作，这个留到下一期再讲。
