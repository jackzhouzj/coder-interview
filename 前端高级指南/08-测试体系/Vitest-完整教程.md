# Vitest测试框架 - 完整教程

## 目录
1. [Vitest简介](#1-vitest简介)
2. [安装与配置](#2-安装与配置)
3. [基础测试](#3-基础测试)
4. [断言API](#4-断言api)
5. [Mock与Spy](#5-mock与spy)
6. [快照测试](#6-快照测试)
7. [UI模式](#7-ui模式)
8. [性能优化](#8-性能优化)
9. [从Jest迁移](#9-从jest迁移)
10. [最佳实践](#10-最佳实践)

---

## 1. Vitest简介

### 1.1 为什么选择Vitest

```javascript
/**
 * Vitest优势
 * @author erik.zhou
 */

// 1. 极速启动 - 基于Vite的HMR
// 2. 开箱即用 - 支持TypeScript、JSX、ESM
// 3. Jest兼容 - 大部分API与Jest相同
// 4. 原生ESM - 无需转译即可测试
// 5. 多线程 - 使用Worker并行运行测试
// 6. UI界面 - 内置可视化测试界面
```

### 1.2 Vitest vs Jest

```javascript
/**
 * Vitest与Jest对比
 * @author erik.zhou
 */

// 相同点：
// - API几乎完全兼容
// - 支持快照测试
// - 支持Mock和Spy
// - 支持覆盖率报告

// 不同点：
// Vitest优势：
// - 启动速度更快（基于Vite）
// - HMR支持（测试文件修改即时更新）
// - 原生ESM支持
// - 内置UI界面
// - 更好的TypeScript支持

// Jest优势：
// - 生态更成熟
// - 社区更大
// - 文档更完善
```

---

## 2. 安装与配置

### 2.1 安装Vitest

```bash
# 使用npm
npm install -D vitest

# 使用pnpm
pnpm add -D vitest

# 使用yarn
yarn add -D vitest

# 安装UI界面
npm install -D @vitest/ui

# 安装覆盖率工具
npm install -D @vitest/coverage-v8
```

### 2.2 基础配置

```javascript
/**
 * Vitest基础配置
 * @author erik.zhou
 */

// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
    test: {
        // 测试环境
        environment: 'node', // 或 'jsdom', 'happy-dom'
        
        // 全局API
        globals: true,
        
        // Setup文件
        setupFiles: ['./test/setup.ts'],
        
        // 覆盖率配置
        coverage: {
            provider: 'v8',
            reporter: ['text', 'json', 'html'],
            exclude: [
                'node_modules/',
                'test/',
                '**/*.spec.ts',
                '**/*.test.ts'
            ]
        },
        
        // 测试匹配模式
        include: ['**/*.{test,spec}.{js,mjs,cjs,ts,mts,cts,jsx,tsx}'],
        exclude: ['node_modules', 'dist', '.idea', '.git', '.cache'],
        
        // 测试超时
        testTimeout: 10000,
        hookTimeout: 10000
    }
});
```

### 2.3 与Vite集成

```javascript
/**
 * Vite + Vitest配置
 * @author erik.zhou
 */

// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [react()],
    test: {
        globals: true,
        environment: 'jsdom',
        setupFiles: './test/setup.ts',
        css: true
    },
    resolve: {
        alias: {
            '@': '/src'
        }
    }
});
```

### 2.4 TypeScript配置

```json
/**
 * TypeScript配置
 * @author erik.zhou
 */

// tsconfig.json
{
    "compilerOptions": {
        "target": "ES2020",
        "module": "ESNext",
        "lib": ["ES2020", "DOM"],
        "types": ["vitest/globals"],
        "moduleResolution": "bundler",
        "resolveJsonModule": true,
        "esModuleInterop": true,
        "skipLibCheck": true
    },
    "include": ["src", "test"]
}
```

### 2.5 Package.json脚本

```json
{
    "scripts": {
        "test": "vitest",
        "test:ui": "vitest --ui",
        "test:run": "vitest run",
        "test:coverage": "vitest run --coverage",
        "test:watch": "vitest watch"
    }
}
```

---

## 3. 基础测试

### 3.1 第一个测试

```typescript
/**
 * 基础测试示例
 * @author erik.zhou
 */

// sum.ts
export function sum(a: number, b: number): number {
    return a + b;
}

// sum.test.ts
import { describe, test, expect } from 'vitest';
import { sum } from './sum';

describe('sum function', () => {
    test('adds 1 + 2 to equal 3', () => {
        expect(sum(1, 2)).toBe(3);
    });
    
    test('adds negative numbers', () => {
        expect(sum(-1, -1)).toBe(-2);
    });
    
    test('adds zero', () => {
        expect(sum(0, 5)).toBe(5);
    });
});
```

### 3.2 测试生命周期

```typescript
/**
 * 测试生命周期示例
 * @author erik.zhou
 */

import { describe, test, beforeAll, afterAll, beforeEach, afterEach } from 'vitest';

describe('Lifecycle Hooks', () => {
    beforeAll(() => {
        console.log('在所有测试前执行一次');
    });
    
    afterAll(() => {
        console.log('在所有测试后执行一次');
    });
    
    beforeEach(() => {
        console.log('在每个测试前执行');
    });
    
    afterEach(() => {
        console.log('在每个测试后执行');
    });
    
    test('test 1', () => {
        expect(1 + 1).toBe(2);
    });
    
    test('test 2', () => {
        expect(2 + 2).toBe(4);
    });
});
```

### 3.3 测试上下文

```typescript
/**
 * 测试上下文示例
 * @author erik.zhou
 */

import { describe, test, beforeEach } from 'vitest';

interface Context {
    user: {
        id: number;
        name: string;
    };
}

describe<Context>('User Tests', () => {
    beforeEach<Context>((context) => {
        context.user = {
            id: 1,
            name: 'John'
        };
    });
    
    test<Context>('should have user in context', ({ user }) => {
        expect(user.id).toBe(1);
        expect(user.name).toBe('John');
    });
    
    test<Context>('can modify user', ({ user }) => {
        user.name = 'Jane';
        expect(user.name).toBe('Jane');
    });
});
```

### 3.4 并发测试

```typescript
/**
 * 并发测试示例
 * @author erik.zhou
 */

import { describe, test } from 'vitest';

describe.concurrent('Concurrent Tests', () => {
    test('test 1', async () => {
        await new Promise(resolve => setTimeout(resolve, 100));
        expect(1 + 1).toBe(2);
    });
    
    test('test 2', async () => {
        await new Promise(resolve => setTimeout(resolve, 100));
        expect(2 + 2).toBe(4);
    });
    
    test('test 3', async () => {
        await new Promise(resolve => setTimeout(resolve, 100));
        expect(3 + 3).toBe(6);
    });
});

// 控制并发数量
describe.concurrent('Limited Concurrency', { concurrent: 2 }, () => {
    // 最多同时运行2个测试
    test('test 1', async () => {});
    test('test 2', async () => {});
    test('test 3', async () => {});
});
```

---

## 4. 断言API

### 4.1 基础断言

```typescript
/**
 * 基础断言示例
 * @author erik.zhou
 */

import { test, expect } from 'vitest';

describe('Basic Assertions', () => {
    test('toBe - 严格相等', () => {
        expect(2 + 2).toBe(4);
        expect('hello').toBe('hello');
    });
    
    test('toEqual - 深度相等', () => {
        const obj = { name: 'John', age: 30 };
        expect(obj).toEqual({ name: 'John', age: 30 });
    });
    
    test('toStrictEqual - 严格深度相等', () => {
        const obj = { name: 'John' };
        expect(obj).toStrictEqual({ name: 'John' });
        // toStrictEqual会检查undefined属性
    });
    
    test('toBeTruthy/toBeFalsy', () => {
        expect(true).toBeTruthy();
        expect(1).toBeTruthy();
        expect('hello').toBeTruthy();
        
        expect(false).toBeFalsy();
        expect(0).toBeFalsy();
        expect('').toBeFalsy();
    });
});
```

### 4.2 数字断言

```typescript
/**
 * 数字断言示例
 * @author erik.zhou
 */

import { test, expect } from 'vitest';

describe('Number Assertions', () => {
    test('toBeGreaterThan/toBeLessThan', () => {
        expect(10).toBeGreaterThan(5);
        expect(5).toBeLessThan(10);
    });
    
    test('toBeGreaterThanOrEqual/toBeLessThanOrEqual', () => {
        expect(10).toBeGreaterThanOrEqual(10);
        expect(5).toBeLessThanOrEqual(5);
    });
    
    test('toBeCloseTo - 浮点数', () => {
        expect(0.1 + 0.2).toBeCloseTo(0.3);
    });
    
    test('toBeNaN', () => {
        expect(NaN).toBeNaN();
        expect(Number('abc')).toBeNaN();
    });
});
```

### 4.3 字符串断言

```typescript
/**
 * 字符串断言示例
 * @author erik.zhou
 */

import { test, expect } from 'vitest';

describe('String Assertions', () => {
    test('toMatch - 正则匹配', () => {
        expect('hello world').toMatch(/world/);
        expect('test@example.com').toMatch(/^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/);
    });
    
    test('toContain - 包含子串', () => {
        expect('hello world').toContain('world');
    });
    
    test('toHaveLength', () => {
        expect('hello').toHaveLength(5);
    });
});
```

### 4.4 数组与对象断言

```typescript
/**
 * 数组与对象断言示例
 * @author erik.zhou
 */

import { test, expect } from 'vitest';

describe('Array and Object Assertions', () => {
    test('toContain - 数组包含', () => {
        const fruits = ['apple', 'banana', 'orange'];
        expect(fruits).toContain('banana');
    });
    
    test('toContainEqual - 对象包含', () => {
        const users = [
            { id: 1, name: 'John' },
            { id: 2, name: 'Jane' }
        ];
        expect(users).toContainEqual({ id: 1, name: 'John' });
    });
    
    test('toHaveLength', () => {
        expect([1, 2, 3]).toHaveLength(3);
    });
    
    test('toHaveProperty', () => {
        const user = { id: 1, name: 'John', age: 30 };
        expect(user).toHaveProperty('name');
        expect(user).toHaveProperty('name', 'John');
    });
    
    test('toMatchObject', () => {
        const user = { id: 1, name: 'John', age: 30 };
        expect(user).toMatchObject({
            name: 'John',
            age: 30
        });
    });
});
```

### 4.5 Promise断言

```typescript
/**
 * Promise断言示例
 * @author erik.zhou
 */

import { test, expect } from 'vitest';

async function fetchUser(id: number) {
    if (id > 0) {
        return { id, name: 'John' };
    }
    throw new Error('Invalid ID');
}

describe('Promise Assertions', () => {
    test('resolves', async () => {
        await expect(fetchUser(1)).resolves.toEqual({ id: 1, name: 'John' });
    });
    
    test('rejects', async () => {
        await expect(fetchUser(-1)).rejects.toThrow('Invalid ID');
    });
    
    test('async/await', async () => {
        const user = await fetchUser(1);
        expect(user).toEqual({ id: 1, name: 'John' });
    });
});
```

---

## 5. Mock与Spy

### 5.1 Mock函数

```typescript
/**
 * Mock函数示例
 * @author erik.zhou
 */

import { test, expect, vi } from 'vitest';

describe('Mock Functions', () => {
    test('should create mock function', () => {
        const mockFn = vi.fn();
        
        mockFn('hello');
        mockFn('world');
        
        expect(mockFn).toHaveBeenCalledTimes(2);
        expect(mockFn).toHaveBeenCalledWith('hello');
        expect(mockFn).toHaveBeenLastCalledWith('world');
    });
    
    test('should mock return value', () => {
        const mockFn = vi.fn();
        mockFn.mockReturnValue('mocked');
        
        expect(mockFn()).toBe('mocked');
    });
    
    test('should mock return value once', () => {
        const mockFn = vi.fn();
        mockFn
            .mockReturnValueOnce('first')
            .mockReturnValueOnce('second')
            .mockReturnValue('default');
        
        expect(mockFn()).toBe('first');
        expect(mockFn()).toBe('second');
        expect(mockFn()).toBe('default');
    });
    
    test('should mock implementation', () => {
        const mockFn = vi.fn((x: number, y: number) => x + y);
        
        expect(mockFn(2, 3)).toBe(5);
        expect(mockFn).toHaveBeenCalledWith(2, 3);
    });
});
```

### 5.2 Mock模块

```typescript
/**
 * Mock模块示例
 * @author erik.zhou
 */

import { test, expect, vi } from 'vitest';

// userService.ts
import axios from 'axios';

export async function getUser(id: number) {
    const response = await axios.get(`/api/users/${id}`);
    return response.data;
}

// userService.test.ts
vi.mock('axios');

describe('User Service', () => {
    test('should fetch user', async () => {
        const mockUser = { id: 1, name: 'John' };
        
        vi.mocked(axios.get).mockResolvedValue({ data: mockUser });
        
        const user = await getUser(1);
        
        expect(user).toEqual(mockUser);
        expect(axios.get).toHaveBeenCalledWith('/api/users/1');
    });
});
```

### 5.3 部分Mock

```typescript
/**
 * 部分Mock示例
 * @author erik.zhou
 */

import { test, expect, vi } from 'vitest';

// utils.ts
export const utils = {
    add: (a: number, b: number) => a + b,
    multiply: (a: number, b: number) => a * b
};

// utils.test.ts
vi.mock('./utils', async () => {
    const actual = await vi.importActual<typeof import('./utils')>('./utils');
    return {
        ...actual,
        multiply: vi.fn((a: number, b: number) => a * b * 2)
    };
});

describe('Utils', () => {
    test('should use real add', () => {
        expect(utils.add(2, 3)).toBe(5);
    });
    
    test('should use mocked multiply', () => {
        expect(utils.multiply(2, 3)).toBe(12);
    });
});
```

### 5.4 Spy函数

```typescript
/**
 * Spy函数示例
 * @author erik.zhou
 */

import { test, expect, vi } from 'vitest';

const calculator = {
    add: (a: number, b: number) => a + b,
    multiply: (a: number, b: number) => a * b
};

describe('Spy Functions', () => {
    test('should spy on method', () => {
        const spy = vi.spyOn(calculator, 'add');
        
        const result = calculator.add(2, 3);
        
        expect(result).toBe(5);
        expect(spy).toHaveBeenCalledWith(2, 3);
        
        spy.mockRestore();
    });
    
    test('should spy and mock implementation', () => {
        const spy = vi.spyOn(calculator, 'multiply')
            .mockImplementation((a, b) => a * b * 2);
        
        expect(calculator.multiply(2, 3)).toBe(12);
        
        spy.mockRestore();
        expect(calculator.multiply(2, 3)).toBe(6);
    });
});
```

### 5.5 定时器Mock

```typescript
/**
 * 定时器Mock示例
 * @author erik.zhou
 */

import { test, expect, vi, beforeEach, afterEach } from 'vitest';

describe('Timer Mocks', () => {
    beforeEach(() => {
        vi.useFakeTimers();
    });
    
    afterEach(() => {
        vi.useRealTimers();
    });
    
    test('should fast forward time', () => {
        const callback = vi.fn();
        
        setTimeout(callback, 1000);
        
        vi.advanceTimersByTime(1000);
        
        expect(callback).toHaveBeenCalledTimes(1);
    });
    
    test('should run all timers', () => {
        const callback = vi.fn();
        
        setTimeout(callback, 1000);
        setTimeout(callback, 2000);
        
        vi.runAllTimers();
        
        expect(callback).toHaveBeenCalledTimes(2);
    });
    
    test('should get system time', () => {
        const now = Date.now();
        
        vi.setSystemTime(new Date('2024-01-01'));
        
        expect(Date.now()).toBe(new Date('2024-01-01').getTime());
    });
});
```



---

## 6. 快照测试

### 6.1 基础快照

```typescript
/**
 * 快照测试示例
 * @author erik.zhou
 */

import { test, expect } from 'vitest';

interface ButtonProps {
    text: string;
    onClick: () => void;
}

function Button({ text, onClick }: ButtonProps) {
    return {
        type: 'button',
        props: {
            className: 'btn',
            onClick,
            children: text
        }
    };
}

describe('Button Snapshot', () => {
    test('should match snapshot', () => {
        const button = Button({ text: 'Click me', onClick: () => {} });
        expect(button).toMatchSnapshot();
    });
    
    test('should match inline snapshot', () => {
        const button = Button({ text: 'Submit', onClick: () => {} });
        expect(button).toMatchInlineSnapshot(`
            {
              "props": {
                "children": "Submit",
                "className": "btn",
                "onClick": [Function],
              },
              "type": "button",
            }
        `);
    });
});
```

### 6.2 文件快照

```typescript
/**
 * 文件快照示例
 * @author erik.zhou
 */

import { test, expect } from 'vitest';
import fs from 'fs';

describe('File Snapshot', () => {
    test('should match file snapshot', () => {
        const content = fs.readFileSync('./config.json', 'utf-8');
        expect(content).toMatchFileSnapshot('./snapshots/config.snap');
    });
});
```

### 6.3 自定义序列化器

```typescript
/**
 * 自定义序列化器示例
 * @author erik.zhou
 */

import { test, expect } from 'vitest';

// 自定义序列化器
expect.addSnapshotSerializer({
    test: (val) => val && typeof val === 'object' && 'type' in val,
    serialize: (val, config, indentation, depth, refs, printer) => {
        return `Component<${val.type}>`;
    }
});

describe('Custom Serializer', () => {
    test('should use custom serializer', () => {
        const component = { type: 'Button', props: {} };
        expect(component).toMatchInlineSnapshot(`"Component<Button>"`);
    });
});
```

---

## 7. UI模式

### 7.1 启动UI模式

```bash
# 启动UI界面
npm run test:ui

# 或直接运行
npx vitest --ui
```

### 7.2 UI功能

```typescript
/**
 * UI模式功能
 * @author erik.zhou
 */

// UI模式提供：
// 1. 可视化测试结果
// 2. 测试文件树状结构
// 3. 实时测试执行
// 4. 代码覆盖率可视化
// 5. 测试过滤和搜索
// 6. 测试详情查看
// 7. 错误堆栈追踪
// 8. 快照对比
```

---

## 8. 性能优化

### 8.1 并行执行

```typescript
/**
 * 并行执行配置
 * @author erik.zhou
 */

// vitest.config.ts
export default defineConfig({
    test: {
        // 启用多线程
        threads: true,
        
        // 最大线程数
        maxThreads: 4,
        
        // 最小线程数
        minThreads: 1,
        
        // 单个测试文件的隔离
        isolate: true
    }
});
```

### 8.2 测试过滤

```typescript
/**
 * 测试过滤示例
 * @author erik.zhou
 */

import { test, describe } from 'vitest';

// 只运行这个测试
test.only('should run only this', () => {
    expect(true).toBe(true);
});

// 跳过这个测试
test.skip('should skip this', () => {
    expect(true).toBe(false);
});

// 条件跳过
test.skipIf(process.env.CI)('skip in CI', () => {
    expect(true).toBe(true);
});

// 条件运行
test.runIf(!process.env.CI)('run locally', () => {
    expect(true).toBe(true);
});
```

### 8.3 测试超时

```typescript
/**
 * 测试超时配置
 * @author erik.zhou
 */

import { test } from 'vitest';

// 单个测试超时
test('should timeout', async () => {
    await new Promise(resolve => setTimeout(resolve, 100));
}, 5000); // 5秒超时

// 全局超时配置
// vitest.config.ts
export default defineConfig({
    test: {
        testTimeout: 10000,
        hookTimeout: 10000
    }
});
```

### 8.4 缓存优化

```typescript
/**
 * 缓存优化配置
 * @author erik.zhou
 */

// vitest.config.ts
export default defineConfig({
    test: {
        // 缓存目录
        cache: {
            dir: 'node_modules/.vitest'
        },
        
        // 监听模式下的文件变化
        watchExclude: [
            '**/node_modules/**',
            '**/dist/**'
        ]
    }
});
```

---

## 9. 从Jest迁移

### 9.1 API兼容性

```typescript
/**
 * Jest到Vitest迁移
 * @author erik.zhou
 */

// Jest代码
import { describe, test, expect, jest } from '@jest/globals';

// Vitest代码（几乎相同）
import { describe, test, expect, vi } from 'vitest';

// 主要区别：
// 1. jest.fn() -> vi.fn()
// 2. jest.mock() -> vi.mock()
// 3. jest.spyOn() -> vi.spyOn()
// 4. jest.useFakeTimers() -> vi.useFakeTimers()
```

### 9.2 配置迁移

```javascript
/**
 * 配置迁移示例
 * @author erik.zhou
 */

// jest.config.js
module.exports = {
    testEnvironment: 'jsdom',
    setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
    moduleNameMapper: {
        '^@/(.*)$': '<rootDir>/src/$1'
    },
    collectCoverageFrom: ['src/**/*.{js,jsx,ts,tsx}']
};

// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
    test: {
        environment: 'jsdom',
        setupFiles: ['./test/setup.ts'],
        alias: {
            '@': '/src'
        },
        coverage: {
            include: ['src/**/*.{js,jsx,ts,tsx}']
        }
    }
});
```

### 9.3 全局API迁移

```typescript
/**
 * 全局API迁移
 * @author erik.zhou
 */

// Jest - 需要导入
import { describe, test, expect } from '@jest/globals';

// Vitest - 可以使用全局API
// vitest.config.ts
export default defineConfig({
    test: {
        globals: true
    }
});

// 然后可以直接使用
describe('test', () => {
    test('should work', () => {
        expect(true).toBe(true);
    });
});
```

### 9.4 Mock迁移

```typescript
/**
 * Mock迁移示例
 * @author erik.zhou
 */

// Jest
jest.mock('./module');
const mockFn = jest.fn();
jest.spyOn(obj, 'method');

// Vitest
vi.mock('./module');
const mockFn = vi.fn();
vi.spyOn(obj, 'method');

// 自动替换
// 可以使用codemod工具自动迁移
// npx vitest-codemods
```

---

## 10. 最佳实践

### 10.1 测试组织

```typescript
/**
 * 测试组织最佳实践
 * @author erik.zhou
 */

// ✅ 好的实践 - 清晰的测试结构
describe('UserService', () => {
    describe('getUser', () => {
        test('should return user when ID is valid', async () => {
            // Arrange
            const userId = 1;
            
            // Act
            const user = await userService.getUser(userId);
            
            // Assert
            expect(user).toBeDefined();
            expect(user.id).toBe(userId);
        });
        
        test('should throw error when ID is invalid', async () => {
            await expect(userService.getUser(-1))
                .rejects
                .toThrow('Invalid user ID');
        });
    });
});

// ❌ 不好的实践 - 混乱的测试
test('user tests', async () => {
    const user1 = await userService.getUser(1);
    expect(user1).toBeDefined();
    
    const user2 = await userService.createUser({ name: 'Jane' });
    expect(user2).toBeDefined();
});
```

### 10.2 使用TypeScript

```typescript
/**
 * TypeScript最佳实践
 * @author erik.zhou
 */

import { test, expect } from 'vitest';

// ✅ 好的实践 - 类型安全
interface User {
    id: number;
    name: string;
    email: string;
}

async function getUser(id: number): Promise<User> {
    return {
        id,
        name: 'John',
        email: 'john@example.com'
    };
}

test('should return typed user', async () => {
    const user = await getUser(1);
    
    // TypeScript会检查类型
    expect(user.id).toBe(1);
    expect(user.name).toBe('John');
    expect(user.email).toBe('john@example.com');
});
```

### 10.3 测试隔离

```typescript
/**
 * 测试隔离最佳实践
 * @author erik.zhou
 */

import { describe, test, beforeEach, afterEach } from 'vitest';

// ✅ 好的实践 - 每个测试独立
describe('Database Tests', () => {
    let db: Database;
    
    beforeEach(async () => {
        db = await createTestDatabase();
        await db.seed();
    });
    
    afterEach(async () => {
        await db.cleanup();
    });
    
    test('should insert user', async () => {
        const user = await db.users.insert({ name: 'John' });
        expect(user).toBeDefined();
    });
    
    test('should delete user', async () => {
        const user = await db.users.insert({ name: 'John' });
        await db.users.delete(user.id);
        
        const found = await db.users.findById(user.id);
        expect(found).toBeNull();
    });
});
```

### 10.4 Mock最小化

```typescript
/**
 * Mock最小化最佳实践
 * @author erik.zhou
 */

import { test, expect, vi } from 'vitest';

// ✅ 好的实践 - 只Mock外部依赖
vi.mock('./api', () => ({
    fetchUser: vi.fn()
}));

import { fetchUser } from './api';
import { getUserProfile } from './userService';

test('should get user profile', async () => {
    vi.mocked(fetchUser).mockResolvedValue({
        id: 1,
        name: 'John'
    });
    
    const profile = await getUserProfile(1);
    
    expect(profile).toBeDefined();
    expect(fetchUser).toHaveBeenCalledWith(1);
});

// ❌ 不好的实践 - 过度Mock
test('should add numbers', () => {
    const add = vi.fn((a, b) => a + b);
    expect(add(2, 3)).toBe(5);
});
```

### 10.5 测试覆盖率

```typescript
/**
 * 测试覆盖率最佳实践
 * @author erik.zhou
 */

// vitest.config.ts
export default defineConfig({
    test: {
        coverage: {
            provider: 'v8',
            reporter: ['text', 'json', 'html'],
            
            // 覆盖率阈值
            thresholds: {
                lines: 80,
                functions: 80,
                branches: 80,
                statements: 80
            },
            
            // 排除文件
            exclude: [
                'node_modules/',
                'test/',
                '**/*.spec.ts',
                '**/*.test.ts',
                '**/*.config.ts',
                '**/types.ts'
            ]
        }
    }
});

// 优先测试关键路径
describe('Payment Service', () => {
    // ✅ 必须测试：核心业务逻辑
    test('should process payment', async () => {
        const result = await processPayment({
            amount: 100,
            currency: 'USD'
        });
        expect(result.status).toBe('success');
    });
    
    // ✅ 必须测试：错误处理
    test('should handle payment failure', async () => {
        await expect(processPayment({
            amount: -100,
            currency: 'USD'
        })).rejects.toThrow();
    });
});
```

### 10.6 Watch模式优化

```typescript
/**
 * Watch模式优化
 * @author erik.zhou
 */

// vitest.config.ts
export default defineConfig({
    test: {
        // Watch模式配置
        watch: true,
        
        // 排除监听的文件
        watchExclude: [
            '**/node_modules/**',
            '**/dist/**',
            '**/.git/**'
        ],
        
        // 文件变化后重新运行测试
        forceRerunTriggers: [
            '**/vitest.config.*/**',
            '**/vite.config.*/**'
        ]
    }
});
```

### 10.7 环境变量

```typescript
/**
 * 环境变量最佳实践
 * @author erik.zhou
 */

// test/setup.ts
import { beforeAll } from 'vitest';

beforeAll(() => {
    // 设置测试环境变量
    process.env.NODE_ENV = 'test';
    process.env.API_URL = 'http://localhost:3000';
});

// vitest.config.ts
export default defineConfig({
    test: {
        env: {
            NODE_ENV: 'test',
            API_URL: 'http://localhost:3000'
        }
    }
});
```

### 10.8 测试工具函数

```typescript
/**
 * 测试工具函数
 * @author erik.zhou
 */

// test/utils.ts
export function createMockUser(overrides = {}) {
    return {
        id: 1,
        name: 'John',
        email: 'john@example.com',
        ...overrides
    };
}

export async function waitFor(
    callback: () => boolean,
    timeout = 1000
): Promise<void> {
    const startTime = Date.now();
    
    while (Date.now() - startTime < timeout) {
        if (callback()) {
            return;
        }
        await new Promise(resolve => setTimeout(resolve, 50));
    }
    
    throw new Error('Timeout waiting for condition');
}

// 使用工具函数
import { test, expect } from 'vitest';
import { createMockUser, waitFor } from './test/utils';

test('should use mock user', () => {
    const user = createMockUser({ name: 'Jane' });
    expect(user.name).toBe('Jane');
    expect(user.email).toBe('john@example.com');
});

test('should wait for condition', async () => {
    let ready = false;
    setTimeout(() => { ready = true; }, 100);
    
    await waitFor(() => ready);
    expect(ready).toBe(true);
});
```

---

## 总结

本教程全面介绍了Vitest测试框架的核心功能和最佳实践：

1. **Vitest简介**：优势、与Jest对比
2. **安装与配置**：基础配置、Vite集成、TypeScript支持
3. **基础测试**：第一个测试、生命周期、测试上下文、并发测试
4. **断言API**：基础断言、数字/字符串/数组/对象/Promise断言
5. **Mock与Spy**：Mock函数、Mock模块、部分Mock、Spy函数、定时器Mock
6. **快照测试**：基础快照、文件快照、自定义序列化器
7. **UI模式**：可视化测试界面
8. **性能优化**：并行执行、测试过滤、超时配置、缓存优化
9. **从Jest迁移**：API兼容性、配置迁移、全局API、Mock迁移
10. **最佳实践**：测试组织、TypeScript、测试隔离、Mock最小化、覆盖率、工具函数

通过本教程的学习，你将能够使用Vitest编写高效、可维护的测试代码。

---

**@author erik.zhou**
**最后更新时间：2026-03-03**
