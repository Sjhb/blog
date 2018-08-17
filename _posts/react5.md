---
title: React学习笔记五：WebPack+React
date: 2018-02-10 00:00:26
tags:
---

### 配置流程

1. 配置Webpack开发环境
2. 加入React

### Webpack配置

这里只是开发使用，不会配置得如同正式那样复杂。

#### 初始化文件夹，安装webpack依赖

```bash
$ npm init react-learn
$ cd react-learn
$ npm install --save-dev webpack
```

#### 准备开发目录

+--react-learn\
|+--config\
|----webpack.dev.js\
|+--node_modules\
|+--src\
|----index.html\
|----index.js\
|--package.json


#### 准备配置文件：webpack.dev.js

1. 加入插件：`html-webpack-plugin`,`clean-webpack-plugin`,`webpack-dev-server`

```dash
$ npm install --save-dev html-webpack-plugin clean-webpack-plugin webpack-dev-server
```

2. 配置入口文件、输出文件路径、上下文、解析
```javascript
context: path.resolve(__dirname, '..'),
entry: {
    app: path.resolve(__dirname, '..', 'src', 'index.js')
}
output: {
    path: path.resolve(__dirname, '..', 'dist'),
    filename: '[name].js',
    publicPath: '/'
},
resolve: {
    extensions: ['.js', 'jsx', 'json'],
    alias: {
    '@': resolve('src')
    }
}
```

3. 插件配置

```javascript
plugins: [
    new HtmlWebpackPlugin (
        {
            title: '[name].bundle.js',
            filename: 'index.html',
            template: path.resolve(__dirname, '..', 'src', 'index.html'),
            hash: true
        }
    ),
    new CleanWebpackPlugin(path.resolve(__dirname, '..', 'dist')),
    new webpack.NamedModulesPlugin(),
    new webpack.HotModuleReplacementPlugin()
]
```

4. 开发服务起配置

```javascript
devServer: {
    contentBase: path.resolve(__dirname, '..', 'dist'),
    publicPath: '/',
    port: '8082',
    historyApiFallback: true,
    hot: true
}
```

### Webpack加入React配置

#### 添加React、React-Dom包以及babel支持

```dash
npm install --save react react-dom
npm install --save-dev babel-cli babel-loader babel-preset-es2015 babel-preset-react react-hot-loader
```
#### 更新`webpack.dev.js`文件，使其支持babel以及热模块更新

1. 更新入口文件配置

```javascript
entry: {
    app: [
        'react-hot-loader/patch',
        // 开启 React 代码的模块热替换(HMR)
        'webpack-dev-server/client?http://localhost:8082',
        // 为 webpack-dev-server 的环境打包代码
        // 然后连接到指定服务器域名与端口，可以换成本机ip
        'webpack/hot/only-dev-server',
        // 为热替换(HMR)打包好代码
        // only- 意味着只有成功更新运行代码才会执行热替换(HMR)
        path.resolve(__dirname, '..', 'src', 'index.js')
        // 入口文件
    ]
}
```

2. 配置loader

```javascript
module: {
    rules: [
        {
            test: /\.jsx?$/,
            use: ['babel-loader'],
            exclude: /node_modules/
        }
    ]
}
```

#### 添加.babelrc文件，使应用支持es6语法、jsx语法以及loader配置

```json
{
  "presets": [
    ["es2015", {
      "modules": false
    }],
    "react"
  ],
  "plugins": [
    "react-hot-loader/babel"
  ]
}
```

#### 模版文件index.html以及入口文件改造

根据我们的配置，webpack会使用src目录下的index.html作为模版生成打包后的主文件，并把index.js作为js入口文件，我们需要改造它们使其支持热模块更新

- index.html

```html
<!DOCTYPE html>
<html lang="CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>React</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```

- index.js

```javascript
import React from 'react'
import ReactDOM from 'react-dom'

import { AppContainer } from 'react-hot-loader'
import App from './App'

// 封装 render 函数
const render = (App) => {
  ReactDOM.render(
    <AppContainer>
      <App />
    </AppContainer>,
    document.getElementById('root')
  )
}

render(App)

if (module.hot) {
    module.hot.accept('./App', () => render(App))
}
```

这时候程序已经成能够正常跑起来了

### 完整配置文件

```javascript
var path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const CleanWebpackPlugin = require('clean-webpack-plugin')
const webpack = require('webpack')

function resolve (dir) {
  return path.join(__dirname, '..', dir)
}
module.exports = {
  context: path.resolve(__dirname, '..'),
  entry: {
    app: [
      'react-hot-loader/patch',
      // 开启 React 代码的模块热替换(HMR)
      'webpack-dev-server/client?http://localhost:8082',
      // 为 webpack-dev-server 的环境打包代码
      // 然后连接到指定服务器域名与端口，可以换成本机ip
      'webpack/hot/only-dev-server',
      // 为热替换(HMR)打包好代码
      // only- 意味着只有成功更新运行代码才会执行热替换(HMR)
      path.resolve(__dirname, '..', 'src', 'index.js')
      // 入口文件
    ]
  },
  output: {
    path: path.resolve(__dirname, '..', 'dist'),
    filename: '[name].js',
    publicPath: '/'
  },
  resolve: {
    extensions: ['.js', 'jsx', 'json'],
    alias: {
      '@': resolve('src')
    }
  },
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        use: ['babel-loader'],
        exclude: /node_modules/
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin (
      {
        title: '[name].bundle.js',
        filename: 'index.html',
        template: path.resolve(__dirname, '..', 'src', 'index.html'),
        hash: true
      }
    ),
    new CleanWebpackPlugin(path.resolve(__dirname, '..', 'dist')),
    new webpack.NamedModulesPlugin(),
    new webpack.HotModuleReplacementPlugin()
  ],
  devServer: {
    contentBase: path.resolve(__dirname, '..', 'dist'),
    publicPath: '/',
    port: '8082',
    historyApiFallback: true,
    hot: true
  }
}
```