---
title: 浅谈Javascript中继承实现
date: 2018-10-15 22:23:04
tags: inherit
---

### 为什么Javascript中使用的是原型继承？

继承的方式主要分为两种：原型继承和类继承，当然还有其它的一些，例如：抽象继承。首先弄清楚什么是原型继承，什么是类继承。

在面向对象的程序设计中，一个普遍的说法继承是解决了代码复用的问题，当然使用不好也常常会产生高耦合的代码。个人认为继承也有表示一个对象的特性的功能，例如在JAVA的抽象继承中，一个方法可能只会接收实现了某个接口的实例。类继承和原型继承是实现继承的两种不同的方式。

可以类比生活中的继承关系，老虎、狮子都属于猫科，猫科动物属于食肉动物，类继承会严谨地进行分类：先构造出基类（食肉动物），然后派生出子类（猫科动物），继续派生出子类（老虎类、狮子类），通过老虎类、狮子类就可以构造出很多狮子、老虎；原型继承中没有类的概念，为了描述狮子、老虎、猫科动物、食肉动物四者的关系，需要使用原型链的概念：需要为上面四个抽象概念声明构造函数，每个构造函数都有一个原型属性（prototype），原型属性描述了继承关系，例如狮子、老虎构造函数的原型都指向了由猫科动物这个构造函数示例化出来的对象。原型继承在具体语言实现起来更加灵活和简单。

Javascript之所以使用原型继承，是因为Brendan Eich在设计Javascript这门语言之初就是希望简单点，只需要处理简单的浏览器端事件即可。而且当时正是面向对象编程最兴盛的时期。Javascript引入对象的概念，但是没有引入类的概念，为了实现继承才使用了原型的概念。

### es5中实现继承

Javascript实现继承方式有很多种，下面简单几种：

准备一个父级构造函数

```javascript
function Cats (name) { // Cats => 猫科动物
 // 属性
  this.name = name;
  // 实例方法
  this.walk = function() {
    console.log(this.name + ' is walking。');
  }
}
// 原型方法
Cats.prototype.eat = function(food) {
  console.log(this.name + ' is eating ' + food);
};
```

- 原型链继承, 这是一种比较简单的做法，但是也是极不推荐的做法，因为：来自原型对象的所有属性被所有实例共享、创建子类实例时，无法向父类构造函数传参

```javascript
function Tiger () {}
Tiger.prototype = new Cats()
Tiger.prototype.name = 'tiger'

let tiger = new Tiger()
tiger.name // tiger
tiger.walk() // tiger is walking。
tiger.eat('monkey') // tiger is eating monkey
tiger instanceof Tiger //true 
tiger instanceof Cats //true
```

- 构造继承：使用父类的构造函数来增强子类实例，等于是复制父类的实例属性给子类

```javascript
function Tiger(name){
  Cats.call(this)
  this.name = name || 'big cat'
}

// Test Code
var tiger = new Tiger()
tiger.name // big cat
tiger.walk() // big cat is walking
tiger.eat('monkey') // tiger.eat is not a function
tiger instanceof Cats // false
tiger instanceof Tiger // true
```

这种方式可以实现多继承，但是缺点也是明显的：实例并不是父类的实例，只是子类的实例、只能继承父类的实例属性和方法，不能继承原型属性/方法、无法实现函数复用，每个子类都有父类实例函数的副本，影响性能。不推荐。

- 实例继承: 为父类实例添加新特性，作为子类实例返回

```javascript
function Tiger(name){
  var instance = new Cats()
  instance.name = name || 'big cat'
  return instance;
}

// Test Code
var tiger = new Tiger()
tiger.name // big cat
tiger.walk() // big cat is walking。
tiger.eat('monkey') // big cat is eating monkey
tiger instanceof Cats // true
tiger instanceof Tiger // false
```

这种做法的缺点是：实例是父类的实例，不是子类的实例、不支持多继承

- 拷贝继承：

```javascript
function Tiger(name){
  var cats = new Cats();
  for(var p in cats){
    Tiger.prototype[p] = cats[p]
  }
  Tiger.prototype.name = name || 'big cat'
}

// Test Code
var tiger = new Tiger()
tiger.name // big cat
tiger.walk() // big cat is walking。
tiger.eat('monkey') // big cat is eating monkey
tiger instanceof Cats // false
tiger instanceof Tiger // true
```

这种做法缺点是：效率较低，内存占用高（因为要拷贝父类的属性）、无法获取父类不可枚举的方法（不可枚举方法，不能使用for in 访问到）

- 组合继承：通过调用父类构造，继承父类的属性并保留传参的优点，然后通过将父类实例作为子类原型，实现函数复用

```javascript
function Tiger(name){
  Cats.call(this, name)
}
Tiger.prototype = new Cats()
Tiger.prototype.constructor = Cats;
// Test Code
var tiger = new Tiger('big cat')
tiger.name // big cat
tiger.walk() // big cat is walking。
tiger.eat('monkey') // big cat is eating monkey
tiger instanceof Cats // true
tiger instanceof Tiger // true

```

这种做法的缺点：调用了两次父类构造函数，生成了两份实例（子类实例将子类原型上的那份屏蔽了），优点也是明显的：弥补了方式2的缺陷，可以继承实例属性/方法，也可以继承原型属性/方法、既是子类的实例，也是父类的实例、不存在引用属性共享问题、可传参、函数可复用.

- 寄生组合继承: 通过寄生方式，砍掉父类的实例属性，这样，在调用两次父类的构造的时候，就不会初始化两次实例方法/属性，避免的组合继承的缺点

```javascript
function Tiger(name){
  Cats.call(this, name)
}
(function(){
  // 创建一个没有实例方法的类
  var Super = function(){};
  Super.prototype = Cats.prototype;
  //将实例作为子类的原型
  Tiger.prototype = new Super();
})();
Tiger.prototype.constructor = Cats; // 需要修复下构造函数

// Test Code
var tiger = new Tiger('big cat')
tiger.name // big cat'
tiger.walk() // big cat is walking。
tiger instanceof Tiger // true
tiger instanceof Cats //true
```

这种做法没有什么副作用，但是实现起来比较复杂。

### es6的class

es6中引入了class类的概念，但是底层实现仍然是原型，所以大家都说class是一个语法糖。当我们声明一个class时，可以等同的认为声明了一个构造函数。