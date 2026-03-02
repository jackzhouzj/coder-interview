# TypeScript基础 - 完整教程

## 目录
1. [TypeScript简介](#typescript简介)
2. [环境搭建](#环境搭建)
3. [基本类型](#基本类型)
4. [接口](#接口)
5. [类](#类)
6. [函数](#函数)
7. [泛型入门](#泛型入门)

## TypeScript简介

### 什么是TypeScript
TypeScript是JavaScript的超集，添加了静态类型系统和其他特性。

### 为什么使用TypeScript
- 类型安全：编译时发现错误
- 更好的IDE支持：智能提示、重构
- 代码可维护性：类型即文档
- 渐进式采用：可以逐步迁移

### TypeScript vs JavaScript
```typescript
// JavaScript
function add(a, b) {
  return a + b;
}
add(1, '2'); // "12" - 运行时才发现问题

// TypeScript
function add(a: number, b: number): number {
  return a + b;
}
add(1, '2'); // 编译错误：类型不匹配
```

## 环境搭建

### 安装TypeScript
```bash
# 全局安装
npm install -g typescript

# 项目安装
npm install --save-dev typescript

# 查看版本
tsc --version
```

### 初始化项目
```bash
# 创建tsconfig.json
tsc --init

# 编译TypeScript文件
tsc index.ts

# 监听模式
tsc --watch
```

### 基础tsconfig.json配置
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## 基本类型

### 原始类型
```typescript
// 布尔值
let isDone: boolean = false;

// 数字
let decimal: number = 6;
let hex: number = 0xf00d;
let binary: number = 0b1010;

// 字符串
let color: string = "blue";
let fullName: string = `Bob Bobbington`;

// 数组
let list1: number[] = [1, 2, 3];
let list2: Array<number> = [1, 2, 3];

// 元组
let x: [string, number] = ["hello", 10];

// 枚举
enum Color {
  Red,
  Green,
  Blue
}
let c: Color = Color.Green;

// Any类型（尽量避免使用）
let notSure: any = 4;
notSure = "maybe a string";

// Void类型
function warnUser(): void {
  console.log("This is a warning message");
}

// Null和Undefined
let u: undefined = undefined;
let n: null = null;

// Never类型
function error(message: string): never {
  throw new Error(message);
}
```

### 类型断言
```typescript
// 尖括号语法
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;

// as语法（推荐，JSX中只能用这种）
let someValue2: any = "this is a string";
let strLength2: number = (someValue2 as string).length;
```

### 联合类型
```typescript
let value: string | number;
value = "hello";
value = 123;

function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id);
  }
}
```

### 字面量类型
```typescript
let direction: "left" | "right" | "up" | "down";
direction = "left"; // ✓
// direction = "forward"; // ✗ 编译错误

type Status = "pending" | "success" | "error";
let status: Status = "pending";
```

## 接口

### 基本接口
```typescript
interface User {
  name: string;
  age: number;
  email?: string; // 可选属性
  readonly id: number; // 只读属性
}

const user: User = {
  id: 1,
  name: "张三",
  age: 25
};

// user.id = 2; // 错误：只读属性
```

### 函数类型接口
```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}

const mySearch: SearchFunc = function(src, sub) {
  return src.search(sub) > -1;
};
```

### 可索引类型
```typescript
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray = ["Bob", "Fred"];

interface StringMap {
  [key: string]: string;
}

let myMap: StringMap = {
  name: "张三",
  city: "北京"
};
```

### 接口继承
```typescript
interface Shape {
  color: string;
}

interface Square extends Shape {
  sideLength: number;
}

let square: Square = {
  color: "blue",
  sideLength: 10
};

// 多重继承
interface PenStroke {
  penWidth: number;
}

interface ColoredSquare extends Shape, PenStroke {
  sideLength: number;
}
```

## 类

### 基本类定义
```typescript
class Person {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  greet(): string {
    return `Hello, I'm ${this.name}`;
  }
}

const person = new Person("张三", 25);
console.log(person.greet());
```

### 访问修饰符
```typescript
class Animal {
  public name: string;      // 公共（默认）
  private age: number;      // 私有
  protected species: string; // 受保护

  constructor(name: string, age: number, species: string) {
    this.name = name;
    this.age = age;
    this.species = species;
  }

  public getAge(): number {
    return this.age;
  }
}

class Dog extends Animal {
  bark(): void {
    console.log(`${this.name} (${this.species}) is barking`);
    // console.log(this.age); // 错误：私有属性
  }
}
```

### 参数属性简写
```typescript
class User {
  constructor(
    public name: string,
    private age: number,
    protected email: string
  ) {}
}

// 等价于
class User2 {
  public name: string;
  private age: number;
  protected email: string;

  constructor(name: string, age: number, email: string) {
    this.name = name;
    this.age = age;
    this.email = email;
  }
}
```

### 抽象类
```typescript
abstract class Department {
  constructor(public name: string) {}

  printName(): void {
    console.log("Department name: " + this.name);
  }

  abstract printMeeting(): void; // 必须在派生类中实现
}

class AccountingDepartment extends Department {
  constructor() {
    super("Accounting");
  }

  printMeeting(): void {
    console.log("The Accounting Department meets each Monday at 10am.");
  }
}
```

## 函数

### 函数类型
```typescript
// 函数声明
function add(x: number, y: number): number {
  return x + y;
}

// 函数表达式
const multiply: (x: number, y: number) => number = function(x, y) {
  return x * y;
};

// 箭头函数
const divide = (x: number, y: number): number => x / y;
```

### 可选参数和默认参数
```typescript
// 可选参数
function buildName(firstName: string, lastName?: string): string {
  if (lastName) {
    return firstName + " " + lastName;
  }
  return firstName;
}

// 默认参数
function buildName2(firstName: string, lastName: string = "Smith"): string {
  return firstName + " " + lastName;
}

// 剩余参数
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, num) => acc + num, 0);
}
```

### 函数重载
```typescript
function reverse(x: string): string;
function reverse(x: number): number;
function reverse(x: string | number): string | number {
  if (typeof x === "string") {
    return x.split("").reverse().join("");
  }
  return Number(x.toString().split("").reverse().join(""));
}

console.log(reverse("hello")); // "olleh"
console.log(reverse(12345));   // 54321
```

## 泛型入门

### 泛型函数
```typescript
// 不使用泛型
function identity(arg: any): any {
  return arg;
}

// 使用泛型
function identity<T>(arg: T): T {
  return arg;
}

let output1 = identity<string>("myString");
let output2 = identity<number>(123);
let output3 = identity("myString"); // 类型推断
```

### 泛型接口
```typescript
interface GenericIdentityFn<T> {
  (arg: T): T;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

### 泛型类
```typescript
class GenericNumber<T> {
  zeroValue: T;
  add: (x: T, y: T) => T;

  constructor(zeroValue: T, add: (x: T, y: T) => T) {
    this.zeroValue = zeroValue;
    this.add = add;
  }
}

let myGenericNumber = new GenericNumber<number>(0, (x, y) => x + y);
```

### 泛型约束
```typescript
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}

loggingIdentity("hello"); // ✓
loggingIdentity([1, 2, 3]); // ✓
// loggingIdentity(3); // ✗ 错误：number没有length属性
```

## 实战练习

### 练习1：用户管理系统
```typescript
interface IUser {
  id: number;
  name: string;
  email: string;
  role: "admin" | "user" | "guest";
}

class UserManager {
  private users: IUser[] = [];

  addUser(user: IUser): void {
    this.users.push(user);
  }

  getUserById(id: number): IUser | undefined {
    return this.users.find(user => user.id === id);
  }

  getUsersByRole(role: IUser["role"]): IUser[] {
    return this.users.filter(user => user.role === role);
  }
}
```

### 练习2：泛型数据容器
```typescript
class DataContainer<T> {
  private data: T[] = [];

  add(item: T): void {
    this.data.push(item);
  }

  get(index: number): T | undefined {
    return this.data[index];
  }

  getAll(): T[] {
    return [...this.data];
  }

  filter(predicate: (item: T) => boolean): T[] {
    return this.data.filter(predicate);
  }
}

const numberContainer = new DataContainer<number>();
numberContainer.add(1);
numberContainer.add(2);
```

## 常见问题

### Q1: any和unknown的区别？
```typescript
// any：完全放弃类型检查
let value1: any = 10;
value1.foo.bar; // 不报错

// unknown：类型安全的any
let value2: unknown = 10;
// value2.foo.bar; // 报错
if (typeof value2 === "string") {
  console.log(value2.toUpperCase()); // 类型守卫后可用
}
```

### Q2: interface和type的区别？
```typescript
// interface可以声明合并
interface User {
  name: string;
}
interface User {
  age: number;
}
// User现在有name和age

// type不能重复声明
type User2 = {
  name: string;
};
// type User2 = { age: number; }; // 错误

// type可以表示联合类型
type ID = string | number;

// interface可以被类实现
interface IAnimal {
  name: string;
  makeSound(): void;
}
class Dog implements IAnimal {
  name: string = "Dog";
  makeSound() {
    console.log("Woof!");
  }
}
```

## 最佳实践

1. **优先使用interface定义对象类型**
2. **使用type定义联合类型、交叉类型**
3. **避免使用any，使用unknown代替**
4. **开启strict模式**
5. **为函数参数和返回值添加类型**
6. **使用类型守卫进行类型收窄**
7. **合理使用泛型提高代码复用性**

## 下一步学习
- TypeScript进阶：高级类型、装饰器
- 类型体操：深入类型编程
- 实战项目：将JavaScript项目迁移到TypeScript

---

**@author erik.zhou**
