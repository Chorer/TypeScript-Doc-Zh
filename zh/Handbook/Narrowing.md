## 类型收缩

假设现在有一个叫做 `padLeft` 的函数：

```ts
function padLeft(padding: number | string, input: string): string {
    trjow new Error('Not implemented yet!')
}
```

如果 `padding` 是 `number` 类型，那么它将作为 `input` 前缀空格的个数，如果它是 `string` 类型，那么它将直接作为 `input` 的前缀。现在我们尝试实现一下相关的逻辑，假定要给 `padLeft` 传入 `number` 类型的 `padding` 参数。

```ts
function padLeft(padding: number | string, input: string) {
  return " ".repeat(padding) + input;
// Argument of type 'string | number' is not assignable to parameter of type 'number'.
//  Type 'string' is not assignable to type 'number'.
}
```

啊这，传入 `padding` 参数的时候报错了。TypeScript 警告我们，将 `number` 添加给 `number | string` 可能会得到期望之外的结果，事实上也的确如此。换句话说，我们没有在一开始显式检查 `padding` 是否是一个 `number`，同时我们也没有处理它是 `string` 的情况。所以我们来改进一下代码吧。

```ts
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

如果你觉得这看起来和无趣的 JavaScript 代码一样，那你可说到点上了。除了我们添加的类型注解之外，这些 TypeScript 代码看起来确实很像 JavaScript。这里的重点在于， TypeScript 的类型系统旨在让开发者尽可能轻松地编写常规的 JavaScript 代码，而不必为了获得类型安全而费尽心思。

虽然看起来可能不多，但实际上这个过程藏着很多秘密。就像 TypeScript 如何使用静态类型分析运行时的值一样，它将类型分析覆盖在类似于 `if/else` 这样的 JavaScript 运行时控制流结构上，同时还包括了三元表达式、循环、真值检查等，这些都能对类型产生影响。

在 `if` 条件检查语句中，TypeScript 发现了 `typeof padding === "number"`，并将其视为一种称之为“类型保护”的特殊代码结构。TypeScript 遵循我们的程序可能到达的执行路径，并在给定的位置分析某个值可能取到的最具体类型。它会查看这些特殊的检查语句（也就是“类型保护”）和赋值语句，并将声明的类型精炼为更具体的类型，这就是所谓的“类型收缩”。在很多编辑器中，我们可以观察到这些类型的变化。

```ts
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
      				^^^^^^^^	                        
				   // (parameter) padding: number
  }
  return padding + input;
         ^^^^^^^  
	    // (parameter) padding: string
}
```

TypeScript 可以理解几种不同的用于收缩类型的构造。

### `typeof` 类型保护

正如我们所看到的，JavaScript 支持的 `typeof` 运算符可以给出关于运行时值的类型的基本信息。同样的，TypeScript 期望该运算符可以返回如下确定的字符串：

- `"string"`
- `"number"`
- `"bigint"`
- `"boolean"`
- `"symbol"`
- `"undefined"`
- `"object"`
- `"function"`

就像我们在 `padLeft` 中看到的，这个运算符经常出现在大量的 JavaScript 库中，而 TypeScript 也能理解这个运算符，从而在不同的分支中收缩类型。

在 TypeScript 中，检查 `typeof` 的返回值就是一种类型保护的方式。因为 TypeScript 可以编码 `typeof` 对不同值的操作方式，所以它也知道这个运算符在 JavaScript 中的一些怪异表现。举个例子，注意看上面的列表，`typeof` 没有返回字符串 `"null"`。再看下面的例子：

```ts
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) {
         		   ^^^^	
		// Object is possibly 'null'.
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // do nothing
  }
}
```

在 `printAll` 函数中，我们试图检查 `strs` 是否是一个对象，从而判断它是不是数组类型（在 JavaScript 中，数组也属于对象类型）。但在 JavaScript 中，`typeof null` 实际上会返回 `"object"`！这是历史遗留 bug 中的其中一个。

有充足经验的开发者可能不会感到很惊讶，但并不是每一个人都曾在 JavaScript 中遇到这个问题。幸运的是，TypeScript 让我们知道 `strs` 只是收缩到 `string[] | null` 类型而不是 `string[]` 类型。

这可能是讲解“真值”检查的一个不错的引子。

### 真值收缩

Truthiness 这个词可能在词典中找不到，但你一定在 JavaScript 中听过这个东西。

在 JavaScript 中，我们可以在条件语句中使用任意的表达式，比如 `&&`、`||`、`if` 语句、布尔值取反（`!`）等。举个例子，`if` 语句并没有要求它的条件一定是 `boolean` 类型。

```ts
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`;
  }
  return "Nobody's here. :(";
}
```

