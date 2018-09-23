---
title: 静态类型检查器：Flow
date: 2018-08-17 18:34:02
tags: check
---

### 写法示例

一个方法：

```javascript
function foo (x, y) {
  return x * y;
}
foo('a', 'b'); // NaN
```

我们声明这个方法时期望函数调用时传入的两个参数,x和y，会是数字。但是JavaScript非常灵活，`*`运算符左右两侧参数可以为任意类型，所以很难去预测foo方法会被传入什么参数。

Flow其中一个功能就解决了这个问题，例子如下：

```javascript
// @flow
function foo (x: number, y: number): number {
  return x * y;
}
foo('a', 'b');
// Cannot call foo with 'a' bound to x because string [1] is incompatible with number [2].

//  [2] 2│ function foo (x: number, y: number): number {
//      3│   return x * y;
//      4│ }
//  [1] 5│ foo('a', 'b'); // flow警告

// Cannot call foo with 'b' bound to y because string [1] is incompatible with number [2].

//  [2] 2│ function foo (x: number, y: number): number {
//      3│   return x * y;
//      4│ }
//  [1] 5│ foo('a', 'b'); // flow警告
```

### flow的主要功能

- 类型检查：可以是增加了类型注解的，也可以是没有增加注解的
- 提高代码提高阅读性，清晰的注解有助于团队项目维护
- 预测程序代码中可能会发生的错误，例如：null.length

### 安装与使用

#### 安装

Flow目前可以支持macOS、Linux(64位)、Windows(64位),这里介绍mac安装方式：

```bash
npm install --save-dev flow-bin
```
或者
```bash
brew update
brew install flow
```

#### 使用

初始化项目：

```
$> mkdir flow_project
$> cd flow_project
$> flow init
```

这是可以看到新建了一个flow的配置文件: .flowconfig，我们可以使用命令：flow check，检测根目录以及所有子目录的代码。

新建文件index.js,里面添加如下内容：

```javascript
// @flow
function foo (x, y) {
  return x * y;
}
foo('a', 'b'); // NaN
```

运行命令：`flow check`，这时候flow会帮助我们去检查代码了。

注意在需要检查代码的文件上方添加注释`// @flow`，当然也可以使用命令：`flow check --all`强制检查所有代码。

#### 使用flow服务：

每次使用`flow check`命令检查代码并不是很好，因为每次检查时都需要启动flow服务，将所有的需要的代码检查遍。其实我们可以使用flow服务，每次只需要检查改变了的代码就好了。

启动命令很简单：`flow`, 关闭命令：`flow stop`,每次更改了文件，只需再次运行`flow`命令，即可检查文件。

### 类型注解

上面已经提到了类型注解，例如`let x: string = 'test'`，这段代码就使用了类型注解，格式为`:`开头，后面跟上类型，可以用来标明变量生命和函数参数，以及函数返回指。

有时候类型注解并不是必须的，例如使用`*`操作符的时候，不用类型注解flow也能自动帮我们识别错误。

>> any注解表示任意类型，一般不推荐使用，除非知道自己在干什么。

### 运行flow代码

因为类型注解不是 Js 规范的一部分，所以我们得移除它，官方文档建议使用 Babel。

转换步骤：

- 安装babel-cli: `npm install -g babel-cli`
- 添加package.json文件
- 安装Flow 转换器（babel插件）： `npm install babel-plugin-transform-flow-strip-types`
- 增加.babelrc配置文件：`echo '{"plugins": ["transform-flow-strip-types"]}' > .babelrc`

现在，运行命令：`babel --watch=index.js --out-dir=./build`, 只要我们的index.js文件发生改变，就会创建相应的正常的Js版本，保存在 build/ 目录下。当然我们可以指定某个文件夹，例如src，这时命令就为：`babel --watch=./src --out-dir=./build`

### flow内置的类型

flow的内置类型包括： 
- 1、Javscript原生类型，例如：number、string、boolean、function；
- 2：any类型，是所有类型的超集，同时也是所有类型的子集，可以简单地理解为任意类型；
- 3： mixed类型，mixed类型与any类型及其相似，但是mixed类型不是所有类型的子集。

理解any类型和mixed类型：

看下面的例子：

```javascript
// @flow
let genericVariable: any = 20;
let numericVariable: number;

numericVariable = genericVariable; // 正常运行
```
但是
```javascript
// @flow
let genericVariable: mixed = 20;
let numericVariable: number;

numericVariable = genericVariable; // 报错
// Cannot assign genericVariable to numericVariable because mixed [1] is incompatible with number [2].
```

还有其他很多类型，这里就不一一列举了，可以查看官方文档：`https://flow.org/en/docs/types/`

### 一些总结

相比较`TypeScript`而言，`flow`的学习成本可以是相当的低了。 一些原来没有使用静态检查工具的项目，想要迁移到`flow`也是相当地简单，例如可以是一个文件一个文件的迁移、Babel 和 ESLint 都有对应的 Flow 插件以支持语法，可以完全沿用现有的构建配置。迁出的成本低，万一哪天不想用 `flow` 了，用`babel-plugin-transform-flow-strip-types`转一下，就得到符合规范的 ES。一个典型的案例就是`Vue`2.0重构代码的时候使用了`flow`，而不是`TypeScript`。`React`源码同样也是采用的`flow`.