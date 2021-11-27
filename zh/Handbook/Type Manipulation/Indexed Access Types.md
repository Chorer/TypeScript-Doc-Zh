## 按索引访问的类型

我们可以访问某个类型上的特定属性，从而获取该属性的类型。这种类型称为按索引访问的类型。

```ts
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"];
     ^^^
    // type Age = number
```

索引类型本身也是一个类型，所以它完全可以和联合类型、`keyof` 或者其它类型搭配使用：

```ts
type I1 = Person["age" | "name"];
     ^
    // type I1 = string | number
 
type I2 = Person[keyof Person];
     ^ 
    // type I2 = string | number | boolean
 
type AliveOrName = "alive" | "name";
type I3 = Person[AliveOrName];
      ^ 
    // type I3 = string | boolean
```

如果尝试索引一个不存在的属性，则会抛出错误：

```ts
type I1 = Person["alve"];	
			    ^^^^^^			
// Property 'alve' does not exist on type 'Person'.
```

此外，我们还可以使用 `number` 获取数组元素的类型。我们可以将其与 `typeof` 相结合，方便地捕获数组字面量的元素类型：

```ts
const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 38 },
];
 
type Person = typeof MyArray[number];
     ^^^^^^  
/* type Person = {
   	    name: string;
        age: number;
   } */
   
type Age = typeof MyArray[number]["age"];
     ^^^
   // type Age = number
// 或者
type Age2 = Person["age"];
     ^^^ 
    // type Age2 = number
```

你只能使用类型作为索引，也就是说，使用 `const` 创建的变量引用是不能作为索引的：

```ts
const key = "age";
type Age = Person[key];
				^^^^
/*                    
Type 'key' cannot be used as an index type.
'key' refers to a value, but is being used as a type here. Did you mean 'typeof key'?
*/
```

不过，你可以改用类型别名重写这段代码：

```ts
type key = "agr";
type Age = Person[key];
```