在 JavaScript 中，类似 `if` 这样的结构会首先将条件“强制转化为”一个 `boolean` 类型的值，从而确保接受的参数是合理的，之后基于结果是 `true` 还是 `false`，会选择对应的分支。

类似下面这样的值经过转化后都会成为 `false`：

- `0`
- `NaN`
- `""` （空字符串）
- `0n` (0 的 `bigint` 版本)
- `null`
- `undefined`

除此之外的其他值经过转化后都会成为 `true`。你总可以通过调用 `Boolean` 函数将值转化为 `boolean` 类型，或者使用更加简短的 `!!`。（后者的优势在于，TypeScript 可以将其推断为一个更具体的字面量布尔值类型 `true`，而前者只能被推断为 `boolean`）

```ts
// 下面的结果都是 true
Boolean("hello"); // type: boolean, value: true
!!"world"; // type: true,    value: true
```

在编码中经常会用到这个特性，尤其多用于防止出现像 `null` 或者 `undefined` 这样的值。举个例子，我们尝试在 `printAll` 函数中使用：

```ts
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === "object") {
    for (const s of strs) {
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  }
}
```

可以看到，通过检查 `strs` 是否是真值，我们成功摆脱了之前出现的报错。这至少可以防止出现像下面这样令人害怕的错误：

```
TypeError: null is not iterable
```

但是请记住，对原始类型的真值检查常常容易出错。举个例子，我们尝试像下面这样改写 `printAll`：

```ts
function printAll(strs: string | string[] | null) {
  // !!!!!!!!!!!!!!!!
  //  不要这样写！
  // !!!!!!!!!!!!!!!!
  if (strs) {
    if (typeof strs === "object") {
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
    }
  }
}
```

我们将整个函数体包裹在一个真值检查中，但这样做其实有一个潜在的问题：我们可能再也无法正确地处理空字符串的情况。

TypeScript 在这里并不会给出报错提示，但如果你不熟悉 JavaScript 的话，这是一个值得关注的事情。TypeScript 总是能够帮助你提前捕获 bug，但如果你选择对某个值不做任何处理，那么在确保不过度约束的前提下，TypeScript 能做的也就只有这么多了。如果你需要的话，可以用一个 linter 确保自己正确处理了类似这样的情况。

关于真值收缩，最后一点要说明的是，布尔值取反 `!` 可以筛选出否定分支：

```ts
function multiplyAll(
  values: number[] | undefined,
  factor: number
): number[] | undefined {
  if (!values) {
    return values;
  } else {
    return values.map((x) => x * factor);
  }
}
```

### 相等性收缩

TypeScript 也使用 `switch` 语句和诸如 `===`、`!==`、`==` 和 `!=` 这样的相等性检查来收缩类型。举个例子：

```ts
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // We can now call any 'string' method on 'x' or 'y'.
    x.toUpperCase();
      ^^^^^^^^^^^^^    
    //(method) String.toUpperCase(): string
    y.toLowerCase();
      ^^^^^^^^^^^^^^      
    //(method) String.toLowerCase(): string
  } else {
    console.log(x);
    		   ^^	           
               //(parameter) x: string | number
    console.log(y);
                ^^
               //(parameter) y: string | boolean
  }
}
```

在上述例子中，当我们通过检查得知 `x` 和 `y` 是相等的时候，TypeScript 知道它们的类型也必须是相等的。由于 `string` 是 `x` 和 `y` 共有的类型，所以 TypeScript 知道 `x` 和 `y` 在第一个逻辑分支中肯定都是 `string` 类型。

同样的，我们也可以检查特定的字面量值（和变量相对）。在前面讲解真值收缩的例子中，我们编写的 `printAll` 函数存在潜在的错误，因为它没有适当地处理空字符串的情况。不妨换一种思路，我们通过一个特定的检查排除 `null`，这样 TypeScript 也仍然可以将 `null` 从 `strs` 的类型中正确地移除。

```ts
function printAll(strs: string | string[] | null) {
  if (strs !== null) {
    if (typeof strs === "object") {
      for (const s of strs) {
                      ^^^^
                     // (parameter) strs: string[]
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
                  ^^^^ 
                 // (parameter) strs: string
    }
  }
}
```

