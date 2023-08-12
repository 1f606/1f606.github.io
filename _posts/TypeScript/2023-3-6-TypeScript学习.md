---
layout:     post
title:      TypeScrip学习
subtitle:   
date:       2023-3-6
author:     sq
header-img: 
catalog: true
tags:
    - TypeScript
---
# TypeScript from scratch
定义类型的方法：在标识符、函数的小括号后添加冒号和类型。

## 环境

```shell
#安装
npm i typescript -g

# 初始化
tsc --init

#实时监听ts文件变化编译为js文件
tsc -w

#编译 ts 文件为 js 文件
tsc fileName
```

```shell
npm i ts-node -g
npm i @types/node -D

# 可以直接运行 typescript 文件
ts-node fileName
```

## 类型
原始类型包括：`number`， `boolean`，`string`，`undefined`， `null`，`bigInt`, `symbol`。

`Object`，`object`，`any`， `number[]`， `Array<number>`，`<string， number>`， `enum`，`void`。

变量定义类型后如果被赋值为其他类型、调用其他类型方法就会报错，`any` 不受影响。

### Number, String, Boolean, Symbol
首字母大写的Number String Boolean Symbol很容易与小写的原始类型number string boolean symbol混淆，前者是相应原始类型的包装对象。

原始类型可以赋值给包装对象类型，包装对象类型无法赋值给对应的原始类型。

不要用包装对象类型注解。
### 大小Object 和 空对象字面量
小object `object` 代表引用类型，可以赋值引用类型的值。

大object `Object` 代表所有拥有toString hasOwnProperty方法的类型。所以，所有原始类型和非原始类型都可以赋值给大Object。

空对象和大Object可以互相代替，它们两的特性一致。

在严格模式下 null 和 undefined 类型不能赋给大小 Object。

大Object包含原始类型，而小object仅包含非原始类型。那么大Object是不是小object的父类型？实际上，大Object不仅是小object的父类型，同时也是小
object的子类型。

```typescript
type FatherType = object extends Object ? true : false; // true
type ChildType  = Object extends object ? true : false; // true
```

### undefined, null and void
`void` 一般用于没有任何返回值的函数。不能用来赋值。

`undefined` 和 `null` 是所有类型的子类型。也就是可以赋值给任何类型。

如果在tsconfig.json里配置了"strictNullChecks": true，null就只能赋值给any、unknown和它本身的类型（null），undefined就只能赋值给any、
unknown、void和它本身的类型（undefined）。

### never
never 类型表示的是那些永不存在的值的类型。never 也是所有类型的子类型。可以赋值给任何类型。但没有任何类型是 never 的子类型。

值会永不存在的两种情况：
- 如果一个函数执行时抛出了异常，那么这个函数就永远不存在返回值。
- 函数中执行无限循环的代码，也就是死循环。

在 typescript 中，可以用 `never` 实现全面性检查。

```typescript
type Type = string | number;

function inspectWithNever (param: Type) {
    if (typeof param === 'string') {
    } else if (typeof param === 'number') {
        // 到这里是 never 类型
        const check: never = param;
    }
}
```

假如后面 Type 被修改为：
```typescript
type TYpe = string | number | boolean;
```

那么编译就会失败，因为 boolean 无法赋值给 never。

通过这种方法，保证了类型的绝对安全。

### unknown, any
所有类型都可以被分配给 `any` 和 `unknown`。

`unknown` 只能赋值给自身或 `any`；`any` 可以赋值给任何类型（never除外）。

如果不缩小类型，无法对 `unknown` 做任何操作。

### narrow缩小类型
缩小类型用 typeof 或类型断言。

```typescript
const a:unknown = '1';

// error
a.split('');

if (typeof a === 'string') a.split();

// 类型断言
(a as string).split('');
```

### 数组的类型

#### 类型+方括号

```typescript
// 数组的项中不允许出现其他的类型
let fibonacci: number[] = [1, 1, 2, 3, 5];

interface m {
    name: string
}
const items: m[] = [{name: ''}];

// 内置伪数组类型
const args: IArguments = arguments;

// 原理
interface IArguments {
    callee: Function;
    length: number;
    [index: number]: any;
}

// 联合类型
let arr: (number | string)[] = [1, '1'];
```

#### 泛型

```typescript
const nums: number[][] = [[1]]

const nums: Array<Array<number>> = [[1]];
```

#### 元组
元组可以限制数组个数和每个元素的类型。

当不确定类型时，只能访问或调用数组内元素共有的属性或方法

```typescript
//  定义了一个长度为2，第一位是string类型，第二位是number类型
let x: [string, number]
x = ['h', 1]
```

##### 可选元素

```typescript
let arr:[string, boolean?];

arr = ['1'];
```

