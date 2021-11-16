## 普通类型

在这一章中，我们的内容会涉及到 JavaScript 代码中最常见的一些数据类型，同时也会解释这些类型在 TypeScript 中的对应描述方式。本章节并不会详尽介绍所有类型，在后续章节中我们还会介绍更多命名和使用其它类型的方法。

类型不仅可以出现在类型注解中，还可以出现在许多其它地方。在学习类型本身的同时，我们也会学习如何在某些地方使用这些类型去组成新的结构。

首先，我们先来回顾一下编写 JavaScript 或者 TypeScript 代码时最基础和最常用的类型。它们稍后将成为更复杂类型的核心组成部分。

### 原始类型：`string`、`number` 和 `boolean`

JavaScript 有三种很常用的[原始类型](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)：`string`、`number` 和 `boolean`。每一种类型在 TypeScript 中都有相对应的类型。正如你所料，它们的名字就和使用 JavaScript 的 `typeof` 运算符得到的字符串一样：

* `string` 表示类似 `"Hello, world!"` 这样的字符串值
* `number` 表示类似 `42` 这样的数值。对于整数，JavaScript 没有特殊的运行时值，所以也就没有 `int` 或者 `float` 类型 —— 所有的数字都是 `number` 类型
* `boolean` 表示布尔值 `true` 和 `false`

> 类型名 `String`、`Number` 和 `Boolean`（大写字母开头）也是合法的，但它们指的是在代码中很少出现的内建类型。请始终使用 `string`、`number` 和 `boolean`

### 数组

为了表示类似 `[1,2,3]` 这样的数组类型，你可以使用语法 `number[]`。这种语法也可以用于任意类型（比如 `string[]` 表示数组元素都是字符串类型）。它还有另一种写法是 `Array<number>`，两者效果是一样的。在后续讲解泛型的时候，我们会再详细介绍 `T<U>` 语法。