JavaScript 的松散相等性检查 `==` 和 `!=` 同样也可以正确地收缩类型。可能你还不太熟悉，检查某个值是否`== null` 的时候，不仅仅是在检查这个值是否确切地等于 `null`，也是在检查这个值是否是潜在的 `undefined`。对于 `== undefined` 也同理：它会检查这个值是否等于 `null` 或者 `undefined`。

```ts
interface Container {
  value: number | null | undefined;
}
 
function multiplyValue(container: Container, factor: number) {
  // 这个检查可以同时移除 null 和 undefined
  if (container.value != null) {
    console.log(container.value);
                          ^^^^^^   
                        // (property) Container.value: number
 
    // 现在可以安全地进行计算
    container.value *= factor;
  }
}
```

### `in` 操作符收缩

JavaScript 的 `in` 操作符可以判断对象是否有某个属性。TypeScript 将其视为一种收缩潜在类型的方式。

举个例子，假定有代码 `"value" in x`，`"value"` 是一个字符串字面量，`x` 是一个联合类型。那么结果为 `true` 的分支会将 `x` 收缩为具有可选属性或必需属性 `value` 的类型，而结果为 `false` 的分支则会将 `x` 收缩为具有可选属性或缺失属性 `value` 的类型。

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };
 
function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }
 
  return animal.fly();
}
```

再次重申，可选的属性在收缩时会同时出现在两个分支中。举个例子，人类既能游泳也能飞（我指的是通过交通工具），因此在 `in` 检查中，这个类型会同时出现在两个分支中：

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { swim?: () => void; fly?: () => void };
 
function move(animal: Fish | Bird | Human) {
  if ("swim" in animal) {
    animal;
    ^^^^^^  
   // (parameter) animal: Fish | Human
  } else {
    animal;
    ^^^^^^   
   // (parameter) animal: Bird | Human
  }
}
```

### `instanceof` 收缩

JavaScript 有一个操作符可以检查某个值是否是另一个值的实例。更具体地说，在 JavaScript 中，`x instanceof Foo` 可以检查 `x` 的原型链上是否包含 `Foo.prototype`。虽然我们在这里不会深入探讨，而且后续讲解类的时候会涉及更多这方面的内容，但它对大多数可以由 `new` 构造的值来说仍然很有用。你可能已经猜到了，`instanceof` 也是类型保护的一种方式，TypeScript 可以在由 `instanceof` 保护的分支里收缩类型。

```ts
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString());
                ^
              // (parameter) x: Date
  } else {
    console.log(x.toUpperCase());
                ^
              // (parameter) x: string
  }
}
```

### 赋值

正如我们先前提到的，当我们给任意变量赋值的时候，TypeScript 会查看赋值语句右部，对左部的变量类型进行合适的收缩。

```ts
let x = Math.random() < 0.5 ? 10 : "hello world!";
 	^  
   // let x: string | number
x = 1;
 
console.log(x);
            ^     
          // let x: number
x = "goodbye!";
 
console.log(x);
            ^ 
          // let x: string
```

注意这些赋值语句都是有效的。虽然在第一次赋值之后，`x` 的可观察类型变成了 `number`，但我们仍然可以给它赋值 `string` 类型的值。这是因为 `x` 的声明类型 —— 也就是 `x` 的初始类型，是 `string | number`，而可赋值性总是会基于声明类型进行检查。

如果我们赋值给 `x` 一个 `boolean` 类型的值，那么就会抛出一个错误，因为在声明类型中并不存在 `boolean` 类型。

```ts
let x = Math.random() < 0.5 ? 10 : "hello world!";
    ^
  // let x: string | number
x = 1;
 
console.log(x);
 		   ^	          
          // let x: number
x = true;
^
// Type 'boolean' is not assignable to type 'string | number'.
 
console.log(x);
            ^        
           // let x: string | number
```

### 控制流分析

到目前为止，我们已经通过一些基本的例子讲解了 TypeScript 是如何在具体的分支中收缩类型的。但除了分析每个变量，在 `if`、`while` 等条件语句中查找类型保护之外，TypeScript 还做了不少其他工作。

