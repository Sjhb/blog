---
title: 一些VUE-CLI 3.X配置技巧
date: 2018-12-15 21:56:56
tags: vue vue-cli3
---

`VUE-CLI 3`出来也蛮久了，这里总结一下一些配置技巧

### 审查已有配置

`VUE-CLI 3`把`webpack`配置都隐藏起来了，但是经常我们需要深度配置，这时候我们就需要审查已有的`webpack`配置。`vue-cli-service`暴露了 `inspect`命令用于审查解析好的 `webpack` 配置, 在实际应用中，我们可以使用命令`vue inspect > output.js`将`webpack`配置、包括链式访问规则和插件的提示输出到`output.js`。

可以在`package.json`文件中增加命令：`"inspect": "vue inspect > output.js --verbose"`，这样每次可以少敲几个单词

这个操作很有用，后面的操作都会用到。

关于`vue inspect`命令参数：

| 命令 | 定义 |
|:--:|:--:|
|--mode <mode>| inspect a specific module rule |
|--plugin <pluginName>| inspect a specific plugin |
|--rules| list all module rule names|
|--plugins|  list all plugin names |
|-v --verbose|  Show full function definitions in output |

### 链式操作

`Vue CLI 3`内部的插件、`loader`配置是通过`webpack-chain`维护的。这个库提供了一个`webpack`原始配置的上层抽象，使其可以定义具名的 loader 规则和具名插件，并有机会在后期进入这些规则并对它们的选项进行修改。

#### 插件配置

通过链式访问怎么配置`webpack`插件呢？首先得明白插件是怎么配置在`webpack`中的,然后再谈怎么通过`webpack-chain`添加插件。下面以`stylelint-webpack-plugin`插件为例，展示`VUE-CLI 3`插件安装过程。


`styleint`是一个工具，可以帮助我们规范css代码，纠正css代码错误。

安装插件包：

```bash
yarn add -dev stylelint-webpack-plugin stylelint stylelint-codeframe-formatter
npm install --save-dev stylelint-webpack-plugin stylelint stylelint-codeframe-formatter
```

准备`stylelint`配置文件，在当前根目录下新建`.stylelintrc`文件，并添加以下内容：

```json
{
  "rules": {
    // 禁止使用命名的颜色名 如white
    "color-named": "never"
  }
}
```

通过链式访问，配置`webpack`插件，添加`stylelint-webpack-plugin`插件及其配置:

```javascript
const StyleLintPlugin = require('stylelint-webpack-plugin')
// CodeframeFormatter 可将出错代码以代码片段的形式展示
const CodeframeFormatter = require('stylelint-codeframe-formatter');

// 配置信息在这里
module.exports = {
  chainWebpack: config => {
      config.plugin('stylelint')
        .use(StyleLintPlugin, [{
          failOnError: false,
          files: ['**/*.less'],
          formatter: CodeframeFormatter,
          fix: true,
          syntax: 'less'
        }])
        .end()
  }
}
```

测试代码：添加一行css代码`color: black`，就可以看到报错信息了：

![](/images/vue-cli3.jpeg)

#### loader配置

除了插件配置，我们可以通过链式操作配置`module`的loader。下面以`less-loader`配置为例：

在实际项目中，经常我们会把公用的`less`属性、通用方法、资源文件、等等统一整理。但是在使用时都需要`import`到当前`less`文件，像这种频繁引入的文件有没有一种机制一次性引入呢？

`style-resources-loader`是一款插件，可以为我们的样式文件（css、less、scss、sass、stylus）注入样式资源，帮助我们管理资源文件。利用这个插件我们可以配置`less-loader`配置默认要引入资源，就可以少些很多代码，具体配置如下：

安装：

```bash
yarn add -dev style-resources-loader
npm install --save-dev style-resources-loader
```

审查已有的`webpack`配置，关注`less-loader`,因为我的`less`代码都是单独保存管理，所以我只需要配置`/* config.module.rule('less').oneOf('vue') */`这条链路即可，如果是单文件组件并且`less`、`javascript`、`html`写在一起的话，则需要配置`/* config.module.rule('less').oneOf('vue-modules') */`这一条链路。

在`vue.config.js`中更改`less-loader`的配置：

```javascript
// 配置信息在这里
module.exports = {
  /* config.module.rule('less').oneOf('vue').use('style-resources-loader') */
  chainWebpack: config => {
    config.module
      .rule('less')
      .oneOf('vue')
      .use('style-resources-loader')
      .loader('style-resources-loader')
      .options({
        patterns: [path.resolve(__dirname, 'src/assets/less/base.less'), path.resolve(__dirname, 'src/pages/Index/assets/image/index.less')],
        injector: 'prepend'
      }).end()
  }
}
```

上面我分别添加了`base.less`和`image/index.less`两个文件管理基础样式和图片，这样我就可以直接在`less`使用诸如`@black`这样的变量而不用频繁引入`base.less`文件。

### babel插件配置

`VUE-CLI 3`中，配置babel有两种方法：

