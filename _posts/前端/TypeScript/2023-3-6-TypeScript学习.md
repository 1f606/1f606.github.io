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

#### truthiness narrowing
0, NaN, 0n, '', null and undefined all coerce to false, and other values get coerced to true.

You can always coerce values to booleans by running them through the Boolean function, or by using the shorter double-Boolean negation. (The latter has the advantage that TypeScript infers a narrow literal boolean type true, while inferring the first as type boolean.)

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

TypeScript only allows an operation if it's valid for every member of the union. If you want to operate one of the 
union's member, narrow the union. The same as we would do in JavaScript.

## Type Aliases
Type alias can name any type. Just make an alias, not a new type.

the syntax for a type alias:
```typescript
type Point = {
  x: number;
  y: number;
};
```

## 交叉类型
交叉类型是将多个类型合并为一个类型，这让我们可以把现有的多种类型叠加到一起成为一种类型。使用&定义交叉类型。

常用于将多个接口类型合并成一个类型，从而实现类似于继承的效果。

```typescript
interface Type1 {
    name: string;
    sex: string;
}

interface Type2 {
    age: number;
}

type NewType = Type1 & Type2;
const person: NewType = {
    name: '金克丝',
    sex: '女',
    age: 19,
    address: '诺克萨斯', // error
}
```

```typescript
// 该例子无意义。因为没有任何类型可以既是string类型又是number类型，两者不能同时满足，Value的类型是never。
type Value = string & number;
```

### 同名属性的处理
1. 如果合并的多个接口类型中存在同名属性，且同名属性的类型不兼容，例如的 `{name: string}&{name: number}`，那么合并后的类型就是
string & number，即never。
2. 如果同名属性的类型兼容，例如一个是number，另一个是number的子类型(数字字面量类型)，合并后 name 属性的类型就是数字字面量类型（两者中的子类型）。
3. 如果同名属性是非基本数据类型。会合并其中类型。

## Interfaces
An interface declaration is another way to define type of object.

Interface works as type alias does. TypeScript only concerned with the structure of the value.
```typescript
interface Point {
  x: number;
  y: number;
  // optional
  z?: number;
  // 索引签名
  [propName: string]: any;
  // 只能创建时初始化值
  readonly id: number
}
```

### 索引签名
索引签名可以省去定义不关注的属性，关注指明的属性类型定义。允许任意名字和类型属性，一个接口中只能有一个。

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

如果接口中有多个类型的属性，可以在任意属性中使用联合类型。

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

interface的 extends 从句是可以跟着多个组合对象，多个组合对象之间用逗号,隔开。类也能作为组合对象。

在 TypeScript 中，类既是值也是类型。 interface extends class 是组合了 class 的类型，忽略了实现代码。

```typescript
class SomeClass {
    private name!: string
    updateName(name: string) {
        this.name = name || '';
    }
}

// 上面的 class 从类型的角度看等同于
interface SomeClass {
    name: string
    updateName: (name:string)=> void
}

interface Parent extends SomeClass {
    value: string
}
```

### implements
implements clause is used to check that a class satisfies a particular interface。Classes may also implement single 
or multiple interfaces at once.

类实现接口后，TypeScript 会校验其类型。但类的类型没有被改变（目前这么解释）。

```typescript
interface Checkable {
    check(name: string): boolean;
}
class NameChecker implements Checkable {
    check(s) {
        // 报错 Parameter 's' implicitly has an 'any' type.
        return s.toLowerCase() === "ok";
    }
}
```

如果是用 interface extends 来实现上面的代码，不会提示上面的报错。说明 extends 也继承了对应 interface 内部函数或属性的类型。

另外一个情况：类 implements 有可选属性的接口，类并不能给可选属性赋值。会提示这个属性在类上并不存在。`extends` 可以。

// TODO
https://juejin.cn/post/6844904034621456398?searchId=2023081711080054732E726A95A8E2D3E1
https://juejin.cn/post/7199447328306364473?searchId=2023081711080054732E726A95A8E2D3E1

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
`type` use the `&` to extend while `interface` use `extends` to extend new Properties. Both of them can extend the other.

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

