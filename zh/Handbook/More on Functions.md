## 函数

无论是本地函数，还是从其它模块导入的函数，或者是类上的方法，函数都是任何应用的基本组成部分。它们同样也是值，就和其它值一样，TypeScript 有很多种描述函数如何被调用的方式。接下来，让我们了解如何编写类型去描述函数吧。

### 函数类型表达式

最简单的描述函数的方式就是使用函数类型表达式。这些类型在语法上和箭头函数非常相似：

```ts
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}
 
function printToConsole(s: string) {
  console.log(s);
}
 
greeter(printToConsole);
```

语法 `(a: string) => void` 表示“某个函数接受名为 `a` 的参数，类型为字符串，且该函数没有返回值”。和函数声明一样，如果没有指定参数类型，那么参数会被隐式推断为 `any` 类型。

> 注意参数名是**必需的**。函数类型 `(string) => void`表示“某个函数接受名为 `string` 的参数，类型为 `any`”

当然，我们也可以使用类型别名为某个函数类型命名：

```ts
type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
  // ...
}
```

### 调用签名

在 JavaScript 中，函数可以拥有属性，以方便进行调用。但是，TypeScript 的函数类型表达式语法不允许声明属性。如果我们想要描述某个可以通过属性被调用的东西，那么我们可以在一个对象类型上编写一个调用签名：

```ts
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
```

注意这个语法和函数类型表达式的语法不太一样。在参数列表和返回值类型之间，它使用的是 `:` 而不是 `=>`。

### 构造签名

JavaScript 函数也可以通过 `new` 运算符进行调用。TypeScript 将这种函数视为构造器，因为它们通常用于创建新对象。你可以在调用签名前面添加 `new` 关键字，从而编写一个构造签名：

```ts
type SomeConstructor = {
  new (s: string): SomeObject;
};
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```

有些对象，比如 JavaScript 的 `Date` 对象，可以直接调用也可以通过 `new` 调用。你可以在同一类型中任意组合调用签名和构造签名：

```ts
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}
```

### 泛型函数

我们经常需要编写某个函数，它的输入值类型和输出值类型相关联，或者两个输入值的类型在某种程度上相关联。假设现在有一个函数，它需要返回某个数组中的第一个元素：

```ts
function firstElement(arr: any[]) {
  return arr[0];
}
```

这个函数可以运行，但不幸的是，它的返回值类型为 `any`。如果返回值类型和数组类型一样，那就更好了。

在 TypeScript 中，当我们想要描述两个值之间的对应关系的时候，可以使用泛型。怎么使用呢？只需要在函数签名中声明一个类型参数：

```ts
function firstElement<Type>(arr: Type[]): Type | undefined {
    return arr[0];
}
```

通过添加一个类型参数 `Type` 到函数中，并在两个地方使用这个参数，我们已经让函数的输入值（数组）和输出值（返回值）建立了一个联系。现在当我们再次调用函数的时候，将会得到一个类型更加具体的返回值：

```ts
// s 的类型是 string
const s = firstElement(["a", "b", "c"]);
// n 的类型是 number
const n = firstElement([1, 2, 3]);
// u 的类型是 undefined
const u = firstElement([]);
```

#### 推断

注意在上面的例子中，我们不需要特殊说明 `Type` 的类型。因为 TypeScript 可以推断 —— 自动选择它的类型。

我们也可以使用多个类型参数。举个例子，可以像下面这样实现一个独立版本的 `map`：

```ts
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}
 
// n 的类型是 string
// parsed 的类型是 number[]
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```

注意在这个例子中，TypeScript 可以基于给定的 `string` 类型数组推断出 `Input` 类型参数的类型，也可以基于函数表达式的返回值类型（`number`）推断出 `Output` 类型参数的类型。

#### 约束

我们目前编写的泛型函数适用于所有类型的值。有时候，我们想要关联两个值，但要求只能对值的某个子集进行操作。这时候，我们可以使用“约束”去限制类型参数可以接受的种类。

