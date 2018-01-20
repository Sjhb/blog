---
title: Nodejs中的进程
date: 2018-01-02 00:40:33
tags:
---

### Nodejs中的进程

众所周知，JavaScript是一门单线程语言，同时采用事件队列的方式来实现异步操作。Node使用JavaScript作为开发语言，因为JavaScript异步特性，赋予了Node处理高并发的能力。

但是，正式因为JavaScript单线程的特点，反过来又会限制Node本身，想想看，在Node服务器中，当有一个线程出现错误，导致当前进程死掉，那么整个服务器也就宕机了。当然这只是一个方面。

其实，我任务Node进程引入多进程的一个重要原因是充分利用CPU资源

### 进程对象：process

`process`是Node提供的进程对象，包含一系列的属性和方法，便于操作进程。它是一个内置对象，不需要模块引入可直接使用。

**`process`常用属性：**

- `execPath`: 用来Node的绝对路径，例如在我的电脑是：/usr/local/bin/node

- `stdin`: 这个属性的值是一个可用于读入标准输入流的对象，拥有Stream输入流对象的方法和属性，但是改对象默认是暂停状态，在使用时需要使用`process.stdin.resume()`方法先将其恢复为可写入状态

- `stdout`: 这个属性的值是一个可用于写入标准输出流的对象，拥有Stream输出流对象的方法和属性。

> `stdin`和`stdout`可用于进程间通信

```javascript

// main.js

process.stdin.resume()

process.stdin.on('data', (chunk) => {

  console.log('输入了：\t' + chunk)

})

// $ node main.js

// test

// 输入了：        test

```

- `stderr`:  这个属性的值是一个可用于写入标准错误输出流的对象

- `argv`: 运行Node程序时的命令行参数，第一个参数值与`execPath`值一样，表示Node程序的地址；第二个参数表示本次执行的文件地址；后面的一次为本次命令的参数

- `env`: 包含一了一系列的执行环境信息，例如在我的电脑上，在REPL中输入`process.env`会打印一下信息

```javascript

{

  TERM_PROGRAM: 'vscode',

  TERM: 'xterm-256color',

  SHELL: '/bin/bash',

  TMPDIR: '/var/folders/cp/019sgmcd0vx2dfqgxcfct87w0000gn/T/',

  Apple_PubSub_Socket_Render: '/private/tmp/com.apple.launchd.Nbdx6iVeXl/Render',

  TERM_PROGRAM_VERSION: '1.19.1',

  USER: 'wanghuan',

  SSH_AUTH_SOCK: '/private/tmp/com.apple.launchd.i8gWatNMR9/Listeners',

  __CF_USER_TEXT_ENCODING: '0x1F5:0x19:0x34',

  VSCODE_NODE_CACHED_DATA_DIR_68043: '/Users/wanghuan/Library/Application Support/Code/CachedData/0759f77bb8d86658bc935a10a64f6182c5a1eeba',

  GOOGLE_API_KEY: 'AIzaSyAQfxPJiounkhOjODEO5ZieffeBv6yft2Q',

  PATH: '/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/mysql/bin:/usr/local/mysql/bin',

  PWD: '/Users/wanghuan/project/node-learn',

  LANG: 'zh_CN.UTF-8',

  XPC_FLAGS: '0x0',

  XPC_SERVICE_NAME: '0',

  SHLVL: '2',

  HOME: '/Users/wanghuan',

  VSCODE_NLS_CONFIG: '{"locale":"zh-cn","availableLanguages":{"*":"zh-cn"}}',

  LOGNAME: 'wanghuan',

  VSCODE_IPC_HOOK: '/Users/wanghuan/Library/Application Support/Code/1.19.1-main.sock',

  VSCODE_PID: '68043',

  _: '/usr/local/bin/node'

}

```

- `pid`: 当前进程的PID

**`process`常用方法**

- `memoryUsage()`: 现实进程内存使用量，其中res: 进程内存消耗总量；heapTotal:V8分配的内存量；heapUsed：V8内存消耗量，单位为字节

```javascript

 process.memoryUsage()

//

// {

//   rss: 21925888,

//   heapTotal: 7708672,

//   heapUsed: 5123248,

//   external: 31956

// }

```

- `nextTick(fun))`: 讲传入的函数：fun推迟到同步方法执行完毕，异步方法之前执行

```javascript

console.log('test')

setTimeout(() => {

  console.log('timeout')

}, 0)

process.nextTick(() => {

  console.log('nextTick')

})

// 输出

// test

// nextTick

// timeout

```

- `abort()`: 使程序异常终止，并向Nodejs应用程序发送SIGABRT信号

- `chdir()`: 修改Node应用程序使用的工作目录

- `cwd`: 返回当前的工作目录

- `exit([code])`: 退出进程，code为可选参数，表示操作系统提供的退出码，默认为0，即正常状态

- `kill(pid, [siganl])`: 结束指定的进程，`pid`表示进程的PID，`signal`表示发送的信号

- `hrtime()`: 测试一段代码运行的时间，如下所示，数组第一个元素单位秒，第二个为纳秒，加起来即为代码运行时间

```javascript

let time = process.hrtime()

setTimeout(() => {

  console.log(process.hrtime(time))

}, 1000)

// 输出：[ 1, 6989549 ]

```

**`process`常用事件**

