---
title: 全局监听处理Promise拒绝
date: 2017-08-04 00:42:15
tags: [web]
categories: es6
---

## 为什么要监听

当Promise使用越来越广泛的时候，promise仍然存在着以下问题：

1. 当一个Promise在没有添加拒绝处理程序（catch/reject）的时候执行了拒绝（reject）操作，则js不会提示任何的失败信息

   >Node 8.0 版本已经有提示警告信息了,如下
   >
   >UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): rejected
   >
   >[DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
   >
   >PromiseRejectionHandledWarning: Promise rejection was handled asynchronously (rejection id: 1)
   >
   >大意为：捕获未处理拒绝/在以后版本中未被处理的Promise拒绝将会直接导致node线程结束/拒绝已经处理了
   >
   >经过测试，Firefox54.0.1和chrome60.0.x已经可以报错了，
   >

   ​

2. 当一个Promise执行了拒绝（reject）行为，Promise的特性导致我们在开发的时候无法得知拒绝（reject）行为进行的时间，也不知道是否被处理



## Node 环境

在Node.js中，Promise拒绝（reject）时会触发process对象的两个事件

- unhandledRejection ： 在事件队列中，当Promise被拒绝，没有提供拒绝处理程序或者拒绝未被及时处理。

  ```javascript
  let rejected
  rejected = Promise.reject('rejected')

  process.on('unhandledRejection', (reason, promise) => {
    console.log(rejected === promise)
    console.log(reason)
  })
  // 结果：
  // true
  //rejected
  ```

  注册unhandledRejection时，事件处理函数接受两个参数：拒绝信息reason和Promise对象

- rejectionHandled ：在事件队列中，当Promise被拒绝，在被拒绝处理程序处理时触发。

  ```javascript
  let rejected
  rejected = Promise.reject('rejected')
  setTimeout(() => {
   rejected.catch(reason => {
      console.log(reason);
    });
  });

  process.on('rejectionHandled', (promise) => {
    console.log(rejected === promise);
  });
  // 结果：
  // true
  //rejected
  ```

  注册rejectionHandled时,事件处理函数接受一个参数：Promise对象

  > 我们可以拿到对应的Promise对象，做一些处理



### 综合例子：完整展现Node监听过程

```javascript
process.on('unhandledRejection', (reason, promise) => {
  console.log('unhandledRejection:');
  console.log(promise);
  console.log(reason);
  console.log('----------------------');
});
process.on('rejectionHandled', (promise) => {
  console.log('rejectionHandled:');
  console.log(promise);
  console.log('----------------------');
});
let rejected = Promise.reject('rejected');

setTimeout(() => {
  rejected.catch(reason => {
    console.log(reason);
  });
}, 1000);
// 输出结果
// unhandledRejection:
// Promise { <rejected> 'rejected' }
// rejected
// ----------------------
// rejectionHandled:
// Promise { <rejected> 'rejected' }
// ----------------------
// rejected
```

拒绝发生未被处理时，触发unhandledRejection ，之后注册了拒绝处理函数又触发rejectionHandled

## 浏览器环境

与Node环境类似，两个事件是在window对象上触发的

- unhandledRejection ： 在事件队列中，当Promise被拒绝，没有提供拒绝处理程序或者拒绝未被及时处理。

- rejectionHandled： 在事件队列中，当Promise被拒绝，在被拒绝处理程序调用时触发。

> 这里接受event对象，与Node环境不同
>
> 也可以使用addEventListener方式

  ```javascript
   window.onunhandledrejection = event => {
      console.log(event.type)
      console.log(event.reason)
      console.log(event.promise === rsss)
      console.log('---------------------')
    }
    window.onrejectionhandled = event => {
      console.log(event.type)
      console.log(event.reason)
      console.log(event.promise === rsss)
      console.log('---------------------')
    }
    let rejected = Promise.rejected('rejected')
    setTimeout(() => {
      rejected.catch(res => {console.log(res)})
    })
  	// 输出结果：
 	//  unhandledrejection
  	//  rejected
  	//  true
  	//  ---------------------
  	//  Uncaught (in promise) rsss // 这里是报错信息 
	//  rejected  //（拒绝处理了）
  	//  rejectionhandled
  	//  rejected
  	//  true
  	//  ---------------------
  ```

  ​

