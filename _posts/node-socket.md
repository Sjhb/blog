---
title: node实现ftp相关操作
date: 2018-12-02 21:37:23
tags: socket ftp node
---

最近在看ftp相关内容，想着使用node实现一下

### 什么是socket

socket是对`TCP/IP`协议族的一种封装，是应用层与`TCP/IP`协议族通信的中间软件抽象层，也就是说，socket就是一组编程接口(API)。通过这些API我们可以实现网络间通信，也可以实现进程间通信。

|Socket| 起源于 Unix ，Unix/Linux 基本哲学之一就是“一切皆文件”，都可以用“打开(open) –> 读写(write/read) –> 关闭(close)”模式来进行操作。因此 Socket 也被处理为一种特殊的文件。|

### Node中的Net模块

net模块提供了基于流的`TCP`或`IPC`服务器(net.createServer())和客户端(net.createConnection()) 的异步网络 API。这个模块下提供的两个类：`net.Server`和`net.Socket`，就是用来建立服务器和客户端所需要用到的。

### 利用Net模块ftp建立服务器

#### 知识准备

ftp（File Transfer Protocol）即**文件传输协议**，是用于在网络上进行文件传输的一套标准协议，使用客户/服务器模式，属于网络传输协议的应用层。

FTP服务一般运行在20和21两个端口。端口20用于在客户端和服务器之间传输数据流，而端口21用于传输控制流，并且是命令通向ftp服务器的进口。当我们连接到ftp服务器的时候使用21端口进行登录操作，而下载和上传文件则是使用21端口。

建立socket连接时，必须要提供地址和端口号，同时还需要用户名和登录密。另外ftp协议中有一种匿名模式：ftp服务器开启了匿名功能后，客户端在连接的时候只需要提供`anonymous`用户名即可登录，这个帐号不需要密码，虽然通常要求输入用户的邮件地址作为认证密码，但这只是一些细节或者此邮件地址根本不被确定。

匿名服务绝大多数情况只允许现在文件，就算能允许上传文件那也只是上传到指定的文件夹。

ftp命令：ftp客户端与服务端是通过特定格式进行通信的，当某客户端连接到FTP 服务器后，客户端发送指令：`<指令> [参数] <命令结束符："\r\n"> `，服务会按以下格式返回：`<状态码> [参数或说明] <命令结束符："\r\n">`。常见命令附后。

ftp状态码: 客户端请求服务器后返回的信息中带有状态码，客户端可以依据状态码执行操作。全状态码附后。


#### 具体实现

**本地建立ftp server**

- 确定状态码数据：

```json
// replyCode.json
{
  "110": "Restart marker reply.",
  "120": "Service ready in nn minutes.",
  "125": "Data Connection already open; transfer starting.",
  "150": "File status okay; about to open data connection.",
  "200": "Command okay.",
  "202": "Command not implemented, superfluous at this site.",
  "211": "System status, or system help reply.",
  "212": "Directory status.",
  "213": "File status.",
  "214": "Help message.",
  "215": "NAME system type.",
  "220": "Service ready for new user.",
  "221": "Service closing control connection.",
  "225": "Data connection open; no transfer in progress.",
  "226": "Closing data connection.",
  "227": "Entering Passive Mode.",
  "230": "User logged in, proceed. This status code appears after the client sends the correct password. It indicates that the user has successfully logged on.",
  "250": "Requested file action okay, completed.",
  "257": "'\"'PATHNAME'\"' created.",
  "331": "User name okay, need password.",
  "332": "Need account for login.",
  "350": "Requested file action pending further information.",
  "421": "Error 421 Service not available, closing control connection.\n            Error 421 User limit reached\n            Error 421 You are not authorized to make the connection\n            Error 421 Max connections reached\n            Error 421 Max connections exceeded",
  "425": "Cannot open data connection.",
  "426": "Connection closed; transfer aborted.",
  "450": "Requested file action not taken.",
  "451": "Requested action aborted: local error in processing.",
  "452": "Requested action not taken. Insufficient storage space in system.",
  "500": "Syntax error, command unrecognized, command line too long.",
  "501": "Syntax error in parameters or arguments.",
  "502": "Command not implemented.",
  "503": "Bad sequence of commands.",
  "504": "Command not implemented for that parameter.",
  "530": "User not logged in.",
  "532": "Need account for storing files.",
  "550": "Requested action not taken. File unavailable, not found, not accessible",
  "552": "Requested file action aborted. Exceeded storage allocation.",
  "553": "Requested action not taken. File name not allowed.",
  "10054": "Connection reset by peer. The connection was forcibly closed by the remote host.",
  "10060": "Cannot connect to remote server.",
  "10061": "Cannot connect to remote server. The connection is actively refused by the server.",
  "10066": "Directory not empty.",
  "10068": "Too many users, server is full."
}
```