##### 剩余元素
元组类型里最后一个元素可以是剩余元素，形式为...x

```typescript
let arr: [number, ...string[]];
```

##### 只读
可以为任何元组类型加上readonly关键字前缀，使其成为只读元组

```typescript
const arr: readonly [string, number] = ['1', 666];
```

### 函数
可以在函数参数的第一个参数定义 `this` 的类型，实际运行会被忽略，用于增强编辑器的提示。

#### 可选参数

```typescript
// 函数表达式
const sum = function(name: string, age?: number) {
    // ...
}
```

注意：可选参数后不能出现必需参数。

#### 参数默认值

```typescript
function query(sex: string = '1') {
}
```

#### 剩余参数

```typescript
function push(arr: any[], ...items: any[]) {
}
```

#### 函数重载
```typescript
function find (id: number):number[]
function find (ids: number[] | number):number[] {
    if (Array.isArray(ids)) {
        return [1];
    } else if (typeof ids === 'number') {
        return [1];
    } else {
        return [2];
    }
}
```

## Union Types
Union Types represents values that may be any one of those types. Each of these type is union's member.

Union Types syntax:
```typescript
function log (s: number | string) {
    console.log(s);
}
```

It's easy to provide a value matching the union type - provide a type matching any of the union's member.

But TypeScript only allows an operation if it's valid for every member of the union. If you want to operate one of the union's member,
narrow the union. The same as we would do in JavaScript.

## Type Aliases
Type alias can name any type such as union type, object type.

the syntax for a type alias:
```typescript
type Point = {
  x: number;
  y: number;
};
```

## Interfaces
An interface declaration is another way to name an object type.

Interface works as type alias does. TypeScript only concerned with the structure of the value.
```typescript
interface Point {
  x: number;
  y: number;
  // optional
  z?: number;
  // 索引签名
  [propName: string]: any;
  readonly id: number
}
```

### 索引签名
索引签名可以省去定义不关注的属性，关注指明的属性类型定义。

需要注意的是：**一旦定义了索引签名`，那么确定属性和可选属性的类型都必须是它的类型的子集**：

例如：
```typescript
interface Person {
    name: string;
    age?: number;
    [propName: string]: string;
}

//  报错
let tom: Person = {
    name: 'Tom',
    age: 25,
    gender: 'male'
};
```

上例中，任意属性的值允许是 `string`，但是可选属性 `age` 的值却是 `number`，`number` 不是 `string` 的子属性，所以报错了。

### extends
继承其他 `interface` 的类型。

```typescript
interface A extends B {
    a: number
}
interface B {
    b: string
}
```

上面的例子会校验 `a` 和 `b` 两个属性。

### 定义函数类型

```typescript
interface Fn {
    (name: string): number[]
}

const fn:Fn = function (name: string) {
    return [1];
}
```

## Differences Between Type Aliases and Interfaces
The key distinction is that a type cannot be re-opened to add new properties vs an interface which is always extendable:
```typescript
// extending an interface
interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}

// extending a type via intersections
type Animal = {
    name: string
}

type Bear = Animal & {
    honey: boolean
}
```

A type cannot be changed after being created while Adding new fields to an existing interface is allowed.
```typescript
interface Window {
  title: string
}

interface Window {
  ts: TypeScriptAPI
}
// it works

type Window = {
    title: string
}

type Window = {
    ts: TypeScriptAPI
}
// Error: Duplicate identifier 'Window'.
```

## 枚举类型
```typescript
enum Color {
  Red,
  Blue,
  Greed
}

let c: Color = Color.Red
```

也能通过值反查键名

```typescript
enum Color {
  Red = 1,
  Green = 2,
  Blue = 4,
}

let r: string = Color[2]
```

## typescript inference
TS 会根据上下文环境自动地推断出变量、函数的返回值等的类型，无需我们再写明类型注解。这是 TS 的类型推断能力。

如果定义的时候没有赋值，不管之后有没有赋值，都会被推断为any类型。

## Literal Inference
TypeScript can infer the type of the property of Object.

For example:
```typescript
const obj = {method: 'GET'};

// And its type is: 
const obj = {
    method: string
}

