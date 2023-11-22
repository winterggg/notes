# TypeScript 类型运算符

## satisfies 运算符

场景：需要检查某个值 obj 符合指定类型 T，但不希望指定为 T

具体的：T 为更宽泛的类型，希望 obj 属于 T 的同时，不丢失其更具体的(推断)类型。

```ts
type Colors = "red" | "green" | "blue";
type RGB = [number, number, number];

const palette: Record<Colors, string|RGB> = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255]
};
const greenComponent = palette.green.substring(1, 6); // 报错

// better way
type Colors = "red" | "green" | "blue";
type RGB = [number, number, number];

const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  bleu: [0, 0, 255] // 报错
} satisfies Record<Colors, string|RGB>;

const greenComponent = palette.green.substring(1); // 不报错
```

## is 运算符

第一种用法，替换 boolean做类型保护。

```ts
function isCat(a:any): a is Cat {
  return a.name === 'kitty';
}

let x:Cat|Dog;

if (isCat(x)) {
  x.meow(); // 正确，因为 x 肯定是 Cat 类型
}
```

第二种用法，用在类（class）的内部，描述类的方法的返回值。

暂时没找到这样做有什么意义，勉强举个例子：

```ts
class Teacher {
  isStudent():this is Student {
    return false;
  }
}

class Student {
  isStudent():this is Student {
    return true;
  }
  write() {

  }
}

type Man = Teacher | Student

const man: Man = new Teacher();

if (man.isStudent()) {
    man.write();
}
man.write() // error
```

## in 运算符

遍历联合类型：

```ts
type U = 'a'|'b'|'c';

type Foo = {
  [Prop in U]: number;
};
// 等同于
type Foo = {
  a: number,
  b: number,
  c: number
};


// [Prop in keyof Obj] 
```

## 方括号运算符

```ts
type Person = {
  age: number;
  name: string;
  alive: boolean;
};

// Age 的类型是 number
type Age = Person['age'];
// number|string
type T = Person['age'|'name'];
// number|string|boolean
type A = Person[keyof Person];

type Obj = {
  [key:string]: number,
};
// number
type T = Obj[string];

// MyArray 的类型是 { [key:number]: string }
const MyArray = ['a','b','c'];

// typeof 优先级高于[]，等同于 (typeof MyArray)[number]
// 返回 string
type Person = typeof MyArray[number];
```

## keyof 运算符

单目运算符，接受一个对象类型作为参数，返回该对象的所有键名组成的联合类型。

```ts
interface T {
  0: boolean;
  a: string;
  b(): void;
}

type KeyT = keyof T; // 0 | 'a' | 'b'
type KeyT = keyof any; // string | number | symbol
type KeyT = keyof object;  // never

type MyObj = {
  foo: number,
  bar: string,
};

type Keys = keyof MyObj;
type Values = MyObj[Keys]; // number|string
```

一些例子：

```ts
interface T {
  [prop: string]: number;
}

// string|number, js 的 feature 属于是
type KeyT = keyof T;

// number | "0" | "1" | "2" | "length" | "pop" | "push" |
type Result = keyof ['a', 'b', 'c'];

type A = { a: string; z: boolean };
type B = { b: string; z: boolean };

// 返回 'z'
type KeyT = keyof (A | B);

type A = { a: string; x: boolean };
type B = { b: string; y: number };

// 返回 'a' | 'x' | 'b' | 'y'
type KeyT = keyof (A & B);

// 相当于
keyof (A & B) ≡ keyof A | keyof B
```

可以用来定义一些精确类型：

```ts
function prop<Obj, K extends keyof Obj>(
  obj:Obj, key:K
):Obj[K] {
  return obj[key];
}

type NewProps<Obj> = {
  [Prop in keyof Obj]: boolean;
};
// 用法
type MyObj = { foo: number; };
// 等于 { foo: boolean; }
type NewObj = NewProps<MyObj>;

type Mutable<Obj> = {
    // -readonly 去除 readonly，也可以 +readonly
  -readonly [Prop in keyof Obj]: Obj[Prop];
};
// 用法
type MyObj = {
  readonly foo: number;
}
// 等于 { foo: number; }
type NewObj = Mutable<MyObj>;

type Concrete<Obj> = {
    // 去除必选，也可以+?
  [Prop in keyof Obj]-?: Obj[Prop];
};
// 用法
type MyObj = {
  foo?: number;
}
// 等于 { foo: number; }
type NewObj = Concrete<MyObj>;
```

## extends...?: 条件运算符 

条件运算符`extends...?:`可以根据当前类型是否符合某种条件，返回不同的类型。

```ts
// 如果 T 可以赋值给 U，结果为 X，否则为 Y
T extends U ? X : Y


interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}

// number
type T1 = Dog extends Animal ? number : string;

// string
type T2 = RegExp extends Animal ? number : string;
```

对于联合类型，有两种写法，对应于展开或者不展开：

```ts
// 示例一
type ToArray<Type> =
  Type extends any ? Type[] : never;

// string[]|number[]
type T = ToArray<string|number>;
// 即
(A|B) extends U ? X : Y

// 等同于

(A extends U ? X : Y) |
(B extends U ? X : Y)

// 示例二
type ToArray<Type> =
  [Type] extends [any] ? Type[] : never;

// (string | number)[]
type T = ToArray<string|number>;
```

嵌套使用：

```ts
type LiteralTypeName<T> =
  T extends undefined ? "undefined" :
  T extends null ? "null" :
  T extends boolean ? "boolean" :
  T extends number ? "number" :
  T extends bigint ? "bigint" :
  T extends string ? "string" :
  never;

// "bigint"
type Result1 = LiteralTypeName<123n>;

// "string" | "number" | "boolean"
type Result2 = LiteralTypeName<true | 1 | 'a'>;
```

## 模板字符串

TypeScript 允许使用模板字符串，构建类型：

```ts
type World = "world";

// "hello world"
type Greeting = `hello ${World}`;

// 模板字符串可以引用的类型一共6种，分别是 string、number、bigint、boolean、null、undefined
type Num = 123;
type Obj = { n : 123 };
type T1 = `${Num} received`; // 正确
type T2 = `${Obj} received`; // 报错

// 联合类型
type T = 'A'|'B';
// "A_id"|"B_id"
type U = `${T}_id`;
type M = '1'|'2';
// 'A1'|'A2'|'B1'|'B2'
type V = `${T}${M}`;
```

## infer 关键字

infer 定义泛型里面推断出来的类型参数，通常结合条件运算符 extends？使用：

```ts
type Flatten<Type> =
  Type extends Array<infer Item> ? Item : Type;

// string
type Str = Flatten<string[]>;

// number
type Num = Flatten<number>;
```

使用`infer`，推断函数的参数类型和返回值类型：

```ts
type ReturnPromise<T> =
  T extends (...args: infer A) => infer R 
  ? (...args: A) => Promise<R> 
  : T;
// 如果不使用infer，就不得不把ReturnPromise<T>写成ReturnPromise<T, A, R>，非常麻烦
```

提取对象指定属性：

```ts
type MyType<T> =
  T extends {
    a: infer M,
    b: infer N
  } ? [M, N] : never;

// 用法示例
type T = MyType<{ a: string; b: number }>;
// [string, number]
```

配合模板字符串：

```ts
type Str = 'foo-bar';

type Bar = Str extends `foo-${infer rest}` ? rest : never // 'bar'
```