- 增加具体操作函数

```javascript
// opration.js
module.exports = {
  AUTH: function() {
    this.send('502 SSL/TLS authentication not allowed.')
  },
  USER: function(username) {
    this.session.username = username
    this.send('331 User name okay, need password.')
  },
  PASS: function(password) {
    let socket    = this
    let username  = socket.username

    if (username == 'newghost' && password == 'dachun') {
      socket.send(230, 'Logged on')
    } else {
      socket.send(450, 'Ensure that you typed the correct user name and password combination.')
    }
  },
  PWD: function(args) {
    this.send('257 "/" is current directory')
  },
  TYPE: function(args) {
    this.send('200 Type set to I')
  },
  EPSV: function(args) {
    this.send('229 Entering Extended Passive Mode (|||30324|).')
  },
  PASV: function(args) {
    this.send('227 Entering Passive Mode (112,124,126,185,165,12).')
  },
  MLSD: function(args) {
    this.send('226 Successfully transferred "/"')
  },
  LIST: function(args) {
    this.send('502 Command not implemented.')

    this.send('502')
  }
}
```

- 主入口文件

```javascript
const REPLY_CODE = require('./reply.json')
const operation = require('./opration.js')
// index.js
let sendHandler = function(type, message) {
  let socket  = this
  let command
  if (arguments.length < 2) {
    if (REPLY_CODE[type]) {
      command = REPLY_CODE[type]
    } else {
      command = type.toString()
    }
  } else {
    command = type + ' ' + message
  }
  console.log('S:', command)
  socket.write(command + '\r\n')
}

let ftpServer = net.createServer(function(socket) {
  socket.session  = {}
  socket.send = sendHandler

  socket.send(220, 'Welcome to OnceDoc FTP Server')

  let onCommand = function(buffer) {
    let buffers = buffer.toString()
    let lines   = buffers.split('\r\n')

    for (let i = 0, l = lines.length; i < l; i++) {
      let line = lines[i]
      if (line) {
        console.log('C:', line)

        let cmds  = line.split(' ')
        let cmd   = cmds[0].toUpperCase()
        let arg   = cmds.slice(1)

        let func  = operation[cmd]

        func ? func.apply(socket, arg) : socket.send(502)
      }
    }
  }

  socket.on('data', onCommand)
    .on('end',  function() {
      console.log('end', arguments)
    })
    .on('close', function () {
      console.log('close', arguments)
    })
    .on('timeout', function () {
      console.log('timeout', arguments)
    })
    .on('error', function (err) {
      console.log('error', arguments)
    })

}).on('error', function(err) {
  // handle errors here
  console.error(err)
})
ftpServer.listen({ port: 21 }, function() {
  console.log('opened server on', ftpServer.address())
})
 ```

这时候，运行index.js文件，就可以启动一个server服务器

**访问ftp server**

编写一个简单测试脚本：