我们来编写一个函数，它可以返回两个值长度较长的那个。为了实现这个功能，我们需要一个 `number` 类型的 `length` 属性。这里，我们通过 `extends` 子句将类型参数约束为具有 `length` 属性的类型：

```ts
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
 
// longerArray 的类型是 number[]
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString 的类型是 'alice' | 'bob'
const longerString = longest("alice", "bob");
// 报错！number 类型的值没有 length 属性
const notOK = longest(10, 100);
// Argument of type 'number' is not assignable to parameter of type '{ length: number; }'.
```

在这个例子中，没有什么有趣的事情值得注意。我们允许 TypeScript 推断 `longest` 函数返回值的类型。返回值的类型推断也适用于泛型函数。

我们将 `Type` 约束为 `{length: number}`，因此得以访问 `a` 参数和 `b` 参数的 `length` 属性。如果没有类型约束，那么我们是无法访问这个属性的，因为传入的参数可能是其它不具备 `length` 属性的类型。

`longerArray` 和 `longerString` 的类型是基于函数参数推断出来的。记住，泛型都是将两个或多个值与同一类型相关联！

#### 使用约束值

下面是使用泛型约束的时候常见的一个错误：

```ts
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minimum };
/*
Type '{ length: number; }' is not assignable to type 'Type'.
  '{ length: number; }' is assignable to the constraint of type 'Type', but 'Type' could be instantiated with a different subtype of constraint '{ length: number; }'.
*/  
  }
}
```

这个函数看起来似乎没毛病 —— `Type` 被约束为 `{length: number}`，函数要么返回 `Type`，要么返回匹配约束条件的值。但问题在于，函数承诺返回一个与传入参数相同类型的对象，而不是某个匹配约束条件的对象。如果这段代码是合法的，那么你很可能写出下面这样无法正常运行的代码：

```ts
// 'arr' 的值是 { length: 6 }
const arr = minimumLength([1, 2, 3], 6);
// 这里会报错，因为 arr 不是数组，没有 slice 方法
console.log(arr.slice(0));
```

#### 指定类型参数

在一次泛型调用中，TypeScript 通常可以推断出预期的类型参数，但也有例外。举个例子，假设你要编写一个合并两个数组的函数：

```ts
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
    return arr1.concat(arr2);
}
```

如果调用该函数的时候传入的两个数组的类型不匹配，那么正常情况下是会抛出错误的：

```ts
const arr = combine([1, 2, 3], ["hello"]);
							 ^^^^^^^
// Type 'string' is not assignable to type 'number'.
```

不过，如果你本意就是想合并两个类型不匹配的数组，那么你可以手动指定 `Type`：

```ts
const arr = combine<string | number>([1,2,3],["hello"]);
```

#### 编写良好泛型函数的指南

编写泛型函数很有意思，并且很容易因为使用类型参数而忘乎所以。使用过多类型参数或者在不需要的时候使用约束条件，会导致类型推断很难成功，对函数的调用者造成困惑。

**抑制类型参数**

下面两种方式编写的函数很相似：

```ts
function firstElement1<Type>(arr: Type[]) {
  return arr[0];
}
 
function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0];
}
 
// a: number (good)
const a = firstElement1([1, 2, 3]);
// b: any (bad)
const b = firstElement2([1, 2, 3]);
```

乍一看这两个函数好像差不多，但 `firstElement1` 函数要更好。它推断得到的返回值类型是 `Type`，而 `firstElement2` 推断得到的返回值类型却是 `any`，因为 TypeScript 需要使用约束类型去解析 `arr[0]` 表达式，而不是在函数调用期间“等着”去解析元素。

> **规则：** 在可能的情况下，请直接使用类型参数，而不是给它设置约束条件

**使用更少的类型参数**

下面是另一对相似的函数：

```ts
function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
  return arr.filter(func);
}
 
function filter2<Type, Func extends (arg: Type) => boolean>(
  arr: Type[],
  func: Func
): Type[] {
  return arr.filter(func);
}
```

