## 映射类型

有时候我们不想重复编写代码，这时候就需要基于某个类型创建出另一个类型。

索引签名用于为那些没有提前声明的属性去声明类型，而映射类型是基于索引签名的语法构建的。

```ts
type OnlyBoolsAndHorses = {
    [key: string]: boolean | Horse;
};
const conforms: OnlyBoolsAndHorses = {    
    del: true,
    rodney: false,
};
```

映射类型也是一种泛型类型，它使用 `PropertyKey`（属性键）的联合类型（通常通过 [keyof](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html) 创建）去遍历所有的键，从而创建一个新的类型：

```ts
type OptionsFlags<Type> = {
    [Property in keyof Type]: boolean;
};
```

在这个例子中，`OptionsFlags` 会接受类型 `Type` 的所有属性，并将它们的值改为布尔值。

```ts
type FeatureFlags = {
    darkMode: () => void;
    newUserProfile: () => void;
};

type FeatureOptions = OptionsFlags<FeatureFlags>;
		   ^
      /* 
      type FeatureOptions = {
            darkMode: boolean;
            newUserProfile: boolean;
      }     
      */ 
```

### 映射修饰符

在映射的时候还有两个附加的修饰符可供使用，也就是 `readonly` 和 `?`，它们分别用于声明属性的只读性和可选性。

要移除或者添加修饰符，只需要给修饰符添加前缀 `-` 或者 `+` 即可。如果没有添加前缀，则默认使用 `+`。

```ts
// 移除类型中属性的只读性
type CreateMutable<Type> = {
    -readonly [Property in keyof Type]: Type[Property];
};

type LockedAccount = {
    readonly id: string;
    readonly name: string;
};

type UnlockedAccount = CreateMutable<LockedAccount>;
		    ^
       /* 
       type UnlockedAccount = {
              id: string;
              name: string;
       }     
       */ 
```

```ts
// 移除类型中属性的可选性
type Concrete<Type> = {
    [Property in keyof Type]-?: Type[Property];
}
type MaybeUser = {
    id: string;
    name?: string;
    age?: number;
};
type User = Concrete<MaybeUser>;
	  ^
  /* 
  type User = {
         id: string;
         name: string;
         age: number;
   }    
   */ 
```

### 通过 `as` 实现键的重新映射

在 TypeScript4.1 或者更高的版本中，你可以在映射类型中使用 `as` 子句实现键的重新映射：

```ts
type MappedTypeWithNewProperties<Type> = {
    [Properties in keyof Type as NewKeyType]: Type[Properties]
}
```

你可以使用诸如[模板字面量类型](https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html)这样的特性从原来的属性名中去创建新的属性名：

```ts
type Getters<Type> = {
    [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};

interface Person {
	name: string;
	age: number;
	location: string;
}
type LazyPerson = Getters<Person>;
		^
	/*
    type LazyPerson = {
        getName: () => string;
        getAge: () => number;
        getLocation: () => string;
    }
    */
```

你可以通过条件类型产生 `never` 类型，从而过滤掉某些键：

```ts
// 移除 kind 属性
type Exclude<T,U> = T extends U ? never : T
type RemoveKindField<Type> = {
    [Property in keyof Type as Exclude<Property,'kind'>]: Type[Property]
};

interface Circle {
    kind: 'circle';
    radius: number;
}

type KindlessCircle = RemoveKindField<Circle>;
		   ^
      /*
      type KindlessCircle = {
      	radius: number;
      }
      */         
```

不仅仅是 `string | number | symbol` 这样的联合类型，任意的联合类型都可以映射：

```ts
type EventConfig<Events extends { kind: string }> = {
    [E in Events as E['kind']]: (event: E) => void;
}

type SqureEvent = { kind: 'squre', x: number, y: number };
type CircleEvent = { kind: 'circle', radius: number };

type Config = EventConfig<SqureEvent | CricleEvent>
	   /**
	   type Config = {
	   	 squre: (event: SqureEvent) => void;
	   	 circle: (event: CircleEvent) => void;
	   }
	   /	
```

### 映射类型的进一步应用

映射类型也可以和本章（类型操控）介绍的其它特性搭配使用。举个例子，下面是一个[使用了条件类型的映射类型](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)，根据对象是否有一个设置为字面量 `true` 的属性 `pii`，它会返回 `true` 或者 `false`：

```ts
type ExtractPII<Type> = {
    [Property in keyof Type]: Type[Property] extends { pii: true } ? true : false
};

type DBFileds = {
    id: { format: 'incrementing' };
    name: { type: string; pii: true };
};

type ObjectsNeedingGDPRDeletion = ExtractPII<DBFileds>;
				^
            /*
            type ObjectsNeedingGDPRDeletion = {
            	id: false;
            	name: true;
            }
            */        
```