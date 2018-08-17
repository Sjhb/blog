---
title: TypeScript学习笔记（一）
date: 2018-06-19 18:45:56
tags: TypeScript
---

### 简单介绍

`TypeScript`是有微软开发的一种开源的变成语言，它是JavasSript的一个严格超集，并添加了可选的静态类型和基于类的面向对象编程。

使用`TypeScript`是一种趋势，使用`TypeScript`编写的代码，有较为严格的类型规范，它拓展了JavasSript的能力。

### 简单体验

#### 安装

```bash
$ npm install -g typescript
```

或者

```bash
$ npm global add typescript
```

#### 准备文件

使用`TypeScript`编写的代码，文件类型应当使用ts。新建一个`TypeScript`文件，命名为greeter.ts，内容如下：

```javascript
function greeter (person) {
  return 'Hello,' + person;
}
let user = 'Jhon';
document.body.innerHtml = greeter(user);
```
#### 编译

然后使用命令`tsc`编译文件greeter.ts:

```bash
$tsc greeter.ts
```

这个，在greeter.ts的同一层级下面应该会新加一个名为greeter.js的文件，打开来看会发现什么都没有改变。不着急，继续往下面走

#### 增加注释类型注解

变更greeter.ts，如下：

```typescript
function greeter (person: string) {
  return 'Hello,' + person;
}
let user = [1, 2, 3];
document.body.innerHtml = greeter(user);
```

再次运行上面的编译命令，就会发现报错了，提示：`Argument of type 'number[]' is not assignable to parameter of type 'string'.`.

因为我们声明`greete`方法的参数应当是一个`string`类型的，而实际调用传递的一个数组，所以就会报错，虽然`greeter.js`仍然编译出来了，但是程序可能并不会按我们开始设定那样运行。

### 基础类型

#### 布尔值

与`JavasSript`一样，`TypeScript`里也叫做`boolean`

```typescript
let disabled: boolean = true;
```

#### 数字

`JavaScript`和`TypeScript`中的数字都是浮点类型。在`TypeScript`中称之为`number`。 除了支持十进制和十六进制字面量，TypeScript还支持ECMAScript 2015中引入的二进制和八进制字面量。

```typescript
let decLiteral: number = 10; // 10 十进制
let hexLiteral: number = 0x10; // 16 十六进制 
let binaryLiteral: number = 0b101; // 5 二进制
let octalLiteral: number = 0o11; // 9 八进制
```

#### 字符串

`TypeScirpt`使用`string`表示字符串

```typescript
let name: string = 'Jhon';
```

当然，也可以使用模版字符串

```typescript
let name: string = 'Jhon';
let fullName: string = `Han ${name}`;
```

#### 数组

`JavaScript`中数组有两种定义方式：

- 在元素类型后面接上`[]`，表示由此类型元素组成的一个数组：

```typescript
let arr: number[] = [1, 2, 3];
```

- 使用数组泛型，`Array<元素类型>`：

```typescript
let arr: Array<number> = [1, 2, 3];
```

#### 元组（Tuple）

元组类型表示一个已知元素数量和类型的数组，各个元素之间的类型可以不必相同。

```typescript
let tuple: [string, number];
tuple = ['haha', 10];
let a: string = tuple[0]; // 正确
let b: string = tuple[1]; // 错误
```

当访问越界的数组元素的时候，元素类型将为联合类型：

```typescript
let tuple: [string, number];
tuple = ['haha', 10];
tuple[2] = 'gg'; // 正确，字符串可以赋值给(string | number)类型
tuple[3] = /test/; // 错误，正则不是（string | number)类型
```

联合类型稍后再做讨论

#### 枚举类型

`enum`类型是对JavaScript标准数据类型的一个补充,使用枚举类型可以为一组数值赋予友好的名字。

```typescript
enum Color {Red = 5, Green = 9, Blue}; // 枚举类型的值既可以是自动赋值，也可以手动赋值
let color: number = Color.Green;
```

编译出来就是

```javascript
var Color;
(function (Color) {
    Color[Color["Red"] = 5] = "Red";
    Color[Color["Green"] = 9] = "Green";
    Color[Color["Blue"] = 10] = "Blue";
})(Color || (Color = {}));
var color = Color.Green;
```

从编译结果我们可以看出来，我们既可以根据枚举类型的名称得到值，也可以根据值得到具体的名称

```typescript
enum Color {Red = 5, Green = 9, Blue};
let color:number = Color.Green;
let colorName:string = Color[10];
console.log(color);
console.log(colorName);

// 运行结果
// 9
// Blue
```

#### Any

`Any`表示任意的一种类型，类型检查器对这些值不会进行检查而是直接让它们通过编译阶段的检查。

```typescript
let x: any = 'may be string'；
x = true;
```

`Any`与`Object`比较：

`Object`类型的变量允许我们任意赋值给它，但是不能调用该变量上的任意方法，即使这个方法该变量确实有

```typescript
let x: Object = 'may be string';
let b: string = x.substr(1 ,5);
console.log(b);
// 错误：Property 'substr' does not exist on type 'Object'.

let x: any = 'may be string';
let b: string = x.substr(1 ,5);
console.log(b);
// 正确：运行结果：ay be
let c: any[] = [1, true, /test/]；
```

#### Void

`Void`表示没有任何类型，注意，`Any`表示任何类型

通常，当一个函数没有返回值的时候，其返回类型就会声明为`Void`

```typescript
function fun (): void {
  console.log('fun');
}
```

一个变量也可以声明为`Void`类型，但是一般没多大用，因为这个变量只能被赋予`null`或者`undefined`

#### Null 和 Undefined

在`TypeScript`中，`null`和`undefined`的类型也是`null`和`undefined`， 和 void相似，它们的本身的类型用处不是很大。

```typescript
let a: null = null;
let b: undefined = undefined;
```

可以把`null`和`undefined`赋值给其他类型的变量，例如`string`或者是`number`

```typescript
let a: number = null;
let b: string = undefined;
```

值得注意的是，官方并不推荐我们这么使用，因为这样使用会导致一些常见的问题，如果我们在编译命令上加入参数：`--strictNullChecks`,那么我们上面的代码就将会编译失败。因为这时，`null`和`undefined`只能赋值给void和它们各自。当然我们也可以使用联合类型：`string | number | null`。

#### Never

`Never`类型简而言之就是永远不存在的类型。例如永远会抛出异常的函数或者是内部有死循环而永远不会有返回的函数。变量也可以声明为`never`,当它们被永不为真的类型保护所约束时。

`never`类型是任何类型的子类型，也可以赋值给任何类型；然而，没有类型是`never`的子类型或可以赋值给`never`类型（除了`never`本身之外）。 即使`any`也不可以赋值给`never`。

```typescript
function a (): never {
 throw new Error('a')
}
```

#### 类型断言

`TypeScript`中，类型断言精髓就在于'更具体',即我比`TypeScript`更了解某个值的详细信息。通过类型断言这种方式可以告诉编译器，“相信我，我知道自己在干什么”。 类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时的影响，只是在编译阶段起作用。

类型断言的两种方式：

- 尖括号形式

```typescript
let a: any = [1,2,3,4];
let b: number = (<number[]>a).length;
```

- as语法

```typescript
let a: any = [1,2,3,4];
let b: number = (a as number[]).length;
```