在第二个函数中，我们创建了一个类型参数 `Func`，但是它并没有关联两个值。这是一个危险的信号，因为这意味着调用者传入实际的类型参数的时候，必须毫无理由地手动指定一个额外的类型参数。`Func` 不但没有帮上任何忙，反而破坏了函数的可读性和合理性。

> **规则：** 总是尽可能地使用更少的类型参数

**类型参数应该出现两次**

有时候我们会忘记某个函数可能是不需要使用泛型的：

```ts
function greet<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}
 
greet("world");
```

上面的函数可以改写为下面这个更简单的版本：

```ts
function greet(s: string) {
  console.log("Hello, " + s);
}
```

记住，类型参数的目的是关联多个值的类型。如果一个类型参数在函数签名中只使用了一次，那么它其实没有关联任何东西。

> **规则：** 如果一个类型参数在某个地方只出现了一次，请重新慎重思考自己是否需要使用类型参数

### 可选参数

JavaScript 中的函数可以接受的参数数量总是可变的。举个例子，`number` 的 `toFixed` 方法可以接受一个可选的参数表示数位：

```ts
function f(n: number) {
  console.log(n.toFixed()); // 0 个参数
  console.log(n.toFixed(3)); // 1 个参数
}
```

在 TypeScript 中，我们可以使用 `?` 标识某个参数是可选的：

```ts
function f(x?: number) {
  // ...
}
f(); // OK
f(10); // OK
```

虽然参数被指定为 `number` 类型，但它实际上是 `number | undefined` 类型，因为在 JavaScript 中，没有指定的参数的默认值就是 `undefined`.

你也可以给参数提供一个默认值：

```ts
function f(x = 10){
	// ...
}
```

现在，`f` 函数体中的 `x` 的类型将为 `number`，因为任何 `undefined` 类型的参数都会被替换为 `10`。注意，当参数可选的时候，调用者总是会传递 `undefined` 给这个参数，从而简单地模拟一个“丢失的”参数。

```ts
declare function f(x? : number): void;
// cut
// All ok
f();
f(10);
f(undefined);
```

#### 回调函数中的可选参数

在你了解了可选参数和函数类型表达式之后，你可能会很容易在编写回调函数的时候犯下面的错误：

```ts
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
    for(let i = 0;i < arr.length;i ++){
        callback(arr[i],i);
    }
}
```

很多人之所以编写 `index?` 以将其作为可选参数，本意是想让下面两种调用都是合法的：

```ts
myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```

这实际上意味着，在调用 `callback` 的时候可能只传入了一个参数。换句话说，上面的函数定义表明了它的内部实现可能是这样的：

```ts
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    // 这次我不想要提供 index
    callback(arr[i]);
  }
}
```

反过来推导，TypeScript 会使用这个内部实现，并抛出一个实际上不可能出现的错误：

```ts
myForEach([1, 2, 3], (a, i) => {
  console.log(i.toFixed());
              ^
// Object is possibly 'undefined'.
});
```

在 JavaScript 中，如果调用函数的时候传入的实参数量多于形参数量，那么多余的参数只会被简单地忽略掉。TypeScript 同样也是这么处理的。参数数量较少的函数总是可以替换参数数量较多的函数（两个函数的参数类型相同）。

> 当为回调函数编写一个函数类型的时候，永远不要使用可选参数，除非你的本意是在调用该函数的时候不传入那个参数。

### 函数重载

在调用某些 JavaScript 函数的时候，传入的参数数量和类型可能是多种多样的。举个例子，编写一个产生 `Date` 的函数时，既可以传入一个时间戳（一个参数），也可以传入年份/月份/天数（三个参数）。

在 TypeScript 中，我们可以编写重载签名来指定一个函数可以通过不同方式调用。要实现重载，你需要先编写一些函数签名（通常是两个或者两个以上），然后跟着一个函数体：

