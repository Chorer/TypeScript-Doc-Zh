## 模块

处理模块化代码的方式很多，JavaScript 在这方面有着悠久的历史。TypeScript 诞生于 2012 年，对许多模块化方案也实现了支持。但随着时间的推移，社区和 JavaScript 规范在一种名为 ES 模块（或者称为 ES6 模块）的方案上达成了共识。你可能听说过它的 `import/export` 语法。

ES 模块于 2015 年被纳入 JavaScript 规范，到了 2020 年，它已经获得了多数 web 浏览器和 JavaScript 运行时的支持。

本手册会重点讲解 ES 模块以及在它之前非常流行的、提供了 `module.exports =` 语法的 CommonJS。在“参考”章节的[模块](https://www.typescriptlang.org/docs/handbook/modules.html)这一小节中，你可以了解到更多关于其它模块化方案的信息。

### JavaScript 的模块是如何定义的

和 ECMAScript 2015 一样，TypeScript 会将任何包含顶层 `import` 或者 `export` 的文件视为一个模块。

反过来，一个不包含顶层 `import` 或者 `export` 声明的文件会被视为一个脚本，它的内容可以在全局作用域中访问到（因此对模块也是可见的）。

模块在自身的作用域而非全局作用域中执行。这意味着在一个模块中声明的变量、函数和类等在模块外面是不可见的，除非使用其中一种导出方式将它们显式导出。反过来，为了使用从某个不同的模块中导出的变量、函数、类等，也需要使用其中一种导入方式将它们导入。

### 非模块

在我们开始之前，有个很重要的事情需要搞清楚，那就是 TypeScript 会将什么视为一个模块。JavaScript 规范表明，任何不包含 `export` 或者顶层 `await` 的 JavaScript 文件都应该被视为一个脚本，而不是一个模块。

在一个脚本文件中声明的变量和类型会位于共享的全局作用域中，而且通常情况下，你会使用 [outFile](https://www.typescriptlang.org/tsconfig#outFile) 编译选项将多个输入文件合并为一个输出文件，或者使用 HTML 文件中的多个 `<script>` 标签去（以正确的顺序！）加载文件。

如果你的文件当前没有任何的 `import` 或者 `export`，但是你想将其视为一个模块，那么可以添加下面这行代码：

```js
export {};
```

这会将文件转化为没有导出任何东西的一个模块。不管你的模块目标是什么，这个语法都可以生效。

### TypeScript 中的模块

在 TypeScript 中编写基于模块的代码时，有三件主要的事情需要考虑：

* **语法：**我想要使用什么语法去进行导入和导出？
* **模块解析：**模块名（或者路径）和磁盘上的文件有什么关系？
* **模块输出目标：**产生的 JavaScript 模块看起来应该是什么样子的？

#### ES 模块语法

一个文件可以通过 `export default` 声明一个主要的导出：

```ts
// @filename: hello.ts
export default function helloWorld() {
  console.log("Hello, world!");
}
```

接着通过下面的语句导入：

```ts
import hello from "./hello.js";
hello();
```

除了默认导出之外，你还可以省略 `default`，直接用 `export` 导出多个变量和函数：

```js
// @filename: maths.ts
export var pi = 3.14;
export let squareTwo = 1.41;
export const phi = 1.61;
 
export class RandomNumberGenerator {}
 
export function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}
```

你可以通过 `import` 语法在另一个文件中导入：

```js
import { pi, phi, absolute } from "./maths.js";
 
console.log(pi);
const absPhi = absolute(phi);
         ^
   // const absPhi: number
```

#### 其它导入语法

可以使用诸如 `import {old as new}` 这样的形式重新命名一个导入：

```js
import { pi as π } from "./maths.js";
 
console.log(π);
            ^
     //(alias) var π: number
    // import π
```

你可以将上述语法混合为单个 `import`：

```js
// @filename: maths.ts
export const pi = 3.14;
export default class RandomNumberGenerator {}
 
// @filename: app.ts
import RNGen, { pi as π } from "./maths.js";
 
RNGen;
  ^
//(alias) class RNGen
//import RNGen
 
console.log(π);
            ^
     // (alias) const π: 3.14
    // import π
```

使用 `* as name`，你可以接受所有导出对象，并将它们放入单个命名空间中：

```js
// @filename: app.ts
import * as math from "./maths.js";
 
console.log(math.pi);
const positivePhi = math.absolute(math.phi);
           ^
    // const positivePhi: number
```

使用 `import "./file"`，你可以在当前模块中仅导入文件，而不导入文件中的任何变量：

```js
// @filename: app.ts
import "./maths.js";
 
console.log("3.14");
```

在这种情况下，`import` 不会做任何事。不过，`math.ts` 中的所有代码都会被执行，这可能会触发副作用并影响其它对象。

##### TypeScript 专属的 ES 模块语法

你可以使用和 JavaScript 值一样的语法将类型进行导出和导入：

```ts
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };
 
export interface Dog {
  breeds: string[];
  yearOfBirth: number;
}
 
// @filename: app.ts
import { Cat, Dog } from "./animal.js";
type Animals = Cat | Dog;
```

TypeScript 为 `import ` 语法拓展了两个用途，让它可以声明类型导入：

**`import type`**

该导入语句只能导入类型：

```ts
// @filename: animal.ts
export type Cat = { breed: string; yearOfBirth: number };
// 'createCatName' cannot be used as a value because it was imported using 'import type'.
export type Dog = { breeds: string[]; yearOfBirth: number };
export const createCatName = () => "fluffy";
 
// @filename: valid.ts
import type { Cat, Dog } from "./animal.js";
export type Animals = Cat | Dog;
 
// @filename: app.ts
import type { createCatName } from "./animal.js";
const name = createCatName();
```

**内联 `type` 导入**

TypeScript 4.5 也允许单个导入使用 `type` 前缀表明导入的引用是一个类型：

```ts
// @filename: app.ts
import { createCatName, type Cat, type Dog } from "./animal.js";
 
export type Animals = Cat | Dog;
const name = createCatName();
```

所有这些都允许诸如 Babel、swc 或者 esbuild 这样的非 TypeScript 转译工具知道哪些导入是可以被安全地移除的。

##### 具备 CommonJS 行为的 ES 模块语法

TypeScript 的 ES 模块语法可以和 CommonJS 与 AMD 的 `require` 直接关联。在大多数情况下，使用 ES 模块的导入与相同环境下使用 `require` 是一样的，但这个语法可以确保你的 TypeScript 文件和 CommonJS 输出存在一对一的匹配：

```ts
import fs = require("fs");
const code = fs.readFileSync("hello.ts", "utf8");
```

你可以在[模块](https://www.typescriptlang.org/docs/handbook/modules.html#export--and-import--require)这一参考章节中了解到更多关于该语法的信息。

### CommonJS 语法

CommonJS 是大多数 npm 包采用的模块化方案。即使你编写代码的时候采用的是 ES 模块语法，简要了解一下 CommonJS 语法的工作方式也有助于简化你的调试过程。

#### 导出

通过给一个名为 `module` 的全局对象设置 `exports` 属性，你可以导出标识符：

```ts
function absolute(num: number) {
  if (num < 0) return num * -1;
  return num;
}
 
module.exports = {
  pi: 3.14,
  squareTwo: 1.41,
  phi: 1.61,
  absolute,
};
```

之后可以通过 `require` 声明导入这些文件：

```ts
const maths = require("maths");
maths.pi;
      ^
   // any
```

或者你也可以使用 JavaScript 的解构语法只导入一部分内容：

```ts
const { squareTwo } = require("maths");
squareTwo;
    ^
  // const squareTwo: any
```

#### CommonJS 和 ES 模块的互操作

由于默认导入和模块命名空间对象导入的差异，CommonJS 和 ES 模块在功能上存在不匹配的地方。TypeScript 提供了一个编译选项 [esModuleInterop](https://www.typescriptlang.org/tsconfig#esModuleInterop)，以减少这两组不同的约束之间的冲突。

### TypeScript 的模块解析选项

模块解析是一个过程，它指的是从 `import` 或者 `require` 声明中提取一个字符串，并确定该字符串所指示的文件。

TypeScript 使用两种解析策略：Classic 和 Node。使用 Classic 策略是为了实现向后兼容，当编译选项 [module](https://www.typescriptlang.org/tsconfig#module) 不是 `commonjs` 的时候，默认采用该策略。Node 策略则复刻了 Node.js 在 CommonJS 模式下的工作方式，并提供了额外的 `.ts` 和 `.d.ts` 检查。

还有很多 TSConfig 选项会影响到 TypeScript 中的模块策略，包括：[moduleResolution](https://www.typescriptlang.org/tsconfig#moduleResolution)、[baseUrl](https://www.typescriptlang.org/tsconfig#baseUrl)、[paths](https://www.typescriptlang.org/tsconfig#paths)、[rootDirs](https://www.typescriptlang.org/tsconfig#rootDirs)。

有关这些策略如何工作的详细信息，请阅读[模块解析](https://www.typescriptlang.org/docs/handbook/module-resolution.html)。

### TypeScript 的模块输出选项

有两个选项会影响到最终输出的 JavaScript：

* [target](https://www.typescriptlang.org/tsconfig#target) 会决定哪些 JS 特性会被降级（进行转化并运行在比较旧的 JavaScript 运行时），哪些 JS 特性会被保留
* [module](https://www.typescriptlang.org/tsconfig#module) 会决定模块之间进行交互所使用的代码

使用哪个 [target](https://www.typescriptlang.org/tsconfig#target)，取决于你希望执行 TypeScript 代码的 JavaScript 运行时可以使用的特性。这样的运行时可以是：你支持的最旧的浏览器，你希望可以运行的最低版本的 Node.js，或者从运行时 —— 比如 Electron 的唯一约束进行考量。

模块之间的所有通信通过一个模块加载器进行，编译选项 [module](https://www.typescriptlang.org/tsconfig#module) 会决定应该使用哪一个。在运行时，模块加载器负责在执行模块之前定位并执行模块的所有依赖。

举个例子，这是一个使用 ES 模块语法的 TypeScript 文件：

```ts
import { valueOfPi } from "./constants.js";
 
export const twoPi = valueOfPi * 2;
```

下面是使用不同的 [module](https://www.typescriptlang.org/tsconfig#module) 选项之后的编译结果：

##### ES2020

```js
import { valueOfPi } from "./constants.js";
export const twoPi = valueOfPi * 2;
```

##### CommonJS

```js
"use strict";
Object.defineProperty(exports, "__esModule", { value: true });
exports.twoPi = void 0;
const constants_js_1 = require("./constants.js");
exports.twoPi = constants_js_1.valueOfPi * 2;
```

##### UMD

```js
(function (factory) {
    if (typeof module === "object" && typeof module.exports === "object") {
        var v = factory(require, exports);
        if (v !== undefined) module.exports = v;
    }
    else if (typeof define === "function" && define.amd) {
        define(["require", "exports", "./constants.js"], factory);
    }
})(function (require, exports) {
    "use strict";
    Object.defineProperty(exports, "__esModule", { value: true });
    exports.twoPi = void 0;
    const constants_js_1 = require("./constants.js");
    exports.twoPi = constants_js_1.valueOfPi * 2;
});
```

> 注意：ES2020 实际上和原先的 index.ts 是一样的。

你可以在 [TSConfig](https://www.typescriptlang.org/tsconfig#module) 这一参考章节的“模块”小节中了解到所有可用的选项以及对应输出的 JavaScript 代码是怎么样的。

### TypeScript 命名空间

TypeScript 有自己的模块格式，名为“命名空间”，它比 ES 模块标准出现得要早。这个语法提供了很多有用的特性以创建复杂的定义文件，并且仍然广泛应用于 [DefinitelyTyped](https://www.typescriptlang.org/dt) 中。虽然该语法还没有被弃用，但鉴于 ES 模块已经拥有了命名空间的大部分特性，我们推荐你使用 ES 模块来跟 JavaScript 保持一致。在[命名空间](https://www.typescriptlang.org/docs/handbook/namespaces.html)的参考章节中，你可以了解到更多相关信息。