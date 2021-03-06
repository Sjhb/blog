---
title: 怎么写出易于维护的代码
date: 2018-10-24 18:49:28
tags: goodcode
---

### 怎么写出易于维护的代码

从我个人角度来讲，代码的维护行体现在两个方面：一是代码可读性，二是架构的合理性。有句话是这样说的：代码是写给人看的，然后才交给机器运行的。代码可维护性的重要程度其实大家都知道，区别在于重视程度罢了。在今天这篇文章里我总结了怎么提高代码可维护性的一些个人心得

#### 代码规范

代码规范啥的都是老生常谈的问题，其能有效提高我们的代码维护性，在一个团队中，每个成员都有自己的编码风格，有些更是可以用“风骚”来形容了。一套完整的技术规范所能呈现的最好效果就是：团队里面的人写出来的代码就像是一个人写出来的一样。

#### 变量、函数命名

尽量短而语义化地命名。

代码规范中肯定会涉及到命名规范这一类的东西，例如驼峰式命名、私有变量采用下滑线开头、全局常量头部声明等等。我这里分享一些我个人觉得比较好的做法：

- 函数名以动宾结构命名，做到语义化：`setName`、`checkUserState`

- 不要随意简写，简写时使用约定俗成的：`address` => `addr`、`document` => `doc`、`window` => `win`

- 常量命名应当与变量区分开来

#### 循环

- 减少中间变量，除非这个变量有助于理解。

- 适当的迭代递归通常都比循环要易读，但是需要使用尾调用去优化：

```javascript
// 一个计算阶乘的例子
function factorial (n) {
  return n > 1 ? n * factorial(n - 1) : 1;
}

function factorialWithVia (n) {
  let total = i = n;
  while (i > 1) {
    total *= (--i);
  }
  return total;
}
```

- 优化条件判断语句，嵌套循环语句配上多个条件判断，简直就是地狱。使用卫语句优化是一个不错的选择：

> 条件表达式通常有2种表现形式。第一：所有分支都属于正常行为。第二：条件表达式提供的答案中只有一种是正常行为，其他都是不常见的情况。
> 这2类条件表达式有不同的用途。如果2条分支都是正常行为，就应该使用形如`if...else...`的条件表达式；如果某个条件极其罕见，就应该单独检查该条件，并在该条件为真时立刻从函数中返回。这样的单独检查常常被称为“卫语句”。

```javascript
function fun (arg) {
  if (A(arg)) {
    B(arg);
  } else {
    if (C(arg)) {
      D(arg)
    } else (E(arg)) {
      F(arg)
      ....
    }
  }
}

function fun (arg) {
  if (A(arg)) return B(arg);
  if (C(arg)) return D(arg);
  F(arg);
  ...
}
```

- 最好的情况是避免嵌套的循环，嵌套循环一般都意味着多位数组。没办法的时候，也许，我们可以把每一层代码抽离为独立的方法。

#### 精简代码

- 短路写法： `if (A()) {B()}` => `A() && B()`。

- 使用`||`设置默认值： `var userName = getUserName() || 'user'`

- 简单条件判断可以用三元表达式替代

#### 学会抽象

抽象其实有点考验程序员代码功底，常常会涉及到组件封装、设计模式、高阶函数之类的东西。我觉得只要保持一个想法即可：这段代码逻辑跟之前写的有点相似，我能不能把公用部分抽出来？看一个程序员的好坏就是要看他会不会思考。

#### 函数式编程

`javaScript`虽然是一门面向对象的语言，但是它也可以进行函数式编程。一个说法就是：函数式编程告诉计算机做什么，命令式编程是告诉计算机怎么做，这也从另外一个角度说明了函数式编程对读代码的程序员会友好一点。

这里需要coder们一定程度上掌握函数式编程，下面简单列举一些对我们有帮助的点

- 函数是引用透明且没有副作用的：简单来说就是给一个函数相同输入值，这个函数就会输出相同的结果，且不会依赖和改变外部环境，也不会修改传入的参数。而且，函数式编程中的函数本身指的是数学上的映射关系，而非我们常见的函数。

- 当功能被拆分为各个函数去实现的时候，我们写测试和调试会更加容易。因为每个函数都是纯函数，多个纯函数组成函数也是纯函数。

那具体到实际写代码的时候这种思维怎么帮到我们呢？我觉得可以这么说：当我们写比较复杂应用的时候，我们可以拆分不同的功能模块，每个模块不依赖外部环境，根据具体参数决定输出什么。在主流程中我们只需要关注主流程即可。每一个功能模块则单独为其写单元测试。另外一方面，使用纯函数减少了这种情况出现的可能性：我只改了这里，为什么那边出现了问题

这里推荐一个知乎回答，我觉得还是讲得蛮到位的：https://www.zhihu.com/question/28292740/answer/40336090

#### 拆分代码

我这里讲的拆分代码指的是：当一个函数或者一段代码行数过多的时候我们应当去拆分它、优化它，一段代码很长那就说明是有问题的。比如switch语句往往会很长，我们是不是可以考虑拆分出来单独写个方法，又比如一个form表单提交会涉及到校验、提交、结果展示，我们是不是可以将用户提示、数据处理拆分出来呢。

#### 架构

架构听起来很厉害，确实，能做架构师的都是思维能力比较强的人，对业务了解、把控很深。现在我们把视野缩小，聚焦到前端这一块。

以前前端不谈架构是因为没有能力，现在前端能做的事情多了，我们也就可以来做做一些简单的架构。首先要明白我们在前端这个一亩三分地內，架构指的是什么，个人认为是对系统整体的抽象描述和各个组件之间的组合。大到复杂的spa应用，小到一个简单的单页，我们都可以谈架构。一个页面分哪几分组成，哪些部分可重用是架构，一个系统分哪几个大型模块，每个模块之间怎么协作，怎么有效组合也是架构

我这里想强调的就是以大局观的眼光审视自己做的东西，以架构的角度取思考的时候就会想到可用性、拓展性、安全性、稳定性、等等一些列的东西。各个组件合理设计，有效结合。设计差劲的代码维护起来很难，避免不了重构的命运。

现在前端其实比较少考验我们架构这方面的东西了，我们常见的react、vue全家桶都帮我们设计好，这也是前端门槛低的一个方面。但当我们涉足到一些新的领域的时候，这些能力就会显现出来。

#### 其它

- 避免`dead code`，即一些永远不会运行的代码。例如在代码中写上开发环境才会运行的代码，在生产环境就不会运行。

- 避免注释，让代码说明功能。

- 适当加一个空行会有意想不到的效果。

- 缩短变量声明与使用其代码的距离。
