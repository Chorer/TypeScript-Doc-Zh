## 对象类型

在 JavaScript 中，最基础的分组和传递数据的方式就是使用对象。在 TypeScript 中，我们则通过对象类型来表示。

正如之前看到的，对象类型可以是匿名的：

```ts
function greet(person: { name: string; age: number }) {
  return "Hello " + person.name;
}
```

或者也可以使用一个接口来命名：

```ts
interface Person {
  name: string;
  age: number;
}
 
function greet(person: Person) {
  return "Hello " + person.name;
}
```

或者使用一个类型别名来命名：

```ts
type Person = {
  name: string;
  age: number;
};
 
function greet(person: Person) {
  return "Hello " + person.name;
}
```

在上面的例子中，我们编写的函数接受的对象包含了 `name` 属性（类型必须是 `string`）和 `age` 属性（类型必须是 `number`）。

### 属性修饰符

对象类型中的每个属性都可以指定一些东西：属性类型、属性是否可选，属性是否可写。

#### 可选属性

大多数时候，我们会发现自己处理的对象可能有一个属性集。这时候，我们可以在这些属性的名字后面加上 `?` 符号，将它们标记为可选属性。

```ts
interface PaintOptions {
  shape: Shape;
  xPos?: number;
  yPos?: number;
}
 
function paintShape(opts: PaintOptions) {
  // ...
}
 
const shape = getShape();
paintShape({ shape });
paintShape({ shape, xPos: 100 });
paintShape({ shape, yPos: 100 });
paintShape({ shape, xPos: 100, yPos: 100 });
```

在这个例子中，`xPos` 和 `yPos` 都是可选属性。这两个属性我们可以选择提供或者不提供，所以上面的 `paintShape` 调用都是有效的。可选性真正想要表达的其实是，如果设置了该属性，那么它最好有一个特定的类型。

这些属性同样可以访问 —— 但如果开启了 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks)，则 TypeScript 会提示我们这些属性可能是 `undefined`。

```ts
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos;
                  ^^^^
			// (property) PaintOptions.xPos?: number | undefined
  let yPos = opts.yPos;
                  ^^^^   
		    // (property) PaintOptions.yPos?: number | undefined
  // ...
}
```

在 JavaScript 中，即使从来没有设置过某个属性，我们也依然可以访问它 —— 值是 `undefined`。我们可以对 `undefined` 这种情况做特殊的处理。

```ts
function paintShape(opts: PaintOptions) {
  let xPos = opts.xPos === undefined ? 0 : opts.xPos;
      ^^^^ 
    // let xPos: number
  let yPos = opts.yPos === undefined ? 0 : opts.yPos;
      ^^^^ 
    // let yPos: number
  // ...
}
```

注意，这种为没有指定的值设置默认值的模式很常见，所以 JavaScript 提供了语法层面的支持。

```ts
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
  console.log("x coordinate at", xPos);
                                 ^^^^ 
							// (parameter) xPos: number
  console.log("y coordinate at", yPos);
                                 ^^^^ 
							// (parameter) yPos: number
  // ...
}
```

这里我们为 `paintShape` 的参数使用了[解构模式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)，同时也为 `xPos` 和 `yPos` 提供了[默认值](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Default_values)。现在，`xPos` 和 `yPos` 在 `paintShape` 函数体中就一定是有值的，且调用该函数的时候这两个参数仍然是可选的。

> 注意，目前没有任何方法可以在解构模式中使用类型注解。这是因为下面的语法在 JavaScript 中有其它的语义

```ts
function draw({ shape: Shape, xPos: number = 100 /*...*/ }) {
  render(shape);
    	^^^^^^
     // Cannot find name 'shape'. Did you mean 'Shape'?
  render(xPos);
         ^^^^^
    // Cannot find name 'xPos'.
}
```

在一个对象解构模式中，`shape: Shape` 表示“捕获 `shape` 属性并将其重新定义为一个名为 `Shape` 的局部变量”。同理，`xPos: number` 也会创建一个名为 `number` 的变量，它的值就是参数中 `xPos` 的值。

