## 泛型

软件工程的一个主要部分是构建组件。组件不仅具有定义良好且一致的 API，而且还具有可重用性。组件如果既能处理现在的数据，又能处理将来的数据，那么它将在构建大型软件系统的时候为你提供最灵活的能力。

在 C# 和 Java 等语言中，创建可重用组件的主要工具之一就是泛型。利用泛型，我们可以创建出适用于多种类型而不是单一类型的组件，从而允许用户在使用这些组件的时候用上自己的类型。

### 初识泛型

初识泛型，让我们先来实现一个 `identity` 函数。`identity` 函数可以接受任意参数并将其返回，你可以认为它的功能类似于 `echo` 指令。

如果不使用泛型，那么我们必须给 `identity` 函数指定一个具体的类型：

```ts
function identity(arg: number): number {
    return arg;
}
```

或者，我们可以使用 `any` 类型去描述 `identity` 函数：

```ts
function identity(arg: any): any {
    return arg;
}
```

使用 `any` 确实是一种通用的做法，因为函数的 `arg` 参数可以接受任意类型或者所有类型的值，但实际上，我们丢失了函数返回值类型的信息。如果我们传递的参数是数字，那么我们唯一能知道的信息是函数可以返回任意类型的值。

相反，我们需要一种方法去捕获参数的类型，这样我们也可以使用它来表示返回值的类型。这里，我们会使用一个类型变量，这种特殊的变量作用于类型而非值。

```ts
function identity<Type>(arg: Type): Type {
    return arg;
}
```

我们现在给 `identity` 函数添加了类型变量 `Type`。`Type` 允许我们捕获用户传入参数的类型（比如说 `number` 类型），这将作为稍后可以利用的信息。接着，我们在返回值类型的声明中也使用了 `Type`。从代码可以看出，参数和返回值都使用了相同的类型，这可以让我们在函数的一侧接受类型信息，并将信息传输给另一侧作为函数的输出。

我们称这个版本的 `identity` 函数是一个泛型函数，因为它适用于一系列的类型。和使用 `any` 不同，这个函数非常地明确（比如说，它没有丢失任何类型信息），效果和之前使用 `number` 作为参数和返回值类型的那个 `identity` 函数一样。

一旦实现了泛型的 `identity` 函数，我们就可以通过两种方式去调用它。第一种方式是给函数传递所有参数，包括类型参数：

```ts
let output = identity<string>("myString");
	^^^^^
        // let output: string
```

这里，我们显式设置 `Type` 为 `string` 并将其作为函数调用的参数。注意包裹参数的是 `<>` 不是 `()`。

第二种方式可能是最常用的。我们在这里使用了类型参数推断 —— 也就是说，我们想要让编译器基于传入参数的类型自动设置 `Type` 的值：

```ts
let output = identity("myString");
	^^^^^
        // let output: string
```

注意，我们并不一定要在 `<>` 中显式传入类型。编译器会查看值 `myString`，并将这个值的类型作为 `Type` 的值。虽然类型参数推断可以有效地保证代码的简洁与可读性，但在更复杂的案例中，编译器可能无法成功推断出类型，这时候你就需要像之前那样显式传入类型参数了。

### 使用泛型的类型变量

在你开始使用泛型之后，你会注意到，每次你创建类似 `identity` 这样的泛型函数之后，编译器就会强行认为你在函数体中正确地使用了任意通用的类型参数。也就是说，它会认为你将这些参数视为任意的、所有的类型。

我们看一下之前编写的 `identity` 函数：

```ts
function identity<Type>(arg: Type): Type {
    return arg;
}
```

如果每次调用的时候，我们都想要将参数 `arg` 的长度打印到控制台中，应该怎么做呢？我们可能会尝试写出下面的代码：

```ts
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
    			 ^^^^^^
// Property 'length' does not exist on type 'Type'.
  return arg;
}
```