```typescript
type Obj1 = {
    x: string;
}

interface Obj2 extends Obj1 {
    y: number;
}

const obj: Obj2 = {
    x: '我的一个跟斗，能翻十万八千里',
    y: 222,
}

interface Obj1 {
    x: string;
}

type Obj2 = Obj1 & {
    y: number;
}

const obj: Obj2 = {
    x: '只要点一下就够了，蠢货！',
    y: 333,
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

`type` just a new name for the type that `type` is using.

we can use `interface` to define same name interface multiple times. These interface will be merged. While `type` can't.

```typescript
interface Obj {
    x: string;
}

interface Obj {
    y: number;
}

const obj: Obj = {x: '奉均衡之命！', y: 666}; // 自动合并为单个接口
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

## []
[] 传入的是类型，例如：
```typescript
type Person = {
  age: number;
  name: string;
  alive: boolean;
};

type Age = Person['age'];
// or
type key = 'age';
type Age = Person[key];
```

TODO 不理解
第一个示例中的最终取得的类型是number，因为含有string类型的索引签名对应的属性类型就是number。

第二个示例中会得到元组中所有元素的类型组成的联合类型，因为其实元组的索引都是number类型的，所以可以一次全部取到所有元素的类型。

示例1：
interface Test {
    [p: string]: number
}
// number
type stringTypes = Test[string]

示例2：
type tuple = ['1', 1, true]
// true | "1" | 1
type allTypes = tuple[number]

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

例如一个根据传入对象类型返回其 key 的方法，如果调用时传入的对象类型是省略过部分字段的 `Partial<Omit<Admin, 'type'>>` ，需要用 `as`。
```typescript
const getObjectKeys = <T>(obj: T) => Object.keys(obj) as (keyof T)[];
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

Boolean literal types. true and false is actually just an alias for the union `true | false`.

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

## let,const
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

### 类型拓宽
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

#### 例子

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

#### const 断言
另一个例子：

```typescript
const obj = {
    x: 250,
}

obj.x = 520; // ok
obj.x = '520'; // Type 'string' is not assignable to type 'number'
obj.y = 1314; // Property 'y' does not exist on type '{ x: number; }'
```

对于对象而言，TS 的拓宽算法会将其内部属性视为赋值给let关键字声明的变量，进而来推断其属性的类型。因此，obj的类型为{x: number}。obj.x的值可以
是任何number类型的值，但不允许是string类型的，同时也不允许给obj对象添加其它的属性。

要解决上面的问题，我们可以使用const断言。当在某个值后面使用了const断言时，TS 会为这个值推断出最窄的类型，没有拓宽。
对于真正的常量，这通常是你想要的。

```typescript
// TS: {x: number; y: number}
const obj1 = {
    x: 1,
    y: 2,
}

// TS: {x: 1; y: number}
const obj2 = {
    x: 1 as const,
    y: 2,
}

// TS: {readonly x: 1; readonly y: 2}
const obj3 = {
    x: 1,
    y: 2,
} as const;

const arr1 = [1, 2, 3]; // TS: number[]
const arr2 = [1, 2, 3] as const; // TS: readonly [1, 2, 3]
```

### 类型缩小
通过一些操作将变量的类型由一个较为宽泛的集合缩小为相对较小、较明确的集合，这就是类型缩小。

```typescript
let fn = (a: any) => {
    if (typeof a === 'string') {
        return a;
    } else if (typeof a === 'number') {
        return a;
    }
    return null;
}
```

## 绕开额外属性检查的方法

### duckType
具有鸭子特征的就认为它是鸭子。所谓的鸭式辨型法，就是通过制定规则来判定对象是否实现这个接口。例如：

```typescript
interface Person {
    name: string;
}

function getPersonInfo(personObj: Person) {
    console.log(personObj.name);
}

getPersonInfo({name: '德莱文', age: 27}); // error
```

```typescript
interface Person {
    name: string;
}

function getPersonInfo(personObj: Person) {
    console.log(personObj.name);
}