```ts
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);
// No overload expects 2 arguments, but overloads do exist that expect either 1 or 3 arguments.
```

在这个例子中，我们编写了两个重载：一个接受单个参数，另一个接受三个参数。前面的这两个签名称为“重载签名”。

之后，我们编写了一个带有兼容签名的函数实现。函数有一个“实现签名”，但是这个签名不能被直接调用。即使函数的一个必需参数后面跟着两个可选参数，调用该函数的时候也不能只传入两个参数！

#### 重载签名和实现签名

这是一个常见的让人困惑的地方。人们通常会写出下面的代码，并且不理解为什么会抛出错误：

```ts
function fn(x: string): void;
function fn() {
  // ...
}
// 这里本应该可以不传入任何参数
fn();
^^^^
// Expected 1 arguments, but got 0.
```

再次重申，用于编写函数体的签名必须不能从外部被“看到”。

> 实现签名不能从外部被“看到”。当编写重载函数的时候，在函数的代码实现部分的上面，必须始终有两个或者两个以上的签名。

此外，实现签名必须与重载签名兼容。举个例子，下面的写法都是错误的，因为实现签名没有正确地匹配重载签名：

```ts
function fn(x: boolean): void;
// 参数类型不对
function fn(x: string): void;
// This overload signature is not compatible with its implementation signature.
function fn(x: boolean) {}
```

```ts
function fn(x: string): string;
// 返回值类型不对
function fn(x: number): boolean;
// This overload signature is not compatible with its implementation signature.
function fn(x: string | number) {
  return "oops";
}
```

#### 编写良好的重载函数

就像泛型一样，在使用重载函数的时候，我们也需要遵循一些规则。遵循这些规则可以让你的函数更容易被调用、理解和实现。

假设有一个函数可以返回某个字符串或者数组的长度：

```ts
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
    return x.length;
}
```

这个函数很好，我们在调用的时候可以传入字符串或者数组。但是，我们无法传入一个可能是字符串或者数组的值，因为 TypeScript 只能将一个函数调用解析为单个重载：

```ts
len(""); // OK
len([0]); // OK
len(Math.random() > 0.5 ? "hello" : [0]);
/*
No overload matches this call.
  Overload 1 of 2, '(s: string): number', gave the following error.
    Argument of type 'number[] | "hello"' is not assignable to parameter of type 'string'.
      Type 'number[]' is not assignable to type 'string'.
  Overload 2 of 2, '(arr: any[]): number', gave the following error.
    Argument of type 'number[] | "hello"' is not assignable to parameter of type 'any[]'.
      Type 'string' is not assignable to type 'any[]'.
*/ 
```

因为两个重载都有相同的参数数量和返回类型，所以实际上可以编写一个不使用重载的函数版本：

```ts
function len(x: any[] | string) {
    return x.length;
}
```

这看起来就好多了！调用者调用该函数的时候可以传入两种参数的任意一种。还有一个额外的好处是，我们不需要搞清楚正确的实现签名。

> 在可能的情况下，请始终使用联合类型参数，而不是重载

#### 在函数中声明 `this`

TypeScript 可以通过代码流分析推断出函数中的 `this` 指向。举个例子：

```ts
const user = {
  id: 123,
 
  admin: false,
  becomeAdmin: function () {
    this.admin = true;
  },
};
```

TypeScript 知道 `user.becomeAdimin` 这个函数对应的 `this` 是外部对象 `user`。在大部分情况下这其实就足够了，但也有一些情况，我们需要更明确地控制 `this` 指向的对象。JavaScript 规范指出，参数不能取名为 `this`，因此 TypeScript 就利用了这个“语法空缺”让开发者声明函数体中 `this` 的类型。

```ts
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}
 
const db = getDB();
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});
```

在回调风格的 API 中，通常会有另一个对象控制函数何时被调用，因此这种模式很常见。注意，你需要使用 `function` 普通函数而不是箭头函数：

