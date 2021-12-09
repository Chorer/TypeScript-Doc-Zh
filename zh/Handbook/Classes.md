> 背景导读：[类（MDN）](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)

## 类

TypeScript 为 ES2015 引入的 `class` 关键字提供了全面的支持。

就像其它的 JavaScript 语言特性一样，TypeScript 也为类提供了类型注解和其它语法，以帮助开发者表示类和其它类型之间的关系。

### 类成员

这是一个最基本的类 —— 它是空的：

```ts
class Point {}
```

这个类目前没有什么用，所以我们给它添加一些成员吧。

#### 字段

声明字段相当于是给类添加了一个公共的、可写的属性：

```ts
class Point {
    x: number;
    y: number;
}
const pt = new Point()
pt.x = 0;
pt.y = 0;
```

和其它特性一样，这里的类型注解也是可选的，但如果没有指定类型，则会隐式采用 `any` 类型。

字段也可以进行初始化，初始化过程会在类实例化的时候自动进行：

```ts
class Point {
  x = 0;
  y = 0;
}
 
const pt = new Point();
// 打印 0, 0
console.log(`${pt.x}, ${pt.y}`);
```

就像使用 `const`、`let` 和 `var` 一样，类属性的初始化语句也会被用于进行类型推断：

```ts
const pt = new Point();
pt.x = "0";
// Type 'string' is not assignable to type 'number'.
```

##### --strictPropertyInitialization