const psObj = {name: '德莱文', age: 27};

getPersonInfo(psObj); // ok
```

上面的栗子中，在参数里写对象就相当于直接给personObj赋值，这个对象有严格的类型定义，所以不能多参或者少参。

而当我们在外面将该对象用另一个变量psObj接收，psObj不会经过额外属性检查，但是会根据类型推论为const psObj: {name: string, age: number} =
{name: '德莱文', age: 27}。然后将psObj再赋值给personObj，此时根据类型的兼容性，参照「鸭式辨型法」，两个类型因为都具有name属性，所以被认定
为相同，故而可以用此方法来绕开多余的类型检查。

### 类型断言
类型断言的意义就等同于你在告诉程序，你很清楚自己在做什么，此时程序就不会再进行额外的属性检查了：

```typescript
interface Person {
    name: string;
    age?: number;
}

const pete: Person = {
    name: 'Pete',
    age: 25,
    sex: '男',
} as Person; // ok
```

### 索引签名

```typescript
interface Person {
    name: string;
    age?: number;
    sex: string;
    [propName: string]: any;
}

const trump: Person = {
    name: 'Trump',
    sex: '男',
    // ok
    address: 'Mars',
    phoneNumber: 123456,
}
```

## 声明
`declare` is used to describe the API the library that is not written in Typescript expose.

First, create a file ends with `.d.ts`. Then use the `module` keyword and quoted name of this module which will be use 
in later import.

a example of the API that nodejs expose.

```typesciprt
// node.d.ts (simplified excerpt)
declare module "url" {
    export interface Url {
        protocol?: string;
        hostname?: string;
        pathname?: string;
    }
    export function parse (
        urlStr: string,
        parseQueryString?,
        slashesDenoteHOst?
    ): Url;
}
```

Now we can `/// <reference> node.d.ts` and then load the modules using `import url = require("url");` or `import * as URL from "url"`

```typescript
/// <reference path="node.d.ts"/>
import * as URL from "url";
let myUrl = URL.parse("https://www.typescriptlang.org");
```

### declare 声明不同变量的方式

```typescript
declare namespace aObject {
  function hi(str: string): void;
  keyA: number
}

declare function methodOne(v: string): string;
declare function methodTwo(v: string[]): string;

declare class Admin {
  constructor(v: string);
  member: string;
  test(): void;
}

declare var variableName: number;


```

### .d.ts
`.d.ts` 用于声明类型，声明后无须手动引入就能使用类型。在 `tsconfig.json` 的 `include` 数组中添加。

.d.ts 文件中的顶级声明必须以 `declare` 或 `export` 修饰符开头，两者共用会导致其他 ts 文件使用时需要手动引入。
其余顶级声明可以不用写 `declare`，也能在其他地方自动引入使用。

`include` 支持 glob 通配符，如：`**/*.ts` 递归匹配子目录 ts 结尾文件。

### Ambient Declarations
在.d.ts后缀的文件中编写声明代码，起到文档的作用。

如果源码更改，声明文件也需要更改，否则会编译报错。

在声明文件顶层，要用 declare 关键字声明。

```typescript
// A 和 B 效果相同，但更推荐B
// Sample A
declare var myPoint: { x: number; y: number; };

// Sample B
interface Point {
    x: number; y: number;
}

declare var myPoint: Point;
```

```typescript
// Lib a.d.ts
interface Point {
x: number; y: number;
}
declare var myPoint: Point;

// Lib b.d.ts
interface Point {
z: number;
}
// todo why?
var myPoint.z; // Allowed!
```

### global.d.ts

```typescript
declare module "*.css";
declare module "*.html";
// 这样声明后就能导入这两种文件
```

## 泛型
泛型能够根据类型自动进行推导和检查。例如一个接收什么类型就返回什么类型的函数：

```typescript
function fn<T>(arg: T): T {
    return arg;
}

fn('哈哈哈');
```

我们定义了一个类型<T>，这个T是一个抽象类型，只有在调用的时候才能确定它的值。当我们传入'哈哈哈'时，T会自动识别传入参数的类型，进而转换为string，
然后再链式传递给参数类型和返回值类型，这样一来就不用将类型写死了。