- `exit`: 进程退出，`process.exit()`方法会触发此事件，可用来监听子进程或者本进程结束，来指定需要执行的回调函数

- 信号事件：用于监听Node应用程序的各种事件，例如'SIGINT'，表示退出s

### Nodejs开启多进程的方式



利用Node提供的`child_process`模块可以开启子进程，并且有三种开启子进程的方式

#### **spawn**

- `child_process.spawn(command, [args], [option])`: 参数command：运行的命令；args：可选参数，表示运行命令的参数；option：可选参数，用于指定开启子进程的选项。

option的属性如下：

1. `cwd`: 指定子进程的工作目录

2. `stdio`: 指定子进程的标准输入流、标准输出流、标准错误流，格式为字符串或者包含三个元素的素组

3. `env`: 环境变量，对象形式

4. `detached`: 布尔类型，表示是否是领头进程，如果是，在父进程结束后子进程仍然会存在，否在会随副进程结束

```javascript

// spawn.js

const cp = require('child_process')

let child = cp.spawn('node', ['./spawn_child.js', '-r', 't'], {

  stdio: 'inherit',

  detached: false

})

child.on('close', (code) => {

  console.log('close事件：code\t' + code + '\n')

})

child.on('exit', (code) => {

  console.log('exit事件：code\t' + code + '\n')

})

child.on('error', (err) => {

  console.log('error事件：' + err)

})

// spawn_child.js

process.stdout.write('子进程PID：' + process.pid + '\n')

process.stdout.write('接收的参数：\n')

process.argv.forEach(element => {

  process.stdout.write( element + '\n')

});

process.exit()

```

运行spawn.js：文件将会输出：

> 子进程PID：90560
> 接收的参数：
> /usr/local/bin/node
> /Users/wanghuan/project/node-learn/spawn_child.js
> -r
> t
> exit事件：code  0
> close事件：code 0

特别的，close事件和exit事件是有区别的，close事件是子进程退出并且输入输出流都关闭时才会触发，例如当子进程和父进程共享输入输出流时，而exit事件仅仅是子进程退出就会触发。

#### **fork**

fork方式是专门用来开启进程来运行Node模块文件的。

- `child_process.fork(modulePath, [args], [option])`: 参数modulePath：模块路径及文件名；args：可选参数，表示运行命令的参数；option：可选参数，用于指定开启子进程的选项。

父子进程通信可使用send方法：

 `child.send(message, [sendHandle])`

 `father.send(message, [sendHandle])`

`message`为自定义的字符串，`sendHandle`为可选参数，可以指定为回调函数，也可以指定为socket对象

然后使用message事件监听信息

 `process('message', (m, setHandle) => {})`

参数m为接收的为发送的message字符串，当send方法传递了一个socket对象时，`setHandle`的值为传递的对象，否则为空

```javascript

// fork.js

const cp = require('child_process')

let child = cp.fork('./spawn_child.js', ['-r', 't'],)

child.send('主进程发送 fork 测试')

child.on('message', (m) => {

  process.stdout.write('主进程接收信息：' + m + '\n')

  process.kill(child.pid)

})

child.on('close', (code) => {

  console.log('close事件：code\t' + code + '\n')

})

child.on('exit', (code) => {

  console.log('exit事件：code\t' + code + '\n')

})

child.on('error', (err) => {

  console.log('error事件：' + err)

})

// fork_child.js

process.stdout.write('子进程PID：' + process.pid + '\n')

process.stdout.write('接收的参数：\n')

process.argv.forEach(element => {

  process.stdout.write( element + '\n')

})

process.on('message', (m) => {

  process.stdout.write('子进程接收信息：' + m + '\n')

  process.send('子进程发送fork测试')

})

```

运行spawn.js：文件将会输出：

> 子进程PID：90637

> 接收的参数：

> /usr/local/bin/node

> /Users/wanghuan/project/node-learn/fork_child.js

> -r

> t

> 子进程接收信息：主进程发送 fork 测试

> 主进程接收信息：子进程发送fork测试

> exit事件：code  null

> close事件：code null

#### **exec**

- `child_process.exec(command, [option], [callback])`: 参数commond：命令；option：可选参数，用于指定开启子进程的选项；callback为回调函数，默认三个参数，error、stdout、stderr，分别表示子进程开启错误时的错误对象，stdout为子进程标准输出流，stderr为子进程标准错误输出流。

option参数可选属性有：

  1. env

  2. cwd

  3. encoding: 输出流的编码格式

  4. timeout: 子进程超时时间

  5. maxbuffer: 指定缓存区缓存标准输出和标准错误输出的最大长度，超出时子进程将会被强制关闭

  6. killSignal: 关闭子进程的信号

```javascript

const cp = require('child_process')

cp.exec('git pull', (error, stdout, stderr) =>{

  console.log('错误信息：' + error)

  console.log('标准错误输出流：' + error)

})

cp.exec('echo hello exec', (error, stdout, stderr) =>{

  console.log('标准输出流:'+stdout)

})

```

运行上面的代码将会输出

> 错误信息：Error: Command failed: git pull
> fatal: Not a git repository (or any of the parent directories): .git
> 标准错误输出流：Error: Command failed: git pull
> fatal: Not a git repository (or any of the parent directories): .git
> 标准输出流:hello exec