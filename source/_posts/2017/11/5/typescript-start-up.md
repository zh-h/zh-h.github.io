---
title: TypeSript 快速上手
date: 2017-11-05
tags: [TypeScript]
categories: [Javascript]
---

## TypeScript 特性

TypeScript 是一种由微软开发的自由和开源的编程语言。它是 JavaScript 的一个超集，而且本质上向这个语言添加了可选的静态类型和基于类的面向对象编程。TypeScript 通常先进与 ECAMScript 标准实现，如当前试验阶段的装饰器语法也会最先得到使用。

使用 TypeScript 带来的好处:
1. 可以使用最新的 ES2017 语言特性
2. 确定类型的智能代码提示,像单纯 Javascrit 还要人肉对比变量名拼写就十分痛苦。
3. 编辑代码时具有及时错误检查功能，可以避免诸如输错函数名这种明显的错误
4. 非常精准的代码重构功能

## 环境配置

### 编译器

使用 npm 全局安装 tsc。

### tsconfig.json 配置文件

每个 TypeScript 项目都需要一个 tsconfig.json 描述，告知编译器进行怎么样的处理：

```json
	{
	"compilerOptions": {
	"module": "commonjs",
	"target": "es5",
	"outDir": "out",
	"lib": [
	"es6"
	],
	"sourceMap": true,
	"rootDir": "src"
	},
	"exclude": [
	"node_modules"
	]
}
```

详细的字段更改请参考：https://zhongsp.gitbooks.io/typescript-handbook/content/doc/handbook/tsconfig.json.html

### 安装声明文件
非 Typescript 编写的模块需要提供 tds 文件供编译器和 IDE 使用。Node.js 的 API 也要提供，使用`npm I @types/node` 进行安装。

### 创建 vscode 运行配置
使用 vscode 对 Typescript 有很好的支持，如果需要执行单一的 Typescrit 源码还要另外设置运行配置。

```json
	{
	"type": "node",
	"request": "attach",
	"name": "Attach by Process ID",
	"processId": "${command:PickProcess}"
	},
	{
	"type": "node",
	"request": "launch",
	"name": "Launch Program",
	"program": "${file}", // 这个需要配置
	"outFiles": [
	"${workspaceFolder}/out/**/*.js" // 这个
	]
}
```

如上面的配置，执行了一个任务在后台运行`tsc -P -w`实时监控文件变更并及时编译，运行配置为执行源码编译输出对应的 js 文件。

## 语法概览

使用 Java 或者 C# 编程经验的人会对此非常熟悉，因为 Typescrit 的设计者就是 C# 的设计者。

### 类型声明

与 Java 最大的不同可能就是类型声明是写在变量名之后。
```javascript
let a:string = 'a'
function test(a:string){
    console.log(a)
}
```
方法声明包括输入参数和返回参数等，每个参数的顺序都是有意义的。
```javascript
function call(cb:(a:string):void){
    cb('lalal')
}
```

### 基础类型

以下是 TypeScript 中的几种基础类型：

- boolean为布尔值类型，如let isDone: Boolean = false
- number为数值类型，如let decimal: number = 6;
- string为字符串类型，如let color: string = 'blue'
- 数组类型，如let list: number[] = [ 1, 2, 3 ]
- 元组类型，如let x: [ string, number ] = [ "hello", 10 ]
- 枚举类型，如enum Color { Red, Green, Blue }; let c: Color = Color.Green
- any为任意类型，如let notSure: any = 4; notSure = "maybe a string instead"
- void为空类型，如let unusable: void = undefined
- null和undefined
- never表示没有值的类型，如function error(message: string): never { throw new Error(message); }

多种类型可以用|隔开，比如number | string表示可以是number或string类型

### 接口（interface）

以下是接口的几种常见形式：
```javascript
// 定义具有 color 和 width 属性的对象
interface SuperConfug {
  color: string;
  width: number;
}

// readonly 表示只读，不能对其属性进行重新赋值
interface Point {
  readonly x: number;
  readonly y: number;
}

// ?表示属性是可选的，
// [propName: string]: any 表示允许 obj[xxx] 这样的动态属性
interface SquareConfig {
  color?: string;
  width?: number;
  [propName: string]: any;
}

// 函数接口
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```
实际上 TypeScript 的接口还有很多种的表示形式，详细信息可以参考这里：TypeScript Hankbook - Interfaces
函数

以下是几种函数接口的定义方式：
```javascript
// 普通函数
function add(a: number, b: number): number {
  return a + b;
}

// 函数参数
function readFile(file: string, callback: (err: Error | null, data: Buffer) => void) {
  fs.readFile(file, callback);
}

// 通过 type 语句定义类型
type CallbackFunction = (err: Error | null, data: Buffer) => void;
function readFile(file: string, callback: CallbackFunction) {
  fs.readFile(file, callback);
}

// 通过 interface 语句来定义类型
interface CallbackFunction {
  (err: Error | null, data: Buffer): void;
}
function readFile(file: string, callback: CallbackFunction) {
  fs.readFile(file, callback);
}
```
以上几种定义方式有着微妙的差别，还是需要在深入实践 TypeScript 后才能合理地运用。详细信息可以参考这里：TypeScript Handbook - Functions
类

TypeScript 的类定义跟 JavaScript 的定义方法类型一样，但是增加了public, private, protected, readonly等访问控制修饰符：
```javascript
class Person {
  protected name: string;
  constructor(name: string) {
    this.name = name;
  }
}

class Employee extends Person {
  private department: string;

  constructor(name: string, department: string) {
    super(name);
    this.department = department;
  }

  public getElevatorPitch() {
    return `Hello, my name is ${this.name} and I work in ${this.department}.`;
  }
}
```
### 泛型

TypeScript 的泛型和接口使得具备较强的类型检查能力的同时，很好地兼顾了 JavaScript 语言的动态特性。以下是使用泛型的简单例子：
```javscript
function identity<T>(arg: T): T {
  return arg;
}

const map = new Map<string, number>();
map.set('a', 123);

function sleep(ms: number): Promise<number> {
  return new Promise<number>((resolve, reject) => {
    setTimeout(() => resolve(ms), ms);
  });
}
```