当我们这么做的时候，编译器会抛出一个错误，指出我们正在访问 `arg` 的 `.length` 成员，但此前没有在任何地方声明 `arg` 有这个成员。记住之前我们说过的，这些类型变量表示任意的、所有的类型，因此使用这个函数的人可能会传递 `number` 类型的值进去，而这种值是没有 `.length` 成员的。

假设我们实际上想要让这个函数处理 `Type` 类型的数组，而不是直接处理 `Type`。既然当前处理的是数组，那么它就肯定有 `.length` 成员了。我们可以像创建其它类型的数组一样进行描述：

```ts
function loggingIdentity<Type>(arg: Type[]): Type[] {
  console.log(arg.length);
  return arg;
}
```

你可以将 `loggingIdentity` 的类型解读为“泛型函数 `loggingIdentity` 接受一个类型参数 `Type`，以及一个参数 `arg`，后者是一个 `Type` 类型的数组。函数最后返回的也是一个 `Type` 类型的数组”。如果我们传入的是 `number` 类型的数组，那么最终返回的也是 `number` 类型的数组，因为 `Type` 会绑定为 `number`。这允许我们将泛型类型变量 `Type` 作为要用到的其中一种类型去使用，而不是作为整个类型去使用，因此给予了我们更大的灵活性。

我们也可以将这个简单的示例改写为如下：

```ts
function loggingIdentity<Type>(arg: Array<Type>): Array<Type> {
  console.log(arg.length); // 数组有 length 属性，所以不会再抛出错误
  return arg;
}
```

你可能已经很熟悉其它语言中的这种类型风格了。在下一节中，我们会介绍如何自己创建类似 `Array<Type>` 这样的泛型。

### 泛型类型

在前面几节中，我们创建了适用于一系列类型的泛型函数 `identity`。在本节中，我们将探索函数本身的类型以及创建泛型接口的方法。

泛型函数类型和非泛型函数类型一样，只是前者会像泛型函数声明一样先列举出类型参数：

```ts
function identity<Type>(arg: Type): Type {
    return arg;
}
let myIdentity: <Type>(arg: Type) => Type = identity;
```

我们也可以给类型中的泛型类型参数使用不同的名字，只要类型变量的个数和使用方式保持不变：

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: <Input>(arg: Input) => Input = identity;
```

我们也可以将泛型类型编写为某个对象字面量类型的调用签名：

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: { <Type>(arg: Type): Type } = identity;
```

基于此，就可以编写我们的第一个泛型接口了。我们将上面代码中的对象字面量提取出来，并把它放到一个接口中：

```ts
interface GenericIdentityFn {
    <Type>(arg: Type): Type;
}
function identity<Type>(arg: Type): Type {
    return arg;
}
let myIdentity: GenericIdentityFn = identity;
```

有时候，我们可能想要将泛型参数移出去，作为整个接口的一个参数。这可以让我们看到泛型具体使用的是哪种类型（比如使用 `Dictionary<string>` 而不是 `Dictionary`）。同时这也让类型参数对接口的其它所有成员都是可见的。

```ts
interface GenericIdentityFn<Type> {
    (arg: Type): Type;
}
 
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: GenericIdentityFn<number> = identity;
```

注意，我们的例子变得和之前不太一样了。现在我们拥有的不再是一个泛型函数，而是一个非泛型函数签名，而它属于某个泛型类型的一部分。现在，当我们使用 `GenericIdentityFn` 的时候，我们需要指定对应的类型参数（这里是：`number`），从而有效地保证它被底层的函数签名所使用。理解什么时候把类型参数直接放到调用签名中，什么时候把类型参数放到接口本身中，对于描述某个类型的哪些地方是泛型有很大的作用。

除了泛型接口以外，我们也可以创建泛型类。注意我们无法创建泛型枚举和命名空间。

### 泛型类

泛型类和泛型接口的结构很相似。泛型类在类名后面会跟着 `<>`，里面是一个泛型类型参数列表。

```ts
class GenericNumber<NumType> {
  zeroValue: NumType;
  add: (x: NumType, y: NumType) => NumType;
}
 
let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function (x, y) {
  return x + y;
};
```

