## Keyof 类型操作符

### `keyof` 类型操作符

`keyof` 类型操作符接受一个对象类型作为参数，并基于它的键产生一个由字符串字面量或者数值字面量组成的联合类型。下面的类型 `P` 等同于类型 `"x" | "y"`：

```ts
type Point = { x: number; y: number };
type P = keyof Point;
	^
   // type P = keyof Point     
```

如果 `keyof` 操作的类型有 `string` 或者 `number` 类型的索引签名，那么 `keyof` 会返回该索引签名的类型：

```ts
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;
     ^
   // type A = number
 
type Mapish = { [k: string]: boolean };
type M = keyof Mapish;
     ^
   // type M = string | number
```

注意，在这个例子中，`M` 表示的类型是 `string | number` —— 这是因为 JavaScript 对象的键总是会被强制转化为一个字符串，因此 `obj[0]` 等同于 `obj["0"]`。

`keyof` 类型和映射类型结合的时候会发挥很大的作用，后续的章节我们也会进行介绍。