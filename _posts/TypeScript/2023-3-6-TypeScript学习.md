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

```typescript
interface Point {
  x: number;
  y: number;
}
```

Interface works as type alias does. TypeScript only concerned with the structure of the value.

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

## Type Assertions
We use `as` or the angle-bracket syntax (except if the code is in a .tsx file).

TypeScript only allows type assertions which convert to a more specific or less specific version of a type. This rule prevents from
converting type to no intersection type unless you convert to `unknown` first.

Sometimes this rule can be too conservative and will disallow more complex coercions that might be valid. If this happens,
you can use two assertions, first to `any` , then to the desired type:

`const a = (expr as any) as T;`

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

## 资料
https://www.typescriptlang.org