这种使用 `GenericNumber` 类的方法非常平实，但你可能注意到了一件事情，那就是我们并没有限制类只能使用 `number` 类型。相反，我们可以使用 `string` 甚至是其它更复杂的对象。

```ts
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function (x, y) {
  return x + y;
};
 
console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

和接口一样，把类型参数放到类本身，可以让我们确保类的所有属性都在使用同一类型。

我们在关于[类的章节](https://www.typescriptlang.org/docs/handbook/2/classes.html)有提到过，类包含两部分：静态部分和实例部分。泛型类的泛型只适用于实例部分而非静态部分，因此在使用泛型类的时候，静态成员无法使用类的类型参数，

### 泛型约束

还记得之前访问参数长度的例子吗？有时候，你可能需要编写一个只作用于某些类型的泛型函数，并且你对这些类型的特征有一定的了解。比如在示例的 `loggingIdentity` 函数中，我们想要访问 `arg` 的 `length` 属性，但是编译器无法验证每个类型都有 `length` 属性，因此它警告我们，我们不能假设所有类型都有该属性。

```ts
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length);
    			 ^^^^^^
// Property 'length' does not exist on type 'Type'.
  return arg;
}
```

我们不想让函数处理任意的类型，而是想将它约束为只能处理任意具备 `length` 属性的类型。只要某个类型具备这个属性，我们就允许传入该类型，反过来，要传入某个类型，那么它至少必须具备这个属性。为了实现这一点，我们必须列举出对 `Type` 的要求，以约束 `Type` 的类型。

为此，我们会创建一个描述约束条件的接口。这里，我们创建了一个具备单属性 `length` 的接口，之后使用该接口和 `extends` 关键字去表示我们的约束：

```ts
interface Lengthwise {
  length: number;
}
 
function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length); // 现在我们知道它一定是有 length 属性的，所以不会抛出错误
  return arg;
}
```

因为泛型函数现在受到了约束，所以它不再可以处理任意的、所有的类型：

```ts
loggingIdentity(3);
			   ^
// Argument of type 'number' is not assignable to parameter of type 'Lengthwise'.                   
```

相反，我们传入的值的类型应该具备所有必需属性：

```ts
loggingIdentity({ length: 10, value: 3 });
```

### 在泛型约束中使用类型参数

你可以声明一个类型参数，让它受到另一个类型参数的约束。举个例子，现在我们需要通过给定属性名访问对象的属性，那么我们必须确保不会意外地访问对象上不存在的属性，因此我们就会在两个类型之间使用一个约束：

```ts
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
    return obj[key];
}
 
let x = { a: 1, b: 2, c: 3, d: 4 };
 
getProperty(x, "a");
getProperty(x, "m");
			   ^	
// Argument of type '"m"' is not assignable to parameter of type '"a" | "b" | "c" | "d"'.
```

### 在泛型中使用类类型

在使用 TypeScript 泛型的前提下，如果使用了工厂模式创建实例，那么有必要通过构造函数引用类类型，从而推断出类的类型。例如：

```ts
function create<Type>(c: { new(): Type }): Type {
  return new c();
}
```

更高级的例子是像下面这样，使用原型属性去推断和约束构造函数和类类型的实例部分之间的关系：

```ts
class BeeKeeper {
  hasMask: boolean = true;
}
 
class ZooKeeper {
  nametag: string = "Mikle";
}
 
class Animal {
  numLegs: number = 4;
}
 
class Bee extends Animal {
  keeper: BeeKeeper = new BeeKeeper();
}
 
class Lion extends Animal {
  keeper: ZooKeeper = new ZooKeeper();
}
 
function createInstance<A extends Animal>(c: new () => A): A {
  return new c();
}
 
createInstance(Lion).keeper.nametag;
createInstance(Bee).keeper.hasMask;
```

这种模式可以用于驱动[混入](https://www.typescriptlang.org/docs/handbook/mixins.html)设计模式。