```ts
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

`padLeft` 在第一个 `if` 块中返回。TypeScript 可以对这段代码进行分析，并发现函数体的剩余部分（`return padding + input;`）在 `padding` 为 `number` 的时候是不可达的。最后，针对函数体的剩余部分，它可以将 `number` 从 `padding` 的类型中移除（也就是将类型 `string | number` 收缩为 `string`）。

这种基于可达性的代码分析称为“控制流分析”。在遇到类型保护和赋值语句的时候，TypeScript 会使用这种流分析去收缩类型。当分析一个变量的时候，控制流可以不断被拆开与重新合并，而我们也可以观察到变量在每个节点有不同的类型。

```ts
function example() {
  let x: string | number | boolean;
 
  x = Math.random() < 0.5;
 
  console.log(x);
              ^
             // let x: boolean
 
  if (Math.random() < 0.5) {
    x = "hello";
    console.log(x);
                ^
               // let x: string
  } else {
    x = 100;
    console.log(x);
                ^ 
              // let x: number
  }
 
  return x;
         ^
        // let x: string | number
}
```

### 使用类型谓词

目前为止，我们都是使用现成的 JavaScript 结构去处理类型收缩，但有时候，你可能想要更加直接地去控制类型在代码中的变化。

要实现一个用户自定义的类型保护，我们只需要定义一个返回类型谓词的函数即可：

```ts
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

在本例中，`pet is Fish` 就是一个类型谓词。类型谓词的形式是 `paramenterName is Type`，`parameterName` 必须是当前函数签名的参数名。

任何时候，只要给 `isFish` 传递参数并调用它，TypeScript 就会在该类型兼容初始类型的时候，将变量类型收缩为该具体的类型。

```ts
// 对 swim 和 fly 的调用都是可以的
let pet = getSmallPet();
 
if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```

注意，TypeScript 不仅知道在 `if` 分支中 `pet` 是 `Fish`，也知道在 `else` 分支中其对应的类型，因为不是 `Fish` 那就肯定是 `Bird` 了。

你也可以使用 `isFish` 这个类型保护从一个 `Fish | Bird` 类型的数组中筛选出一个仅包含 `Fish` 类型的数组：

```ts
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()];
const underWater1: Fish[] = zoo.filter(isFish);
// 或者使用
const underWater2: Fish[] = zoo.filter(isFish) as Fish[];
 
// 在更复杂的例子中，可能需要重复类型谓词
const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
  if (pet.name === "sharkey") return false;
  return isFish(pet);
});
```

