# TypeScript进阶 - 完整教程

## 目录
1. [高级类型](#高级类型)
2. [类型守卫](#类型守卫)
3. [装饰器](#装饰器)
4. [模块系统](#模块系统)
5. [声明文件](#声明文件)
6. [编译器API](#编译器api)

## 高级类型

### 交叉类型（Intersection Types）
```typescript
interface Person {
  name: string;
  age: number;
}

interface Employee {
  employeeId: number;
  department: string;
}

// 交叉类型：同时具有两个类型的所有属性
type Staff = Person & Employee;

const staff: Staff = {
  name: "张三",
  age: 30,
  employeeId: 1001,
  department: "技术部"
};
```

### 联合类型进阶
```typescript
// 可辨识联合类型
interface Square {
  kind: "square";
  size: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

interface Circle {
  kind: "circle";
  radius: number;
}

type Shape = Square | Rectangle | Circle;

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "square":
      return shape.size * shape.size;
    case "rectangle":
      return shape.width * shape.height;
    case "circle":
      return Math.PI * shape.radius ** 2;
  }
}
```

### 索引类型（Index Types）
```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = {
  name: "张三",
  age: 30
};

const name = getProperty(person, "name"); // string
const age = getProperty(person, "age");   // number
// getProperty(person, "email"); // 错误：email不存在
```

### 映射类型（Mapped Types）
```typescript
// 将所有属性变为可选
type Partial<T> = {
  [P in keyof T]?: T[P];
};

// 将所有属性变为只读
type Readonly<T> = {
  [P in keyof T]: readonly T[P];
};

// 将所有属性变为必填
type Required<T> = {
  [P in keyof T]-?: T[P];
};

interface User {
  name: string;
  age?: number;
}

type PartialUser = Partial<User>;     // { name?: string; age?: number; }
type ReadonlyUser = Readonly<User>;   // { readonly name: string; readonly age?: number; }
type RequiredUser = Required<User>;   // { name: string; age: number; }
```

### 条件类型（Conditional Types）
```typescript
// T extends U ? X : Y
type NonNullable<T> = T extends null | undefined ? never : T;

type A = NonNullable<string | null>;  // string
type B = NonNullable<number | undefined>; // number

// 提取函数返回类型
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;

function getUserInfo() {
  return { name: "张三", age: 30 };
}

type UserInfo = ReturnType<typeof getUserInfo>; // { name: string; age: number; }
```

### 模板字面量类型
```typescript
type EventName = "click" | "scroll" | "mousemove";
type EventHandler = `on${Capitalize<EventName>}`;
// "onClick" | "onScroll" | "onMousemove"

type PropEventSource<T> = {
  on<K extends string & keyof T>(
    eventName: `${K}Changed`,
    callback: (newValue: T[K]) => void
  ): void;
};

declare function makeWatchedObject<T>(obj: T): T & PropEventSource<T>;

const person = makeWatchedObject({
  firstName: "张",
  lastName: "三",
  age: 30
});

person.on("firstNameChanged", (newName) => {
  console.log(`新名字：${newName}`);
});
```

## 类型守卫

### typeof类型守卫
```typescript
function padLeft(value: string, padding: string | number) {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  if (typeof padding === "string") {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}
```

### instanceof类型守卫
```typescript
class Bird {
  fly() {
    console.log("鸟在飞");
  }
}

class Fish {
  swim() {
    console.log("鱼在游");
  }
}

function move(animal: Bird | Fish) {
  if (animal instanceof Bird) {
    animal.fly();
  } else {
    animal.swim();
  }
}
```

### 自定义类型守卫
```typescript
interface Cat {
  meow(): void;
}

interface Dog {
  bark(): void;
}

// 类型谓词
function isCat(animal: Cat | Dog): animal is Cat {
  return (animal as Cat).meow !== undefined;
}

function makeSound(animal: Cat | Dog) {
  if (isCat(animal)) {
    animal.meow();
  } else {
    animal.bark();
  }
}
```

### in操作符类型守卫
```typescript
interface A {
  x: number;
}

interface B {
  y: string;
}

function doStuff(q: A | B) {
  if ("x" in q) {
    // q是A类型
    console.log(q.x);
  } else {
    // q是B类型
    console.log(q.y);
  }
}
```

## 装饰器

### 类装饰器
```typescript
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }
}

// 装饰器工厂
function classDecorator<T extends { new(...args: any[]): {} }>(constructor: T) {
  return class extends constructor {
    newProperty = "new property";
    hello = "override";
  };
}

@classDecorator
class MyClass {
  hello: string;
  constructor(m: string) {
    this.hello = m;
  }
}
```

### 方法装饰器
```typescript
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`调用 ${propertyKey} 方法，参数：`, args);
    const result = originalMethod.apply(this, args);
    console.log(`${propertyKey} 返回：`, result);
    return result;
  };
  
  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(2, 3);
// 输出：
// 调用 add 方法，参数： [2, 3]
// add 返回： 5
```

### 属性装饰器
```typescript
function format(formatString: string) {
  return function(target: any, propertyKey: string) {
    let value: string;
    
    const getter = function() {
      return value;
    };
    
    const setter = function(newVal: string) {
      value = formatString.replace("{value}", newVal);
    };
    
    Object.defineProperty(target, propertyKey, {
      get: getter,
      set: setter,
      enumerable: true,
      configurable: true
    });
  };
}

class User {
  @format("姓名：{value}")
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
}

const user = new User("张三");
console.log(user.name); // "姓名：张三"
```

### 参数装饰器
```typescript
function required(target: Object, propertyKey: string, parameterIndex: number) {
  const existingRequiredParameters: number[] = 
    Reflect.getOwnMetadata("required", target, propertyKey) || [];
  existingRequiredParameters.push(parameterIndex);
  Reflect.defineMetadata("required", existingRequiredParameters, target, propertyKey);
}

function validate(target: any, propertyName: string, descriptor: PropertyDescriptor) {
  const method = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    const requiredParameters: number[] = 
      Reflect.getOwnMetadata("required", target, propertyName) || [];
    
    for (const parameterIndex of requiredParameters) {
      if (parameterIndex >= args.length || args[parameterIndex] === undefined) {
        throw new Error(`参数 ${parameterIndex} 是必需的`);
      }
    }
    
    return method.apply(this, args);
  };
}

class UserService {
  @validate
  createUser(@required name: string, age: number) {
    console.log(`创建用户：${name}, ${age}岁`);
  }
}
```

## 模块系统

### ES6模块
```typescript
// math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export default class Calculator {
  multiply(a: number, b: number): number {
    return a * b;
  }
}

// app.ts
import Calculator, { add, subtract } from './math';

const calc = new Calculator();
console.log(add(1, 2));
console.log(calc.multiply(3, 4));
```

### 命名空间
```typescript
namespace Validation {
  export interface StringValidator {
    isValid(s: string): boolean;
  }
  
  export class EmailValidator implements StringValidator {
    isValid(s: string): boolean {
      return /^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/.test(s);
    }
  }
  
  export class PhoneValidator implements StringValidator {
    isValid(s: string): boolean {
      return /^1[3-9]\d{9}$/.test(s);
    }
  }
}

const emailValidator = new Validation.EmailValidator();
console.log(emailValidator.isValid("test@example.com"));
```

### 模块解析
```typescript
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@/*": ["*"],
      "@components/*": ["components/*"],
      "@utils/*": ["utils/*"]
    }
  }
}

// 使用路径别名
import Button from '@components/Button';
import { formatDate } from '@utils/date';
```

## 声明文件

### 全局声明
```typescript
// global.d.ts
declare global {
  interface Window {
    myCustomProperty: string;
  }
  
  var MY_GLOBAL_VAR: string;
  
  function myGlobalFunction(param: string): void;
}

export {};
```

### 模块声明
```typescript
// lodash.d.ts
declare module 'lodash' {
  export function chunk<T>(array: T[], size: number): T[][];
  export function debounce<T extends (...args: any[]) => any>(
    func: T,
    wait: number
  ): T;
}

// 使用
import { chunk, debounce } from 'lodash';
```

### 第三方库声明
```typescript
// jquery.d.ts
declare namespace JQuery {
  interface AjaxSettings {
    url?: string;
    method?: string;
    data?: any;
  }
}

declare function $(selector: string): JQuery;

declare namespace $ {
  function ajax(settings: JQuery.AjaxSettings): void;
}
```

### UMD模块声明
```typescript
// my-lib.d.ts
export as namespace myLib;

export function doSomething(): void;

export interface Options {
  timeout: number;
}
```

## 编译器API

### 程序分析
```typescript
import * as ts from 'typescript';

function compile(fileNames: string[], options: ts.CompilerOptions): void {
  const program = ts.createProgram(fileNames, options);
  const emitResult = program.emit();
  
  const allDiagnostics = ts
    .getPreEmitDiagnostics(program)
    .concat(emitResult.diagnostics);
  
  allDiagnostics.forEach(diagnostic => {
    if (diagnostic.file) {
      const { line, character } = diagnostic.file.getLineAndCharacterOfPosition(
        diagnostic.start!
      );
      const message = ts.flattenDiagnosticMessageText(
        diagnostic.messageText,
        '\n'
      );
      console.log(
        `${diagnostic.file.fileName} (${line + 1},${character + 1}): ${message}`
      );
    } else {
      console.log(
        ts.flattenDiagnosticMessageText(diagnostic.messageText, '\n')
      );
    }
  });
}
```

### AST遍历
```typescript
import * as ts from 'typescript';

function visit(node: ts.Node) {
  if (ts.isFunctionDeclaration(node) && node.name) {
    console.log(`函数名：${node.name.text}`);
  }
  
  ts.forEachChild(node, visit);
}

const sourceCode = `
  function hello() {
    console.log("Hello");
  }
  
  function world() {
    console.log("World");
  }
`;

const sourceFile = ts.createSourceFile(
  'test.ts',
  sourceCode,
  ts.ScriptTarget.Latest,
  true
);

visit(sourceFile);
```

## 实战案例

### 案例1：类型安全的事件系统
```typescript
type EventMap = {
  'user:login': { userId: string; timestamp: number };
  'user:logout': { userId: string };
  'data:update': { id: string; data: any };
};

class TypedEventEmitter<T extends Record<string, any>> {
  private listeners: {
    [K in keyof T]?: Array<(data: T[K]) => void>;
  } = {};
  
  on<K extends keyof T>(event: K, listener: (data: T[K]) => void): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(listener);
  }
  
  emit<K extends keyof T>(event: K, data: T[K]): void {
    const eventListeners = this.listeners[event];
    if (eventListeners) {
      eventListeners.forEach(listener => listener(data));
    }
  }
}

const emitter = new TypedEventEmitter<EventMap>();

emitter.on('user:login', (data) => {
  console.log(`用户 ${data.userId} 在 ${data.timestamp} 登录`);
});

emitter.emit('user:login', {
  userId: '123',
  timestamp: Date.now()
});
```

### 案例2：类型安全的API客户端
```typescript
interface ApiEndpoints {
  '/users': {
    GET: { response: User[] };
    POST: { body: CreateUserDto; response: User };
  };
  '/users/:id': {
    GET: { params: { id: string }; response: User };
    PUT: { params: { id: string }; body: UpdateUserDto; response: User };
    DELETE: { params: { id: string }; response: void };
  };
}

class ApiClient<T extends Record<string, any>> {
  async request<
    Path extends keyof T,
    Method extends keyof T[Path]
  >(
    path: Path,
    method: Method,
    options?: T[Path][Method] extends { body: infer B } ? { body: B } : {}
  ): Promise<T[Path][Method] extends { response: infer R } ? R : never> {
    // 实现API请求逻辑
    throw new Error('Not implemented');
  }
}

const api = new ApiClient<ApiEndpoints>();

// 类型安全的API调用
const users = await api.request('/users', 'GET');
const newUser = await api.request('/users', 'POST', {
  body: { name: '张三', email: 'zhang@example.com' }
});
```

## 最佳实践

1. **优先使用类型推断，避免过度注解**
2. **使用严格模式（strict: true）**
3. **合理使用泛型约束**
4. **避免使用any，使用unknown代替**
5. **使用类型守卫进行类型收窄**
6. **为复杂类型创建类型别名**
7. **使用映射类型减少重复代码**
8. **装饰器需要开启experimentalDecorators**

## 下一步学习
- 类型体操：深入类型编程技巧
- 实战项目：构建类型安全的应用
- 源码阅读：学习优秀库的类型设计

---

**@author erik.zhou**
