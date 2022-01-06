# TS基础学习

## 基础类型

```typescript
// 布尔类型
let isDone: boolean = true;

// 数字类型
let count: number = 10;

// 字符串类型
let name: string = "funny";

// symbol类型
let sym: symbol = Symbol('a');

// 数组类型
let list: number[] = [1, 2, 3];
// 这里用到了泛型的概念，后面会详细介绍泛型
let list: Array<number> = [1, 2, 3];
// 数组里含多个类型，可以使用联合类型
let list: Array<number | string> = [1, 2, 'c'];

// 元组类型Tuple: 允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。
let map : [string, number]
map = ['a', 1]; 				// ok
map = [1, 'a']; 				// error

// 枚举类型enum
enum Color {
    Red,
    Blue,
    Green
}

// any类型
let notSure: any = 'notSure';
notSure = false;    			// ok

// unknown类型
let unknown: unknown = 3;
let number : number = 4;
number = unknown        		//Error

// void类型
// 表示没有任何类型，通常在函数没有返回值时使用
function say(): void {
    console.log("hello funny");
}

// null,undefined类型
let u: undefined = undefined;
let n: null = null;

// never类型,返回never的函数必须存在无法达到的终点
function error(message: string): never {
    throw new Error(message);
}

// object类型
declare function create(o: object | null): void;
create({ prop: 0 }); 			// OK
create(null); 					// OK
create(42); 					// Error
```

上述为基本类型的使用方法，现在我们对于其中的相关概念做一些补充：

### 枚举类型

枚举类型默认从0开始为元素编号的，当然你也可以手动指定。同时枚举类型会被编译为一个双向Map结构。数字枚举具备自映射，而字符串枚举不具备。

### any、unknown、never类型

**any**类型表示任意类型，没有任何约束，可以绕过静态类型检查。通常当我们不希望类型检查器对这些值进行检查而是直接让它们通过编译阶段的检查。 那么我们可以使用 any类型来标记这些变量。

**unknown**类型是any的安全类型，但当我们需要对unknown类型的值进行操作之前，要检查其类型，因为unknown类型只能赋给any类型和unknown类型。

**never**类型表示永远不存在的值,即和any相反。需要注意的是**never类型是任何类型的子类型**，也可以赋值给任何类型；然而，没有类型是never的子类型或可以赋值给never类型（除了never本身之外）。

```typescript
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
    throw new Error(message);
}

// 推断的返回值类型为never
function fail() {
    return error("Something failed");
}

// 返回never的函数必须存在无法达到的终点
function infiniteLoop(): never {
    while (true) {
    }
}
```

### null和undefined类型

- 默认情况：null和undefined是所有类型的子类型。即我们可以把null和undefined赋值给其他类型的变量比如number。
- 指定了--strictNullChecks标记情况：null和undefined只能赋值给void和它们各自。

## 接口类型和类型别名

我们一般用 interface 来代表接口， 用 type 来表示类型别名。他俩的区别其实并不大，都可以用来约束对象和函数

```typescript
interface Person {
    name: string,
    age: number
}

// 更多情况下我们使用type来定义函数
interface getPersonName {       
    (person: Person) : string
}

type Person1 = {
    name: string,
    age: number
}

type getPerson1Name = (person: Person1) => string;
```

### interface

接口里可定义只读属性，多选属性，接口也可以做多余属性的检查

```typescript
interface Person {
    name: string;
    readonly age: number; // 可选属性
    hobby?: string; // 只读属性
}
```

多余属性检查：

```typescript
let p: Person = { name: 'funny', age: 20, sex: '男' }; 
// error TS2322: Type '{ name: string; age: number; sex: string; }' is not assignable to type 'Person'.
```

### 类型别名type

类型别名的本质即给类型起一个新名字。其功能主要用于解决接口类型无法覆盖的场景，比如联合类型、交叉类型、提取接口属性类型、引用自身。

```typescript
// 联合类型：retcode
type retcode = string | number;
// 类型复用BrandCard，GItemProps实现了动态生成类型
interface BrandCard { 
    id: string;
    name: string;
}
// 交叉类型：GItemProps
type GItemProps = BrandCard & {
    onReportClick: () => void;
    onShow: () => void;
};
// 提取接口属性类型
type BrandNameType = BrandCard['name']; // BrandNameType 是 string类型 的别名
type DOMCalcMethod = () => ();
// 引用自身：DOMAccessTree、TreeNode
interface DOMAccessTree { // DOMAccessTree 是一种递归的数据结构
    [key: string]: DOMAccessTree | DOMCalcMethod;
}
// 类型别名也可是泛型
type TreeNode<T> = {
    value: T;
    leftNode: TreeNode<T>;
    rightNode: TreeNode<T>;
}
```

### 区别

- interface 遇到同名的 interface、class 会自动聚合；interface 仅能声明 object、function 类型
- type 不可重复声明；type 声明不受限；

## 泛型

用于提高函数和组件的通用性。

```typescript
type User = {
	name: string;
	age: number;
}
```

### 一些关键字

```typescript
// keyof 关键字
// 对于一个object类型，可以得到这个类型的所有属性名构成的联合类型
type TA = keyof User;	// "name" | "age"
let a:TA = "age";


// typeof 关键字
// 可以让我们得到某个类型实例的类型
const fn = () => ({name: 'funny', age: 20});
type TB = typeof fn;	// () => { name: string;  age: number; }


// extends 关键字
// 这个关键字在ts中比较特殊，其是判断一个类型是否可以被赋值给另一个类型，可用作类型约束
interface ILengthy {
    length: number;
}
function logLength2<T extends ILengthy>(arg: T) {
    console.log(arg.length)
}


// infer 关键字
// infer 一般用于类型提取
type UserPromise = Promise<User>;
// 在这里可以让我infer声明了一个类型V
type UnPromisify<T> = T extends Promise<infer V> ? V : never;
type InferedUser = UnPromisify<UserPromise>;	//  InferedUser = {name: string;age: number;}
```