```ts
interface DB {
  filterUsers(filter: (this: User) => boolean): User[];
}
 
const db = getDB();
const admins = db.filterUsers(() => this.admin);
//The containing arrow function captures the global value of 'this'.
//Element implicitly has an 'any' type because type 'typeof globalThis' has no //index signature.
```

### 其它需要了解的类型

在使用函数类型的时候，我们还需要了解一些额外的类型。它们和之前介绍过的类型一样，也可以在任何地方使用，但和函数上下文的关联尤其紧密。

#### `void`

`void` 表示没有返回任何值的函数的返回值。只要某个函数没有 `return` 语句，或者 `return` 语句中没有返回任何显式的值，那么函数的返回值类型就会被推断为 `void`：

```ts
// 返回值的类型被推断为 void
function noop() {
  return;
}
```

在 JavaScript 中，没有返回值的函数会隐式返回 `undefined`。但是，在 TypeScript 中，`void` 和 `undefined` 是不一样的东西。在本章节的最后，我们会进一步讲解相关细节。

> `void` 和 `undefined` 是不一样的

#### `object`

特殊类型 `object` 代表的是任意非原始类型的值（`string`、 `number`、`bigint`、`boolean`、 `symbol`、`null` 或者 `undefined`）。它和空对象类型 `{}` 不一样，和全局类型 `Object` 也不一样。很可能你永远也不会用到 `Object`。

> `object` 不是 `Object`，请**始终**使用 `object`！

注意，在 JavaScript 中，函数也是对象：它们也有属性，原型链也包含 `Object.prototype`，并且 `instanceof Object` 返回 `true`，你还可以给 `Object.keys` 传递函数等。基于这些理由，TypeScript 中的函数类型被归类为 `object`。

#### `unknown`

`unknown` 类型表示任意值。它和 `any` 类型很相似，但更加安全，因为对 `unknown` 类型的值执行任何操作都是不合法的：

```ts
function f1(a: any) {
  a.b(); // OK
}
function f2(a: unknown) {
  a.b();
  ^  
//Object is of type 'unknown'.
}
```

在描述函数类型的时候，这很有用，因为你可以描述一个能接受任意值的函数，同时又避免在函数体中出现任何 `any` 类型的值。

同样的，你可以描述一个返回 `unknown` 类型值的函数：

```ts
function safeParse(s: string): unknown {
  return JSON.parse(s);
}
 
// 注意 obj 的类型！
const obj = safeParse(someRandomString);
```

#### `never`

有些函数从来不会返回任何值：

```ts
function fail(msg: string): never {
  throw new Error(msg);
}
```

`never` 类型表示从来不会被看到的值。当返回值是 `never` 类型的时候，意味着函数抛出了一个异常，或者终止了程序的执行。

当 TypeScript 确定联合类型中没有其它剩余类型的时候，也会用到 `never`。

```ts
function fn(x: string | number) {
  if (typeof x === "string") {
    // 执行某些操作
  } else if (typeof x === "number") {
    // 执行某些操作
  } else {
    x; // 类型为 never！
  }
}
```

 #### `Function`

全局类型 `Function` 描述了诸如 `bind`、`call`、`apply` 这样的属性，以及其它出现在 JavaScript 中所有函数值上面的属性。它还有一个特点，就是 `Function` 类型的值总是可以被调用的，并且都会返回 `any`：

```ts
function doSomething(f: Function) {
  return f(1, 2, 3);
}
```

这是一个非类型化的函数调用，最好避免这种用法，因为返回值为 `any` 是不安全的。

如果你需要接受一个任意类型的函数，但不打算调用它，那么使用类型 `() => void` 会更加安全。

### 剩余参数和展开运算符

#### 剩余参数

除了使用可选参数和重载让函数接受固定数量的多个参数以外，我们也可以定义一个函数，通过剩余参数让它接受数量不固定的参数。

剩余参数出现在所有参数后面，使用 `...` 语法：