T 代表 Type，在定义泛型时通常用作第一个类型变量名称。是约定俗成的写法。还有常见的泛型变量名：
* K（Key）：表示对象中的键类型
* V（Value）：表示对象中的值类型
* E（Element）：表示元素类型 泛型变量也可以定义多个：

```typescript
function fn<T, U>(message: T, value: U): U {
    console.log(message);
    return value;
}

console.log(fn<string, number>('我喜欢你', 520));

// 上面的调用形式是为泛型变量显式设定值，更常见的做法是让编译器自动推导这些类型。我们可以省略尖括号，使代码更加简洁
console.log(fn('我喜欢你', 520));
```

### 泛型约束

```typescript
function fn<T>(arg: T): T {
    console.log(arg.size); // Property 'size' does not exist on type 'T'
    return arg;
}
```

我们想打印出参数的 size 属性，但是 TS 报错了。原因在于 T 理论上可以是任何类型，跟 any 相反，无论使用它的什么属性或方法都会报错（除非这个属性和
方法是所有集合共有的）。

想要解决这个问题，我们需要对类型进行约束，限定传给函数的参数类型应该要有 size 类型。

使用extends关键字可以做到这一点，简单说就是我们先定义一个类型，然后通过extends关键字让 T 实现它即可：

```typescript
interface ArgType {
    size: number;
}

function fn<T extends ArgType>(arg: T): T {
    console.log(arg.size);
    return arg;
}
```

// TODO 直接将函数的参数限定为 ArgType 类型，会有类型丢失的风险？

## generic interfaces
In this section, we’ll explore the type of a identity function and how to create generic interfaces.

```typescript
// similarly to function declaration
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: <Type>(arg: Type) => Type = identity;
```

We can also write the generic type as a call signature of an object literal type:

```typescript
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: { <Type>(arg: Type): Type } = identity;
```

let's move the previous example to an interface:

```typescript
interface GenericIdentityFn {
  <Type>(arg: Type): Type;
}
 
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: GenericIdentityFn = identity;
```

we can make the generic parameter to be the parameter of the whole interface. This makes the type parameter visible to all the other members of the interface.

```typescript
interface GenericIdentityFn<Type> {
  (arg: Type): Type;
}
 
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: GenericIdentityFn<number> = identity;
```

### 泛型工具类型
为了方便开发者，TS 内置了一些常见的工具类型，例如：Partial、Required、Readonly、Record 等等。在具体学习工具类型之前，我们先得了解一些基础知识。

#### typeof
typeof的主要用途是在类型上下文中获取变量、属性、对象或函数的类型：

```typescript
interface Person {
    name: string;
    age: number;
}

const lzl: Person = {
    name: '林志玲',
    age: 18,
}

// 后续可以使用 LzlType 类型
type LzlType = typeof lzl;

function fn(x: string): string[] {
    return [x];
}

type FnType = typeof fn; // (x: string) => string[]
```

#### keyof
keyof 操作符可以用来获取某种类型的所有键，其返回类型是联合类型：

```typescript
interface Person {
    name: string;
    age: number;
}

type P = keyof Person; // 'name' | 'age'
```

由于 JS 是动态类型语言，有时在静态类型系统中捕获某些操作的语义可能会比较麻烦，例如：

```typescript
function fn(obj, key) {
    return obj[key];
}
```

该函数接收 obj 和 key 两个参数，并返回对应属性的值。对象上的不同属性，可以具有完全不同的类型，我们甚至都不知道 obj 对象长什么样子。

那么该如何定义 fn 函数的类型呢？我们来尝试一下：

```typescript
function fn(obj: object, key: string) {
    return obj[key];
}
// Element implicitly has an 'any' type because expression of type 'string' can't be used to index type '{}'
```

元素隐式地拥有any类型，因为string类型不能被用于索引类型{}。解决这个问题最暴力的方式就是使用 any大法：

```typescript
function fn(obj: object, key: string) {
    return (obj as any)[key];
}
```

