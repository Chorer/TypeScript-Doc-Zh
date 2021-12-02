## 条件类型

在大多数应用的核心中，我们需要基于输入决定执行哪一个逻辑。JavaScript 应用也是如此，但由于值很容易自省（译者注：自省指的是代码能够自我检查、访问内部属性，获得代码的底层信息），所以具体要执行哪个逻辑也得看输入数据的类型。条件类型就可以用于描述输入类型和输出类型之间的联系。

```ts
interface Animal {
    live(): void;
}
interface Dog extends Animal {
    woof(): void;
}
type Example1 = Dog extends Animal ? number : string;
		^
        // type Example1 = number
type Example2 = RegExp extends Animal ? number : string;
	    ^
        // type Example2 = string
```

条件类型的形式有点像 JavaScript 中的条件表达式（`条件 ? 真分支:假分支`）：

```ts
SomeType extends OtherType ? TrueType : FalseType;
```

当 `extends` 左边的类型可以赋值给右边的类型时，最终得到的就是第一个分支（真分支）中的类型，否则得到第二个分支（假分支）中的类型。

仅从上面的例子来看，条件类型看起来并不是很有用 —— 就算不依靠它，我们自己也能知道 `Dog extends Animal` 是否成立，然后选择对应的 `number` 类型或者 `string` 类型！但如果把条件类型和泛型结合使用，那它就能发挥巨大的威力了。

举个例子，我们看看下面的 `createLabel` 函数：

```ts
interface IdLabel {
    id: number            /* 一些属性 */
}
interface NameLabel {
    name: string          /* 其它属性 */  
}
function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel {
    throw "unimplemented";
}
```

`createLabel` 函数使用了重载，基于不同的输入类型选择不同的输出类型。请注意，这样做存在一些问题：

1. 如果一个库必须在整个 API 中反复做出相同的选择，那么这将变得很繁杂。
2. 我们需要创建三个重载：前两个分别针对具体的输入类型（`string` 和 `number`），最后一个则针对最通用的情况（输入类型为 `string | number `）。一旦 `createLabel` 增加了能够处理的新类型，那么重载的数量将以指数形式增加。

所以不妨换一种方式，我们可以将上面代码的逻辑编码到一个条件类型中：

```ts
type NameOrId<T extends number | string> = T extends nummber
? IdLabel
: NameLabel;
```

接着，我们可以使用这个条件类型将原来的重载函数简化为一个没有重载的函数：

```ts
function createLabel<T extends number | string>(idOrName: T): NameOrId<T> {
    throw "unimplemented";
}
let a = createLabel("typescript");
    ^
   // let a: NameLabel
 
let b = createLabel(2.8);
    ^
  // let b: IdLabel
 
let c = createLabel(Math.random() ? "hello" : 42);
    ^
 // let c: NameLabel | IdLabel
```

### 条件类型约束

通常情况下，条件类型中的检查会给我们提供一些新的信息。就像使用类型保护实现的类型收缩可以得到一个更具体的类型一样，条件类型的真分支可以通过我们检查的类型进一步地去约束泛型。

以下面的代码为例：

```ts
type MessageOf<T> = T['message'];
//Type '"message"' cannot be used to index type 'T'.
```

在这段代码中，TypeScript 抛出了一个错误，因为它无法确定 `T` 是否有 `message` 属性。我们可以对 `T` 进行约束，这样 TypeScript 就不会再报错了：

```ts
type MessageOf<T extends { message: unknown }> = T['message'];

interface Email {
    message: string;
}

type EmailMessageContents = MessageOf<Emial>;
			^
          // type EmailMessageContents = string      
```

不过，如果我们想要让 `MessageOf` 可以接受任意类型，并在 `message` 属性不存在的时候默认使用 `never` 类型，应该怎么做呢？我们可以把约束条件移出去，然后引入条件类型：

