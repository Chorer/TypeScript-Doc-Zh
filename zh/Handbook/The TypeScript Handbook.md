## TypeScript 手册

### 关于本手册

JavaScript 被引入编程社区二十多年后，现在已成为有史以来最广泛使用的跨平台语言之一。起初它还只是一门为网页添加零星交互性的小型脚本语言，现在则成为了各种规模的前后端应用的首选语言。虽然用 JavaScript 编写的应用的大小、范围和复杂性呈指数级增长，但JavaScript 语言表达不同代码单元之间关系的能力却没有增强。再加上 JavaScript 非常独特的运行时语义，语言和程序复杂性之间的这种不匹配使得 JavaScript 开发成为难以大规模管理的任务。

程序员编码时最常见的错误可以说就是类型错误：期望某种值，但使用的却是另一种值。导致这种错误的原因很多，可能是简单的拼写错误，对库的 API 接口理解错误，对运行时行为的理解不正确，或者是其它的错误。TypeScript 的目标就是成为 JavaScript 应用的一个静态类型检查器 —— 换句话说，它是一个在代码执行前先运行的工具（静态），可以确保应用的类型是正确的（类型检查）。

如果你接触 TypeScript 的时候还没有学过 JavaScript，并且打算将 TypeScript 作为你的第一语言，那么我推荐你阅读一下 [Microsoft Learn JavaScript tutorial](https://docs.microsoft.com/javascript/) 的文档或者 [JavaScript at the Mozilla Web Docs](https://developer.mozilla.org/docs/Web/JavaScript/Guide)。如果你有其他语言的学习经验，那么阅读手册之后你应该就能很快掌握 JavaScript 的语法了。

### 本手册的结构

本手册包括两个部分：

* **手册**

  TypeScript 手册旨在成为一份向普通程序员解释 TypeScript 的综合文档。你可以在左边导航栏按照从上到下的顺序阅读手册。

  每一章或每一页都能让你对给定的概念有一个深刻的理解。TypeScript 手册不是一份完整的语言规范，但它旨在成为一份介绍语言所有特性和行为的综合性指南。

  阅读完本手册的读者应该能够：

  * 阅读并理解常用的 TypeScript 语法和模式
  * 解释重要的编译选项的效果
  * 在大多数情况下正确地预测类型系统的行为

  为了清晰和简洁起见，本手册的主要内容不会探讨所介绍特性的每一种边缘情况或细节。你可以在参考文章中找到特定概念的更多细节。

* **参考文件**

  导航栏中手册下方的参考部分旨在让你更深入地了解 TypeScript 的特定部分是如何工作的。你可以从上到下阅读，但每一部分都旨在对一个概念进行更深入的解释 —— 这意味着阅读起来可能是不连贯的。

#### 非目标

本手册也是一份可以在几个小时内轻松阅读的简明文档。为了保持简洁，我们不会涵盖某些特定的主题。

确切地说，本手册没有完全介绍核心的 JavaScript 基础知识，比如函数、类和闭包等。在适当的情况下，我们会给出背景导读的链接供你了解相关概念。

本手册也不打算取代语言规范。在某些情况下，我们会跳过边缘案例或行为的正式描述，以带来更高层次、更易理解的解释。不过，仍然会有单独的参考页去更精确和正式地描述TypeScript 行为的方方面面。参考页不适用于不熟悉 TypeScript 的读者，因此它们可能会使用你尚未阅读过的高级术语或参考主题。

最后，除了必要的地方，本手册不会介绍 TypeScript 如何与其它工具交互。诸如如何使用Webpack、rollup、packet、react、babel、closure、lerna、rush、bazel、preact、vue、angular、svelte、jquery、warn 或 npm 去配置 TypeScript 等主题，不在手册的讨论范围之内 —— 你可以在网上其它地方找到这些资源。

### 开始

在开始阅读[基础](https://www.typescriptlang.org/docs/handbook/2/basic-types.html)之前，我们建议你先阅读下面的其中一个介绍页。这些介绍旨在强调 TypeScript 和你喜欢的编程语言之间关键的相似性和差异性，并澄清这些语言特有的常见误解。

- [为新手级程序员准备的 TypeScript 入门指南](https://www.typescriptlang.org/docs/handbook/typescript-from-scratch.html)
- [为 JavaScript 程序员准备的 TypeScript 入门指南](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)
- [为 OOP 程序员准备的 TypeScript 入门指南](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-oop.html)
- [为函数式编程程序员准备的 TypeScript 入门指南](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-func.html)



