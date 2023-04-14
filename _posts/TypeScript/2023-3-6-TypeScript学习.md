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

## 基础

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

### 环境

```shell
npm i ts-node -g
npm i @types/node -D

# 使用
ts-node fileName
```

## 基础类型
`Object`，`object`， `number`， `boolean`， `undefined`， `null`， `string`， `any`， `number[]`， `Array<number>`， `<string， number>`， `enum`，
`void` 和 `null`。

变量定义类型后如果被赋值为其他类型、调用其他类型方法就会报错，`any` 不受影响。

`Object` 和 `{}` 在 typescript 中代表任何类型，可以赋值任何类型的值。

`object` 代表引用类型，可以赋值引用类型的值。

### undefined, null and void
`void` 用于没有任何返回值的函数。

`undefined` 和 `null` 是所有类型的子类型。也就是说 `undefined` 类型的变量，可以赋值给 `number` 类型的变量：

而 `void` 类型的变量不能赋值给 `number` 类型的变量

### unknown, any
`any` 和 `unknown` 可以接收任何类型的值。

可以对 `any` 做任何操作和赋值，typescript 不校验。

`unknown` 只能赋值给自身或 `any`，无法读取其任何属性。

### 数组的类型

#### 类型+方括号

```typescript
// 数组的项中不允许出现其他的类型
let fibonacci: number[] = [1, 1, 2, 3, 5];
```

#### 元组
元组和联合类型相似，当不确定类型时，只能访问或调用数组内元素共有的属性或方法

```typescript
//  定义了一个长度为2，第一位是string类型，第二位是number类型
let x: [string, number]
x = ['h', 1]
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

## Type Assertions
Type Assertions allows us to override typescript inference.

We use `as` or the angle-bracket syntax (except if the code is in a .tsx file).

TypeScript only allows type assertions which convert to a more specific or less specific version of a type. This rule prevents from
converting type to no intersection type unless you convert to `unknown` first.

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

## Literal Types
Literal Types allow us to refer to specific strings and numbers in type positions.

For example:
```
const str = 'hello world'; // string type

let str: 'hello world' = 'hello world' // 'hello world' type

const str: 'hello world' // same as above code
```
now, str can not assign other value except 'hello world'.

It's useful to express a type of certain set of known values.

For example:
```typescript
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
function compare(a: string, b: string): -1 | 0 | 1 {
    return a === b ? 0 : a > b ? 1 : -1;
}
```

you can combine these with non-literal types.

Note that there’s one more kind of literal type: boolean literals, with only two boolean literal types. true and false.
It is actually just an alias for the union true | false.

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

## null and undefined
`null` and `undefined` are used to signal absent or uninitialized value.

TypeScript has two corresponding types by the same names. How these types behave depends on whether you have the 
`strictNullChecks` option on.

### strictNullChecks off
With strictNullChecks off, values that might be null or undefined can still be accessed normally, and the values null 
and undefined can be assigned to a property of any type.

### strictNullChecks on
With strictNullChecks on, when a value is null or undefined, you will need to test for those values before using methods
or properties on that value.

## Non-null Assertion Operator (Postfix`!`)
TypeScript also has a special syntax for removing null and undefined from a type without doing any explicit checking. 
Writing `!` after any expression is effectively a type assertion that the value isn’t null or undefined:

```typescript
function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
```


## 资料
https://www.typescriptlang.org