使用[映射修饰符](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#mapping-modifiers)可以移除可选属性。

#### 只读属性

在 TypeScript 中，我们可以将属性标记为 `readonly`，表示这是一个只读属性。虽然这不会改变运行时的任何行为，但标记为 `readonly` 的属性在类型检查期间无法再被重写。

```ts
interface SomeType {
  readonly prop: string;
}
 
function doSomething(obj: SomeType) {
  // 可以读取 obj.prop
  console.log(`prop has the value '${obj.prop}'.`);
 
  // 但无法重新给它赋值
  obj.prop = "hello";
// Cannot assign to 'prop' because it is a read-only property.
}
```

使用 `readonly` 修饰符并不一定意味着某个值是完全不可修改的 —— 或者换句话说，并不意味着它的内容是不可修改的。`readonly` 仅表示属性本身不可被重写。

```ts
interface Home {
  readonly resident: { name: string; age: number };
}
 
function visitForBirthday(home: Home) {
  // 我们可以读取并更新 home.resident 属性
  console.log(`Happy birthday ${home.resident.name}!`);
  home.resident.age++;
}
 
function evict(home: Home) {
  // 但我们无法重写 Home 类型的 resident 属性本身
  home.resident = {
       ^^^^^^^^
// Cannot assign to 'resident' because it is a read-only property.
    name: "Victor the Evictor",
    age: 42,
  };
}
```

理解 `readonly` 的含义非常重要。在使用 TypeScript 进行开发的过程中，它可以有效地表明一个对象应该如何被使用。TypeScript 在检查两个类型是否兼容的时候，并不会考虑它们的属性是否是只读的，所以只读属性也可以通过别名进行修改。

```ts
interface Person {
  name: string;
  age: number;
}
 
interface ReadonlyPerson {
  readonly name: string;
  readonly age: number;
}
 
let writablePerson: Person = {
  name: "Person McPersonface",
  age: 42,
};
 
// 可以正常执行
let readonlyPerson: ReadonlyPerson = writablePerson;
 
console.log(readonlyPerson.age); // 打印 42
writablePerson.age++;
console.log(readonlyPerson.age); // 打印 43
```

使用[映射修饰符](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#mapping-modifiers)可以移除只读属性。

#### 索引签名

有时候你无法提前知道某个类型所有属性的名字，但你知道这些属性值的类型。在这种情况下，你可以使用索引签名去描述可能值的类型。举个例子：

```ts
interface StringArray {
    [index: number]: string
}
const myArray: StringArray = getStringArray();
const secondItem = myArray[1];
      ^^^^^^^^^^    
     // const secondItem: string
```

上面的代码中，`StringArray` 接口有一个索引签名。这个索引签名表明当 `StringArray` 被 `number` 类型的值索引的时候，它将会返回 `string` 类型的值。

一个索引签名的属性类型要么是 `string`，要么是 `number`。

当然，也可以同时支持两种类型……

但前提是，数值型索引返回的类型必须是字符串型索引返回的类型的一个子类型。这是因为，当使用数值索引对象属性的时候，JavaScript 实际上会先把数值转化为字符串。这意味着使用 `100`（数值）进行索引与使用 `"100"`（字符串）进行索引，效果是一样的，因此这两者必须一致。

```ts
interface Animal {
  name: string;
}
 
interface Dog extends Animal {
  breed: string;
}
 
// Error: indexing with a numeric string might get you a completely separate type of Animal!
interface NotOkay {
  [x: number]: Animal;
// 'number' index type 'Animal' is not assignable to 'string' index type 'Dog'.
  [x: string]: Dog;
}
```

不过，如果索引签名所描述的类型本身是各个属性类型的联合类型，那么就允许出现不同类型的属性了：

```ts
interface NumberOrStringDictionary {
  [index: string]: number | string;
  length: number; // length 是数字，可以
  name: string; // name 是字符串，可以
}
```

最后，可以设置索引签名是只读的，这样可以防止对应索引的属性被重新赋值：

```ts
interface ReadonlyStringArray {
  readonly [index: number]: string;
}
 
let myArray: ReadonlyStringArray = getReadOnlyStringArray();
myArray[2] = "Mallory";
// Index signature in type 'ReadonlyStringArray' only permits reading.
```

因为索引签名设置了只读，所以无法再更改 `myArray[2]` 的值。

### 拓展类型

基于某个类型拓展出一个更具体的类型，这是一个很常见的需求。举个例子，我们有一个 `BasicAddress` 类型用于描述邮寄信件或者包裹所需要的地址信息。

```ts
interface BasicAddress {
    name?: string;
    street: string;
    city: string;
    country: string;
    postalCode: string;
}
```

通常情况下，这些信息已经足够了，不过，如果某个地址的建筑有很多单元的话，那么地址信息通常还需要有一个单元号。这时候，我们可以用一个 `AddressWithUnit` 来描述地址信息：

```ts
interface AddressWithUnit {
    name?: string;
    unit: string;
    street: string;
    city: string;
    country: string;
    postalCode: string;
}
```

这当然没问题，但缺点就在于：虽然只是单纯添加了一个域，但我们却不得不重复编写 `BasicAddress` 中的所有域。那么不妨改用一种方法，我们拓展原有的 `BasicAddress` 类型，并且添加上 `AddressWithUnit` 独有的新的域。

```ts
interface BasicAddress {
  name?: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}
 
interface AddressWithUnit extends BasicAddress {
  unit: string;
}
```

跟在某个接口后面的 `extends` 关键字允许我们高效地复制来自其它命名类型的成员，并且添加上任何我们想要的新成员。这对于减少我们必须编写的类型声明语句有很大的作用，同时也可以表明拥有相同属性的几个不同类型声明之间存在着联系。举个例子，`AddressWithUnit` 不需要重复编写 `street` 属性，且由于 `street` 属性来自于 `BasicAddress`，开发者可以知道这两个类型之间存在着某种联系。

接口也可以拓展自多个类型：

```ts
interface Colorful {
  color: string;
}
 
interface Circle {
  radius: number;
}
 
interface ColorfulCircle extends Colorful, Circle {}
 
const cc: ColorfulCircle = {
  color: "red",
  radius: 42,
};
```

### 交叉类型

接口允许我们通过拓展原有类型去构建新的类型。TypeScript 还提供了另一种称为“交叉类型”的结构，可以用来结合已经存在的对象类型。

通过 `&` 操作符可以定义一个交叉类型：

```ts
interface Colorful {
    color: string;
}
interface Circle {
    radius: number;
}
type ColorfulCircle = Colorful & Circle;
```

这里，我们结合 `Colorful` 和 `Circle`类型，产生了一个新的类型，它拥有 `Colorful` 和 `Circle` 的所有成员。

```ts
function draw(circle: Colorful & Circle) {
  console.log(`Color was ${circle.color}`);
  console.log(`Radius was ${circle.radius}`);
}
 
// 可以运行
draw({ color: "blue", radius: 42 });
 
// 不能运行
draw({ color: "red", raidus: 42 });
/*
Argument of type '{ color: string; raidus: number; }' is not assignable to parameter of type 'Colorful & Circle'.
  Object literal may only specify known properties, but 'raidus' does not exist in type 'Colorful & Circle'. Did you mean to write 'radius'?
*/ 
```

### 接口 VS 交叉类型

目前，我们了解到了可以通过两种方式去结合两个相似但存在差异的类型。使用接口，我们可以通过 `extends` 子句拓展原有类型；使用交叉类型，我们也可以实现类似的效果，并且使用类型别名去命名新类型。这两者的本质区别在于它们处理冲突的方式，而这个区别通常就是我们在接口和交叉类型的类型别名之间选择其一的主要理由。

### 泛型对象类型

假设我们有一个 `Box` 类型，它可能包含任何类型的值：`string`、`number`、`Giraffe` 等。

```ts
interface Box {
    contents: any;
}
```

现在，`contents` 属性的类型是 `any`，这当然没问题，但使用 `any` 可能会导致类型安全问题。

因此我们可以改用 `unknown`。但这意味着只要我们知道了 `contents` 的类型，我们就需要做一个预防性的检查，或者使用容易出错的类型断言。

```ts
interface Box {
  contents: unknown;
}
 
let x: Box = {
  contents: "hello world",
};
 
// 我们可以检查 x.contents
if (typeof x.contents === "string") {
  console.log(x.contents.toLowerCase());
}
 
// 或者使用类型断言
console.log((x.contents as string).toLowerCase());
```

还有另一种确保类型安全的做法是，针对每种不同类型的 `contents`，创建不同的 `Box` 类型。

```ts
interface NumberBox {
  contents: number;
}
 
interface StringBox {
  contents: string;
}
 
interface BooleanBox {
  contents: boolean;
}
```

但这意味着我们需要创建不同的函数，或者是函数的重载，这样才能操作不同的 `Box` 类型。

```ts
function setContents(box: StringBox, newContents: string): void;
function setContents(box: NumberBox, newContents: number): void;
function setContents(box: BooleanBox, newContents: boolean): void;
function setContents(box: { contents: any }, newContents: any) {
  box.contents = newContents;
}
```

这会带来非常多的样板代码。而且，我们后续还可能引入新的类型和重载，这未免有些冗余，毕竟我们的 `Box` 类型和重载只是类型不同，实质上是一样的。

不妨改用一种方式，就是让 `Box` 类型声明一个类型参数并使用泛型。

```ts
interface Box<Type> {
    contents: Type;
}
```

你可以将这段代码解读为“`Box` 的类型是 `Type`，它的 `contents` 的类型是 `Type`”。接着，当我们引用 `Box` 的时候，我们需要传递一个类型参数用于替代 `Type`。

```ts
let box: Box<string>;
```

如果把 `Box` 看作是实际类型的模板，那么 `Type` 就是一个会被其它类型代替的占位符。当 TypeScript 看到 `Box<string>` 的时候，它会将 `Box<Type>` 中的所有 `Type` 替换为 `string`，得到一个类似 `{ contents: string }` 的对象。换句话说，`Box<string>` 和之前例子中的 `StringBox` 是等效的。

```ts
interface Box<Type> {
  contents: Type;
}
interface StringBox {
  contents: string;
}
 
let boxA: Box<string> = { contents: "hello" };
boxA.contents;
     ^^^^^^^^   
    // (property) Box<string>.contents: string
 
let boxB: StringBox = { contents: "world" };
boxB.contents;
     ^^^^^^^^   
    // (property) StringBox.contents: string
```

因为 `Type` 可以被任何类型替换，所以 `Box` 具有可重用性。这意味着当我们的 `contents` 需要一个新类型的时候，完全无需再声明一个新的 `Box` 类型（虽然这么做没有任何问题）。

```ts
interface Box<Type> {
    contents: Type;
}
interface Apple {
    //...
}
// 和 { contents: Apple } 一样
type AppleBox = Box<Apple>;
```

这也意味着，通过使用[泛型函数](https://www.typescriptlang.org/docs/handbook/2/functions.html#generic-functions)，我们可以完全避免使用重载。

```ts
function setContents<Type>(box: Box<Type>, newContents: Type) {
    box.contents = newContents;
}
```

值得注意的是，类型别名也可以使用泛型。之前定义的 `Box<Type>` 接口：

```ts
interface Box<Type> {
    contents: Type;
}
```

可以改写为下面的类型别名：

```ts
type Box<Type> = {
    contents: Type;
};
```

类型别名和接口不一样，它不仅仅可以用于描述对象类型。所以我们也可以使用类型别名编写其它的泛型工具类型。

```ts
type OrNull<Type> = Type | null;
type OneOrMany<Type> = Type | Type[];
type OneOrManyOrNull<Type> = OrNull<OneOrMany<Type>>;
	 ^^^^^^^^^^^^^^
    //type OneOrManyOrNull<Type> = OneOrMany<Type> | null
type OneOrManyOrNullStrings = OneOrManyOrNull<string>;
     ^^^^^^^^^^^^^^^^^^^^^^          
    // type OneOrManyOrNullStrings = OneOrMany<string> | null         
```

我们稍后会再绕回来讲解类型别名。

#### 数组类型

泛型对象类型通常是某种容器类型，独立于它们所包含的成员的类型工作。数据结构以这种方式工作非常理想，即使数据类型不同也可以重复使用。

实际上，在这本手册中，我们一直都在和一个泛型打交道，那就是 `Array` （数组）类型。我们编写的 `number[]` 类型或者 `string[]` 类型，实际上都是 `Array<number>` 和 `Array<string>` 的简写。

```ts
function doSomething(value: Array<string>) {
  // ...
}
 
let myArray: string[] = ["hello", "world"];
 
// 下面两种写法都可以！
doSomething(myArray);
doSomething(new Array("hello", "world"));
```

就和前面的 `Box` 类型一样，`Array` 本身也是一个泛型：

```ts
interface Array<Type> {
  /**
   * 获取或者设置数组的长度
   */
  length: number;
 
  /**
   * 移除数组最后一个元素，并返回该元素
   */
  pop(): Type | undefined;
 
  /**
   * 向数组添加新元素，并返回数组的新长度
   */
  push(...items: Type[]): number;
 
  // ...
}
```

现代 JavaScript 也提供了其它同样是泛型的数据结构，比如 `Map<K,V>`、`Set<T>` 和 `Promise<T>`。这其实意味着，`Map`、`Set` 和 `Promise` 的表现形式使得它们能够处理任意的类型集。

#### 只读数组类型

`ReadonlyArray`（只读数组） 是一种特殊的类型，它描述的是无法被修改的数组。

```ts
function doStuff(values: ReadonlyArray<string>) {
  // 我们可以读取 values 
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);
 
  // ...但是无法修改 values
  values.push("hello!");
    	 ^^^^
        // Property 'push' does not exist on type 'readonly string[]'.
}
```

就像属性的 `readonly` 修饰符一样，它主要是一种用来表明意图的工具。当我们看到一个函数返回 `ReadonlyArray` 的时候，意味着我们不打算修改这个数组；当我们看到一个函数接受 `ReadonlyArray` 作为参数的时候，意味着我们可以传递任何数组给这个函数，而无需担心数组会被修改。

和 `Array` 不一样，`ReadonlyArray` 并没有对应的构造函数可以使用。

```ts
new ReadonlyArray("red", "green", "blue");
	^^^^^^^^^^^^^
// 'ReadonlyArray' only refers to a type, but is being used as a value here.
```

不过，我们可以把普通的 `Array` 赋值给 `ReadonlyArray`。

```ts
const roArray: ReadonlyArray<string> = ["red","green","blue"];
```

TypeScript 不仅为 `Array<Type>` 提供了简写 `Type[]`，也为 `ReadonlyArray<Type>` 提供了简写 `readonly Type[]`。

```ts
function doStuff(values: readonly string[]) {
  // 我们可以读取 values
  const copy = values.slice();
  console.log(`The first value is ${values[0]}`);
 
  // ...但无法修改 values
  values.push("hello!");
    	^^^^^
      // Property 'push' does not exist on type 'readonly string[]'.
}
```

最后一件需要注意的事情是，和 `readonly` 属性修饰符不同，普通的 `Array` 和 `ReadonlyArray` 之间的可赋值性不是双向的。

```ts
let x: readonly string[] = [];
let y: string[] = [];
 
x = y;
y = x;
^
// The type 'readonly string[]' is 'readonly' and cannot be assigned to the mutable type 'string[]'.
```

#### 元组类型

元组类型是一种特殊的 `Array` 类型，它的元素数量以及每个元素对应的类型都是明确的，

```ts
type StringNumberPair = [string, number];
```

这里，`StringNumberPair` 是一个包含 `string` 类型和 `number` 类型的元组类型。和 `ReadonlyArray` 一样，它没有对应的运行时表示，但对于 TypeScript 仍非常重要。对于类型系统而言，`StringNumberPair` 描述了这样的一个数组：下标为 `0` 的位置包含了一个 `string` 类型的值，下标为 `1` 的位置包含了一个 `number` 类型的值。

```ts
function doSomething(pair: [string, number]) {
  const a = pair[0];
        ^
     //const a: string
  const b = pair[1];
	    ^        
     // const b: number
  // ...
}
 
doSomething(["hello", 42]);
```

如果访问元组元素的时候下标越界，那么会抛出一个错误。

```ts
function doSomething(pair: [string, number]) {
  // ...
 
  const c = pair[2];
    		    ^	
// Tuple type '[string, number]' of length '2' has no element at index '2'.
}
```

我们也可以使用 JavaScript 的数组解构去[解构元组](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Array_destructuring)。

```ts
function doSomething(stringHash: [string, number]) {
  const [inputString, hash] = stringHash;
 
  console.log(inputString);
              ^^^^^^^^^^^    
		// const inputString: string
 
  console.log(hash);
              ^^^^   
			// const hash: number
}
```

> 元组类型在高度基于约定的 API 中很有用，因为每个元素的含义都是“明确的”。这给予了我们一种灵活性，让我们在解构元组的时候可以给变量取任意名字。在上面的例子中，我们可以给下标为 0 和 1 的元素取任何名字。
>
> 不过，怎么才算“明确”呢？每个开发者的见解都不一样。也许你需要重新考虑一下，在 API 中使用带有描述属性的对象是否会更好。

除了长度检查之外，类似这样的简单元组类型其实等价于一个对象，这个对象声明了特定下标的属性，且包含了数值字面量类型的 `length` 属性。

```ts
interface StringNumberPair {
  // 特定的属性
  length: 2;
  0: string;
  1: number;
 
  // 其它 Array<string | number> 类型的成员
  slice(start?: number, end?: number): Array<string | number>;
}
```

另一件你可能感兴趣的事情是，元组类型也可以拥有可选元素，只需要在某个元素类型后面加上 `?`。可选的元组元素只能出现在最后面，并且会影响该类型的长度。

```ts
type Either2dOr3d = [number, number, number?];
 
function setCoordinate(coord: Either2dOr3d) {
  const [x, y, z] = coord;
               ^
			// const z: number | undefined
 
  console.log(`Provided coordinates had ${coord.length} dimensions`);
                							 ^^^^^^                                
										// (property) length: 2 | 3
}
```

元组也可以使用展开运算符，运算符后面必须跟着数组或者元组。

```ts
type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
type BooleansStringNumber = [...boolean[], string, number];
```

- `StringNumberBooleans` 表示这样的一个元组：它的前两个元素分别是 `string` 和 `number` 类型，同时后面跟着若干个 `boolean` 类型的元素。
- `StringBooleansNumber` 表示这样的一个元组：它的第一个元素是 `string` 类型，接着跟着若干个 `boolean` 类型的元素，最后一个元素是 `number`。
- `BooleansStringNumber` 表示这样的一个元组：它前面有若干个 `boolean` 类型的元素，最后两个元素分别是 `string` 和 `number` 类型。

使用展开运算符的元组没有明确的长度 —— 可以明确的只是它的不同位置有对应类型的元素。

```ts
const a: StringNumberBooleans = ["hello", 1];
const b: StringNumberBooleans = ["beautiful", 2, true];
const c: StringNumberBooleans = ["world", 3, true, false, true, false, true];
```

为什么可选元素和展开运算符很有用呢？因为它允许 TypeScript 将参数列表对应到元组上。在[剩余参数和展开运算符](https://www.typescriptlang.org/docs/handbook/2/functions.html#rest-parameters-and-arguments)中可以使用元组，所以下面这段代码：

```ts
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args;
  // ...
}
```

和这段代码是等效的：

```ts
function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}
```

有时候，我们使用剩余参数接受数量可变的若干个参数，同时又要求参数不少于某个数量，且不想为此引入中间变量，这时候上面的这种写法就非常方便了。

#### 只读元组类型

关于元组类型还有最后一点需要注意的，那就是 —— 元组类型也可以是只读的，通过在元组前面加上 `readonly` 修饰符，我们可以声明一个只读的元组类型 —— 就像只读数组的简写一样。

```ts
function doSomething(pair: readonly [string, number]) {
  // ...
}
```

在 TypeScript 中无法重写只读元组的任何属性。

```ts
function doSomething(pair: readonly [string, number]) {
  pair[0] = "hello!";
       ^
// Cannot assign to '0' because it is a read-only property.
}
```

在大部分的代码中，元组被创建后就不需要修改了，所以将元组注解为只读类型是一个不错的默认做法。还有一点很重要的是，使用 `const` 断言的数组字面量将会被推断为只读元组。

```ts
let point = [3, 4] as const;
 
function distanceFromOrigin([x, y]: [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2);
}
 
distanceFromOrigin(point);
				 ^^^^^^ 	
/* 
Argument of type 'readonly [3, 4]' is not assignable to parameter of type '[number, number]'.
  The type 'readonly [3, 4]' is 'readonly' and cannot be assigned to the mutable type '[number, number]'. 
*/
```

这里，`distanceFromOrigin` 没有修改元组的元素，但它期望接受一个可变的元组。由于 `point` 的类型被推断为 `readonly [3,4]`，所以它和 `[number, number]` 是不兼容的，因为后者无法保证 `point` 的元素不会被修改。