> 注意 `[number]`和普通数组不同，它表示的是[元组](https://www.typescriptlang.org/docs/handbook/2/objects.html#tuple-types)

### `any`

TypeScript 还有一种特殊的 `any` 类型。当你不想要让某个值引起类型检查错误的时候，可以使用 `any`。

当某个值是 `any` 类型的时候，你可以访问它的任意属性（这些属性也会是 `any` 类型），可以将它作为函数调用，可以将它赋值给任意类型的值（或者把任意类型的值赋值给它），或者是任何语法上合规的操作：

```ts
let obj: any = { x: 0 };
// 下面所有代码都不会引起编译错误。使用 any 将会忽略类型检查，并且假定了
// 你比 TypeScript 更了解当前环境
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```

当你不想要写一长串类型让 TypeScript 确信某行代码没问题的时候，`any` 类型很管用。

#### `noImplicitAny`

当你没有显式指定一个类型，同时 TypeScript 也无法从上下文中进行类型推断的时候，编译器会默认将其作为 `any` 类型处理。

不过，通常你会避免这种情况的发生，因为 `any` 是会绕过类型检查的。启用 [noImplicitAny](https://www.typescriptlang.org/tsconfig#noImplicitAny) 配置项可以将任意隐式推断得到的 `any` 标记为一个错误。

### 变量的类型注解

当你使用 `const`、`var` 或者 `let` 声明变量的时候，你可以选择性地添加一个类型注解以显式指定变量的类型：

```ts
let myName: string = 'Alice';
```

> TypeScript 没有采用类似 `int x = 0` 这样“在表达式左边声明类型”的风格。类型注解总是跟在要声明类型的东西后面。

不过，在大多数情况下，注解并不是必需的。TypeScript 会尽可能地在你的代码中自动进行类型推断。举个例子，变量的类型是基于它的初始值推断出来的：

```ts
// 不需要添加类型注解 —— myName 会被自动推断为 string 类型
let myName = 'Alice';
```

多数情况下，你不需要刻意去学习类型推断的规则。如果你还是初学者，请尝试尽可能少地使用类型注解 —— 你可能会惊讶地发现，TypeScript 完全理解所发生的事情所需要的注解是如此之少。

### 函数

函数是 JavaScript 中传递数据的主要方式。TypeScript 允许你指定函数的输入和输出的类型。

#### 参数类型注解

当你声明一个函数的时候，你可以在每个参数后面添加类型注解，从而声明函数可以接受什么类型的参数。参数的类型注解跟在每个参数名字的后面：

```ts
// 参数类型注解
function greet(name: string){
    console.log('Hello, ' + name.toUpperCase() + '!!');
}
```

当函数的某个参数有类型注解的时候，TypeScript 会对传递给函数的实参进行类型检查：

```ts
// 如果执行，会有一个运行时错误！
greet(42);
// Argument of type 'number' is not assignable to parameter of type 'string'.
```

> 即使没有给参数添加类型注解，TypeScript 也会检查你传递的参数的个数是否正确

#### 返回值类型注解

你也可以给返回值添加类型注解。返回值类型注解出现在参数列表后面：

```ts
function getFavourNumber(): number {
    return 26;
}
```

和变量的类型注解一样，通常情况下我们不需要给返回值添加一个类型注解，因为 TypeScript 会基于 `return` 语句推断出函数返回值的类型。上述例子中的类型注解不会改变任何事情。一些代码库会显式指定返回值的类型，这可能是出于文档编写的需要，或者是为了防止意外的修改，或者只是个人喜好。

#### 匿名函数

匿名函数和函数声明有点不同。当一个函数出现在某个地方，且 TypeScript 可以推断它是如何被调用的时候，该函数的参数会被自动分配类型。

比如：

```ts
// 这里没有类型注解，但 TypeScript 仍能在后续代码找出 bug
const names = ["Alice", "Bob", "Eve"];
 
// 基于上下文推断匿名函数参数的类型
names.forEach(function (s) {
  console.log(s.toUppercase());
  			   ^^^^^^^^^^^^  
// Property 'toUppercase' does not exist on type 'string'. Did you mean 'toUpperCase'?
});
 
// 对于箭头函数，也可以正确推断
names.forEach((s) => {
  console.log(s.toUppercase());
    		   ^^^^^^^^^^^^^
//Property 'toUppercase' does not exist on type 'string'. Did you mean 'toUpperCase'?
});
```

即使这里没有给参数 `s` 添加类型注解，TypeScript 也可以基于 `forEach` 函数的类型，以及对于 `name` 数组类型的推断，来决定 `s` 的类型。

这个过程叫做**上下文类型推断**，因为函数调用时所处的上下文决定了它的参数的类型。

和推断规则类似，你不需要刻意学习这个过程是怎么发生的，但明确这个过程确实会发生之后，你自然就清楚什么时候不需要添加类型注解了。稍后我们会看到更多的例子，了解到一个值所处的上下文是如何影响它的类型的。

### 对象类型

除了原始类型之外，最常见的类型就是对象类型了。它指的是任意包含属性的 JavaScript 值。要定义一个对象类型，只需要简单地列举它的属性和类型即可。

举个例子，下面是一个接受对象类型作为参数的函数：

```ts
// 参数的类型注解是一个对象类型
function printCoord(pt: { x: number; y: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
printCoord({ x: 3, y: 7 });
```

这里，我们为参数添加的类型注解是一个包含 `x` 和 `y` 两个属性（类型都是 `number`）的对象。你可以使用 `,` 或者 `;` 分隔每个属性，最后一个属性的分隔符可加可不加。

每个属性的类型部分同样也是可选的，如果你没有指定类型，那么它会采用 `any` 类型。

#### 可选属性

对象类型也可以指定某些或者全部属性是可选的。你只需要在对应的属性名后面添加一个 `?` 即可：

```ts
function printName(obj: { first: string; last?: string }) {
  // ...
}
// 下面两种写法都行
printName({ first: "Bob" });
printName({ first: "Alice", last: "Alisson" });
```

在 JavaScript 中，如果你访问了一个不存在的属性，你将会得到 `undefined` 而不是一个运行时错误。因此，在你读取一个可选属性的时候，你需要在使用它之前检查它是否为 `undefined`。

```ts
function printName(obj: { first: string; last?: string }) {
  // 如果 obj.last 没有对应的值，可能会报错！
  console.log(obj.last.toUpperCase());
// Object is possibly 'undefined'.
  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }
 
  // 下面是使用现代 JavaScript 语法的另一种安全写法：
  console.log(obj.last?.toUpperCase());
}
```

### 联合类型

TypeScript 的类型系统允许你基于既有的类型使用大量的运算符创建新的类型。既然我们已经知道了如何编写基本的类型，是时候开始用一种有趣的方式将它们结合起来了。

#### 定义一个联合类型

第一种结合类型的方式就是使用联合类型。联合类型由两个或者两个以上的类型组成，它代表的是可以取这些类型中任意一种类型的值。每一种类型称为联合类型的成员。

我们来编写一个可以处理字符串或者数字的函数：

```ts
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");
// 报错
printId({ myID: 22342 });
// Argument of type '{ myID: number; }' is not assignable to parameter of type 'string | number'.
//  Type '{ myID: number; }' is not assignable to type 'number'.
```

#### 使用联合类型

提供一个匹配联合类型的值非常简单 —— 只需要提供一个与联合类型某个成员相匹配的类型即可。如果有一个值是联合类型，你要怎么使用它呢？

TypeScript 会限制你对联合类型可以采取的操作，仅当该操作对于联合类型的每个成员都生效的时候，操作才会生效。举个例子，如果你有联合类型 `string | number`，那么你将无法使用只能由 `string` 调用的方法：

```ts
function printId(id: number | string) {
  console.log(id.toUpperCase());
// Property 'toUpperCase' does not exist on type 'string | number'.
//  Property 'toUpperCase' does not exist on type 'number'.
}
```

解决方案就是在代码中去收窄联合类型，这和没有使用类型注解的 JavaScript 的做法一样。当 TypeScript 能够基于代码结构推断出一个更具体的类型时，就会发生收窄。

举个例子，TypeScript 知道只有 `string` 类型的值使用 `typeof` 之后会返回 `"string"`：

```ts
function printId(id: number | string) {
  if (typeof id === "string") {
    // 在这个分支中，id 的类型是 string
    console.log(id.toUpperCase());
  } else {
    // 这里，id 的类型是 number
    console.log(id);
  }
}
```

另一个例子是使用类似 `Array.isArray` 这样的函数：

```ts
function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // 这里，x 是 string[]
    console.log("Hello, " + x.join(" and "));
  } else {
    // 这里，x 是 string
    console.log("Welcome lone traveler " + x);
  }
}
```

注意，在 `else` 分支中，我们不需要做额外的判断 —— 如果 `x` 不是 `string[]`，那它就一定是 `string`。

有时候，联合类型的所有成员可能存在共性。举个例子，数组和字符串都有 `slice` 方法。如果一个联合类型的每个成员都有一个公共的属性，那么你可以不需要进行收窄，直接使用该属性：

```ts
// 返回值会被推断为 number[] | string
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3);
}
```

> 联合类型的各个类型的属性存在交集，你可能会觉得有点困惑。实际上这并不让人意外，“联合”这个名词来自于类型理论。联合类型 `number | string` 是由每个类型的值的联合组成的。假设给定两个集合以及各自对应的事实，那么只有事实的交集可以应用于集合的交集本身。举个例子，有一个屋子的人都很高，而且戴帽子，另一个屋子的人都是西班牙人，而且也戴帽子，那么两个屋子的人放到一起，我们可以得到的唯一事实就是：每个人肯定都戴着帽子。

### 类型别名

目前为止，我们都是在类型注解中直接使用对象类型或者联合类型的。这很方便，但通常情况下，我们更希望通过一个单独的名字多次引用某个类型。

类型别名就是用来做这个的 —— 它可以作为指代任意一种类型的名字。类型别名的语法如下：

```ts
type Point = {
  x: number;
  y: number;
};
 
// 效果和之前的例子完全一样
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
```

不止是对象类型，你可以给任意一种类型使用类型别名。举个例子，你可以命名联合类型：

```ts
type ID = number | string;
```

注意，别名就只是别名而已 —— 你不能使用类型别名去创建同一类型的不同“版本”。当你使用别名的时候，效果就和你直接编写实际的类型一样。换句话说，代码看起来是不合法的，但在 TypeScript 里这是没问题的，不管是别名还是实际类型，都指向同一个类型：

```ts
type UserInputSanitizedString = string;
 
function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str);
}
 
// 创建一个输入
let userInput = sanitizeInput(getInput());
 
// 可以重新给它赋值一个字符串
userInput = "new input";
```

### 接口

接口声明是另一种命名对象类型的方式：

```ts
interface Point {
  x: number;
  y: number;
}
 
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
```

就和上面使用类型别名一样，这个例子也可以正常运行，它的效果和直接使用一个匿名对象类型一样。TypeScript 只关心我们传递给 `printCoord` 的值的结构 —— 它只关心这个值是否有期望的属性。正是因为这种只关注类型的结构和能力的特点，所以我们说 TypeScript 是一个结构性的、类型性的类型系统。

#### 类型别名和接口的区别

类型别名和接口很相似，多数情况下你可以任意选择其中一个去使用。接口的所有特性几乎都可以在类型别名中使用。两者关键的区别在于类型别名无法再次“打开”并添加新的属性，而接口总是可以拓展的。

```ts
// 接口可以自由拓展
interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}

const bear = getBear() 
bear.name
bear.honey

// 类型别名需要通过交集进行拓展
type Animal = {
  name: string
}

type Bear = Animal & { 
  honey: boolean 
}

const bear = getBear();
bear.name;
bear.honey;

// 向既有的接口添加新的属性
interface Window {
  title: string
}

interface Window {
  ts: TypeScriptAPI
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});

// 类型别名一旦创建，就不能再修改了
type Window = {
  title: string
}

type Window = {
  ts: TypeScriptAPI
}

 // Error: Duplicate identifier 'Window'
```

在稍后的章节中，你会学到更多关于这方面的知识，所以现在还不太理解也没关系。

* 在 TypeScript 4.2 版本之前，类型别名的名字可能会出现在报错信息中，有时会代替等效的匿名类型（可能需要，也可能不需要）。而接口的名字则始终出现在报错信息中
* 类型别名无法进行[声明合并，但接口可以](https://www.typescriptlang.org/play?#code/PTAEEEDtQS0gXApgJwGYEMDGjSfdAIx2UQFoB7AB0UkQBMAoEUfO0Wgd1ADd0AbAK6IAzizp16ALgYM4SNFhwBZdAFtV-UAG8GoPaADmNAcMmhh8ZHAMMAvjLkoM2UCvWad+0ARL0A-GYWVpA29gyY5JAWLJAwGnxmbvGgALzauvpGkCZmAEQAjABMAMwALLkANBl6zABi6DB8okR4Jjg+iPSgABboovDk3jjo5pbW1d6+dGb5djLwAJ7UoABKiJTwjThpnpnGpqPBoTLMAJrkArj4kOTwYmycPOhW6AR8IrDQ8N04wmo4HHQCwYi2Waw2W1S6S8HX8gTGITsQA)
* 接口只能用于[声明对象的形状，无法为原始类型命名](https://www.typescriptlang.org/play?#code/PTAEAkFMCdIcgM6gC4HcD2pIA8CGBbABwBtIl0AzUAKBFAFcEBLAOwHMUBPQs0XFgCahWyGBVwBjMrTDJMAshOhMARpD4tQ6FQCtIE5DWoixk9QEEWAeV37kARlABvaqDegAbrmL1IALlAEZGV2agBfampkbgtrWwMAJlAAXmdXdy8ff0Dg1jZwyLoAVWZ2Lh5QVHUJflAlSFxROsY5fFAWAmk6CnRoLGwmILzQQmV8JmQmDzI-SOiKgGV+CaYAL0gBBdyy1KCQ-Pn1AFFplgA5enw1PtSWS+vCsAAVAAtB4QQWOEMKBuYVUiVCYvYQsUTQcRSBDGMGmKSgAAa-VEgiQe2GLgKQA)
* 在报错信息中，接口的名字将始终以原始形式出现，但只限于它们作为名字被使用的时候

大多数情况下，你可以根据个人喜好选择其中一种使用，TypeScript 也会告诉你它是否需要使用另一种声明方式。如果你喜欢启发式，那你可以使用接口，等到需要使用其他特性的时候，再使用类型别名。

### 类型断言

有时候，你会比 TypeScript 更了解某个值的类型。

举个例子，如果你使用 `document.getElementById`，那么 TypeScript 只知道这个调用会返回某个 `HTMLElement`，但你却知道你的页面始终存在一个给定 ID 的 `HTMLCanvasElement`。

在这种情况下，你可以使用类型断言去指定一个更具体的类型：

```ts
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

就像类型注解一样，编译器最终会移除类型断言，保证它不会影响到代码的运行时行为。

你也可以使用等效的尖括号语法（前提是代码不是在一个 `.tsx` 文件中）：

```ts
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

> 记住：因为编译期间会移除类型断言，所以不存在和类型断言相关的运行时检查。即使类型断言是错误的，也不会抛出异常或者产生 `null`

TypeScript 只允许断言之后的类型比之前的类型更具体或者更不具体。这个规则可以防止出现下面这样“不可能存在的”强制类型转换：

```ts
const x = "hello" as number;
// 类型 "string" 到类型 "number" 的转换可能是错误的，因为两种类型不能充分重叠。如果这是有意的，请先将表达式转换为 "unknown"
```

有时候，这个规则可能过于保守了，会阻碍我们进行更复杂的有效转换操作。如果是这样，那么可以使用两步断言，先断言为 `any`（或者 `unknown`，稍后再介绍），再断言为期望的类型：

```ts
const a = (expr as any) as T;
```

### 字面量类型

除了通用的 `string` 和 `number` 类型之外，我们也可以将具体的字符串或者数字看作一种类型。

怎么理解呢？其实我们只需要考虑 JavaScript 声明变量的不同方式即可。`var` 和 `let` 声明的变量都可以修改，但 `const` 不行。这种特点反映在 TypeScript 是如何为字面量创建类型的。

```ts
let changingString = "Hello World";
changingString = "Olá Mundo";
// 因为 changingString 可以表示任意可能的字符串，这是 TypeScript 
// 在类型系统中描述它的方式
changingString;
^^^^^^^^^^^^^^
    // let changingString: string
      
let changingString: string
 
const constantString = "Hello World";
// 因为 constantString 只能表示一种可能的字符串，所以它有一个
// 字面量类型的表示形式
constantString;
^^^^^^^^^^^^^^^
    // const constantString: "Hello World"
      
```

只是单独使用的话，字面量类型的用处并不大：

```ts
let x: "hello" = "hello";
// OK
x = "hello";
// ...
x = "howdy";
// Type '"howdy"' is not assignable to type '"hello"'.
```

上面的例子中，变量只有一个可能的值，这是没有意义的！

但是通过将字面量类型结合为联合类型，你可以表示一个更有实用价值的概念 —— 举个例子，声明一个只接受某些固定值的函数：

```ts
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
						^^^^^^		
// Argument of type '"centre"' is not assignable to parameter of type '"left" | "right" | "center"'.
```

数值型字面量类型也同理：

```ts
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

当然，联合类型中也可以包含非字面量类型：

```ts
interface Options {
  width: number;
}
function configure(x: Options | "auto") {
  // ...
}
configure({ width: 100 });
configure("auto");
configure("automatic");
		  ^^^^^^^^^^	
// Argument of type '"automatic"' is not assignable to parameter of type 'Options | "auto"'.
```

还有一种字面量类型：布尔值字面量。只有两种布尔值字面量类型，也就是 `true` 和 `false`。`boolean` 类型本身其实就是联合类型 `true | false` 的一个别名。

#### 字面量推断

当你初始化一个变量为某个对象的时候，TypeScript 会假定该对象的属性稍后可能会发生变化。比如下面的代码：

```ts
const obj = { counter: 0 };
if (someCondition) {
  obj.counter = 1;
}
```

TypeScript 不觉得将之前值为 0 的属性赋值为 1 是一个错误。另一种理解角度是，`obj.counter` 必须是 `number` 类型，而不是 0，因为类型可以用来决定读写行为。

对于字符串也同理：

```ts
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
// Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.
```

(译者注：这里的 `handleRequest` 签名为 `(url: string, method: "GET" | "POST") => void`)

在上面的例子中，`req.method` 被推断为 `string`，而不是 `"GET"`。因为在创建 `req` 和调用 `handleRequest` 之间可能会执行其它代码，`req.method` 也许会被赋值为类似 `"GUESS"` 这样的字符串，因此 TypeScript 会认为这样的代码是存在错误的。

有两种方式可以解决这个问题：

1. 通过添加类型断言改变类型的推断结果：

   ```ts
   // 方法一：
   const req = { url: "https://example.com", method: "GET" as "GET" };
   // 方法二：
   handleRequest(req.url, req.method as "GET");
   ```

​	方法一表示“我有意让 `req.method` 一直采用字面量类型 `"GET"`”，从而阻止后续将其赋值为其它字符串；方法二表示“出于某种理由，我确信 `req.method` 的值一定是`“GET”`”。

2. 你还可以使用 `as const` 将整个对象转化为字面量类型：

   ```ts
   const req = { url: "https://example.com", method: "GET" } as const;
   handleRequest(req.url, req.method);
   ```

   `as const` 后缀和 `const` 的效果很像，但用于类型系统中。它可以确保对象的所有属性都被赋予了一个字面量类型，而不是采用类似 `string` 或者 `number` 这样较为通用的类型。

### `null` 和 `undefined`

JavaScript 中有两个原始值用于表示缺少的或者没有初始化的值：`null` 和 `undefined`。

TypeScript 对应地也有两个名字和它们一样的类型。它们的行为取决于你是否启用了 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 选项。

#### 禁用 `strictNullChecks`

禁用 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 选项之后，你仍然可以正常访问可能为 `null` 和 `undefined` 的值，这两个值也可以被赋值给任何一种类型。这种行为表现和缺少空值检查的语言（比如 C#、Java）很像。缺少对这些值的检查可能是大量 bug 的来源，在可行的前提下，我们推荐开发者始终启用 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 选项。

#### 启用 `strictNullChecks`

启用 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 选项之后，当一个值是 `null` 或者 `undefined` 的时候，你需要在使用该值的方法或者属性之前首先对其进行检查。就和使用可选属性之前先检查它是否为 `undefined` 一样，我们可以使用类型收窄去检查某个值是否可能为 `null`：

```ts
function doSomething(x: string | null) {
  if (x === null) {
    // do nothing
  } else {
    console.log("Hello, " + x.toUpperCase());
  }
}
```

#### 非空值断言操作符（`!` 后缀）

TypeScript 也提供了一种特殊的语法，可以在不显式进行检查的情况下，将 `null` 和 `undefined` 从类型中排除。在任意表达式后面添加后缀 `!`，可以有效地断言某个值不可能为 `null` 或者 `undefined`：

```ts
function liveDangerously(x?: number | null) {
  // 不会报错
  console.log(x!.toFixed());
}
```

和其它的类型断言一样，非空值断言也不会改变代码的运行时行为，所以切记：仅在你确定某个值不可能为 `null` 或者 `undefined` 的时候，才去使用 `!`。

### 枚举

枚举是 TypeScript 添加到 JavaScript 中的一项特性。它允许描述一个值，该值可以是一组可能的命名常量中的一个。与大多数的 TypeScript 特性不同，枚举不是在类型层面添加到 JavaScript 中的，而是添加到语言本身和它的运行时中。正因如此，你应该了解这个特性的存在，但除非你确定，否则你可能需要推迟使用它。你可以在[枚举引用页面](https://www.typescriptlang.org/docs/handbook/enums.html)中了解到有关枚举的更多信息。

### 其它不常见的原始类型

值得一提的是，JavaScript 的其它原始类型在类型系统中也有对应的表示形式。不过在这里我们不会深入进行探讨。

#### BigInt

ES2020 引入了 `BigInt`，用于表示 JavaScript 中非常大的整数：

```ts
// 通过 BigInt 函数创建大整数
const oneHundred: bigint = BigInt(100);
 
// 通过字面量语法创建大整数
const anotherHundred: bigint = 100n;
```

你可以在 [TypeScript 3.2 发布日志](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-2.html#bigint) 中了解到关于 BigInt 的更多信息。

#### symbol

在 JavaScript 中，我们可以通过函数 `Symbol()` 创建一个全局唯一的引用：

```ts
const firstName = Symbol("name");
const secondName = Symbol("name");
 
if (firstName === secondName) {
// 此条件将始终返回 "false"，因为类型 "typeof firstName" 和 "typeof secondName" 没有重叠。    
}
```

你可以在 [Symbol 引用页面](https://www.typescriptlang.org/docs/handbook/symbols.html) 了解到更多相关信息。
