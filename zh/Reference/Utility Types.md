## 工具类型

TypeScript 提供了一些工具类型用于实现常见的类型转换。这些工具类型是全局可用的。

### `Partial<Type>`

构建出一个将 `Type` 的所有属性设置为可选属性的类型。该工具类型会返回一个代表了给定类型的所有子集的类型。

**示例**

```ts
interface Todo {
  title: string;
  description: string;
}
 
function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}
 
const todo1 = {
  title: "organize desk",
  description: "clear clutter",
};
 
const todo2 = updateTodo(todo1, {
  description: "throw out trash",
});
```

### `Required<Type>`

构建出一个将 `Type` 的所有属性设置为必选属性的类型。它的作用和 [Partial](https://www.typescriptlang.org/docs/handbook/utility-types.html#partialtype) 相反。

**示例**

```ts
interface Props {
  a?: number;
  b?: string;
}
 
const obj: Props = { a: 5 };
 
const obj2: Required<Props> = { a: 5 };
	   ^
// Property 'b' is missing in type '{ a: number; }' but required in type 'Required<Props>'.
```

### `Readonly<Type>`

构建出一个将 `Type` 的所有属性设置为只读属性的类型，这意味着构造出来的类型的属性不可以重新被赋值。

**示例**

```ts
interface Todo {
  title: string;
}
 
const todo: Readonly<Todo> = {
  title: "Delete inactive users",
};
 
todo.title = "Hello";
       ^
// Cannot assign to 'title' because it is a read-only property.
```

这个工具类型可以有效地表示会在运行时失败（比如试图给[冻结对象](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)的属性重新赋值）的赋值表达式。

**`Object.freeze`**

```ts
function freeze<Type>(obj: Type): Readonly<Type>;
```

### `Record<Keys,Type>`

构建出一个对象类型，它的属性键是 `Keys`，属性值是 `Type`。该工具类型可以将属性从某个类型映射为另一种类型。

**示例**

```ts
interface CatInfo {
  age: number;
  breed: string;
}
 
type CatName = "miffy" | "boris" | "mordred";
 
const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: "Persian" },
  boris: { age: 5, breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};
 
cats.boris;
 ^
// const cats: Record<CatName, CatInfo>
```

### `Pick<Type,Keys>`

从 `Type` 中挑出属性集 `Keys`（字符串字面量或者字符串字面量的联合类型），从而构建类型。

**示例**

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}
 
type TodoPreview = Pick<Todo, "title" | "completed">;
 
const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
 
todo;
  ^
// const todo: TodoPreview
```

### `Omit<Type,Keys>`

从 `Type` 中挑出所有属性并移除 `Keys`（字符串字面量或者字符串字面量的联合类型），从而构建类型。

**示例**

```ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
  createdAt: number;
}
 
type TodoPreview = Omit<Todo, "description">;
 
const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
  createdAt: 1615544252770,
};
 
todo;
 ^
// const todo: TodoPreview
 
type TodoInfo = Omit<Todo, "completed" | "createdAt">;
 
const todoInfo: TodoInfo = {
  title: "Pick up kids",
  description: "Kindergarten closes at 5pm",
};
 
todoInfo;
   ^
// const todoInfo: TodoInfo
```

### `Exclude<Type,ExcludedUnion>`

从 `Type` 中排除所有可以赋值给 `ExcludedUnion` 的联合类型成员。

**示例**

```ts
type T0 = Exclude<"a" | "b" | "c", "a">;
      ^
// type T0 = "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">;
      ^  
   // type T1 = "c"
type T2 = Exclude<string | number | (() => void), Function>;
     ^
  // type T2 = string | number
```

### `Extract<Type,Union>`

从 `Type` 中提取出所有可以赋值给 `Union` 的联合类型成员，从而构建类型。

**示例**

```ts
type T0 = Extract<"a" | "b" | "c", "a" | "f">;
      ^
// type T0 = "a"
type T1 = Extract<string | number | (() => void), Function>;
      ^
// type T1 = () => void
```

### `NonNullable<Type>`

从 `Type` 中排除 `null` 和 `undefined`，从而构建类型。

**示例**

```ts
type T0 = NonNullable<string | number | undefined>;
     ^
// type T0 = string | number
type T1 = NonNullable<string[] | null | undefined>;
      ^  
// type T1 = string[]
```

### `Parameters<Type>`

从函数类型 `Type` 的参数使用的类型中构建一个元组类型。

**示例**

```ts
declare function f1(arg: { a: number; b: string }): void;
 
type T0 = Parameters<() => string>;
     ^
// type T0 = []
type T1 = Parameters<(s: string) => void>;
     ^ 
// type T1 = [s: string]
type T2 = Parameters<<T>(arg: T) => T>;
     ^  
// type T2 = [arg: unknown]
type T3 = Parameters<typeof f1>;
      ^  
/*
type T3 = [arg: {
    a: number;
    b: string;
}]
*/
type T4 = Parameters<any>;
      ^   
// type T4 = unknown[]
type T5 = Parameters<never>;
      ^  
// type T5 = never
type T6 = Parameters<string>;
     ^
// Type 'string' does not satisfy the constraint '(...args: any) => any'.     
// type T6 = never
type T7 = Parameters<Function>;
     ^ 
