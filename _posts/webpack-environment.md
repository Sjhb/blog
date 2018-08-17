---
title: Webpack中打包环境的问题
date: 2018-07-28 20:45:43
tags: webpack environment
---

#### 开发和部署中的环境问题

前端代码运行环境往往是有多个的，常见的，开发环境、测试环境和正式环境。在这些环境中我们的项目有一些不同的、需要注意的地方，例如：代码压缩、请求地址、source map选项等等。如何区分这些环境呢？这些配置项是应该写进业务代码中还是集中起来作配置文件呢？配置文件怎么起作用？

#### 思考

需要明确的一点：代码组织应当是不区分环境的，也就是说不能根据当前的环境变量定义、书写不同的代码，这样会使代码开发线上运行的是两套代码，代码运行的结果变得不可预料。

代码打包过程中其实一般只区分打包前的**开发模式**以及打包后的**正式模式**，这里使用模式而不是环境，对应的环境变量`NODE_ENV`为`development`和`production`。根据这两个环境变量，我们可以确定是否开启代码压缩、模块相对路径展示、Gzip压缩、等等。至于测试环境，那是人为业务需要，而非代码本身应当关注的。

那么，开发环境、测试环境和正式环境有很多不同的地方，我们又如何去定义和区分它们呢？答案就是全局常量。可以全局定义请求地址、静态资源地址，而不用在代码中去区分运行环境了。

Webpack官方提供了一个`DefinePlugin`插件，可以用来创建可配置的全局常量。我们可以将在不同环境中不同的值使用配置文件保存起来。问题是如何告诉Webpack使用那个配置文件呢，或者是配置文件中的哪个变量呢？

结合Node相关的知识，其实很容易就想到进程`process`。`process`是一个Node提供的全局变量，提供当前js脚本运行的进程相关信息和操作进程的方法。这里我们只需要关注`argv`这个属性，从这个属性中可以取到运行当前js脚本所使用的命令。例如有一个build.js文件：

```javascript
console.log(process.arv)
```

运行它

```bash
node build.js --production
```

就会输出：
>> [ '/usr/local/bin/node', 
>> '/Users/wanghuan/mainto/project/maintovision-manage-fed/build.js',
>> '--production' ]

每次打包其实就是运行Wbpack脚本，我们可以通过识别命令中的参数来达到动态加载配置的目的

#### 具体实现

利用上面的提到的东西我们来动手实现一下吧，这里以静态资源地址、请求地址为例，首先是在Config中定义配置项：

```javascript
module.exports = {
  // 公共配置
  common: {
    assetsRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
  },
  development: {
    API_HOST: '"/api"',
    assetsPublicPath: '/',
  },
  production: {
    API_HOST: '"http://www..production.com"',
    assetsPublicPath: 'http://fed.dev.production.com',
  },
  opsproduction: {
    API_HOST: '"http://www.dev.opsproduction.com"',
    assetsPublicPath: 'http://fed.opsproduction.com',
  },
```

修改Webpack打包配置

```javascript
import config from './Config'
let param = process.argv[2]
let envConfig = config[param]
module.exports = {
  // ...
  entry: {
    'index': './src/module/pc/main.js'
  },
  output: {
    path: config.common.assetsRoot,
    filename: '[name].js',
    publicPath: envConfig.assetsPublicPath,
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    new webpack.DefinePlugin({
      'API_HOST': envConfig.API_HOST
    }),
  ]
}
```

修改aixios配置文件：

```javascript
// ...
// 默认请求host
axios.defaults.baseURL = process.env.API_HOST
// ...
```

修改打包命令:

```json
{
  "scripts": {
    "dev": "node build/dev,js development",
    "build": "node build/build.js production",
    "opsbuild": "node build/build.js opsproduction"
  }
}
```

#### 总结

值得注意的是，如果我们使用`webpack-dev-server`时，使用`webpack-dev-server --inline --progress --config build/webpack.dev.conf.js` 这样的命令启动，而不是文件内启动，那我们就需要单独为起写配置了。