```ts
function multiply(n: number, ...m: number[]) {
  return m.map((x) => n * x);
}
// a 的值是 [10, 20, 30, 40]
const a = multiply(10, 1, 2, 3, 4);
```

在 TypeScript 中，这些参数的类型注解隐式为 `any[]` 而不是 `any`，任何给定的类型注解也必须是 `Array<T>` 或者 `T[]` 的形式，或者使用元组类型（稍后会学习）。

#### 展开运算符

反过来，我们可以使用展开语法从数组中提供数量可变的参数。举个例子，数组的 `push` 方法可以接受任意数量的参数：

```ts
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
```

注意，通常情况下，TypeScript 不会假定数组是不可变的。这可能会导致一些令人惊讶的行为：

```ts
// 推断的类型是 number[]，也就是一个包含 0 个或更多数字的数组，而
// 不是一个只有两个数字的数组
const args = [8, 5];
const angle = Math.atan2(...args);
// A spread argument must either have a tuple type or be passed to a rest parameter.
```

针对这种情况，最好的解决方案需要取决于你编写的代码。但总的来说，`const` 上下文是最直接的解决方案：

```ts
// 推断为长度为 2 的元组
const args = [8, 5] as const;
// OK
const angle = Math.atan2(...args);
```

针对比较旧的运行时，使用展开运算符可能需要开启 [downlevelIteration](https://www.typescriptlang.org/tsconfig#downlevelIteration)。

### 参数解构

> 相关阅读：[解构赋值](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)

使用参数解构，你可以方便地将单个参数对象解包为函数体中的一个或多个局部变量。在 JavaScript 中，代码看起来是下面这样的：

```js
function sum({ a, b, c }) {
  console.log(a + b + c);
}
sum({ a: 10, b: 3, c: 9 });
```

对象的类型注解跟在解构语法后面：

```ts
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
```

这看起来有点冗长，因此你也可以在这里使用类型别名：

```ts
// 和上面的效果是一样的
type ABC = { a: number; b: number; c: number };
function sum({ a, b, c }: ABC) {
  console.log(a + b + c);
}
```

### 函数的可赋值性

#### 返回值类型为 `void`

函数的返回值类型为 `void` 的时候，可以产生一些不寻常的、但在意料之中的行为。

返回值类型为 `void` 的上下文类型并**不会**强迫函数**不**返回任何东西。换句话说，在实现一个返回值类型为 `void` 的上下文函数类型（`type vf = () => void`）的时候，它其实可以返回任意值，只是这些值会被忽略而已。

因此，类型 `() => void` 的下述实现都是有效的：

```TS
type voidFunc = () => void;
 
const f1: voidFunc = () => {
  return true;
};
 
const f2: voidFunc = () => true;
 
const f3: voidFunc = function () {
  return true;
};
```

当其中某个函数的返回值被赋值给另一个变量的时候，该变量仍会保持 `void` 类型：

```ts
const v1 = f1();
 
const v2 = f2();
 
const v3 = f3();
```

因为有这种行为的存在，所以下面的代码也是有效的。虽然 `Array.prototype.forEach` 方法期望接受一个返回值为 `void` 的函数，而 `Array.prototype.push` 返回的是数字。

```ts
const src = [1, 2, 3];
const dst = [0];
 
src.forEach((el) => dst.push(el));
```

这里还需要注意一种特殊情况，当某个字面量函数定义的返回值为 `void` 的时候，该函数**不能**返回任何东西。

```ts
function f2(): void {
  // 报错
  return true;
}
 
const f3 = function (): void {
  // 报错
  return true;
};
```

关于 `void` 的更多信息，请查阅下面其它的文档：

- [v1 手册](https://www.typescriptlang.org/docs/handbook/basic-types.html#void)
- [v2 手册](https://www.typescriptlang.org/docs/handbook/2/functions.html#void)
- [FAQ - “为什么返回值不是 void 的函数可以赋值给返回值是 void 的函数？”](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-are-functions-returning-non-void-assignable-to-function-returning-void)