/*
Type 'Function' does not satisfy the constraint '(...args: any) => any'.
  Type 'Function' provides no match for the signature '(...args: any): any'.     
type T7 = never
*/
```

### `ConstructorParameters<Type>`

从一个构造函数类型的类型中构建一个元组或者数组类型。它会通过所有的参数类型产生一个元组类型（或者，如果 `Type` 不是函数，则返回 `never` 类型）。

**示例**

```ts
type T0 = ConstructorParameters<ErrorConstructor>;
      ^
// type T0 = [message?: string]
type T1 = ConstructorParameters<FunctionConstructor>;
      ^  
// type T1 = string[]
type T2 = ConstructorParameters<RegExpConstructor>;
      ^  
// type T2 = [pattern: string | RegExp, flags?: string]
type T3 = ConstructorParameters<any>;
      ^  
// type T3 = unknown[]
 
type T4 = ConstructorParameters<Function>;
	 ^
/*
Type 'Function' does not satisfy the constraint 'abstract new (...args: any) => any'.
  Type 'Function' provides no match for the signature 'new (...args: any): any'.     
type T4 = never
*/
```

### `ReturnType<Type>`

构建一个由函数 `Type` 的返回值类型组成的类型。

**示例**

```ts
declare function f1(): { a: number; b: string };
type T0 = ReturnType<() => string>;
      ^
    // type T0 = string
type T1 = ReturnType<(s: string) => void>;
      ^
// type T1 = void
type T2 = ReturnType<<T>() => T>;
      ^  
// type T2 = unknown
type T3 = ReturnType<<T extends U, U extends number[]>() => T>;
      ^  
// type T3 = number[]
type T4 = ReturnType<typeof f1>;
      ^  
/*
type T4 = {
    a: number;
    b: string;
}
*/
type T5 = ReturnType<any>;
      ^
// type T5 = any
type T6 = ReturnType<never>;
      ^  
// type T6 = never
type T7 = ReturnType<string>;
      ^
// Type 'string' does not satisfy the constraint '(...args: any) => any'.     
// type T7 = any
type T8 = ReturnType<Function>;
      ^
/*
Type 'Function' does not satisfy the constraint '(...args: any) => any'.
  Type 'Function' provides no match for the signature '(...args: any): any'.     
type T8 = any          
*/
```

### `ThisParameterType<Type>`

从一个函数类型中提取出 [this](https://www.typescriptlang.org/docs/handbook/functions.html#this-parameters) 参数的类型，或者在函数类型没有 `this` 参数的情况下提取出 [unknown](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html#new-unknown-top-type) 类型。

**示例**

```ts
function toHex(this: Number) {
  return this.toString(16);
}
 
function numberToString(n: ThisParameterType<typeof toHex>) {
  return toHex.apply(n);
}
```

### `OmitThisParameter<Type>`

从 `Type` 中移除 [this](https://www.typescriptlang.org/docs/handbook/functions.html#this-parameters) 参数。如果 `Type` 没有显式声明 `this` 参数，那么直接返回 `Type`，否则会基于 `Type` 创建一个没有 `this` 参数的新的函数类型。泛型会被抹除，并且只有最后一个重载签名会传播到新的函数类型中。

**示例**

```ts
function toHex(this: Number) {
  return this.toString(16);
}
 
const fiveToHex: OmitThisParameter<typeof toHex> = toHex.bind(5);
 
console.log(fiveToHex());
```

### `ThisType<Type>`

这个工具类型并没有返回一个被转化的类型，而是作为一个上下文 [this](https://www.typescriptlang.org/docs/handbook/functions.html#this) 类型的标记。注意，必须在启用 [noImplicitThis](https://www.typescriptlang.org/tsconfig#noImplicitThis) 配置项之后才能使用该工具类型。

**示例**

```ts
type ObjectDescriptor<D, M> = {
  data?: D;
  methods?: M & ThisType<D & M>; // methods 中 this 的类型是 D & M
};
 
function makeObject<D, M>(desc: ObjectDescriptor<D, M>): D & M {
  let data: object = desc.data || {};
  let methods: object = desc.methods || {};
  return { ...data, ...methods } as D & M;
}
 
let obj = makeObject({
  data: { x: 0, y: 0 },
  methods: {
    moveBy(dx: number, dy: number) {
      this.x += dx; // Strongly typed this
      this.y += dy; // Strongly typed this
    },
  },
});
 
obj.x = 10;
obj.y = 20;
obj.moveBy(5, 5);
```

在上面的示例中，传给 `makeObject` 的参数中的 `methods` 对象有一个包含 `ThisType<D & M>` 的上下文类型，因此在 `methods` 对象的方法中，[this](https://www.typescriptlang.org/docs/handbook/functions.html#this) 的类型是  `{ x: number, y: number } & { moveBy(dx: number, dy: number): number }`。注意 `methods` 属性的类型既是一个推断目标，也是方法中 `this` 类型的来源。

`ThisType<T>` 标记接口只是一个在 `lib.d.ts` 中声明的空接口。除了在一个对象字面量的上下文类型中被识别之外，它的表现和任何的空接口一样。

### 内置的字符串操控类型

`Uppercase<StringType>`

`Lowercase<StringType>`

`Capitalize<StringType>`

`Uncapitalize<StringType>`

为了方便操控模板字符串字面量，TypeScript 引入了一系列可以在类型系统的字符串操控中使用的类型。你可以在文档的[模板字面量类型](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html#uppercasestringtype)中找到这些工具类型。