但很明显这并不是一个好方案。我们来回顾一下 fn 函数的作用，该函数用于获取某个对象中指定属性的值，因此我们期望传入的属性是对象中已经存在的属性。那
么如何限制属性名的范围内？这时候就需要keyof。

```typescript
function fn<T extends object, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
}
```

在上例中，我们使用了泛型和泛型约束，还有keyof操作符。首先定义类型 T，并使用extends关键字约束T类型必须是object类型的子类型，然后使用keyof操
作符获取T类型的所有键，其返回值是联合类型，最后利用extends 关键字约束K类型必须是keyof T联合类型的子类型。这样定义的话就能够正确推导出指定键
对应的类型了。

此时这个函数调用时访问不存在的属性也会报错。

#### in
in 用来获取联合类型，主要用于数组或对象的构建。不要用在 interface，会报错，用在 type 上。

```typescript
type Keys = 'x' | 'y' | 'z';

type Obj = {
    [k in Keys]: string;
} 
//
type Obj = {
    x: string;
    y: string;
    z: string;
}
```

#### extends
有时我们不想定义的泛型过于灵活，可以通过extends关键字添加泛型约束：

```typescript
interface ArgType {
    id: number;
}

function fn<T extends ArgType>(arg: T): T {
    console.log(arg.id);
    return arg;
}
```

#### 内置的工具类型
##### Partial
将类型的属性变成可选。

定义：
```typescript
type Partial<T> = {
    [P in keyof T]?: T[P];
}
```

先通过keyof T拿到T的所有属性名，然后使用in进行遍历，将值赋给P，再通过T[P]获取相应属性值的类型。?用于将所有属性变成可选。

举个例子：

```typescript
interface Person {
    name: string;
    age: number;
}

type NewPerson = Partial<Person>;
const zhl: NewPerson = {
    name: '钟汉良',
}

// 等同于
interface NewPerson {
    name?: string;
    age?: number;
}
```

注意：Partial<T>只支持处理第一层的属性：

```typescript
interface Person {
    name: string;
    age: number;
    address: {
        province: string;
        city: string;
    };
}

type NewPerson = Partial<Person>;
const wyz: NewPerson = {
    name: '吴彦祖',
    address: { // Property 'city' is missing in type '{ province: string; }' but required in type '{ province: string; city: string; }'
        province: '香港省',
    },
}
```

##### DeepPartial

```typescript
interface Person {
    name: string;
    age: number;
    address: {
        province: string;
        city: string;
    };
}

type DeepPartial<T> = {
    [K in keyof T]?: T[K] extends object
        ? DeepPartial<T[K]>
        : T[K];
}

type NewPerson = DeepPartial<Person>;
const wyz: NewPerson = {
    name: '吴彦祖',
    address: { // ok
        province: '香港省',
    },
}
```

##### Required
将类型的属性变成必选。

定义：
```typescript
type Required<T> = {
    [K in keyof T]-?: T[K];
}
```

`-?`代表移除可选特性。

```typescript
interface Person {
    name?: string;
    age?: string;
}

type NewPerson = Required<Person>;
const zjl: NewPerson = { // Property 'age' is missing in type '{ name: string; }' but required in type 'Required<Person>'
    name: '周杰伦',
}
```

##### Readonly
将类型的属性变成只读。

定义：
```typescript
type Readonly<T> = {
    readonly [K in keyof T]: T[K];
}
```

```typescript
interface Person {
    name: string;
    age: number;
}

type NewPerson = Readonly<Person>;
const hg: NewPerson = {
    name: '胡歌',
    age: 18,
}

hg.age = 40; // Cannot assign to 'age' because it is a read-only property
```

##### Record
`Record<K extends keyof any, T>`将 K 中所有属性的值转化为 T 类型。

定义：
```typescript
type Record<K extends keyof any, T> = {
    [P in K]: T;
}
```

```typescript
interface PersonInfo {
    name: string;
}

type Person = 'zxy' | 'ldh' | 'zgr';

const ny: Record<Person, PersonInfo> = {
    zxy: {name: '张学友'},
    ldh: {name: '刘德华'},
    zgr: {name: '张国荣'},
}
```

