---
title: typeScript2
date: 2018-06-22 23:24:55
tags: typeScript
---

### 函数声明

`TypeScript`中函数声明推崇使用ES6中函数声明的方式，即使用`let`、`const`来代替`var`。

其次`TypeScript`还引入了其它的ES6新特性，例如：解构赋值、对象和数组展开、函数默认值。

### 接口

`TypeScript`的核心原则之一是对值所具有的*结构*进行类型检查。 在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。

`TypeScript`中的接口与其他语言中的接口是不同的，它并不能被实现，只是用来约束数据结构

一个简单的例子

```typescript
interface inter {
  label: number
}
function func(obj: inter){
  console.log(obj.label);
}
let testObj = {
  label: 10,
  test: 'other data'
};
func(testObj)
```

#### 可选属性

上面的例子中，接口`inter`中的label是必须的，我们可以为`inter`设置一些可选属性，方式如下：

```typescript
interface inter {
  label: number;
  text?: string;
}
function func(obj: inter){
  console.log(obj.label);
  console.log(obj.text);
}
let testObj = {
  label: 10
};
func(testObj)
```

这时候，如果不声明`text`属性为可选属性，调用时会直接报错。定义可选属性主要是：如果存在属性，能约束类型。

#### 只读属性

使用`readonly`关键词可将接口中的属性声明为只读的

```typescript
interface Person {
  readonly name: string;
}
let tom: Person = { name: 'tom'};
tom.name = 'tom cat'; // error
```

使用`ReadonlyArray<T>`类型可以将一个数组声明为只读的，不能对其任何赋值操作

```typescript
let a: [number] = [1,2,3];
let arr: ReadonlyArray<number> = a;
arr[0] = 10;
// Index signature in type 'ReadonlyArray<number>' only permits reading.
arr.push('11');
// roperty 'push' does not exist on type 'ReadonlyArray<number>'.
arr = [4,5];
/* Type '[number, number, number]' is not assignable to type '[number]'.
* Types of property 'length' are incompatible.
* Type '3' is not assignable to type '1'.
*/
```

但是下面这样是可以编译通过的

```typescript
let arr: ReadonlyArray<number> = [1,2,3];
arr = [4,5];
```