```javascript
// client.js
const Net = require('net');
let socket = new Net.Socket();

let host = '127.0.0.1'
let port = 21

socket.on('connect', () => {
  console.log('已连接')
  socket.write('USER anonymous \r\n');
});
socket.on('end', (e) => {
  console.log('已断开', e)
});
socket.on('error', (e) => {
  console.log('发生错误', e)
});
socket.on('data', buffer => {
  let str = buffer.toString()
  console.log(str)
  if (/331/.test(str)) {
    socket.write('PWD anonymous@ \r\n');
  }
})
socket.connect(port, host);
```

运行上面代码将会得到如下结果

```bash
$ node client.js
已连接
220 Welcome to OnceDoc FTP Server

331 User name okay, need password.

257 "/" is current directory

```

### 总结

Node中的Net模块负责网络相关功能，它提供了TCP/IP协议族一整套的API，在上面中我们只是实现了FTP的登录功能，其它的文件下载、上传将在并没有实现。`net.Socket`类提供的各类API，我们不管是想实现`Webscoket`还是`Http`，只需对其二次封装即可。非常方便。


### 参考信息

#### 全命令


| 命令   | 描述   |
| :--- | :--- |
|ABOR	| (ABORT)此命令使服务器终止前一个FTP服务命令以及任何相关数据传输。|
|ACCT|(ACCOUNT)此命令的参数部分使用一个Telnet字符串来指明用户的账户。|
|ADAT|		(AUTHENTICATION/SECURITY DATA)认证/安全数据|
|ALLO|		为接收一个文件分配足够的磁盘空间|
|APPE|		增加|
|AUTH|	认证/安全机制|
|CCC|	清除命令通道|
|CDUP|		改变到父目录|
|CONF|	机密性保护命令|
|CWD|		改变工作目录|
|DELE|		删除文件|
|ENC|	隐私保护通道|
|EPRT|	为服务器指定要连接的扩展地址和端口|
|EPSV|	进入扩展被动模式|
|FEAT|	获得服务器支持的特性列表|
|HELP|		如果指定了命令，返回命令使用文档；否则返回一个通用帮助文档|
|LANG|	语言协商|
|LIST|		如果指定了文件或目录，返回其信息；否则返回当前工作目录的信息|
|LPRT|	为服务器指定要连接的长地址和端口|
|LPSV|	进入长被动模式|
|MDTM|	返回指定文件的最后修改时间|
|MIC|	完整性保护命令|
|MKD|		创建目录|
|MLSD|	如果目录被命名，列出目录的内容|
|MLST|	提供命令行指定的对象的数据|
|MODE|		设定传输模式（流、块或压缩）|
|NLST|		返回指定目录的文件名列表|
|NOOP|		无操作（哑包；通常用来保活）|
|OPTS|	为特性选择选项|
|PASS|		认证密码|
|PASV|		进入被动模式|
|PBSZ|	保护缓冲大小|
|PORT|		指定服务器要连接的地址和端口|
|PROT|	数据通道保护级别|
|PWD|		打印工作目录，返回主机的当前目录|
|QUIT|		断开连接|
|REIN|		重新初始化连接|
|REST|		从指定点重新开始传输|
|RETR|		传输文件副本|
|RMD|		删除目录|
|RNFR|		从...重命名|
|RNTO|		重命名到...|
|SITE|		发送站点特殊命令到远端服务器|
|SIZE|	返回文件大小|
|SMNT|		挂载文件结构|
|STAT|		返回当前状态|
|STOR|		接收数据并且在服务器站点保存为文件|
|STOU|		唯一地保存文件|
|STRU|		设定文件传输结构|
|SYST|		返回系统类型|
|TYPE|		设定传输模式（ASCII/二进制).|
|USER|		认证用户名|
|XCUP|	改变之当前工作目录的父目录|
|XMKD|	创建目录|
|XPWD|	打印当前工作目录|
|XRCP|	|
|XRMD|	删除目录|
|XRSQ|	|
|XSEM|	发送，否则邮件|
|XSEN|	发送到终端|

#### 全状态码

