<!-- markdownlint-disable MD033 -->

# 《Effective TypeScript》读书笔记

## 正文

**[认识 TypeScript](<#认识\ TypeScript>)**  
[1. 理解 TypeScript 与 JavaScript 之间的关系](<#1.\ 理解\ TypeScript\ 与\ JavaScript\ 之间的关系>)

### 认识 TypeScript

#### 1. 理解 TypeScript 与 JavaScript 之间的关系

**在语法层面上**，TypeScript 是 JavaScript 的超集。任何一段 JavaScript 程序，只要它没有语法错误，那么它也是一段合法的 TypeScript 程序(当然 TypeScript 的类型检查器可能会提示你一些类型相关的错误，但那只是一些具体问题，并不影响它”**是不是** TypeScript“ 这一问题的结论)。

TypeScript 使用 `.ts`(或 `.tsx`)作为文件扩展名。这并不意味着 TypeScript 是一种完全不同的语言。由于 TypeScript 是 JavaScript 的超集，因此你所有 `.js` 文件中的代码其实都是 TypeScript 代码。是否将 `main.js` 重命名为 `main.ts` 并不影响这一点。

所有的 JavaScript 程序都是 TypeScript 程序，这一论述的逆向说法并不成立：有的 TypeScript 程序携带了额外的声明类型的语法，而这部分语法是 JavaScript 不支持的。例如下面这段 TypeScript 程序：

```typescript
function greet(who: string) {
  console.log("Hello", who);
}
```

如果你使用 `node` 命令执行，那么你会得到一个 `SyntaxError: Unexpected token :` 的错误。 上述代码中的 `: string` 是一种类型注释。如果你使用了这种语法，那么你就走出了 JavaScript 的领域。

<img src="./assets/1.png" width="394" height="243" />
