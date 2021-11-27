## Typeof 类型操作符

### `typeof` 类型操作符

JavaScript 中本身就有一个可用于表达式上下文的 `typeof` 操作符：

```js
// 打印 "string"
console.log(typeof "Hello world");
```

TypeScript 则添加了一个可用于类型上下文的 `typeof` 操作符，让你可以引用某个变量或者属性的类型：

```ts
let s = "hello";
let n: typeof s;
	^
    // let n: string
```

像上面这样用于基本类型，作用并不是很大，但如果把 `typeof` 和其它类型操作符结合使用，就可以方便地表示多种模式了。举个例子，我们先来看一下预定义类型 `ReturnType<T>`。它可以接受一个函数类型并将它的返回值类型返回出去：

```ts
type Predicate = (x: unknown) => boolean;
type K = ReturnType<Predicate>;
	 ^
    // type K = boolean     
```

如果直接把函数名作为参数传递给 `ReturnType`，那么我们会看到一个指示性错误：

```ts
function f() {
  return { x: 10, y: 3 };
}
type P = ReturnType<f>;
				  ^	
// 'f' refers to a value, but is being used as a type here. Did you mean 'typeof f'?
```

记住，值和类型是不一样的，这里应该传入类型而不是值，因此我们可以改用 `typeof` 去引用值 `f` 的类型：

```ts
function f(){
    return {
        x: 10,
        y: 3
    };
}
type P = ReturnType<typeof f>;
	 ^
   /* type P = {
    	x: number;
    	y: number;
	 }            */   
```

#### 限制

TypeScript 有意限制 `typeof` 可以操作的表达式类型。具体地说，`tyepof` 只能作用于标识符（比如变量名）或者它们的属性。这可以避免让开发者编写他们认为可以执行的但实际上不能执行的代码：

```ts
// 这里应该改用 = ReturnType<typeof msgbox>
let shouldContinue: typeof msgbox("Are you sure you want to continue?");
							^			
// ',' expected.
```