1xx - 肯定的初步答复
这些状态代码指示一项操作已经成功开始，但客户端希望在继续操作新命令前得到另一个答复。 • 110 重新启动标记答复。 
- 120 服务已就绪，在 nnn 分钟后开始。 
- 125 数据连接已打开，正在开始传输。 
- 150 文件状态正常，准备打开数据连接。 

2xx - 肯定的完成答复
一项操作已经成功完成。客户端可以执行新命令。 • 200 命令确定。 
- 202 未执行命令，站点上的命令过多。 
- 211 系统状态，或系统帮助答复。 
- 212 目录状态。 
- 213 文件状态。 
- 214 帮助消息。 
- 215 NAME 系统类型，其中，NAME 是 Assigned Numbers 文档中所列的正式系统名称。 
- 220 服务就绪，可以执行新用户的请求。 
- 221 服务关闭控制连接。如果适当，请注销。 
- 225 数据连接打开，没有进行中的传输。 
- 226 关闭数据连接。请求的文件操作已成功（例如，传输文件或放弃文件）。 
- 227 进入被动模式 (h1,h2,h3,h4,p1,p2)。 
- 230 用户已登录，继续进行。 
- 250 请求的文件操作正确，已完成。 
- 257 已创建“PATHNAME”。 

3xx - 肯定的中间答复
该命令已成功，但服务器需要更多来自客户端的信息以完成对请求的处理。 • 331 用户名正确，需要密码。 
- 332 需要登录帐户。 
- 350 请求的文件操作正在等待进一步的信息。 

4xx - 瞬态否定的完成答复
该命令不成功，但错误是暂时的。如果客户端重试命令，可能会执行成功。 • 421 服务不可用，正在关闭控制连接。如果服务确定它必须关闭，将向任何命令发送这一应答。 
- 425 无法打开数据连接。 
- 426 Connection closed; transfer aborted. 
- 450 未执行请求的文件操作。文件不可用（例如，文件繁忙）。 
- 451 请求的操作异常终止：正在处理本地错误。 
- 452 未执行请求的操作。系统存储空间不够。 

5xx - 永久性否定的完成答复
该命令不成功，错误是永久性的。如果客户端重试命令，将再次出现同样的错误。 • 500 语法错误，命令无法识别。这可能包括诸如命令行太长之类的错误。 
- 501 在参数中有语法错误。 
- 502 未执行命令。 
- 503 错误的命令序列。 
- 504 未执行该参数的命令。 
- 530 未登录。 
- 532 存储文件需要帐户。 
- 550 未执行请求的操作。文件不可用（例如，未找到文件，没有访问权限）。 
- 551 请求的操作异常终止：未知的页面类型。 
- 552 请求的文件操作异常终止：超出存储分配（对于当前目录或数据集）。 
- 553 未执行请求的操作。不允许的文件名。 
常见的 FTP 状态代码及其原因
- 150 - FTP 使用两个端口：21 用于发送命令，20 用于发送数据。状态代码 150 表示服务器准备在端口 20 上打开新连接，发送一些数据。 
- 226 - 命令在端口 20 上打开数据连接以执行操作，如传输文件。该操作成功完成，数据连接已关闭。 
- 230 - 客户端发送正确的密码后，显示该状态代码。它表示用户已成功登录。 
- 331 - 客户端发送用户名后，显示该状态代码。无论所提供的用户名是否为系统中的有效帐户，都将显示该状态代码。 
- 426 - 命令打开数据连接以执行操作，但该操作已被取消，数据连接已关闭。 
- 530 - 该状态代码表示用户无法登录，因为用户名和密码组合无效。如果使用某个用户帐户登录，可能键入错误的用户名或密码，也可能选择只允许匿名访问。如果使用匿名帐户登录，IIS 的配置可能拒绝匿名访问。 
- 550 - 命令未被执行，因为指定的文件不可用。例如，要 GET 的文件并不存在，或试图将文件 PUT 到您没有写入权限的目录。