此外，类可以使用 [this is Type](https://www.typescriptlang.org/docs/handbook/2/classes.html#this-based-type-guards) 去收缩类型。

### 可辨识联合类型

目前为止，我们看到的大多数例子都是将单个变量收缩为简单类型，诸如 `string`、`boolean` 和 `number` 等。虽然这很常见，但在 JavaScript 中，我们很多时候需要处理稍微复杂一些的结构。

假设我们现在需要编码表示圆形和正方形的形状，圆形需要用到半径，正方形需要用到边长。我们会使用 `kind` 域表明当前正在处理的形状。以下是第一种定义 `Shape` 的方式：

```ts
interface Shape {
    kind: "circle" | "square";
    radius?: number;
    sideLength?: number;
}
```

注意我们这里使用了字符串字面量类型的联合： `"circle"` 和 `"square"`。它可以告诉我们当前正在处理的形状是圆形还是正方形。通过使用 `"circle" | "square"` 而不是 `string`，我们可以避免拼写错误。

```ts
function handleShape(shape: Shape) {
  // oops!
  if (shape.kind === "rect") {
// 该条件始终返回 false，因为类型 "circle" | "square" 和类型 "rect" 不存在重叠。
    // ...
  }
}
```

我们可以编写一个 `getArea` 函数，它可以基于当前处理的形状的类型使用对应的逻辑。首先我们来处理一下圆形：

```ts
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2;
// 对象可能是 'undefined'
}
```

在启用 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 的情况下会抛出一个错误 —— 这是合理的，毕竟 `radius` 可能没有定义。但如果我们对 `kind` 属性进行合理的检查呢？

```ts
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
// 对象可能是 'undefined'
  }
}
```

emm，TypeScript 仍然无从下手。我们这里刚好遇到了一个场景，那就是我们掌握的关于这个值的信息比类型检查器要多。因此，这里可以使用一个非空值断言（给 `shape.radius` 添加后缀 `!`）表明 `radius` 一定是存在的。

```ts
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius! ** 2;
  }
}
```

但这种处理方式似乎不是很理想。我们不得不给类型检查器添加大量的非空值断言（`!`），让它确信 `shape.radius` 已经被定义好了，但如果把代码移除，这些断言就很容易造成错误。此外，在禁用 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 的情况下，我们可能会意外地访问到其它域（毕竟读取可选属性的时候，TypeScript 会假定它们是存在的）。总而言之，应当有更好的处理方式。

`Shape` 的编码方式的问题在于，类型检查器完全无法基于 `kind` 属性去判断 `radius` 和 `sideLength` 是否存在。我们必须把自己知道的信息传达给类型检查器。知晓这一点之后，让我们再次定义 `Shape`。

```ts
interface Circle {
  kind: "circle";
  radius: number;
}
 
interface Square {
  kind: "square";
  sideLength: number;
}
 
type Shape = Circle | Square;
```

这里，我们将 `Shape` 适当地划分为两个类型，它们有不同的 `kind` 属性值，但 `radius` 和 `sideLength` 在对应的类型中成为了必需属性。

我们来看下试图访问 `Shape` 的 `radius` 会发生什么事：

```ts
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2;
                         ^^^^^^
// Property 'radius' does not exist on type 'Shape'.
  // Property 'radius' does not exist on type 'Square'.
}
```

就像之前第一次定义 `Shape` 的时候一样，仍然抛出了一个错误。之前，当 `radius` 是可选属性的时候，我们看到了一个报错（仅在启用 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 的情况下），因为 TypeScript 无从得知这个属性是否真的存在。而现在 `Shape` 已经是一个联合类型了，TypeScript 告诉我们 `shape` 可能是 `Square`，而 `Square` 是没有定义 `radius` 属性的！两种解释都是合理的，但只有后者会在禁用 [strictNullChecks](https://www.typescriptlang.org/tsconfig#strictNullChecks) 的情况下仍然抛出一个错误。

那么，如果这时候我们再次检查 `kind` 属性会怎么样呢？

```ts
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
                     ^^^^^  
                   // (parameter) shape: Circle
  }
}
```

代码不再报错了！当联合类型中的每个类型都包含一个字面量类型的公共属性的时候，TypeScript 会将其视为一个可辨识的联合类型，并通过收缩确认类型为联合类型的某个成员。

在本例中，`kind` 就是那个公共属性（也就是 `Shape` 的一个可辨识属性）。通过检查 `kind` 属性是否为 `"circle"`，我们可以排除掉 `Shape` 中所有 `kind` 属性值不为 `"circle"` 的类型。也就是说，可以将 `shape` 类型收缩为 `Circle` 类型。

同理，这种检查也可以用于 `switch` 语句中。现在我们可以编写一个完整的 `getArea` 函数了，而且它没有任何麻烦的 `!` 非空值断言符号。

```ts
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
                       ^^^^^ 
                      // (parameter) shape: Circle
    case "square":
      return shape.sideLength ** 2;
             ^^^^^ 
            // (parameter) shape: Square
  }
}
```

这个例子的重点在于 `Shape` 的编码。将重要的信息传达给 TypeScript 非常重要，我们得告诉它，`Circle` 和 `Square` 是两种不同的类型，有各自的 `kind` 属性值。这样我们就可以编写类型安全的 TypeScript 代码，它看起来与我们编写的 JavaScript 没有什么不同。知道了这一点之后，类型系统也可以做“正确的”处理，在 `switch` 的每个分支中弄清具体的类型。

> 顺便一提，你可以尝试编写上面的示例并删除一些返回关键字。你将看到，在 switch 语句中意外遇到不同子句时，类型检查可以有效避免 bug 的出现

可辨识联合类型的用处非常大，不仅仅是用在本例的圆形和正方形中。它们还适用于表示 JavaScript 中任意类型的消息传递方案，比如在网络上发送消息（客户端/服务端通信）或在状态管理框架中的 mutation 进行编码等。

### `never` 类型

在收缩类型的时候，你可以将联合类型减少到一个仅存的类型，这时候，你基本上已经排除了所有的可能性，并且没有剩余的类型可选了。此时，TypeScript 会使用 `never` 类型去表示一个不应该存在的状态。

### 穷举检查

`never` 类型可以赋值给任意一个类型，但是，除了 `never` 本身，没有任意一个类型可以赋值给 `never`。这意味着你可以使用类型收缩和 `never` 在一个 `swicth` 语句块中进行穷举检查。

举个例子，在 `getArea` 函数的 `default` 分支中，我们可以把 `shape` 赋值给 `never` 类型的值。这样，当任意一个可能的情况没有在前面的分支得到处理的时候，在这个分支中就必然会抛出错误。

```ts
type Shape = Circle | Square;
 
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

给 `Shape` 联合类型添加一个新成员，将会导致 TypeScript 抛出一个错误：

```ts
interface Triangle {
  kind: "triangle";
  sideLength: number;
}
 
type Shape = Circle | Square | Triangle;
 
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
		   ^^^^^^^^^^^^^^^^	
          //Type 'Triangle' is not assignable to type 'never'.
      return _exhaustiveCheck;
  }
}
```