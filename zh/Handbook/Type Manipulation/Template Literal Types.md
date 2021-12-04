## 模板字面量类型

模板字面量类型基于[字符串字面量类型](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types)构建，可以通过联合类型拓展成多种字符串。

其语法和 [JavaScript 中的模板字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)一样，但在 TypeScript 中用于表示类型。和具体的字面量类型一起使用的时候，模板字面量会通过拼接内容产生一个新的字符串字面量类型。

```ts
type World = 'world';

type Greeting = `hello ${World}`;
	    ^
     // type Greeting = 'hello world'      
```

当在模板字面量的插值位置使用联合类型时，最终得到的类型是一个由联合类型每个成员可以表示的字符串字面量构成的集合：

```ts
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";
 
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
          ^
// type AllLocaleIDs = "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"
```

如果模板字面量有多个插值位置，那么各位置上的联合类型之间会进行叉积运算，从而得到最终的类型：

```ts
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
type Lang = "en" | "ja" | "pt";
 
type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`;
           ^ 
// type LocaleMessageIDs = "en_welcome_email_id" | "en_email_heading_id" | "en_footer_title_id" | "en_footer_sendoff_id" | "ja_welcome_email_id" | "ja_email_heading_id" | "ja_footer_title_id" | "ja_footer_sendoff_id" | "pt_welcome_email_id" | "pt_email_heading_id" | "pt_footer_title_id" | "pt_footer_sendoff_id"
```

对于大型的字符串联合类型，我们推荐你提前生成。对于较小的字符串联合类型，则可以使用上面例子中的方法生成。

### 类型中的字符串联合类型

模板字面量的强大之处在于它能够基于类型中的已有信息定义一个新的字符串。

假设现在有一个 `makeWatchedObject` 函数，它可以给传入的对象添加一个 `on` 方法。在 JavaScript 中，该函数的调用形如：`makeWatchedObject(baseObject)`。其中，传入的对象参数类似下面这样：

```ts
const passedObject = {
    firstName: 'Saoirse',
    lastName: 'Ronan',
    age: 26,
};
```

即将添加给对象的 `on` 方法会接受两个参数，一个是 `eventName`（字符串），一个是 `callBack`（回调函数）。

`eventName` 的形式类似于 `attributeInThePassedObject + 'Changed'`。比如说，传入对象有个 `firstName` 属性，那么对应就会有一个叫做 `firstNameChanged` 的 `eventName`。

而 `callBack` 回调函数，在被调用的时候会：

* 接受一个参数，参数的类型和 `attributeInThePassedObject` 的类型相关联。比如说，`firstName` 的类型是 `string`，那么 `firstNameChanged` 这个事件对应的回调函数在被调用的时候也期望接受一个 `string` 类型的参数。同理，和 `age` 相关联的事件回调函数在被调用的时候应该接受一个 `number` 类型的参数。
* 返回值类型为 `void`（为了方便例子的讲解）

`on()` 的简易版函数签名可能是这样的：`on(eventName: string, callBack: (newValue: any) => void)`。不过，从上面的描述来看，我们发现代码中还需要实现很重要的类型约束。而模板字面量类型正好就可以帮助我们做到这一点。

```ts
const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26,
});
 
// makeWatchedObject 函数给匿名对象添加了 on 方法
 
person.on("firstNameChanged", (newValue) => {
  console.log(`firstName was changed to ${newValue}!`);
});
```

注意，`on` 监听的事件是 `"firstNameChanged"`，而不是 `"firstName"`。如果我们要确保符合条件的事件名的集合受到对象属性名（末尾加上“Changed”）的联合类型的约束，那么我们的简易版 `on()` 方法还需要进一步完善才行。虽然在 JavaScript 中我们可以很方便地实现这个效果，比如使用 `Object.keys(passedObject).map(x => ${x}Changed)`，不过，类型系统中的模板字面量也提供了一种类似的操控字符串的方法：

```ts
type PropEventSource<Type> = {
    on(eventName: `${string & keyof Type}Changed`, callback: (newValue: any) => void): void;
};
 
// 创建一个带有 on 方法的监听对象，从而监听对象属性的变化
declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;
```

这样，当传入错误的参数时，TypeScript 会抛出一个错误：

```ts
const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});
 
person.on("firstNameChanged", () => {});
 
// 预防常见的人为错误（错误地使用了对象的属性名而不是事件名）
person.on("firstName", () => {});
// Argument of type '"firstName"' is not assignable to parameter of type '"firstNameChanged" | "lastNameChanged" | "ageChanged"'.
 