##### ReturnType
用来获取一个函数的返回值类型。

定义：
```typescript
type ReturnType<T extends (...args: any[]) => any> = T extends (
  ...args: any[]
) => infer R
  ? R
  : any;
```

infer用于提取函数返回值的类型。

```typescript
type Fn = (v: string) => number;

let x: ReturnType<Fn> = 888;
x = '888'; // Type 'string' is not assignable to type 'number'
```

ReturnType提取到Fn的返回值类型为number，所以变量x只能被赋予number类型的值。

##### Pick
从对象结构的类型中挑出一些指定的属性，来构造一个新类型。

定义：
```typescript
type Pick<T, U extends keyof T> = {
    [P in U]: T[P];
}
```

```typescript
interface Person {
    name: string;
    age: number;
    sex: string;
}

type NewPerson = Pick<Person, 'name' | 'sex'>;
const ldh: NewPerson = {
    name: '刘德华',
    sex: '男',
}

// 等同于
type NewPerson = {
    name: string;
    sex: string;
}
```

##### Omit
从对象结构的类型中排除掉指定的属性，从而构造一个新类型。

定义：
```typescript
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;

```

```typescript
interface Person {
    name: string;
    age: number;
    sex: string;
}

type NewPerson = Omit<Person, 'sex'>;
const ldh: NewPerson = {
    name: '刘德华',
    age: 18,
} 
// 等同于
type NewPerson = {
    name: string;
    age: number;
}
```

##### Extract
Extract<T, U>，从 T 中提取出 U。

```typescript
type Extract<T, U> = T extends U ? T : never;
```

```typescript
type A = Extract<'x' | 'y' | 'z', 'y'>; // 'y'
type B = Extract<string | number | (() => void), Function>; // () => void
```

##### Exclude
Exclude<T, U>，从 T 中移除 U。

```typescript
type Exclude<T, U> = T extends U ? never : T;
```

```typescript
type A = Exclude<'x' | 'y' | 'z', 'y'>; // 'x' | 'z'
type B = Exclude<string | number | (() => void), Function>; // string | number
```

##### NonNullable
过滤掉类型中的 null 和 undefined 类型。

定义：
```typescript
type NonNullable<T> = T extends null | undefined ? never : T;
```

```typescript
type A = NonNullable<string | null | undefined>; // string
```

## 优化点
### 减少重复代码

```typescript
interface Person {
    name: string;
    age: number;
}

interface NewPerson {
    name: string;
    age: number;
    sex: string;
}

// 可以改成
interface NewPerson extends Person {
    sex: string;
}
// 或
type NewPerson = Person & {sex: string}
```

有时候想定义一个类型来匹配一个初始配置对象的“形状”：

```typescript
const jsy = {
    name: '江疏影',
    age: 18,
    sex: '女',
}

interface Person {
    name: string;
    age: number;
    sex: string;
}

// 可以改成
type Person = typeof jsy;
```

还有重复的类型。例如多个函数拥有相同的类型签名：

```typescript
function getList(current: number, pageSize: number): Promise<Response>
function getDetailList(current: number, pageSize: number): Promise<Response>

// 改成
type QueryList = (current: number, pageSize: number) => Promise<Response>;
const getList: QueryList = (current, pageSize) => {};
const getDetailList: QueryList = (current, pageSize) => {};
```

### 精确定义类型
例如：

```typescript
interface Person {
    name: string;
    age: number;
    sex: string;
    birthDate: string;
    income: string;
}
```

假如我们希望日期是 YYYY-MM-DD，收入范围是low, middle 和 high。

```typescript
interface Person {
    name: string;
    age: number;
    sex: string;
    birthDate: Date;
    income: 'low' | 'middle' | 'high';
}

const xdd: Person = {
    name: '徐冬冬',
    age: 32,
    sex: '女',
    birthDate: new Date(1990-02-16), // ok
    income: 'middle', // ok
}
```

## 资料
https://www.typescriptlang.org
