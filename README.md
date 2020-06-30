<!-- markdownlint-disable MD033 -->

# [《Effective TypeScript》](https://effectivetypescript.com/)读书笔记

<a href="https://effectivetypescript.com/">
  <img src="https://effectivetypescript.com/images/cover.jpg" width="267" height="350"/>
</a>

## 目录

第一章 初识 TypeScript

- [01. 理解 TypeScript 与 JavaScript 之间的关系](#01-理解-TypeScript-与-JavaScript-之间的关系)
- [02. 明确你使用的 TypeScript 选项](#02-明确你使用的-TypeScript-选项)
- [03. 理解代码生成与类型系统是相互独立的](#03-理解代码生成与类型系统是相互独立的)
- [04. 理解 TypeScript 中的结构化类型](#04-理解-TypeScript-中的结构化类型)
- [05. 限制 `any` 类型的使用](#05-限制-any-类型的使用)

第二章 TypeScript 的类型系统

- [06. 利用编辑器探索类型系统](#06-利用编辑器探索类型系统)
- [07. 把类型看做值的集合](#07-把类型看做值的集合)
- [08. 学会判断一个符号是在类型空间还是值空间](#08-学会判断一个符号是在类型空间还是值空间)
- [09. 优先使用类型声明而不是类型断言](#09-优先使用类型声明而不是类型断言)

## 正文

### 01. 理解 TypeScript 与 JavaScript 之间的关系

**在语法层面上**，TypeScript 是 JavaScript 的超集。任何一段 JavaScript 程序，只要它没有语法错误，那么它也是一段合法的 TypeScript 程序(当然 TypeScript 的类型检查器可能会提示你一些类型相关的错误，但那只是一些具体问题，并不影响它”**是不是** TypeScript“ 这一问题的结论)。

TypeScript 使用 `.ts`(或 `.tsx`)作为文件扩展名。这并不意味着 TypeScript 是一种完全不同的语言。由于 TypeScript 是 JavaScript 的超集，因此你所有 `.js` 文件中的代码其实都是 TypeScript 代码。是否将 `main.js` 重命名为 `main.ts` 并不影响这一点。

所有的 JavaScript 程序都是 TypeScript 程序，这一论述的逆向说法并不成立：有的 TypeScript 程序携带了额外的声明类型的语法，而这部分语法是 JavaScript 不支持的。例如下面这段 TypeScript 程序：

```typescript
function greet(who: string) {
  console.log("Hello", who);
}
```

如果你使用 `node` 命令执行，那么你会得到一个 `SyntaxError: Unexpected token :` 的错误。 上述代码中的 `: string` 是一种类型注释。一旦你使用了这种语法，那么你就走出了 JavaScript 的范畴。

<img src="./assets/1.png" width="394" height="243" />

#### TypeScript 可以为纯 JavaScript 代码带来哪些收益

先看下面这段 JavaScript 代码：

```javascript
let city = "new york city";
console.log(city.toUppercase());
```

这代码在运行时会产生一个错误 `TypeError: city.toUppercase is not a function`。虽然代码里没有任何类型注释，但 TypeScript 的类型检查器还是会提示你下面的错误：

```typescript
let city = "new york city";
console.log(city.toUppercase());
//               ~~~~~~~~~~~ Property 'toUppercase' does not exist on type
//               'string'. Did you mean 'toUpperCase'?
```

这得益于 TypeScript 的**类型推断**机制。它可以从 `city` 变量的初始值中推断出该变量的类型。

TypeScript 类型系统的一个目标是在**不执行代码**的前提下检测出代码的运行时异常。这也是 TypeScript 被称为**静态**类型语音的原因。

#### 类型推断并不总是能够按照预期方式工作

假设有代码：

```javascript
const states = [
  { name: "Alabama", capital: "Montgomery" },
  { name: "Alaska", capital: "Juneau" },
  { name: "Arizona", capital: "Phoenix" },
];
for (const state of states) {
  console.log(state.capitol); // 这里有一个拼写错误
}
```

这段代码在运行时会输出：

```shell
undefined
undefined
undefined
```

这显然不是我们期望的结果，但同时这段代码也没有任何语法错误，所以 JavaScript 解释器在面对这类错误时并不能提供给我们更多信息。

如果我们把上述的代码交给 TypeScript 类型检查器，我们会得到一个非常明确的错误信息：

```typescript
for (const state of states) {
  console.log(state.capitol);
  //                ~~~~~~~ Property 'capitol' does not exist on type
  //                '{ name: string; capital: string; }'.
  //                Did you mean 'capital'?
}
```

从错误信息中我们可以明确的看到 TypeScript 推断出来的类型，以及拼写错误的属性名称。

下面让我们把上一段代码稍微修改一下：

```typescript
const states = [
  { name: "Alabama", capitol: "Montgomery" },
  { name: "Alaska", capitol: "Juneau" },
  { name: "Arizona", capitol: "Phoenix" },
];
for (const state of states) {
  console.log(state.capital);
  //                ~~~~~~~ Property 'capital' does not exist on type
  //                '{ name: string; capitol: string; }'.
  //                Did you mean 'capitol'?
}
```

这一次 TypeScript 的类型检查器依然给出了错误提示，但它并没有很好的理解我们的真实意图。我们真正想要拼写的是 `capital` 而非 `capitol`。这时候**类型注释**就派上用场了。

```typescript
interface State {
  name: string;
  capital: string;
}
const states: State[] = [
  { name: "Alabama", capitol: "Montgomery" },
  //                 ~~~~~~~~~~~~~~~~~~~~~
  { name: "Alaska", capitol: "Juneau" },
  //                ~~~~~~~~~~~~~~~~~
  { name: "Arizona", capitol: "Phoenix" },
  //                 ~~~~~~~~~~~~~~~~~~ Object literal may only specify known
  //                         properties, but 'capitol' does not exist in type
  //                         'State'. Did you mean to write 'capital'?
];
for (const state of states) {
  console.log(state.capital);
}
```

通过类型注释，我们明确的告诉 TypeScirpt 类型检查器某个变量的类型应该是什么。

#### 通过了 TypeScript 的类型检查并不意味着不会出现运行时错误

看下面这段代码：

```typescript
const names = ["Alice", "Bob"];
console.log(names[2].toUpperCase());
```

虽然这段代码在 TypeScript 的类型检查器眼里并没有什么不妥，但我们一眼就能看出这段代码在运行时会产一个 `TypeError: Cannot read property 'toUpperCase' of undefined` 的异常。TypeScript 会假定数组访问在数组范围之内，但事实并非始终如此。产生这类异常的根本原因在于 TypeScript 对值类型的理解与现实存在偏差。

### 02. 明确你使用的 TypeScript 选项

TypeScript 提供了大量的配置项供我们设置，其中一部分用于设置待编译文件的路径，另一部分被称为”[编译选项(Compiler Options)](https://www.typescriptlang.org/docs/handbook/compiler-options.html)“的配置项则决定了 TypeScript 语言自身的行为。

举个例子：

```typescript
function add(a, b) {
  return a + b;
}
add(10, null);
```

这段代码是否报错，就取决于配置项 `noImplicitAny` 是打开还是关闭。当该配置项为 `false` 时，上述代码中我们没有明确注明类型的参数 `a` 和 `b` 均会被赋予一个**隐式**的 `any` 类型。因此在 `a + b` 处，TypeScript 类型检查器不会报告类型错误。而当 `noImplicitAny` 为 `true` 时，任何隐式的 `any` 类型都会引发错误。

在项目中你应该始终开启 `noImplicitAny` 配置项，唯一的例外是当你将旧项目从 JavaScript 迁移到 TypeScript 的过程中。

再看另一个很重要的配置项：`strictNullChecks`。它决定了在每种数据类型中 `null` 和 `undefined` 是否是合法的值。当开启该参数后，`null` 和 `undefined` 就不能再赋值给除了他们自身以及 `any` 之外的任何其他数据类型。

```typescript
// strictNullChecks 开启前
const x: number = null; // OK, null is a valid number

// strictNullChecks 开启后
const x: number = null;
// ~ Type 'null' is not assignable to type 'number'
```

我们应该始终开启该选项，当你想表达某个变量可以接受 `null` 或 `undefined` 时，你可以在类型注释中明确指定：

```typescript
const x: number | null = null;
```

总结：

1. 在与他人讨论 TypeScript 问题时，需要首先明确各自使用的配置项是否一致。
2. 团队中应该使用 `tsconfig.json` 文件来确保成员间使用的配置项一致。
3. 新项目应该开启 `noImplicitAny` 和 `strictNullChecks` 两个配置项。

### 03. 理解代码生成与类型系统是相互独立的

从宏观上说，`tsc` 做了两件事：

1. 它会将基于”下一代“语法编写的 TyepScript/JavaScript 代码转换成能在浏览器中运行的低版本的 JavaScript 代码(转义)。
2. 它会检查代码中的类型错误。

与我们日常认知不太相符的是：**这两件事之间是完全独立的。**这也就意味着代码里的类型并不能影响编译出的 JavaScript 代码，也不能影响代码的运行时行为。

#### 类型错误并不影响代码的编译结果

在 TypeScript 中类型错误并不会阻断构建流程，这一点和 C 或 Java 很不一样。TypeScript 中的类型错误更像是这两种语言中的 `warning`。

通常在执行 `tsc` 的过程中如果遇到类型错误，我们会说”代码编译失败了“。实际上这一说法并不严谨，`tsc` 依然会输出编译得到的 JavaScript，更严谨的说法是”我们的程序里有类型错误“。

如果在遇到类型错误时不想输出编译结果，可以在 `tsconfig.json` 中开启 `noEmitOnError` 选项。

#### 不能在运行时进行 TypeScript 类型检查

看这段代码：

```typescript
interface Square {
  width: number;
}
interface Rectangle extends Square {
  height: number;
}
type Shape = Square | Rectangle;
function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    //                 ~~~~~~~~~ 'Rectangle' only refers to a type,
    //                            but is being used as a value here
    return shape.width * shape.height;
    //                         ~~~~~~ Property 'height' does not exist
    //                                on type 'Shape'
  } else {
    return shape.width * shape.width;
  }
}
```

为何会提示错误呢？因为 `instanceof` 操作发生在运行时，而 `Rectangle` 是一个类型，所有类型信息(`interface`，`type` 以及类型注释)在编译阶段会被统一”抹去“。

上面的错误消息里提到 `Rectangle` 是一个类型，但我们把它当做值来使用了。那么在 TypeScript 中哪些操作会创建值，哪些操作会创建类型呢？在[官方文档](https://www.typescriptlang.org/docs/handbook/declaration-merging.html#basic-concepts)中有详细解释，感兴趣可以详细阅读。

要在运行时检查类型，你可以**通过判断特定属性是否存在**：

```typescript
function calculateArea(shape: Shape) {
  if ("height" in shape) {
    shape; // Type is Rectangle
    return shape.width * shape.height;
  } else {
    shape; // Type is Square
    return shape.width * shape.width;
  }
}
```

或是**给数据结构额外增加一个标识数据类型的字段**：

```typescript
interface Square {
  kind: "square";
  width: number;
}
interface Rectangle {
  kind: "rectangle";
  height: number;
  width: number;
}
type Shape = Square | Rectangle;
function calculateArea(shape: Shape) {
  if (shape.kind === "rectangle") {
    shape; // Type is Rectangle
    return shape.width * shape.height;
  } else {
    shape; // Type is Square
    return shape.width * shape.width;
  }
}
```

再或是**通过创建类，来解决**(在 TypeScript 中 `class` 既是值又是类型)：

```typescript
class Square {
  constructor(public width: number) {}
}
class Rectangle extends Square {
  constructor(public width: number, public height: number) {
    super(width);
  }
}
type Shape = Square | Rectangle;
function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    shape; // Type is Rectangle
    return shape.width * shape.height;
  } else {
    shape; // Type is Square
    return shape.width * shape.width; // OK
  }
}
```

#### 类型操作不会影响运行时变量值

假设你有函数：

```typescript
function asNumber(val: number | string): number {
  return val as number;
}
```

这里的 `as number` 操作仅限于类型检查时，它并不会在运行时将变量 `val` 的值转换为数字。这段代码在编译后得到的 JavaScript 代码是：

```javascript
function asNumber(val) {
  return val;
}
```

如果你需要在运行时进行类型转换，那么你必须明确使用 `Number` 函数对入参进行转换，像下面这样：

```typescript
function asNumber(val: number | string): number {
  return typeof val === "string" ? Number(val) : val;
}
```

#### 运行时变量类型可能与代码中声明的类型不一致

这一点比较好理解，代码中声明的类型更像是我们代码库内一种”契约“。它描述的是我们期望的入参、出餐的类型。但当我们的程序在运行时有外部输入时，我们并不能保证外部输入的值总是符合我们的”预期“。你为后端接口定义了完备的数据结构，并不能组织线上运行时后端在一个布尔值字段上返回了字符串 `true` 或 `false`。因此，对于程序中重要的接口或其他外部输入，你可能需要考虑建立针对性的数据校验机制。

#### 你不能基于 TypeScript 类型重载函数

在其他语言中(例如 C++)，你可以为一个函数不同的参数类型定义多种实现。这一特性被称为[”函数重载“](https://en.wikipedia.org/wiki/Function_overloading)。

如果你在 TypeScript 中进行相同的操作，你会得到一个错误：

```typescript
function add(a: number, b: number) {
  //     ~~~ Duplicate function implementation
  return a + b;
}
function add(a: string, b: string) {
  //     ~~~ Duplicate function implementation
  return a + b;
}
```

原因其实很简单，因为 TypeScript 最终会编译成 JavaScript，在 JavaScript 你不能对同一个函数实现两个版本。

TypeScript 实现了另外一种形式的函数重载：**你可以为同一个函数提供多个类型定义**。像下面这样：

```typescript
function add(a: number, b: number): number;
function add(a: string, b: string): string;
function add(a, b) {
  return a + b;
}
const three = add(1, 2); // Type is number
const twelve = add("1", "2"); // Type is string
```

#### TypeScript 类型不会影响运行时性能

这一点比较好理解，类型信息在构建过程中都被移除了，运行时执行的其实是纯 JavaScript 代码，因此不会有性能影响。

### 04. 理解 TypeScript 中的结构化类型

JavaScript 语言本质上是[鸭子类型](https://zh.wikipedia.org/wiki/%E9%B8%AD%E5%AD%90%E7%B1%BB%E5%9E%8B)的：当你向函数传递参数对象时，只要这个对象可以满足函数所需的全部属性，函数并不关心这个对象是如何创建的。换句话说，一个对象有效的语义，不是由继承自特定的类或实现特定的接口，而是由"当前方法和属性的集合"决定。

TypeScript 在类型检查时实现了同样的行为(因为 TypeScript 最终要被编译为 JavaScript)。“鸭子类型”机制和我们理解的常规意义的类型系统之间可能会有一些“冲突”，了解这其中的关系有助于我们更好的理解哪些写法会导致 TypeScript 类型错误以及如何写出更健壮的代码。

假设我们有一个向量类型以及计算向量长度的函数：

```typescript
interface Vector2D {
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}
```

然后我们又定义了一个新的类型命名向量：

```typescript
interface NamedVector {
  name: string;
  x: number;
  y: number;
}
```

是否可以通过 `calculateLength` 函数给 `NamedVector` 类型的数据计算长度呢？答案是“可以”。你可以在[这里](https://www.typescriptlang.org/play/index.html?ssl=8&ssc=2&pln=1&pc=1#code/JYOwLgpgTgZghgYwgAgGoQWA9lATAEWQG8AoZZADwC5kQBXAWwCNoBuM5ATxvubZIC+JEjDohMwLCGQI4AGwR05cSABkIIAOZgAFgAoAbjXSYcBAJTEOUCGDpRpAWRU6AdAGcAjlDCHXFZAAqZAN-ZABqENdOIKjOc3YhElBIWEQUADk4BggAExNsKCtyEGyIGncwKFBNdnJqWkYWKDquHib+JIQpSpCaLJz8jELkAF5iShoAZgAaNuQAFjnSnJoAIgAtCAg15AF2WQUlFQh1LV1DBOQAemvkAHkAaTmbdyUwZGB3ZABWEiA)查看结果。

我们并没有声明 `Vector2D` 与 `NamedVector` 之间的关系，也没有对 `calculateLength` 方法基于两种数据类型进行重载。为何 TypeScript 的类型检查器没有提示我们传入参数的类型与函数声明的参数类型不匹配呢？因为 `NamedVector` 类型的数据结构与 `Vector2D` 的数据结构是兼容的。

但“鸭子类型”并不总是带来便利，在其它场景中可能就会带来意想不到的问题。

假设我们又引入了一个 `Vector3D` 类型：

```typescript
interface Vector3D {
  x: number;
  y: number;
  z: number;
}
```

然后写了一个函数来对这个三维向量做[归一化](https://baike.baidu.com/item/%E5%90%91%E9%87%8F%E5%BD%92%E4%B8%80%E5%8C%96%E6%B3%95/22779174?fr=aladdin)：

```typescript
function normalize(v: Vector3D) {
  const length = calculateLength(v);
  return {
    x: v.x / length,
    y: v.y / length,
    z: v.z / length,
  };
}
```

你会发现这个函数没有类型错误，但它的计算结果也不正确，因为 `calculateLength` 只计算了 `x` 和 `y` 两个坐标。**当你开发了一段时间 TypeScript 后，像这种不涉及类型错误的程序逻辑问题，很容易在开发阶段被我们忽视**。

“鸭子类型”另一个可能会引起问题的场景是，我们通常会认为函数收到数据的类型与我们声明的类型是严格一致的，然而实际上由于 “鸭子类型”的存在，我们的收的实际数据类型可能是我们声明的类型的“超集”。参数的类型是“开放的”而不是“封闭”、“精确”的。

举个例子：

```typescript
function calculateLengthL1(v: Vector3D) {
  let length = 0;
  for (const axis of Object.keys(v)) {
    const coord = v[axis];
    //            ~~~~~~~ Element implicitly has an 'any' type because ...
    //                    'string' can't be used to index type 'Vector3D'
    length += Math.abs(coord);
  }
  return length;
}
```

按照常规想法，`axis` 的值只会取 `x`、`y`、`z`，对应在 `Vector3D` 类型上的值也都是数字，为什么在错误信息中会提示 `v[axis]` 是隐含的 `any` 类型呢？这是因为 TypeScript 考虑到了“鸭子类型”机制，当我们循环传入参数的属性时，我们可能会遇到 `x`、`y`、`z` 之外的属性，而这些属性的值我们不能确定，因此就是隐含的 `any` 类型。

更合理的实现方式是我们只操作我们可以明确类型的属性，像这样：

```typescript
function calculateLengthL1(v: Vector3D) {
  return Math.abs(v.x) + Math.abs(v.y) + Math.abs(v.z);
}
```

需要额外注意的是，以上说明的“鸭子类型”机制，在使用类时也同样适用：

```typescript
class C {
  foo: string;
  constructor(foo: string) {
    this.foo = foo;
  }
}
const c = new C("instance of C");
const d: C = { foo: "object literal" }; // OK!
```

### 05. 限制 `any` 类型的使用

给变量设置 `any` 类型后会有以下影响：

- 相关代码不再受类型检查保护。
- 该变量在 IDE 中不再享受任何语言服务（智能提示、一键重构等）。
- 在重构代码时，容易引入 Bug。
- 从长远来讲，任用 `any` 类型会打击团队成员对代码质量的信心。团队成员在开发时不得不一边修复类型错误，一边在大脑里记住变量的真实类型。

### 06. 利用编辑器探索类型系统

在安装 TypeScript 时，我们实际上安装了两个可执行文件：

- tsc - TypeScript 编译器
- tsserver - TypeScript 语言服务

通常我们直接使用 `tsc` 来编译代码，而很少用到 `tsserver`。实际上编辑器针对 TypeScript 提供的代码补全、类型展示、跳转以及重构等进阶功能都是倚赖 `tsserver` 实现的。

在使用 VS Code 打开一个 TypeScript 项目时，在底部状态栏左侧可以看到 TypeScript 语言服务启动的提示。

![VS Code 的 TypeScript 语言服务启动中](./assets/2.png)

在 VS Code 的右下角可以查看当前启用的 TypeScript 版本：

![查看当前使用的 TypeScript 版本](./assets/3.png)

### 07. 把类型看做值的集合

在运行时，每一个变量都会有一个从 JavaScript 全部值范围中选择的具体值。但当 TypeScript 检查代码中的类型错误时，变量有的只是一个**类型**。我们可以把该类型视为变量**所有可能取值的集合**。这个集合被称为类型的“域”。举个例子，我们可以把 `number` 类型看做是全部数字的集合。`42` 与 `-37.25` 都在这个集合中，但 `'Canada'` 不在。取决于 `strictNullChecks` 选项的值，`null` 和 `undefined` 可能在也可能不在这个集合中。

最小的集合是空集合，也就是不包含任何值。这对应 TypeScript 中的 `never` 类型。任何值都不能赋值给一个类型是 `never` 的变量。

```typescript
const x: never = 12;
//    ~ Type '12' is not assignable to type 'never'
```

接下来的最小集合是那些只包含单个值的集合。这对应 TypeScript 中的字面量类型，也被称为单位类型(unit types)。

```typescript
type A = "A";
type B = "B";
```

要创建一个包含两个或三个值的类型，你可以合并这些字面量类型：

```typescript
typeAB = "A" | "B";
typeAB12 = "A" | "B" | 12;
```

单词 “assignable(可赋值)” 经常出现在各种 TypeScript 错误中。在值的集合的上下文中，它表示 “X 是 Y 的成员”（用于值和类型之间的关系）或 “X 是 Y 的子集”（用于两种类型之间的关系）:

```typescript
const a: AB = "A"; // ✅ 值 'A' 是集合 {'A', 'B'} 的一个成员
const c: AB = "C";
//    ~ Type '"C"' is not assignable to type 'AB'
```

要声明一个类型，我们要么通过明确列出所有的可能值：

```typescript
type Int = 1 | 2 | 3 | 4 | 5;
```

要么通过描述这些值的成员属性：

```typescript
interface Identified {
  id: string;
}
```

可以把接口想象成是对它值集合成员的描述。如果一个对象有 `id` 属性且值是 `string` 类型，那么它就是 `Identified` 所表示的集合中的一员。

把类型想象成值的集合有助于我们理解对类型的操作。`&` 操作符表示两个类型的交集。

```typescript
interface Person {
  name: string;
}
interface Lifespan {
  birth: Date;
  death?: Date;
}
type PersonSpan = Person & Lifespan;
```

想想看 `PersonSpan` 类型对应的值的集合中都包含哪些值？它包含的是既在 `Person` 类型的值集合中又在 `Lifespan` 类型的值集合中的值。进一步的，我们可以确定是那些同时拥有 `name`，`birth`，`death` 属性的对象。

再看看两个类型的并集：

```typescript
type K = keyof (Person | Lifespan);
```

类型 `K` 表示的集合中都有哪些值呢？首先 `Person | Lifespan` 表示 `Person` 与 `Lifespan` 的并集，这里包含了双方集合的全部值。接着 `keyof` 运算符表示我们想要取得这个并集中**全部对象所共有**的属性。显然根据两个类型的定义，他们之间没有公共属性。因此 `K` 表示的类型就是 `never`。

以上讨论的逻辑可以表示为：

```typescript
keyof (A&B) = (keyof A) | (keyof B)
keyof (A|B) = (keyof A) & (keyof B)
```

另一种声明 `PersonSpan` 类型的方式是利用 `extends` 操作符：

```typescript
interface Person {
  name: string;
}
interface PersonSpan extends Person {
  birth: Date;
  death?: Date;
}
```

这里 `extends` 表达的语义可以从两个视角来解释：

- 从继承角度，`PersonSpan` 从 `Person` 继承了 `name` 属性并新增了 `birth` 属性。
- 从集合角度，我们可以说 `PersonSpan` 是 `Person` 表达的集合中所有拥有 `birth` 属性的成员形成的一个**子集**。

我们可以通过两种不同的关系图来看待这两个视角：

假设有:

```typescript
interface Vector1D {
  x: number;
}
interface Vector2D extends Vector1D {
  y: number;
}
interface Vector3D extends Vector2D {
  z: number;
}
```

基于“继承”或是“集合”两个角度，我们可以画出两种关系图。

<img src="./assets/4.png" width="300" />

在实际场景中，从集合的角度出发更具“普适性”与“表现力”。因为很多不适用于”继承“的场景，依然可以通过集合的角度很好的表达。比如对 `string|number` 和 `string|Date` 进行 `&` 操作的结果，可以通过集合的方式很直观的展示出来：

<img src="./assets/5.png" width="300" />

### 08. 学会判断一个符号是在类型空间还是值空间

在 TypeScript 中存在两个空间：

- 类型空间
- 值空间

一个符号可以同时出现在这两个空间里，它在不同空间中表达的含义完全不同：

```typescript
interface Cylinder {
  radius: number;
  height: number;
}
const Cylinder = (radius: number, height: number) => ({ radius, height });
```

`interface Cylinder` 在类型空间引入一个符号。`const Cylinder` 在值空间引入相同名字的符号。它们二者之间没有任何联系。TypeScript 会根据上下文来确定这个符号表示的具体含义。

```typescript
function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape.radius;
    //    ~~~~~~ Property 'radius' does not exist on type '{}'
  }
}
```

由于 `instanceof` 是一个 JavaScript 运行时操作符，它只能操作“值”，因此这里的 `Cylinder` 表示的是我们定义的函数，而不是类型。

如果你不能一眼看出某个符号存在于类型空间还是值空间，那么可以利用 [TypeScript Playground](https://www.typescriptlang.org/play) 这个工具。

<img src="./assets/6.png" width="500" />

由于在编译过程中类型信息会被移除，如果一个符号在编译结果中消失了，那么它很可能就存在于类型空间里。

TypeScript 中的语句可能会跨越两种空间。

```typescript
const p: Person = { first: "Jane", last: "Jacobs" };
//    -           --------------------------------- 值空间
//       ------ 类型空间
```

`class` 与 `enum` 会同时引入类型与值。在第一个例子中，如果我们把 `Cylinder` 定义为一个类：

```typescript
class Cylinder {
  radius = 1;
  height = 1;
}
function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape; // ✅ 类型是 Cylinder
    shape.radius; // ✅ 类型是 number
  }
}
```

类引入的 TypeScript 类型是由它的属性与方法决定的。类引入的值就是它的构造函数。

许多运算符在不同上下文中有不同的语义，以 `typeof` 为例：

```typescript
interface Person {
  first: string;
  last: string;
}
const p: Person = { first: "Jane", last: "Jacobs" };

function email(p: Person, subject: string, body: string): Response {}

type T1 = typeof p; // 取变量 p 的 TypeScript 类型，得到 Person
type T2 = typeof email; // 取函数 email 的 TypeScript 类型，得到 (p: Person, subject: string, body: string) => Response
const v1 = typeof p; // 取变量 p 的运行时类型(JavaScript 类型)，得到 "object"
const v2 = typeof email; // 取函数 email 的运行时类型(JavaScript 类型)，得到 "function"
```

对类进行 `typeof` 操作时，基于上下文我们有:

```typescript
const v = typeof Cylinder; // 取类 Cylinder 的运行时类型(JavaScript 类型)，也就是构造函数的 JavaScript 类型，得到 "function"
type T = typeof Cylinder; // 取类 Cylinder 的 TypeScript 类型，得到 typeof Cylinder。注意不是 Cylinder, Cylinder 只是该类的实例的类型。
```

如果你想通过 `Cylinder` 的类型取得该类实例的类型，你可以使用泛型 `InstanceType`:

```typescript
type C = InstanceType<typeof Cylinder>; // C 的类型就是 Cylinder
```

除了 `typeof` 以外，还有很多其他的关键字/运算符在不同上下文中拥有不同的含义：

- `this` 在值空间中表示 JavaScript 的 `this` 关键字。作为类型，`this` 表示“多态 this” 类型。这对于使用子类实现方法链很有帮助。
- 在值空间，`&` 和 `|` 表示位运算符。在类型空间，它们表示交集与并集操作。
- `const` 在值空间会创建一个新变量，而 [`as const` 断言](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions)会改变一个字面量的推断类型。
- `extends` 可以用来定义子类(`class A extends B`)，子类型(`interface A extends B`)或是给泛型增加约束条件(`Generic<T extends number>`)。

### 09. 优先使用类型声明而不是类型断言

要在 TypeScript 中给一个变量赋值同时指定其类型有两种方式：

```typescript
interface Person {
  name: string;
}
const alice: Person = { name: "Alice" }; // Type is Person
const bob = { name: "Bob" } as Person; // Type is Person
```

虽然结果上看起来一样，但在执行流程上有很大不同。前者(类型声明)首先给变量指定类型，然后会检查即将赋的值确保它符合指定的类型。后者(类型断言)则是告诉 TypeScript 虽然它推断出来值的类型是 `A`，但我们要把这个值当成类型 `B` 使用(**注意这里 `A` 与 `B` 之间是有约束条件的，要么 `A` 是 `B` 的子类型，要么`B` 是 `A` 的子类型**)。

总的来说，**应该始终优先使用类型声明而不是类型断言**。因为类型声明总是会进行严格的类型检查，而类型断言会在某些场景会忽略一部分类型错误。

例如：

```typescript
const alice: Person = {};
//    ~~~~~ Property 'name' is missing in type '{}'
//          but required in type 'Person'
const bob = {} as Person; // 这里不会报错！！
```

以及：

```typescript
const alice: Person = {
  name: "Alice",
  occupation: "TypeScript developer",
  // ~~~~~~~~~ Object literal may only specify known properties
  //           and 'occupation' does not exist in type 'Person'
};
const bob = {
  name: "Bob",
  occupation: "JavaScript developer",
} as Person; // 这里不会报错！！
```

再来看类型断言的约束，举个例子：

```typescript
interface Person {
  name: string;
}
const body = document.body;
const el = body as Person;
//         ~~~~~~~~~~~~~~ Conversion of type 'HTMLElement' to type 'Person'
//                        may be a mistake because neither type sufficiently
//                        overlaps with the other. If this was intentional,
//                        convert the expression to 'unknown' first
```

因为 `HTMLElement` 与 `Person` 两个类型并不满足其中一个是另一个子类型的约束条件，因此不能直接使用类型断言进行类型转换。如果一定要这样赋值，可以通过先转换为 `unknown` 类型的方式实现：

```typescript
const el = (document.body as unknown) as Person; // 虽然这样不会报错，但在实际使用中强烈不推荐这样做。
```

类型断言还有一种叫做“非 `null`” 断言的特殊形式。

```typescript
const elNull = document.getElementById("foo"); // Type is HTMLElement | null
const el = document.getElementById("foo")!; // Type is HTMLElement
```

给一个变量或表达式结尾加一个 `!`，相当于告诉 TypeScript 它的值一定不是 `null`。需要注意的是这只是给类型检查器**在静态类型检查阶段**的一个提示，在运行时是否会出现 `null` 值 TypeScript 是无法保证的，这需要由开发人员在业务逻辑中保证。

最后，看一个适用于类型断言的场景：

```typescript
document.querySelector("#myButton")!.addEventListener("click", (e) => {
  e.currentTarget; // Type is EventTarget
  const button = e.currentTarget as HTMLButtonElement;
  button; // Type is HTMLButtonElement
});
```

由于 TypeScript 无法访问 DOM，因此它无法知道 `#myButton` 是一个 `button` 元素。这时候我们可通过类型断言来明确 `currentTarget` 的类型，方便后续对其操作。