配置项 [strictPropertyInitialization](https://www.typescriptlang.org/tsconfig#strictPropertyInitialization) 用于控制类的字段是否需要在构造器中进行初始化。

```ts
class BadGreeter {
  name: string;
    ^
// Property 'name' has no initializer and is not definitely assigned in the constructor.
}
class GoodGreeter {
  name: string;
 
  constructor() {
    this.name = "hello";
  }
}    
```

注意，字段需要在构造器自身内部进行初始化。TypeScript 不会分析在构造器中调用的方法以检测初始化语句，因为派生类可能会重写这些方法，导致初始化成员失败。

如果你坚持要使用除了构造器之外的方法（比如使用一个外部库填充类的内容）去初始化一个字段，那么你可以使用确定赋值断言运算符 `！`：

```ts
class OKGreeter {
  // 没有初始化，但不会报错
  name!: string;
}
```

#### `readonly`

字段可以加上 `readonly` 修饰符作为前缀，以防止在构造器外面对字段进行赋值。

```ts
class Greeter {
  readonly name: string = "world";
 
  constructor(otherName?: string) {
    if (otherName !== undefined) {
      this.name = otherName;
    }
  }
 
  err() {
    this.name = "not ok";
      	   ^
// Cannot assign to 'name' because it is a read-only property.
  }
}
const g = new Greeter();
g.name = "also not ok";
    ^
// Cannot assign to 'name' because it is a read-only property.
```

#### 构造器

类的构造器和函数很像，你可以给它的参数添加类型注解，可以使用参数默认值或者是函数重载：

```ts
class Point {
    x: number;
    y: number;
    // 使用了参数默认值的正常签名
    constructor(x = 0, y = 0) {
        this.x = x;
        this.y = y;
    }
}
class Point {
  // 使用重载
  constructor(x: number, y: string);
  constructor(s: string);
  constructor(xs: any, y?: any) {
    // TBD
  }
}
```

类的构造器签名和函数签名只有一点区别：

* 构造器不能使用类型参数 —— 类型参数属于类声明的部分，稍后我们会进行学习
* 构造器不能给返回值添加类型注解 —— 它返回的类型始终是类实例的类型

##### `super` 调用

和 JavaScript 一样，如果你有一个基类和一个派生类，那么在派生类中使用 `this.` 访问类成员之前，必须先在构造器中调用 `super();`：

```ts
class Base {
  k = 4;
}
 
class Derived extends Base {
  constructor() {
    // ES5 下打印出错误的值，ES6 下报错
    console.log(this.k);
                  ^
// 'super' must be called before accessing 'this' in the constructor of a derived class.
    super();
  }
}
```

在 JavaScript 中，忘记调用 `super` 是一个常见的错误，但 TypeScript 会在必要时给你提醒。

#### 方法

类的属性可能是一个函数，这时候我们称其为方法。方法和函数以及构造器一样，也可以使用各种类型注解：

```ts
class Point {
  x = 10;
  y = 10;
 
  scale(n: number): void {
    this.x *= n;
    this.y *= n;
  }
}
```

除了标准的类型注解之外，TypeScript 没有给方法添加什么新的东西。

注意，在方法体中，必须通过 `this.` 才能访问到类的字段和其它方法。在方法体中使用不合规的名字，将会被视为是在访问邻近作用域中的变量：

```ts
let x: number = 0;
 
class C {
  x: string = "hello";
 
  m() {
    // 下面这句是在试图修改第一行的 x，而不是类的属性
    x = "world";
    ^  
// Type 'string' is not assignable to type 'number'.
  }
}
```

#### Getters/Setters

类也可以有访问器：

```ts
class C {
    _length = 0;
    get length(){
        return this._length;
    }
    set length(value){
        this._length = value;
    }
}
```

> 注意：在 JavaScript 中，一个没有额外逻辑的 get/set 对是没有什么作用的。如果在执行 get/set 操作的时候不需要添加额外的逻辑，那么只需要将字段暴露为公共字段即可。

对于访问器，TypeScript 有一些特殊的推断规则：

* 如果 `get` 存在而 `set` 不存在，那么属性会自动成为只读属性
* 如果没有指定 setter 参数的类型，那么会基于 getter 返回值的类型去推断参数类型
* getter 和 setter 必须具备相同的[成员可见性](#成员可见性)。

从 [TypeScript 4.3](https://devblogs.microsoft.com/typescript/announcing-typescript-4-3/) 开始，访问器的 getter 和 setter 可以使用不同的类型。

```ts
class Thing {
  _size = 0;
 
  get size(): number {
    return this._size;
  }
 
  set size(value: string | number | boolean) {
    let num = Number(value);
 
    // 不允许使用 NaN、Infinity 等
 
    if (!Number.isFinite(num)) {
      this._size = 0;
      return;
    }
 
    this._size = num;
  }
}
```

#### 索引签名

类可以声明索引签名，其工作方式和[其它对象类型的索引签名](https://www.typescriptlang.org/docs/handbook/2/objects.html#index-signatures)一样：

```ts
class MyClass {
    [s: string]: boolean | ((s: string) => boolean);
    check(s: string) {
        return this[s] as boolean;
    }
}
```

因为索引签名类型也需要捕获方法的类型，所以要有效地使用这些类型并不容易。通常情况下，最好将索引数据存储在另一个位置，而不是类实例本身。

### 类继承

和其它面向对象语言一样，JavaScript 中的类可以继承自基类。

#### `implements` 子句

你可以使用一个 `implements` 子句去检查类是否符合某个特定的接口。如果类没有正确地实现这个接口，那么就会抛出一个错误：

```ts
interface Pingable {
  ping(): void;
}
 
class Sonar implements Pingable {
  ping() {
    console.log("ping!");
  }
}
 
class Ball implements Pingable {
        ^
/*
Class 'Ball' incorrectly implements interface 'Pingable'.
  Property 'ping' is missing in type 'Ball' but required in type 'Pingable'.
*/  
  pong() {
    console.log("pong!");
  }
}
```

类可以实现多个接口，比如 `class C implements A,B {`。

##### 注意事项

有个要点需要理解，那就是 `implements` 子句只是用于检查类是否可以被视为某个接口类型，它完全不会改变类的类型或者它的方法。常见的错误是认为 `implements` 子句会改变类的类型 —— 实际上是不会的！

```ts
interface Checkable {
  check(name: string): boolean;
}
 
class NameChecker implements Checkable {
  check(s) {
        ^
//Parameter 's' implicitly has an 'any' type.
    // 注意这里不会抛出错误
    return s.toLowercse() === "ok";
                 ^
              // any
  }
}
```

在这个例子中，我们可能会认为 `s` 的类型会受到接口中 `check` 的 `name: string` 参数的影响。但实际上不会 —— `implements` 子句不会对类内容体的检查以及类型推断产生任何影响。

同理，实现一个带有可选属性的接口，并不会创建该属性：

```ts
interface A {
  x: number;
  y?: number;
}
class C implements A {
  x = 0;
}
const c = new C();
c.y = 10;
  ^
// Property 'y' does not exist on type 'C'.
```

#### `extends` 子句

类可以继承自某个基类。派生类拥有基类的所有属性和方法，同时也可以定义额外的成员。

```ts
class Animal {
  move() {
    console.log("Moving along!");
  }
}
 
class Dog extends Animal {
  woof(times: number) {
    for (let i = 0; i < times; i++) {
      console.log("woof!");
    }
  }
}
 
const d = new Dog();
// 基类方法
d.move();
// 派生类方法
d.woof(3);
```

##### 重写方法

派生类也可以重写基类的字段或者属性。你可以使用 `super.` 语法访问基类的方法。注意，由于 JavaScript 的类只是一个简单的查找对象，所以不存在“父类字段”的概念。

TypeScript 强制认为派生类总是基类的一个子类。

比如，下面是一个合法的重写方法的例子：

```ts
class Base {
  greet() {
    console.log("Hello, world!");
  }
}
 
class Derived extends Base {
  greet(name?: string) {
    if (name === undefined) {
      super.greet();
    } else {
      console.log(`Hello, ${name.toUpperCase()}`);
    }
  }
}
 
const d = new Derived();
d.greet();
d.greet("reader");
```

很重要的一点是，派生类会遵循基类的约束。通过一个基类引用去引用一个派生类，是很常见（并且总是合法的！）的一种做法：

```ts
// 通过一个基类引用去命名一个派生类实例
const b: Base = d;
// 没有问题
b.greet();
```

如果派生类 `Derived` 没有遵循基类 `Base` 的约束，会怎么样呢？

```ts
class Base {
  greet() {
    console.log("Hello, world!");
  }
}
 
class Derived extends Base {
  // 让这个参数成为必选参数
  greet(name: string) {
    ^  
/*
Property 'greet' in type 'Derived' is not assignable to the same property in base type 'Base'.
  Type '(name: string) => void' is not assignable to type '() => void'.
*/  
    console.log(`Hello, ${name.toUpperCase()}`);
  }
}
```

如果无视错误并编译代码，那么下面的代码执行后会报错：

```ts
const b: Base = new Derived();
// 因为 name 是 undefined，所以报错
b.greet();
```

##### 初始化顺序

JavaScript 类的初始化顺序在某些情况下可能会让你感到意外。我们看看下面的代码：

```ts
class Base {
  name = "base";
  constructor() {
    console.log("My name is " + this.name);
  }
}
 
class Derived extends Base {
  name = "derived";
}
 
// 打印 base 而不是 derived
const d = new Derived();
```

这里发生了什么事呢？

根据 JavaScript 的定义，类初始化的顺序是：

* 初始化基类的字段
* 执行基类的构造器
* 初始化派生类的字段
* 执行派生类的构造器

这意味着，因为基类构造器执行的时候派生类的字段尚未进行初始化，所以基类构造器只能看到自己的 `name` 值。

##### 继承内置类型

> 注意：如果你不打算继承诸如 Array、Error、Map 等内置类型，或者你的编译目标显式设置为 ES6/ES2015 或者更高的版本，那么你可以跳过这部分的内容。

在 ES2015 中，返回实例对象的构造器会隐式地将 `this` 的值替换为 `super(...)` 的任意调用者。有必要让生成的构造器代码捕获 `super(...)` 的任意潜在的返回值，并用 `this` 替换它。

因此，`Error`、`Array` 等的子类可能无法如预期那样生效。这是因为诸如 `Error`、`Array` 这样的构造函数使用了 ES6 的 `new.target` 去调整原型链，但是，在 ES5 中调用构造器函数的时候，没有类似的方法可以确保 `new.target` 的值。默认情况下，其它底层编译器通常也具有相同的限制。

对于一个像下面这样的子类：

```ts
class MsgError extends Error {
  constructor(m: string) {
    super(m);
  }
  sayHello() {
    return "hello " + this.message;
  }
}
```

你可能会发现：

* 调用子类之后返回的实例对象，其方法可能是 `undefined`，所以调用 `sayHello` 将会抛出错误
* 子类实例和子类之间的 `instanceof` 可能被破坏，所以 `(new MsgError()) instanceof MsgError` 将会返回 `false`。

推荐的做法是，在任意的 `super(...)` 调用后面手动地调整原型链：

```ts
class MsgError extends Error {
  constructor(m: string) {
    super(m);
    // 显式设置原型链
    Object.setPrototypeOf(this, MsgError.prototype);
  }
 
  sayHello() {
    return "hello " + this.message;
  }
}
```

不过，`MsgError` 的任意子类也需要手动设置原型。对于不支持 [Object.setPrototypeOf](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf) 的运行时，你可以改用 [`__proto__`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/proto)。

糟糕的是，[这些变通方法在 IE10 或者更旧的版本上无法使用](https://msdn.microsoft.com/en-us/library/s4esdbwz(v=vs.94).aspx)。你可以手动将原型上的方法复制到实例上（比如将 `MsgError.prototype` 的方法复制给 `this`），但原型链本身无法被修复。

### 成员可见性

你可以使用 TypeScript 控制特定的方法或属性是否在类的外面可见。

#### `public`

类成员的默认可见性是公有的（`public`）。公有成员随处可以访问：

```ts
class Greeter {
    public greet(){
        console.log('hi!');
    }
}
const g = new Greeter();
g.greet();
```

由于成员的可见性默认就是公有的，所以你不需要在类成员前面进行显式声明，但出于代码规范或者可读性的考虑，你也可以这么做。

#### `protected`

受保护（`protected`）成员只在类的子类中可见。

```ts
class Greeter {
  public greet() {
    console.log("Hello, " + this.getName());
  }
  protected getName() {
    return "hi";
  }
}
 
class SpecialGreeter extends Greeter {
  public howdy() {
    // 这里可以访问受保护成员
    console.log("Howdy, " + this.getName());
  }
}
const g = new SpecialGreeter();
g.greet(); // OK
g.getName();
	^
// Property 'getName' is protected and only accessible within class 'Greeter' and its subclasses.
```

##### 公开受保护成员

派生类需要遵循其基类的约束，但可以选择公开具有更多功能的基类的子类。这包括了让受保护成员变成公有成员：

```ts
class Base {
    protected m = 10;
}
class Derived extends Base {
    // 没有修饰符，所以默认可见性是公有的
    m = 15;
}
const d = new Dervied();
console.log(d.m);  // OK
```

注意 `Dervied` 已经可以自由读写成员 `m` 了，所以这么写并不会改变这种情况的“安全性”。这里需要注意的要点是，在派生类中，如果我们无意公开其成员，那么需要添加 `protected` 修饰符。

##### 跨层级访问受保护成员

对于通过一个基类引用访问受保护成员是否合法，不同的 OOP 语言之间存在争议：

```ts
class Base {
    protected x: number = 1;
}
class Derived1 extends Base {
    protected x: number = 5;
}
class Derived2 extends Base {
    f1(other: Derived2) {
        other.x = 10;
    }
    f2(other: Base) {
        other.x = 10;
        	  ^
// Property 'x' is protected and only accessible through an instance of class 'Derived2'. This is an instance of class 'Base'.                  
    }
}
```

举个例子，Java 认为上述代码是合法的，但 C# 和 C++ 则认为上述代码是不合法的。

TypeScript 也认为这是不合法的，因为只有在 `Derived2` 的子类中访问 `Derived2` 的 `x` 才是合法的，但 `Derived1` 并不是 `Derived2` 的子类。而且，如果通过 `Derived1` 引用访问 `x` 就已经是不合法的了（这确实应该是不合法的！），那么通过基类引用访问它也同样应该是不合法的。

关于 C# 为什么会认为这段代码是不合法的，可以阅读这篇文章了解更多信息：[为什么我无法在一个派生类中去访问一个受保护成员？](https://blogs.msdn.microsoft.com/ericlippert/2005/11/09/why-cant-i-access-a-protected-member-from-a-derived-class/)

#### `private`

`private` 和 `protected` 一样，但声明了 `private` 的私有成员即使在子类中也无法被访问到：

```ts
class Base {
    private x = 0;
}
const b = new Base();
// 无法在类外面访问
console.log(b.x);
// Property 'x' is private and only accessible within class 'Base'.
class Derived extends Base {
  showX() {
    // 无法在子类中访问
    console.log(this.x);
      			    ^	
// Property 'x' is private and only accessible within class 'Base'.
  }
}
```

 由于私有成员对派生类不可见，所以派生类无法提高其可见性：

```ts
class Base {
    private x = 0;
}
class Dervied extends Base {
/*
Class 'Derived' incorrectly extends base class 'Base'.
  Property 'x' is private in type 'Base' but not in type 'Derived'.    
*/  
    x = 1;
}
```

##### 跨实例访问私有成员

对于同一个类的不同实例互相访问对方的私有成员是否合法，不同的 OOP 语言之间存在争议。Java、C#、C++、Swift 和 PHP 允许这么做，但 Ruby 则认为这样做是不合法的。

TypeScript 允许跨实例访问私有成员：

```ts
class A {
    private x = 10;
    public sameAs(other: A) {
        // 不会报错
        return other.x === this.x;
    }
}
```

##### 注意事项

和 TypeScript 类型系统中的其它东西一样，`private` 和 `protected` [只在类型检查期间生效](https://www.typescriptlang.org/play?removeComments=true&target=99&ts=4.3.4#code/PTAEGMBsEMGddAEQPYHNQBMCmVoCcsEAHPASwDdoAXLUAM1K0gwQFdZSA7dAKWkoDK4MkSoByBAGJQJLAwAeAWABQIUH0HDSoiTLKUaoUggAW+DHorUsAOlABJcQlhUy4KpACeoLJzrI8cCwMGxU1ABVPIiwhESpMZEJQTmR4lxFQaQxWMm4IZABbIlIYKlJkTlDlXHgkNFAAbxVQTIAjfABrAEEC5FZOeIBeUAAGAG5mmSw8WAroSFIqb2GAIjMiIk8VieVJ8Ar01ncAgAoASkaAXxVr3dUwGoQAYWpMHBgCYn1rekZmNg4eUi0Vi2icoBWJCsNBWoA6WE8AHcAiEwmBgTEtDovtDaMZQLM6PEoQZbA5wSk0q5SO4vD4-AEghZoJwLGYEIRwNBoqAzFRwCZCFUIlFMXECdSiAhId8YZgclx0PsiiVqOVOAAaUAFLAsxWgKiC35MFigfC0FKgSAVVDTSyk+W5dB4fplHVVR6gF7xJrKFotEk-HXIRE9PoDUDDcaTAPTWaceaLZYQlmoPBbHYx-KcQ7HPDnK43FQqfY5+IMDDISPJLCIuqoc47UsuUCofAME3Vzi1r3URvF5QV5A2STtPDdXqunZDgDaYlHnTDrrEAF0dm28B3mDZg6HJwN1+2-hg57ulwNV2NQGoZbjYfNrYiENBwEFaojFiZQK08C-4fFKTVCozWfTgfFgLkeT5AUqiAA)。

这意味着 JavaScript 运行时的一些操作，诸如 `in` 或者简单的属性查找仍然可以访问私有成员或者受保护成员：

```ts
class MySafe {
    private serectKey = 123345;
}
// 在 JavaScript 文件中会打印 12345
const s = new MySafe();
console.log(s.secretKey);
```

而即使是在类型检查期间，我们也可以通过方括号语法去访问私有成员。因此，在进行诸如单元测试这样的操作时，访问私有字段会比较容易，但缺点就是这些字段是“弱私有的”，无法保证严格意义上的私有性。

```ts
class MySafe {
  private secretKey = 12345;
}
 
const s = new MySafe();
 
// 在类型检查期间，不允许这样访问私有成员
console.log(s.secretKey);
				^
// Property 'secretKey' is private and only accessible within class 'MySafe'.
 
// 但是可以通过方括号语法访问
console.log(s["secretKey"]);
```

和 TypeScript 用 `private` 声明的私有成员不同，JavaScript 用 `#` 声明的[私有字段](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields)在编译之后也仍然是私有的，并且没有提供像上面那样的方括号语法用于访问私有成员，所以 JavaScript 的私有成员是“强私有的”。

```ts
class Dog {
    #barkAmount = 0;
    personality = 'happy';
    
    constructor() {}
}
```

以下面这段 TypeScript 代码为例：

```ts
"use strict";
class Dog {
    #barkAmount = 0;
    personality = "happy";
    constructor() { }
}
 
```

把它编译为 ES2021 或者更低版本的代码之后，TypeScript 会使用 WeakMap 代替 `#`。

```ts
"use strict";
var _Dog_barkAmount;
class Dog {
    constructor() {
        _Dog_barkAmount.set(this, 0);
        this.personality = "happy";
    }
}
_Dog_barkAmount = new WeakMap();
```

如果你需要保护类中的值不被恶意修改，那么你应该使用提供了运行时私有性保障的机制，比如闭包、WeakMap 或者私有字段等。注意，这些在运行时添加的私有性检查可能会影响性能。

### 静态成员

> 背景导读：[静态成员（MDN）](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static)

类可以拥有静态（`static`）成员。这些成员和类的特定实例无关，我们可以通过类构造器对象本身访问到它们：

```ts
class MyClass {
    static x = 0;
    static printX(){
        console.log(MyClass.x);
    }
}
console.log(MyClass.x);
MyClass.printX();
```

静态成员也可以使用 `public`、`protected` 和 `private` 等可见性修饰符：

```ts
class MyClass {
    private static x = 0;
}
console.log(MyClass.x);
				  ^
// Property 'x' is private and only accessible within class 'MyClass'.           
```

静态成员也可以被继承：

```ts
class Base {
  static getGreeting() {
    return "Hello world";
  }
}
class Derived extends Base {
  myGreeting = Derived.getGreeting();
}
```

#### 特殊的静态成员名字

重写 `Function` 原型的属性通常是不安全/不可能的。因为类本身也是一个可以通过 `new` 调用的函数，所以无法使用一些特定的静态成员名字。诸如 `name`、`length` 和 `call` 这样的函数属性无法作为静态成员的名字：

```ts
class S {
    static name = 'S!';
    		^
// Static property 'name' conflicts with built-in property 'Function.name' of constructor function 'S'.                
}
```

#### 为什么没有静态类？

TypeScript（和 JavaScript）并没有像 C# 和 Java 那样提供静态类这种结构。

C# 和 Java 之所以需要静态类，是因为这些语言要求所有的数据和函数必须放在一个类中。因为在 TypeScirpt 中不存在这个限制，所以也就不需要静态类。只拥有单个实例的类在 JavaScript/TypeScirpt 中通常用一个普通对象表示。

举个例子，在 TypeScript 中我们不需要“静态类”语法，因为一个常规的对象（甚至是顶层函数）也可以完成相同的工作：

```ts
// 不必要的静态类
class MyStaticClass {
    static doSomething() {}
}
// 首选（方案一）
function doSomething() {}

// 首选（方案二）
const MyHelperObject = {
  dosomething() {},
};
```

### 类中的静态块

静态块允许你编写一系列声明语句，它们拥有自己的作用域，并且可以访问包含类中的私有字段。这意味着我们能够编写初始化代码，这些代码包含了声明语句，不会有变量泄漏的问题，并且完全可以访问类的内部。

```ts
class Foo {
    static #count = 0;
	get count(){
        return Foo.#count;
    }
	static {
        try {
            const lastInstances = loadLastInstances();
            Foo.#count += lastInstances.length;
        }
        catch {}
    }
}
```

### 泛型类

类和接口一样，也可以使用泛型。当用 `new` 实例化一个泛型类的时候，它的类型参数就像在函数调用中那样被推断出来：

```ts
class Box<Type> {
    contents: Type;
    constructor(value: Type){
        this.contents = value;
    }
}
const b = new Box('hello!');
	  ^	
    // const b: Box<string>      
```

类可以像接口那样使用泛型约束和默认值。

#### 静态成员中的类型参数

下面的代码是不合法的，但原因可能不那么明显：

```ts
class Box<Type> {
    static defaultValue: Type;
    					^
//  Static members cannot reference class type parameters.                       
}
```

记住，类型在编译后总是会被完全抹除的！在运行时，只有一个 `Box.defaultValue` 属性插槽。这意味着设置 `Box<string>.defaultValue`（如果可以设置的话）也会改变 `Box<number>.defaultValue` —— 这是不行的。泛型类的静态成员永远都不能引用类的类型参数。

### 类的运行时 `this`

有个要点需要记住，那就是 TypeScript 不会改变 JavaScript 的运行时行为。而众所周知，JavaScript 拥有一些特殊的运行时行为。

JavaScript 对于 `this` 的处理确实是很不寻常：

```ts
class MyClass {
  name = "MyClass";
  getName() {
    return this.name;
  }
}
const c = new MyClass();
const obj = {
  name: "obj",
  getName: c.getName,
};
 
// 打印 "obj" 而不是 "MyClass"
console.log(obj.getName());
```

长话短说，默认情况下，函数中 `this` 的值取决于**函数是如何被调用的**。在这个例子中，由于我们通过 `obj` 引用去调用函数，所以它的 `this` 的值是 `obj`，而不是类实例。

这通常不是我们期望的结果！TypeScript 提供了一些方法让我们可以减少或者防止这种错误的发生。

#### 箭头函数

如果你的函数在被调用的时候经常会丢失 `this` 上下文，那么最好使用箭头函数属性，而不是方法定义：

```ts
class MyClass {
    name = 'MyClass';
    getName = () => {
        return this.name;
    };
}
const c = new MyClass();
const g = c.getName;
// 打印 MyClass 
console.log(g());
```

这种做法有一些利弊权衡：

* 在运行时可以保证 `this` 的值是正确的，即使对于那些没有使用 TypeScript 进行检查的代码也是如此
* 这样会占用更多内存，因为以这种方式定义的函数，会导致每个类实例都有一份函数副本
* 你无法在派生类中使用 `super.getName`，因为在原型链上没有入口可以去获取基类的方法

#### `this` 参数

在 TypeScript 的方法或者函数定义中，第一个参数的名字如果是 `this`，那么它有特殊的含义。这样的参数在编译期间会被抹除：

```ts
// TypeScript 接受 this 参数
function fn(this: SomeType, x: number) {
    /* ... */
}
// 输出得 JavaScript 
function fn(x) {
    /* ... */
}
```

TypeScript 会检查传入 `this` 参数的函数调用是否位于正确的上下文中。这里我们没有使用箭头函数，而是给方法定义添加了一个 `this` 参数，以静态的方式确保方法可以被正确调用：

```ts
class MyClass {
  name = "MyClass";
  getName(this: MyClass) {
    return this.name;
  }
}
const c = new MyClass();
// OK
c.getName();
 
// 报错
const g = c.getName;
console.log(g());
// The 'this' context of type 'void' is not assignable to method's 'this' of type 'MyClass'.
```

这种方法的利弊权衡和上面使用箭头函数的方法相反：

* JavaScript 的调用方可能仍然会在没有意识的情况下错误地调用类方法
* 只会给每个类定义分配一个函数，而不是给每个类实例分配一个函数
* 仍然可以通过 `super` 调用基类定义的方法

### `this` 类型

在类中，名为 `this` 的特殊类型可以动态地引用当前类的类型。我们看一下它是怎么发挥作用的：

```ts
class Box {
    contents: string = "";
    set(value: string){
     ^
    // (method) Box.set(value: string): this
         this.contents = value;
        return this;
    }
}
```

这里，TypeScript 将 `set` 的返回值类型推断为 `this`，而不是 `Box`。现在我们来创建一个 `Box` 的子类：

```ts
class ClearableBox extends Box {
    clear() {
        this.contents = "";
    }
}
const a = new ClearableBox();
const b = a.set("hello");
      ^
// const b: ClearableBox
```

你也可以在参数的类型注解中使用 `this`：

```ts
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}
```

这和使用 `other: Box` 是不一样的 —— 如果你有一个派生类，那么它的 `sameAs` 方法将只会接受该派生类的其它实例：

```ts
class Box {
  content: string = "";
  sameAs(other: this) {
    return other.content === this.content;
  }
}
 
class DerivedBox extends Box {
  otherContent: string = "?";
}
 
const base = new Box();
const derived = new DerivedBox();
derived.sameAs(base);
			    ^
/*
Argument of type 'Box' is not assignable to parameter of type 'DerivedBox'.
  Property 'otherContent' is missing in type 'Box' but required in type 'DerivedBox'.
*/  
```

#### 基于 `this` 的类型保护

你可以在类和接口的方法的返回值类型注解处使用 `this is Type`。该语句和类型收缩（比如说 `if` 语句）一起使用的时候，目标对象的类型会被收缩为指定的 `Type`。

```ts
class FileSystemObject {
  isFile(): this is FileRep {
    return this instanceof FileRep;
  }
  isDirectory(): this is Directory {
    return this instanceof Directory;
  }
  isNetworked(): this is Networked & this {
    return this.networked;
  }
  constructor(public path: string, private networked: boolean) {}
}
 
class FileRep extends FileSystemObject {
  constructor(path: string, public content: string) {
    super(path, false);
  }
}
 
class Directory extends FileSystemObject {
  children: FileSystemObject[];
}
 
interface Networked {
  host: string;
}
 
const fso: FileSystemObject = new FileRep("foo/bar.txt", "foo");
 
if (fso.isFile()) {
  fso.content;
   ^
 // const fso: FileRep
} else if (fso.isDirectory()) {
  fso.children;
   ^ 
 // const fso: Directory
} else if (fso.isNetworked()) {
  fso.host;
   ^ 
 // const fso: Networked & FileSystemObject
}
```

基于 `this` 的类型保护的常见用例是允许特定字段的延迟验证。以下面的代码为例，当 `hasValue` 被验证为 true 的时候，可以移除 `Box` 中为 `undefined` 的 `value` 值：

```ts
class Box<T> {
  value?: T;
 
  hasValue(): this is { value: T } {
    return this.value !== undefined;
  }
}
 
const box = new Box();
box.value = "Gameboy";
 
box.value;
      ^
    // (property) Box<unknown>.value?: unknown
 
if (box.hasValue()) {
  box.value;
        ^   
   // (property) value: unknown
}
```

### 参数属性

TypeScript 提供了一种特殊的语法，可以将构造器参数转化为具有相同名字和值的类属性。这种语法叫做参数属性，实现方式是在构造器参数前面加上 `public`、`private`、`protected` 或者 `readonly` 等其中一种可见性修饰符作为前缀。最终的字段将会获得这些修饰符：

```ts
class Params {
    constructor(
        public readonly x: number,
        protected y: number,
        private z: number    
    ) {
        // 没有必要编写构造器的函数体     
    }    
}
const a = new Params(1,2,3);
console.log(a.x);
			 ^
            // (property) Params.x: number
console.log(a.z);
			 ^
// Property 'z' is private and only accessible within class 'Params'.           
```

### 类表达式

> 背景导读：[类表达式（MDN）](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/class)

类表达式和类声明非常相似。唯一的不同在于，类表达式不需要名字，但我们仍然可以通过任意绑定给类表达式的标识符去引用它们：

```ts
const someClass = class<Type> {
    content: Type;
    constructor(value: Type) {
        this.content = value;
    }
};

const m = new someClass("Hello, world");
	  ^
    // const m: someClass<string>      
```

### 抽象类和成员

在 TypeScript 中，类、方法和字段可能是抽象的。

抽象方法或者抽象字段在类中没有对应的实现。这些成员必须存在于一个无法直接被实例化的抽象类中。

抽象类的角色是充当一个基类，让其子类去实现所有的抽象成员。当一个类没有任何抽象成员的时候，我们就说它是具体的。

来看一个例子：

```ts
abstract class Base {
    abstract getName(): string;
    printName(){
        console.log("Hello, " + this.getName());
    }
}

const b = new Base();
// Cannot create an instance of an abstract class.
```

因为 `Base` 是一个抽象类，所以我们不能使用 `new` 去实例化它。相反地，我们需要创建一个派生类，让它去实现抽象成员：

```ts
class Derived extends Base {
    getName() {
        rteurn "world";
    }
}

const d = new Derived();
d.printName();
```

注意，如果我们忘记实现基类的抽象成员，那么会抛出一个错误：

```ts
class Derived extends Base {
    	^
// Non-abstract class 'Derived' does not implement inherited abstract member 'getName' from class 'Base'.
  // 忘记实现抽象成员
}
```

#### 抽象构造签名

有时候你想要接受一个类构造器函数作为参数，让它产生某个类的实例，并且这个类是从某个抽象类派生过来的。

举个例子，你可能想要编写下面这样的代码：

```ts
function greet(ctor: typeof Base) {
  const instance = new ctor();
// Cannot create an instance of an abstract class.
  instance.printName();
}
```

TypeScript 会正确地告诉你，你正试图实例化一个抽象类。毕竟，根据 `greet` 的定义，编写这样的代码理应是完全合法的，它最终会构造一个抽象类的实例：

```ts
// 不行！
greet(Base);
```

但它实际上会报错。所以，你编写的函数所接受的参数应该带有一个构造签名：

```ts
function greet(ctor: new () => Base) {
  const instance = new ctor();
  instance.printName();
}
greet(Derived);
greet(Base);
	   ^	
/*
Argument of type 'typeof Base' is not assignable to parameter of type 'new () => Base'.
  Cannot assign an abstract constructor type to a non-abstract constructor type.
*/  
```

现在 TypeScript 可以正确地告知你哪个类构造器函数可以被调用了 —— `Derived` 可以被调用，因为它是一个具体类，而 `Base` 不能被调用，因为它是一个抽象类。

### 类之间的联系

在大多数情况下，TypeScript 中的类是在结构上进行比较的，就跟其它类型一样。

举个例子，下面这两个类可以互相替代对方，因为它们在结构上是一模一样的：

```ts
class Point1 {
  x = 0;
  y = 0;
}
 
class Point2 {
  x = 0;
  y = 0;
}
 
// OK
const p: Point1 = new Point2();
```

类似地，即使没有显式声明继承关系，类和类之间也可以存在子类联系：

```ts
class Person {
  name: string;
  age: number;
}
 
class Employee {
  name: string;
  age: number;
  salary: number;
}
 
// OK
const p: Person = new Employee();
```

这听起来很简单易懂，但还有一些情况会比较奇怪。

空类没有成员。在一个结构化的类型系统中，一个没有成员的类型通常是任何其它类型的超类。所以如果你编写了一个空类（不要这么做！），那么你可以用任何类型去替代它：

```ts
class Empty {}

function fn(x: Empty) {
    // 无法对 x 执行任何操作，所以不建议这么写
}

// 这些参数都是可以传入的！
fn(window);
fn({});
fn(fn);
```