### 类型的“函数”和类型的变量

上面代码中UnPromisify，就是这样一个类型的”函数“，即针对类型定义的一个函数，因为尖括号中的 T是不确定的，因此称为泛型。上面UnPromisify<UserPromise>，则是这个“函数”的执行结果，可以理解为类型的”变量“。

### 介绍几个工具函数

形如 type ToolType<T,....>=R。为了便于理解我们可以将其看做是用type关键字定义的一个封装好的针对类型的“函数”。传给工具类型的，被包裹在尖括号之内的泛型 T，就是函数的参数。等号右边的，就是这个“函数”的返回值。

- Partial<T>: 输入一个object类型， 返回一个新的object类型， 即将其每个属性变为可选的
- Record: 接受两个类型作为”参数“，其中第一个参数 K是一个任意字符串、数字或 Symbol 的联合类型，第二个“参数” T可以为任意类型。最终得到一个由 K中每个值作为键，值类型为 T的新的 object 类型。
- Pick: 从给定的类型 T中 pick 出特定的键和键类型
- Exclude<T, U>: 终得到联合类型中存在于 T中但不存在于 U中的项
- InstanceType<T>:   形如 new(args:any)=>any类型的函数，被称为构造函数类型。InstanceType,用于取得构造函数的返回的实例的类型。
- ReturnType<T>：ReturnType工具类型用于提取泛型 T的返回值。
- Parameters<T>： Parameters工具类型用于提取泛型 T的参数。

```typescript
// MyPartial<T>
type MyPartial<T> = {
    // K必须是这个联合类型中的一个
    [K in keyof T]?: T[K];
};
type PartialUser = MyPartial<User>;


// Record<K, T>
type MyRecord<K extends keyof any, T> = {
    [P in K]: T;
}
type TKeys = "a" | "b" | 0;
type TKeysUser = MyRecord<TKeys, User>;


// MyPick<T, K>
type MyPick<T, K extends keyof T> = {
    [P in K]: T[P];
};
type TNameKey = "name";
type TUserName = MyPick<User, TNameKey>;


// MyExclude
type MyExclude<T, U> = T extends U ? never : T;
type TC = "a" | "b" | "c";
type TD = "a" | "c" | "e";
type TE = MyExclude<TC, TD>;	// b


// 下面三个类型函数，你只要把握住 infer的位置和匿名的函数类型 (...args:any)=>any这两个要点即可。

// MyInstanceType
type MyInstanceType<T extends new (...args: any) => any> = T extends new (
    ...args: any
) => infer R
    ? R
    : any;

// MyReturnType
type MyReturnType<T extends (...args: any) => any> = T extends (
    ...args: any
  ) => infer R
    ? R
    : never;

// MyParameters
type MyParameters<T extends (...args: any) => any> = T extends (
    ...args: infer P
  ) => any
    ? P
    : never;
```

## 类型保护

类型保护就是一些**表达式**，它们会在运行时检查以确保在某个**作用域内**的类型。

```typescript
interface Bird {
  fly(): any;
  layEggs(): any;
}

interface Fish {
  swim(): any
  layEggs(): any
}

type Pet = Bird | Fish;
```

### 无类型保护时

```typescript
function fn(pet: Pet) {  
  pet.layEggs(); // okay
  pet.swim();    // Error: Property 'swim' does not exist on type 'Bird | Fish'.
}
```

### 类型断言

```typescript
function fn(pet: Fish | Bird) {
  if ((<Fish>pet).swim) {
    (<Fish>pet).swim();
  } else {
    (<Bird>pet).fly();
  }
}
```

可以使用 pet的 **类型断言**，来告诉 TS 目标对象是什么类型。

缺点： 不方便，需要在各处都进行引用

注意：如果在编写 tsx 时，你需要将 (pet) 写成 (pet as Fish)，因为在 tsx 中尖括号 <>有特殊的含义。

### is 关键字

```typescript
function isFish<T>(pet: Fish | Bird): pet is Fish {
  return !!(<Fish>pet).swim;
}
```

这个代码我们封装了一下if内的判断，从而获得更加方便的类型保护，此函数使用了

parameterName is Type的 **类型谓语**，切其返回值必须为boolean类型

```typescript
function fn(pet: Fish | Bird ) {  
  if (isFish(pet)) {
    pet.swim(); // 因 is 语句的生效，此语句块中类型 let pet: Fish
  } else {
    pet.fly();  // 排除 Fish 之外的类型，此语句块中类型 let pet: Bird
  }
}
```

### in关键字

```typescript
function fn(pet: Fish | Bird ){
    if ('fly' in pet) {
      pet.fly()
    } else {
      pet.swim()
    }
}
```

### 使用typeof进行类型保护

如果子类型是只是 number、string、boolean、symbol 这几种数据类型，则可以直接使用 typeof关键字，TS 能够检测并提供类型保护。

```typescript
function padLeft(value, padding): string {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;   // let padding: number
    }
    else {
        return padding + value;   // let padding: string
    }
}
    
const t1 = padLeft("world", 6);        // "      world"
const t2 = padLeft("world", "hello "); // "hello world"
```

### 使用 instanceof 进行类型保护

```typescript
interface IA { x(): void; }
interface IB { y(): void; }

class A implements IA {
  x() { }
}
class B implements IB {
  y() { }
}

function fn(e: A | B) {
  if (e instanceof A)
    e.x(); // let e: A
  else
    e.y(); // let e: 
}
```