- 配置`.babelrc`文件

- 更改`VUE-CLI`配置文件，即`vue.config.js`文件

例如，我们假设要配置`@babel/plugin-proposal-class-properties`这个babel插件，这个插件将会把class的静态属性转化为赋值式的声明语法。

安装：`yarn add -dev @babel/plugin-proposal-class-properties`

如果我们配置在`vue.config.js`文件中，就需要进行链式操作，链式操作具体可以查看文档：https://cli.vuejs.org/zh/guide/`webpack`.html#%E9%93%BE%E5%BC%8F%E6%93%8D%E4%BD%9C-%E9%AB%98%E7%BA%A7。

首先审查原来的babel配置：

```javascript
/* config.module.rule('js') */
{
  test: /\.jsx?$/,
  exclude: [
    filepath => {
      // ...
    }
  ],
  use: [
    /* config.module.rule('js').use('cache-loader') */
    {
      loader: 'cache-loader',
      options: {
        cacheDirectory: '/Volumes/***/node_modules/.cache/babel-loader',
        cacheIdentifier: 'b6ffa844'
      }
    },
    /* config.module.rule('js').use('babel-loader') */
    {
      loader: 'babel-loader'
    }
  ]
}
```

可以关注注释，注释已经表明了此处链式结构： `/* config.module.rule('js').use('babel-loader') */`。下面，添加`vue.config.js`，并添加一下内容：

```js
module.exports = {
  chain`webpack`: config => {
    /* config.module.rule('js').use('babel-loader') */
    config.module
      .rule('js')
      .use('babel-loader')
      .options({
        plugins: ['@babel/plugin-proposal-class-properties']
      })
  }
}
```

使用`.babelrc`配置就简单了，直接添加即可：` plugins: ['@babel/plugin-proposal-class-properties']`

### babel编译`node_module`中的包

在实际项目中，有一些npm包，例如`axios`，这些包本身并不提供像下兼容的版本，这就需要进行编译。一种做法是显示地增加`promise-ployfill`这样的额外包到我们的项目中，为这些包提供兼容；一种是在打包过程中，连同`axios`这样的包一起编译。

默认的，`VUE-CLI 3`不会编译`node_module`中的包。如果要编译`node_module`中的包，可以在`vue.config.js`中增加配置：`transpileDependencies: ['axios']`，这里以`axios`为例。

### 多页配置

`VUE-CLI 3`给我们提供了多页模式，我们直接根据项目配置即可，具体操作下面的例子：

假设现在有`page1`、`page2`两个文件夹，分别对应两个单页，具体的文件目录：

```
.
├── pages
│   ├── page1
│   │   ├── App.vue
│   │   └── main.js
|   ├── page2
│   │   ├── App.vue
│   │   └── main.js
└── vue.config.js
```

1. 在public文件夹中增加HTML文件模版：`page1.html`、`page2.html`

2. 配置`vue.config.js`文件如下：

```javascript
module.exports = {
  // 多页应用配置，对应src/pages文件夹，默认入口为index
  pages: {
    page1: {
      entry: 'src/pages/page1/main.js',
      template: 'public/page1.html',
      filename: 'page1.html',
      // 当使用 title 选项时,template 中的 title 标签需要是 <title><%= html`webpack`Plugin.options.title %></title>
      title: 'page1',
      // 在这个页面中包含的块，默认情况下会包含 提取出来的通用 chunk 和 vendor chunk。
      chunks: ['chunk-vendors', 'chunk-common', 'page1']
    },
    page2: {
      entry: 'src/pages/page2/main.js',
      template: 'public/page2.html',
      filename: 'page2.html',
      // 当使用 title 选项时,template 中的 title 标签需要是 <title><%= html`webpack`Plugin.options.title %></title>
      title: 'page2'
      // 在这个页面中包含的块，默认情况下会包含 提取出来的通用 chunk 和 vendor chunk。
      chunks: ['chunk-vendors', 'chunk-common', 'page2']
    }
  }
}
```

上面的配置更像是`html-`webpack`-plugin`和wepack配置的结合体

### postcss插件管理

`postcss`非常强大，在`VUE-CLI 3`中只是对其进行了一些基础配置，这里简单介绍一下它的插件配置。例如现在要增加一个`postcss`插件：`postcss-pxtorem`，这个插件会将css代码中的`px`按照设置好的比例转换为`rem`

- 需要安装：`yarn add -dev postcss-pxtorem`

- 找到`postcss`的配置，根据个人实际喜好，`postcss`的配置有可能是单独的配置文件：`postcss.config.js`,也可能是在`package.json`文件中。

- 增加配置，以`package.json`中配置为例

```json
{
  "postcss": {
    "plugins": {
      "postcss-pxtorem": {
        "rootValue": 37.5,
        "propList": [
          "*"
        ]
      }
    }
  }
}
```

### 总结

`VUE-CLI 3`配置起来并不难，在掌握了`webpack`配置方法后，`vue-cli-service`更像是一个工具去管理`webpack`配置。