<!-- markdownlint-disable MD033 -->

# [《Effective TypeScript》](https://effectivetypescript.com/)读书笔记

<img src="https://effectivetypescript.com/images/cover.jpg" width="267" height="350"/>

## 目录

- [01. 理解 TypeScript 与 JavaScript 之间的关系](#01-理解-TypeScript-与-JavaScript-之间的关系)
- [02. 明确你使用的 TypeScript 选项](#02-明确你使用的-TypeScript-选项)

<!-- markdownlint-disable MD033 -->

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