// 拼写错误
person.on("frstNameChanged", () => {});
// Argument of type '"frstNameChanged"' is not assignable to parameter of type '"firstNameChanged" | "lastNameChanged" | "ageChanged"'.
```

### 模板字面量的推断

注意，目前为止我们还没有完全利用传入对象提供的信息。`firstName` 改变的时候（触发 `firstNameChanged` 事件），我们期望回调函数会接受一个 `string` 类型的参数。同理，`age` 改变的时候，对应的回调函数也会接受一个 `number` 类型的参数。但目前，我们仅仅只是用 `any` 作为回调函数参数的类型而已。这里我们需要再次使用模板字面量类型，它可以确保属性的数据类型和属性对应的回调函数的参数类型保持一致。

实现这一点的关键在于：我们可以使用一个带有泛型的函数，从而确保：

1. 第一个参数中的字面量可以被捕获为一个字面量类型
2. 泛型的有效属性会构成一个联合类型，可以验证捕获的字面量类型是否是该联合类型的一个成员
3. 可以在泛型结构中通过按索引访问的方式去查看已验证属性的类型
4. 该类型信息可以被进一步利用，以保证回调函数的参数也是相同的类型

```ts
type PropEventSource<Type> = {
    on<Key extends string & keyof Type>(eventName: `${Key}Changed`, callback: (newValue: Type[Key]) => void): void;
}

declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;

const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});
 
person.on("firstNameChanged", newName => {
                                 ^
                     // (parameter) newName: string
    console.log(`new name is ${newName.toUpperCase()}`);
});
 
person.on("ageChanged", newAge => {
                           ^     
                 // (parameter) newAge: number
    if (newAge < 0) {
        console.warn("warning! negative age");
    }
})
```

这里我们让 `on` 变成了一个泛型方法。

当开发者通过字符串 `"firstNameChanged"` 调用了 `on` 方法的时候，TypeScript 会尝试推断出 `Key` 的正确类型。具体地说，它会将 `Key` 和 `"Changed"` 前面的部分进行匹配，并推断出字符串 `"firstName"`。一旦 TypeScript 推断完成，`on` 方法就可以取出原对象的 `firstName` 属性的类型 —— 即 `string` 类型。同理，当通过 `"ageChanged"` 调用方法的时候，TypeScript 也会发现 `age` 属性的类型是 `number`。

推断有多种不同的结合方式，通常用于解构字符串，并以不同的方式对字符串进行重构。

### 内建的字符串操控类型

为了方便操控字符串，TypeScript 引入了一些相关的类型。为了提高性能，这些类型是内建到编译器中的，并且无法在 TypeScript 附带的 `.d.ts` 文件中找到。

#### `Uppercase<StringType>`

将字符串中的每个字符转化为大写形式。

示例：

```ts
type Greeting = 'Hello, world'
type ShoutyGreeting = Uppercase<Greeting>
		^
        // type ShoutyGreeting = 'HELLO, WORLD'    
type ASCIICacheKey<Str extends string> = `ID-${Uppercase<Str>}`
type MainID = ASCIICacheKey<'my_app'>
	  ^
	  // type MainID = 'ID-MY_APP'
```

#### `Lowercase<StringType>`

将字符串中的每个字符转化为小写形式。

示例：

```ts
type Greeting = "Hello, world"
type QuietGreeting = Lowercase<Greeting>
          ^ 
      // type QuietGreeting = "hello, world"
 
type ASCIICacheKey<Str extends string> = `id-${Lowercase<Str>}`
type MainID = ASCIICacheKey<"MY_APP">
        ^ 
    // type MainID = "id-my_app"
```

#### `Capitalize<StringType>`

将字符串的首字符转化为大写形式。

示例：

```ts
type LowercaseGreeting = 'hello, world';
type Greeting = Capitalize<LowercaseGreeting>;
		^
       // type Greeting = 'Hello, world'     
```

#### `Uncapitalize<StringType>`

将字符串的首字符转化为小写形式。

示例：

```ts
type UppercaseGreeting = 'HELLO WORLD';
type UncomfortableGreeting = Uncapitalize<UppercaseGreeting>;
		   ^	
          // type UncomfortableGreeting = "hELLO WORLD"     
```

关于内建字符串操控类型的一些技术细节：

从 TypeScript 4.1 开始，这些内建函数的实现直接使用了 JavaScript 的字符串运行时函数进行操作，并且无法做到本地化识别。

```ts
function applyStringMapping(symbol: Symbol, str: string) {
    switch (intrinsicTypeKinds.get(symbol.escapedName as string)) {
        case IntrinsicTypeKind.Uppercase: return str.toUpperCase();
        case IntrinsicTypeKind.Lowercase: return str.toLowerCase();
        case IntrinsicTypeKind.Capitalize: return str.charAt(0).toUpperCase() + str.slice(1);
        case IntrinsicTypeKind.Uncapitalize: return str.charAt(0).toLowerCase() + str.slice(1);
    }
    return str;
}
```