// and we have an request method here
function request(url: string, method: 'GET' | 'POST'): void
```

when we call request and pass the obj.method as parameter. It has an error:

`Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.`

There are two ways to work around this.

You can change the inference by adding a type assertion in either location:
```typescript
// Change 1:
const req = { url: "https://example.com", method: "GET" as "GET" };
// Change 2
handleRequest(req.url, req.method as "GET");
```

You can use as const to convert the entire object to be type literals:

```typescript
const req = { url: "https://example.com", method: "GET" } as const;
```
The as const suffix acts like const but for the type system

## Type Assertions
Type Assertions allows us to override typescript inference.

```typescript
const arr: number[] = [1, 2, 3];
const res: number = arr.find(num => num > 2); // Type 'undefined' is not assignable to type 'number'
```

We use `as` or the angle-bracket syntax (except if the code is in a .tsx file), so better use `as`.

```typescript
const res: number = arr.find(num => num > 2) as number;
```

TypeScript only allows type assertions which convert to a more specific or less specific version of a type. This rule 
prevents from converting type to no intersection type unless you convert to `unknown` first.

Sometimes this rule can be too conservative and will disallow more complex coercions that might be valid. If this happens,
you can use two assertions, first to `any` , then to the desired type:

`const a = (expr as any) as T;`

```typescript
const foo = {};
foo.bar = 123; // Error: 'bar' 属性不存在于 ‘{}’
foo.bas = 'hello'; // Error: 'bas' 属性不存在于 '{}'
```

这里的代码发出了错误警告，因为 `foo` 的类型推断为 `{}`，即是具有零属性的对象。因此，你不能在它的属性上添加 `bar` 或 `bas`，你可以通过类型断言
来避免此问题：

```typescript
interface Foo {
  bar: number;
  bas: string;
}

const foo = {} as Foo;
foo.bar = 123;
foo.bas = 'hello';
```

## Non-null Assertion Operator (Postfix`!`)
TypeScript has a special syntax for removing null and undefined from a type without doing any explicit checking.
Writing `!` after any expression is effectively a type assertion that the value isn’t null or undefined:

```typescript
function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
```

## 确定赋值断言
通过let x!: number确定赋值断言，TS 编译器就会知道该属性会被明确地赋值。

```typescript
let x: number;
init();

console.log(x + 1); // Variable 'x' is used before being assigned

function init() {
    x = 1;
}

// 确定赋值断言
let x!:number;
```

## Literal Types
Literal Types allow us to refer to specific strings, numbers or boolean in type positions.

Literal Type is subType of string type, so type string is not assignable to Literal Type, while Literal Type is assignable 
to string type.

It's usually used in union type to describe explicit members.

Boolean literal types. true and false is actually just an alias for the union true | false.

For example:
```typescript
const str = 'hello world'; // string type

let str: 'hello world' = 'hello world' // 'hello world' type

const str: 'hello world' // same as above code
```

```typescript
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
function compare(a: string, b: string): -1 | 0 | 1 {
    return a === b ? 0 : a > b ? 1 : -1;
}
```

### let,const
TS 的字面量子类型自动转换成对应的原始值类型的这种设计称之为字面量类型的拓宽

```typescript
// 用const定义不可变的常量，在没有添加类型注解的情况下，TS 推断出常量的类型为赋值字面量的类型。
const str = '我还以为你从来都不会选我呢'; // str: '我还以为你从来都不会选我呢'
const num = 1; // num: 1
const bool = true; // bool: true

// 使用let定义的变量显式地添加类型注解，但是变量的类型自动地转换成了赋值字面量类型的原始值类型。
let str = '我还以为你从来都不会选我呢'; // str: string
let num = 1; // num: number
let bool = true; // bool: boolean
```

## 类型拓宽
所有通过let和var定义的变量、函数的形参、对象的非只读属性，如果满足指定了初始值且未显式添加类型注解的条件，那么它们推断出来的类型就是指定的初始值
字面量类型拓宽后的类型，这就是字面量类型拓宽。

```typescript
let fn = (x = '奉均衡之命!') => x; // fn: (x?: string) => string

const a = '明智之选'; // a: '明智之选'
let b = a; // b: string
let func = (c = a) => c; // func: (c?: string) => string 
```

对null和undefined的类型也会进行拓宽，通过let var定义的变量如果满足未显式添加类型注解且被赋予了null或undefined值，则推断出这些变量的类型为any.

```typescript
let x = null; // x: any
let y = undefined; // y: any

const a = null; // a: null;
const b = undefined; // b: undefined
```

### 例子

```typescript
type ObjType = {
    a: number;
    b: number;
    c: number;
}

type KeyType = 'a' | 'b' | 'c';

function fn(object: ObjType, key: KeyType) {
    return object[key];
}

let object = {a: 123, b: 456, c: 789};
let key = 'a';
fn(object, key); // Argument of type 'string' is not assignable to parameter of type '"a" | "b" | "c"'
```

这是因为变量key的类型被推断成了string类型（类型拓宽），但是函数期望它的第二个参数是一个更具体的类型，所以报错。

TS 提供了一些控制拓宽过程的方法，其中一种是使用const，如果用const声明一个变量，那么它的类型会更窄。

所以在本例中将 `let key = 'a'` 改成 `const` 声明即可解决。

## 资料
https://www.typescriptlang.org