```ts
type MessageOf<T> = T extends { message: unknown } ? T['message'] : never

interface Email {
    message: string;
}

interface Dog {
    bark(): void;
}

type EmialMessageContents = MessageOf<Email>;
			^
          // type EmialMessageContents = string
type DogMessageContents = MessageOf<Dog>;
             ^
          // type DogMessageContents = never                
```

在条件类型的真分支中，TypeScript 知道 `T` 将会有 `message` 属性。

再来看一个例子。我们可以编写一个 `Flatten` 函数，它可以将数组类型扁平化为数组中元素的类型，对于非数组类型则保留其原类型：

```ts
type Flatten<T> = T extends any[] ? T[number] : T;

// 提取元素类型
type Str = Flatten<string[]>;
	  ^
    // type Str = string

// 保留原类型          
type Num = Flatten<number>;
	  ^
    // type Num = number      
```

当 `Flatten` 接受数组类型的时候，它会使用 `number` 按索引访问，从而提取出数组类型 `string[]` 中的元素类型；如果它接受的不是数组类型，则直接返回给定的原类型。

### 在条件类型中进行推断

在上面的例子中，我们使用条件类型去应用约束并提取出类型。由于这种操作很常见，所以条件类型提供了一种更简单的方式来完成。

条件类型提供了 `infer` 关键字，让我们可以推断出条件中的某个类型，并应用到真分支中。举个例子，在上面的 `Flatten` 函数中，我们可以直接推断出数组元素的类型，而不是通过索引访问“手动”提取出元素的类型：

```ts
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;
```

这里，我们使用 `infer` 关键字声明式地引入了一个新的泛型类型变量 `Item`，而不是在真分支中指定如何提取出 `T` 数组的元素类型。这使我们不必再去考虑如何找出我们感兴趣的类型的结构。

我们可以使用 `infer` 关键字编写一些有用的工具类型别名。举个例子，在一些简单的情况下，我们可以从函数类型中提取出返回值的类型：

```ts
type GetReturnType<Type> = Type extends (...args: never[]) => infer Rerturn
? Return
: never;

type Num = GetReturnType<() => number>;
	  ^
   // type Num = number      
type Str = GetReturnType<(x: string) => string>;
	  ^
   // type Str = string
type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]>;
	  ^
   // type Bools = boolean[]            
```

如果在一个具备多个调用签名的类型（比如某个重载函数的类型）中进行推断，那么推断只会针对最后一个签名（也就是最通用的情况）。

```ts
declare function stringOrNum(x: string): number;
declare function stringOrNum(x: number): string;
declare function stringOrNum(x: string | number): string | number;
 
type T1 = ReturnType<typeof stringOrNum>;
     ^
   // type T1 = string | number
```

### 可分配的条件类型

条件类型作用于泛型上时，如果给定一个联合类型，那么这时候的条件类型是可分配的。举个例子，看下面的代码：

```ts
type ToArray<Type> = Type extends any ? Type[] : never;
```

如果我们给 `toArray` 传入一个联合类型，那么条件类型将会应用给联合类型的每一个成员。

```ts
type ToArray<Type> = Type extends any ? Type[] : never;
type StrArrOrNumArr = ToArray<string | number>;
		   ^
         // type StrArrOrNumArr = string[] | number[]      
```

`StrArrOrNumArr` 在下面的联合类型中发生了分配：

```ts
string | number
```

然后将联合类型的每一个成员有效地映射为如下的数组：

```ts
ToArray<string> | ToArray<number>;
```

最终得到如下的数组：

```ts
string[] | number[];
```

通常情况下，这是我们期望的行为。如果想要规避这种行为，你可以将 `extends` 关键字的左右两边各用一个方括号包裹起来。

```ts
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never;

// 'StrArrOrNumArr' 不再是一个联合类型
type StrArrOrNumArr = ToArrayNonDist<string | number>;
		  ^
        // type StrArrOrNumArr = (string | number)[]   
```
