---
title: Stylelint规范
date: 2017-10-14 13:38:17
tags: css
---

### 是什么

`Stylelint`是一种样式格式检查工具，能够在开发阶段有效帮助开发人员规范代码，减少错误。

`Stylelint`的`Github`地址：`https://github.com/stylelint/stylelint`

###  怎么用

官网提供了`Stylelint`的各种实现，这里以`webpack`为例。

在项目目录下安装`stylelint-webpack-plugin`插件

```javascript
PS D:\workspace> cd myproject
PS D:\workspace\myproject> npm install --save-dev stylelint stylelint-webpack-plugin
```

上面的代码表示在`myproject`件夹中安装`stylelint`插件。

打开`webpack`的配置文件，使用插件

```javascript
const StyleLintPlugin = require('stylelint-webpack-plugin')
module.exports = {
  plugins:[
    new StyleLintPlugin({
    // options
    })
  ]
}
```

上面提到的options是我们提供给插件的配置项。具体如下

- `configFile`: 指明`stylelint`插件的配置文件地址，默认`undefined`，这时插件会去索引整个项目文件夹查找配置文件，默认三种形式：1、在`package.json`中以`stylelint`为键名的属性；2、`.stylelintrc`文件；3、`stylelint.config.js`命名并输出JS对象。
- `context`: 上下文，默认继承`webpack`配置文件。
- `emitErrors`: 以错误形式显示代码中`Stylelint`错误，而不是警告的方式，默认为 `true`。
- `failOnError`: 在`webapck`构建（build）时，当`stylelint`检查到错误，终止`webpack`的构建进程. 默认: `false。`
- `files`:指定要检查的文件地址，必须相对于`options.context`选项. 默认: `['**/*.s?(a|c)ss']`
- `formatter`: `stylint`错误在`console`中显示的方式. 默认: `require('stylelint').formatters.string`                                   
- `lintDirtyModulesOnly`: 只检查修改过后的文件并在启动时跳过检查. 默认: `false`
- `syntax`: 指定样式文件的语法，例如sass. 默认: `undefined`
- `quiet`:直接打印错误到console面板. 默认: `true`.

> 配置项举例

```示例
options: {
  configFile: '../.stylelintrc',
  context: '../src/assets/',
  emitErrors: true,
  failOnError: false,
  files: 'less/*.less',
  formatter: require('stylelint').formatters.string,
  lintDirtyModulesOnly: false,
  syntax: 'less',
  quiet: true  
}
```

现在配置了插件，但是最核心的规则（rules）我们还没有配置，规则在上面提到的`configFiles`文件中配置。具体的配置规则可参考官网提供的规则`https://stylelint.io/user-guide/rules/`

> 下面是我自己使用的规则

```ini
{
  "rules": {
    # 允许空类
    "block-no-empty": null,
    # 不允许无效color名
    "color-no-invalid-hex": true,
    # 声明模块不允许重复属性
    "declaration-block-no-duplicate-properties": true,
    # 禁止未知伪类选择器,自定义伪类可以使用‘my-’开头
    "selector-pseudo-class-no-unknown": [true, {
      "ignorePseudoClasses": ["/^dir-/", "pseudo-class"]
    }],
    # 禁止未知伪元素选择器
    "selector-pseudo-element-no-unknown": true,
    # 禁止未知元素选择器,自定义伪类可以使用‘dir-’开头
    "selector-type-no-unknown": [true, {
      "ignore": ["custom-elements", "default-namespace"],
      "ignoreNamespaces": ["/^dir-/", "custom-namespace"]
    }],
    # 禁止未知媒体查询功能标签
    "media-feature-name-no-unknown": true,
    # 禁止空注释
    "comment-no-empty": true,
    # 高层级的类名须放在低层级类前面，比如a{}放在a+a{}前面
    "no-descending-specificity": true,
    # 禁止重复命名选择器
    "no-duplicate-selectors": true,
    # 禁止额外的分号
    "no-extra-semicolons": true,
    # 禁止使用双斜杠注释
    "no-invalid-double-slash-comments": true,
    # 禁止未知的动画
    "no-unknown-animations": true,
    # 禁止使用命名的颜色名 如white
    "color-named": "never",
    # url地址禁止双斜杠开头
    'function-url-no-scheme-relative': true,
    # 数字类型只允许两位小数
    'number-max-precision': 2,
    # 限制每行定义的属性
    'declaration-block-single-line-max-declarations': 1,
    # 限制类名的命名方式：只能小写字母并且以-连接
    'selector-class-pattern': "^[a-z]+-*[a-z]*-*[a-z]*$",
    # 限制id的命名方式：只能小写字母并且以-连接
    'selector-id-pattern': "^[a-z]+-*[a-z]*-*[a-z]*$",
    # 选择器只允许小写
    'selector-type-case': 'lower',
    # 伪类只小写
    'selector-pseudo-element-case': 'lower',
    # 限制类与类之间的空行数
    'selector-max-empty-lines': 1,
    # 限制颜色代码使用小写
    'color-hex-case': lower,
    # 颜色代码能使用简写时使用简写
    'color-hex-length': short,
    # 方法里面逗号后面需要空格
    'function-comma-space-after': "always",
    # 方法里面逗号前面不要空格
    'function-comma-space-before': "never",
    # 使用双引号
    'string-quotes': double,
    # 属性内容使用小写
    'value-keyword-case': lower,
    # 属性名使用小写
    'property-case': lower,
    # 类名前面需要空行
    "comment-empty-line-before": [ "always", {
      "ignore": ["stylelint-commands", "after-comment"]
    } ],
    # 区块结束花括号前不允许空行
    "block-closing-brace-empty-line-before": "never",
    # 属性冒号后面需要空格
    "declaration-colon-space-after": "always",
    # 属性冒号前面不要空格
    "declaration-colon-space-before": "never",
    # 缩进使用两个空格
    "indentation": 2,
    # 只允许一行空格
    "max-empty-lines": 1,
    # 类前面要空行
    "rule-empty-line-before": [ "always", {
      "except": ["first-nested"],
      "ignore": ["after-comment"]
    } ],
    # 限制单位
    "unit-whitelist": ["em", "rem", "%", "s", "px"]
  }
}

```

