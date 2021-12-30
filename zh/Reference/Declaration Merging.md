## 声明合并

### 介绍

TypeScript 中的某些独有概念在类型层面描述了 JavaScript 对象的形状。其中一个非常独有的概念就是“声明合并”。理解这个概念可以帮助你在处理现有的 JavaScript 时更加得心应手，同时，它也开启了通往更高级的抽象概念的大门。

对于本文而言，“声明合并”指的是编译器会将分开的两个相同名字的声明合并为一个声明。合并后的声明同时具有合并前声明的所有特性。任意数量的声明都可以合并到一起，不止局限于两个声明。

### 基础概念

在 TypeScript 中，一个声明创建的实体至少是下面三种类型的其中一种：命名空间、类型或者值。命名空间式的声明会创建一个命名空间，它包含的名字可以通过点访问符进行访问。类型式的声明会创建一个对已声明形状可见的类型，并且会绑定到给定的名字上。最后，值式的声明会创建在输出的 JavaScript 中可见的值。

| Declaration Type | Namespace | Type | Value |
| :--------------: | :-------: | :--: | :---: |
|    Namespace     |     X     |      |   X   |
|      Class       |           |  X   |   X   |
|       Enum       |           |  X   |   X   |
|    Interface     |           |  X   |       |
|    Type Alias    |           |  X   |       |
|     Function     |           |      |   X   |
|     Variable     |           |      |   X   |

理解每个声明会创建什么，才能更好地理解进行声明合并时会合并什么。

### 合并接口

最简单、并且可能是最常见的声明合并类型就是接口合并。在大多数情况下，合并操作会机械地将两个声明的所有成员放到一个同名的接口中。

```ts
interface Box {
    height: number;
    width: number;
}
interface Box {
    scale: number;
}
let box: Box = { height: 5, width: 6, scale: 10 };
```

接口的非函数成员应该是唯一的。如果它们不是唯一的，那么类型必须相同。如果两个接口都声明了一个同名但不同类型的非函数成员，那么编译器会抛出一个错误。

对于函数类型的成员，每一个同名的函数成员都会被视为是同个函数的一个重载。还需要注意的是，如果前面的接口 `A` 和后面的接口 `A` 合并，那么第二个接口的优先级会比第一个接口高。

举个例子：

```ts
interface Cloner {
    clone(animal: Animal): Animal;
}
interface Cloner {
    clone(animal: Sheep): Sheep;
}
interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
}
```

这三个接口将会合并为单个声明，如下所示：

```ts
interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
    clone(animal: Sheep): Sheep;
    clone(animal: Animal): Animal;
}
```

注意，每个接口里面的成员都会保持原有的顺序，但是接口原先越靠前，在重载中的位置就越靠后。

这个规则有一个例外，那就是使用专有签名的时候。如果某个签名的参数类型是一个单独的字符串字面量类型（而不是字符串字面量的联合类型），那么这个签名会“冒泡”到达合并后的重载列表的顶端。

举个例子，下面的接口会进行合并：

```ts
interface Document {
  createElement(tagName: any): Element;
}
interface Document {
  createElement(tagName: "div"): HTMLDivElement;
  createElement(tagName: "span"): HTMLSpanElement;
}
interface Document {
  createElement(tagName: string): HTMLElement;
  createElement(tagName: "canvas"): HTMLCanvasElement;
}
```

合并后的 `Document` 声明如下所示：

```ts
interface Document {
  createElement(tagName: "canvas"): HTMLCanvasElement;
  createElement(tagName: "div"): HTMLDivElement;
  createElement(tagName: "span"): HTMLSpanElement;
  createElement(tagName: string): HTMLElement;
  createElement(tagName: any): Element;
}
```

### 合并命名空间

和接口类似，同名的命名空间的所有成员也会合并到一起。由于命名空间会创建命名空间和值，所以我们需要理解它们各自是怎么合并的。

对于命名空间的合并，在每个命名空间中声明的导出接口的类型定义自身会进行合并，形成一个单独的、包含合并后的接口定义的命名空间。

对于命名空间中的值的合并，如果给定名字的命名空间已经存在了，那么它会进行拓展，即接受已有的命名空间，同时将第二个命名空间的导出成员添加到第一个命名空间中。

下面例子中的 `Animals`：

```ts
namespace Animals {
  export class Zebra {}
}
namespace Animals {
  export interface Legged {
    numberOfLegs: number;
  }
  export class Dog {}
}
```

进行声明合并后，等同于：

```ts
namespace Animals {
  export interface Legged {
    numberOfLegs: number;
  }
  export class Zebra {}
  export class Dog {}
}
```

这种模式的命名空间合并很好理解，但我们还需要了解非导出成员的情况。非导出成员只在原始的（未合并的）命名空间中可见，这意味着在合并之后，来自其它声明的合并成员无法访问非导出成员。

看下面的例子会更加直观：

```ts
namespace Animal {
  let haveMuscles = true;
  export function animalsHaveMuscles() {
    return haveMuscles;
  }
}
namespace Animal {
  export function doAnimalsHaveMuscles() {
    return haveMuscles; // Error, because haveMuscles is not accessible here
  }
}
```

因为 `haveMuscles` 没有导出，所以只有共享相同的未合并命名空间的 `animalsHaveMuscles` 函数才能访问它。`doAnimalsHaveMuscles` 函数虽然是合并后的 `Animal` 命名空间的一部分，但它无法访问这个未导出的成员。

### 合并命名空间和类、函数、枚举

命名空间非常灵活，它也能和其它类型的声明合并。要做到这一点，命名空间声明必须跟在要合并的声明后面。最终的声明将具有两个声明类型的所有属性。TypeScript 使用这种能力对 JavaScript 和其它编程语言中的模式进行复刻。

### 合并命名空间和类

这为开发者提供了一种描述内部类的方式：

```ts
class Album {
  label: Album.AlbumLabel;
}
namespace Album {
  export class AlbumLabel {}
}
```

合并成员的可见性规则和[合并命名空间这一小节](#合并命名空间)描述的一样，所以我们必须导出 `AlbumLabel` 这个类，以方便合并后的类去访问它。最终我们会得到一个类，其内部包含另一个类。你也可以使用命名空间向已有的类添加更多的静态成员。

除了内部类这种模式，你还可能熟悉使用 JavaScript 创建一个函数，之后通过添加属性对函数进行拓展。TypeSCript 借助声明合并，以一种类型安全的方式去构建这种模式。

```ts
function buildLabel(name: string): string {
  return buildLabel.prefix + name + buildLabel.suffix;
}
namespace buildLabel {
  export let suffix = "";
  export let prefix = "Hello, ";
}
console.log(buildLabel("Sam Smith"));
```

类似地，命名空间也可以用于为枚举拓展静态成员：

```ts
enum Color {
  red = 1,
  green = 2,
  blue = 4,
}
namespace Color {
  export function mixColor(colorName: string) {
    if (colorName == "yellow") {
      return Color.red + Color.green;
    } else if (colorName == "white") {
      return Color.red + Color.green + Color.blue;
    } else if (colorName == "magenta") {
      return Color.red + Color.blue;
    } else if (colorName == "cyan") {
      return Color.green + Color.blue;
    }
  }
}
```

### 不允许的合并

在 TypeScript 中，不是所有的合并都是允许的。就目前而言，类无法和其它类或者变量合并。关于模拟类合并的信息，可以查阅[ TypeScript 中的混入](https://www.typescriptlang.org/docs/handbook/mixins.html)这一小节。

### 模块增强

虽然 JavaScript 的模块不支持合并，但是你可以通过导入并更新模块，来实现对已有对象的增强。我们来看一个简易的观察者示例：

```ts
// observable.ts
export class Observable<T> {
  // ......
}
// map.ts
import { Observable } from "./observable";
Observable.prototype.map = function (f) {
  // ......
};
```

在 TypeScript 中，这段代码也能运行，但是编译器并不了解 `Observable.prototype.map`。你可以使用模块增强告知编译器它的信息：

```ts
// observable.ts
export class Observable<T> {
  // ......
}
// map.ts
import { Observable } from "./observable";
declare module "./observable" {
  interface Observable<T> {
    map<U>(f: (x: T) => U): Observable<U>;
  }
}
Observable.prototype.map = function (f) {
  // ......
};
// consumer.ts
import { Observable } from "./observable";
import "./map";
let o: Observable<number>;
o.map((x) => x.toFixed());
```

模块名的解析方式和 `import/export` 中的模块修饰符的解析方式相同。查阅[模块](https://www.typescriptlang.org/docs/handbook/modules.html)这一章以了解更多信息。模块增强中的声明会被合并，就好像它们是在原文件中声明的一样。

但是，这里有两个限制需要注意：

1. 你不能在模块增强中去创建一个顶级声明 —— 你只能增强已有的声明
2. 默认导出无法被增强，只有命名导出才能被增强（因为你需要通过导出的名字对导出进行增强，而 `default` 是一个保留字 —— 查阅 [#14080](https://github.com/Microsoft/TypeScript/issues/14080) 了解更多细节）。

### 全局增强

你也可以从模块内部向全局作用域添加声明：

```ts
// observable.ts
export class Observable<T> {
  // ......
}
declare global {
  interface Array<T> {
    toObservable(): Observable<T>;
  }
}
Array.prototype.toObservable = function () {
  // ...
};
```

全局增强和模块增强具有一样的行为和限制。