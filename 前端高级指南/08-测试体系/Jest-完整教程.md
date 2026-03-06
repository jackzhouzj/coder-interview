# Jest测试框架 - 完整教程

## 目录
1. [Jest基础](#1-jest基础)
2. [测试配置](#2-测试配置)
3. [匹配器与断言](#3-匹配器与断言)
4. [异步测试](#4-异步测试)
5. [Mock函数](#5-mock函数)
6. [Mock模块](#6-mock模块)
7. [快照测试](#7-快照测试)
8. [测试覆盖率](#8-测试覆盖率)
9. [高级特性](#9-高级特性)
10. [最佳实践](#10-最佳实践)

---

## 1. Jest基础

### 1.1 安装与初始化

```bash
# 安装Jest
npm install --save-dev jest

# 安装TypeScript支持
npm install --save-dev @types/jest ts-jest

# 初始化配置
npx jest --init
```

### 1.2 第一个测试

```javascript
/**
 * 基础测试示例
 * @author erik.zhou
 */

// sum.js
function sum(a, b) {
    return a + b;
}

module.exports = sum;

// sum.test.js
const sum = require('./sum');

describe('sum function', () => {
    test('adds 1 + 2 to equal 3', () => {
        expect(sum(1, 2)).toBe(3);
    });
    
    test('adds -1 + 1 to equal 0', () => {
        expect(sum(-1, 1)).toBe(0);
    });
});
```

### 1.3 测试结构

```javascript
/**
 * 测试结构示例
 * @author erik.zhou
 */

describe('Calculator', () => {
    // 测试套件级别的setup
    beforeAll(() => {
        console.log('在所有测试前执行一次');
    });
    
    afterAll(() => {
        console.log('在所有测试后执行一次');
    });
    
    // 每个测试前后执行
    beforeEach(() => {
        console.log('在每个测试前执行');
    });
    
    afterEach(() => {
        console.log('在每个测试后执行');
    });
    
    describe('addition', () => {
        test('should add two positive numbers', () => {
            expect(2 + 2).toBe(4);
        });
        
        test('should add negative numbers', () => {
            expect(-1 + -1).toBe(-2);
        });
    });
    
    describe('subtraction', () => {
        test('should subtract numbers', () => {
            expect(5 - 3).toBe(2);
        });
    });
});
```



---

## 2. 测试配置

### 2.1 jest.config.js配置

```javascript
/**
 * Jest配置文件
 * @author erik.zhou
 */

module.exports = {
    // 测试环境
    testEnvironment: 'node', // 或 'jsdom' 用于浏览器环境
    
    // 测试文件匹配模式
    testMatch: [
        '**/__tests__/**/*.[jt]s?(x)',
        '**/?(*.)+(spec|test).[jt]s?(x)'
    ],
    
    // 忽略的文件
    testPathIgnorePatterns: [
        '/node_modules/',
        '/dist/',
        '/build/'
    ],
    
    // 覆盖率配置
    collectCoverageFrom: [
        'src/**/*.{js,jsx,ts,tsx}',
        '!src/**/*.d.ts',
        '!src/index.js'
    ],
    
    // 覆盖率阈值
    coverageThreshold: {
        global: {
            branches: 80,
            functions: 80,
            lines: 80,
            statements: 80
        }
    },
    
    // 模块路径映射
    moduleNameMapper: {
        '^@/(.*)$': '<rootDir>/src/$1',
        '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
        '\\.(jpg|jpeg|png|gif|svg)$': '<rootDir>/__mocks__/fileMock.js'
    },
    
    // 转换配置
    transform: {
        '^.+\\.(ts|tsx)$': 'ts-jest',
        '^.+\\.(js|jsx)$': 'babel-jest'
    },
    
    // Setup文件
    setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
    
    // 清除mock
    clearMocks: true,
    
    // 详细输出
    verbose: true
};
```

### 2.2 TypeScript配置

```javascript
/**
 * TypeScript + Jest配置
 * @author erik.zhou
 */

// jest.config.js
module.exports = {
    preset: 'ts-jest',
    testEnvironment: 'node',
    roots: ['<rootDir>/src'],
    testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],
    transform: {
        '^.+\\.ts$': 'ts-jest'
    },
    moduleFileExtensions: ['ts', 'js', 'json', 'node'],
    globals: {
        'ts-jest': {
            tsconfig: {
                esModuleInterop: true,
                allowSyntheticDefaultImports: true
            }
        }
    }
};

// tsconfig.json
{
    "compilerOptions": {
        "target": "ES2020",
        "module": "commonjs",
        "lib": ["ES2020"],
        "types": ["jest", "node"],
        "esModuleInterop": true,
        "skipLibCheck": true
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules", "**/*.spec.ts", "**/*.test.ts"]
}
```

### 2.3 Setup文件

```javascript
/**
 * Jest Setup文件
 * @author erik.zhou
 */

// jest.setup.js

// 扩展匹配器
import '@testing-library/jest-dom';

// 全局变量
global.API_URL = 'http://localhost:3000';

// Mock全局对象
global.localStorage = {
    getItem: jest.fn(),
    setItem: jest.fn(),
    removeItem: jest.fn(),
    clear: jest.fn()
};

// 自定义匹配器
expect.extend({
    toBeWithinRange(received, floor, ceiling) {
        const pass = received >= floor && received <= ceiling;
        if (pass) {
            return {
                message: () => `expected ${received} not to be within range ${floor} - ${ceiling}`,
                pass: true
            };
        } else {
            return {
                message: () => `expected ${received} to be within range ${floor} - ${ceiling}`,
                pass: false
            };
        }
    }
});

// 全局beforeEach
beforeEach(() => {
    jest.clearAllMocks();
});

// 全局afterEach
afterEach(() => {
    jest.restoreAllMocks();
});
```

---

## 3. 匹配器与断言

### 3.1 基础匹配器

```javascript
/**
 * 基础匹配器示例
 * @author erik.zhou
 */

describe('Basic Matchers', () => {
    test('toBe - 严格相等', () => {
        expect(2 + 2).toBe(4);
        expect('hello').toBe('hello');
    });
    
    test('toEqual - 深度相等', () => {
        const data = { name: 'John', age: 30 };
        expect(data).toEqual({ name: 'John', age: 30 });
    });
    
    test('not - 取反', () => {
        expect(2 + 2).not.toBe(5);
    });
    
    test('toBeTruthy/toBeFalsy', () => {
        expect(true).toBeTruthy();
        expect(1).toBeTruthy();
        expect('hello').toBeTruthy();
        
        expect(false).toBeFalsy();
        expect(0).toBeFalsy();
        expect('').toBeFalsy();
        expect(null).toBeFalsy();
        expect(undefined).toBeFalsy();
    });
    
    test('toBeNull/toBeUndefined/toBeDefined', () => {
        expect(null).toBeNull();
        expect(undefined).toBeUndefined();
        expect('hello').toBeDefined();
    });
});
```

### 3.2 数字匹配器

```javascript
/**
 * 数字匹配器示例
 * @author erik.zhou
 */

describe('Number Matchers', () => {
    test('toBeGreaterThan/toBeLessThan', () => {
        expect(10).toBeGreaterThan(5);
        expect(5).toBeLessThan(10);
    });
    
    test('toBeGreaterThanOrEqual/toBeLessThanOrEqual', () => {
        expect(10).toBeGreaterThanOrEqual(10);
        expect(5).toBeLessThanOrEqual(5);
    });
    
    test('toBeCloseTo - 浮点数比较', () => {
        expect(0.1 + 0.2).toBeCloseTo(0.3);
        expect(0.1 + 0.2).not.toBe(0.3); // 浮点数精度问题
    });
});
```

### 3.3 字符串匹配器

```javascript
/**
 * 字符串匹配器示例
 * @author erik.zhou
 */

describe('String Matchers', () => {
    test('toMatch - 正则匹配', () => {
        expect('hello world').toMatch(/world/);
        expect('hello@example.com').toMatch(/^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/);
    });
    
    test('toContain - 包含子串', () => {
        expect('hello world').toContain('world');
    });
    
    test('toHaveLength - 字符串长度', () => {
        expect('hello').toHaveLength(5);
    });
});
```

### 3.4 数组与对象匹配器

```javascript
/**
 * 数组与对象匹配器示例
 * @author erik.zhou
 */

describe('Array and Object Matchers', () => {
    test('toContain - 数组包含元素', () => {
        const fruits = ['apple', 'banana', 'orange'];
        expect(fruits).toContain('banana');
    });
    
    test('toContainEqual - 数组包含对象', () => {
        const users = [
            { id: 1, name: 'John' },
            { id: 2, name: 'Jane' }
        ];
        expect(users).toContainEqual({ id: 1, name: 'John' });
    });
    
    test('toHaveLength - 数组长度', () => {
        expect([1, 2, 3]).toHaveLength(3);
    });
    
    test('toHaveProperty - 对象属性', () => {
        const user = { id: 1, name: 'John', age: 30 };
        expect(user).toHaveProperty('name');
        expect(user).toHaveProperty('name', 'John');
        expect(user).toHaveProperty('age', 30);
    });
    
    test('toMatchObject - 部分匹配', () => {
        const user = { id: 1, name: 'John', age: 30, email: 'john@example.com' };
        expect(user).toMatchObject({
            name: 'John',
            age: 30
        });
    });
});
```

### 3.5 异常匹配器

```javascript
/**
 * 异常匹配器示例
 * @author erik.zhou
 */

describe('Exception Matchers', () => {
    function throwError() {
        throw new Error('Something went wrong');
    }
    
    test('toThrow - 抛出异常', () => {
        expect(() => throwError()).toThrow();
        expect(() => throwError()).toThrow(Error);
        expect(() => throwError()).toThrow('Something went wrong');
        expect(() => throwError()).toThrow(/went wrong/);
    });
    
    test('not.toThrow - 不抛出异常', () => {
        expect(() => {
            return 'success';
        }).not.toThrow();
    });
});
```

---

## 4. 异步测试

### 4.1 Promise测试

```javascript
/**
 * Promise测试示例
 * @author erik.zhou
 */

// 异步函数
function fetchUser(id) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (id > 0) {
                resolve({ id, name: 'John' });
            } else {
                reject(new Error('Invalid ID'));
            }
        }, 100);
    });
}

describe('Promise Tests', () => {
    // 方式1：返回Promise
    test('should fetch user - return promise', () => {
        return fetchUser(1).then(user => {
            expect(user).toEqual({ id: 1, name: 'John' });
        });
    });
    
    // 方式2：使用resolves
    test('should fetch user - resolves', () => {
        return expect(fetchUser(1)).resolves.toEqual({ id: 1, name: 'John' });
    });
    
    // 方式3：使用rejects
    test('should reject with invalid ID', () => {
        return expect(fetchUser(-1)).rejects.toThrow('Invalid ID');
    });
});
```

### 4.2 Async/Await测试

```javascript
/**
 * Async/Await测试示例
 * @author erik.zhou
 */

describe('Async/Await Tests', () => {
    test('should fetch user - async/await', async () => {
        const user = await fetchUser(1);
        expect(user).toEqual({ id: 1, name: 'John' });
    });
    
    test('should handle error - async/await', async () => {
        await expect(fetchUser(-1)).rejects.toThrow('Invalid ID');
    });
    
    test('should fetch multiple users', async () => {
        const users = await Promise.all([
            fetchUser(1),
            fetchUser(2),
            fetchUser(3)
        ]);
        
        expect(users).toHaveLength(3);
        expect(users[0]).toHaveProperty('id', 1);
    });
});
```

### 4.3 Callback测试

```javascript
/**
 * Callback测试示例
 * @author erik.zhou
 */

function fetchUserCallback(id, callback) {
    setTimeout(() => {
        if (id > 0) {
            callback(null, { id, name: 'John' });
        } else {
            callback(new Error('Invalid ID'));
        }
    }, 100);
}

describe('Callback Tests', () => {
    test('should fetch user - callback', (done) => {
        fetchUserCallback(1, (error, user) => {
            expect(error).toBeNull();
            expect(user).toEqual({ id: 1, name: 'John' });
            done();
        });
    });
    
    test('should handle error - callback', (done) => {
        fetchUserCallback(-1, (error, user) => {
            expect(error).toBeInstanceOf(Error);
            expect(error.message).toBe('Invalid ID');
            expect(user).toBeUndefined();
            done();
        });
    });
});
```

### 4.4 定时器测试

```javascript
/**
 * 定时器测试示例
 * @author erik.zhou
 */

function delayedGreeting(name, delay) {
    return new Promise((resolve) => {
        setTimeout(() => {
            resolve(`Hello, ${name}!`);
        }, delay);
    });
}

describe('Timer Tests', () => {
    // 使用真实定时器
    test('should greet after delay - real timers', async () => {
        const greeting = await delayedGreeting('John', 100);
        expect(greeting).toBe('Hello, John!');
    });
    
    // 使用fake timers
    test('should greet after delay - fake timers', () => {
        jest.useFakeTimers();
        
        const promise = delayedGreeting('John', 1000);
        
        // 快进时间
        jest.advanceTimersByTime(1000);
        
        return promise.then(greeting => {
            expect(greeting).toBe('Hello, John!');
        });
    });
    
    test('should run all timers', () => {
        jest.useFakeTimers();
        
        const callback = jest.fn();
        setTimeout(callback, 1000);
        
        // 运行所有定时器
        jest.runAllTimers();
        
        expect(callback).toHaveBeenCalledTimes(1);
    });
    
    afterEach(() => {
        jest.useRealTimers();
    });
});
```

---

## 5. Mock函数

### 5.1 创建Mock函数

```javascript
/**
 * Mock函数基础
 * @author erik.zhou
 */

describe('Mock Functions', () => {
    test('should create mock function', () => {
        const mockFn = jest.fn();
        
        mockFn('hello');
        mockFn('world');
        
        // 检查调用次数
        expect(mockFn).toHaveBeenCalledTimes(2);
        
        // 检查调用参数
        expect(mockFn).toHaveBeenCalledWith('hello');
        expect(mockFn).toHaveBeenLastCalledWith('world');
        
        // 检查所有调用
        expect(mockFn.mock.calls).toEqual([['hello'], ['world']]);
    });
    
    test('should mock return value', () => {
        const mockFn = jest.fn();
        
        mockFn.mockReturnValue('mocked value');
        
        expect(mockFn()).toBe('mocked value');
        expect(mockFn()).toBe('mocked value');
    });
    
    test('should mock return value once', () => {
        const mockFn = jest.fn();
        
        mockFn
            .mockReturnValueOnce('first')
            .mockReturnValueOnce('second')
            .mockReturnValue('default');
        
        expect(mockFn()).toBe('first');
        expect(mockFn()).toBe('second');
        expect(mockFn()).toBe('default');
        expect(mockFn()).toBe('default');
    });
});
```

### 5.2 Mock实现

```javascript
/**
 * Mock实现示例
 * @author erik.zhou
 */

describe('Mock Implementation', () => {
    test('should mock implementation', () => {
        const mockFn = jest.fn((x, y) => x + y);
        
        expect(mockFn(1, 2)).toBe(3);
        expect(mockFn(5, 10)).toBe(15);
    });
    
    test('should mock implementation once', () => {
        const mockFn = jest.fn();
        
        mockFn
            .mockImplementationOnce((x) => x * 2)
            .mockImplementationOnce((x) => x * 3)
            .mockImplementation((x) => x * 4);
        
        expect(mockFn(5)).toBe(10);
        expect(mockFn(5)).toBe(15);
        expect(mockFn(5)).toBe(20);
    });
    
    test('should mock resolved value', async () => {
        const mockFn = jest.fn();
        
        mockFn.mockResolvedValue('success');
        
        await expect(mockFn()).resolves.toBe('success');
    });
    
    test('should mock rejected value', async () => {
        const mockFn = jest.fn();
        
        mockFn.mockRejectedValue(new Error('failed'));
        
        await expect(mockFn()).rejects.toThrow('failed');
    });
});
```

### 5.3 Mock属性

```javascript
/**
 * Mock属性示例
 * @author erik.zhou
 */

describe('Mock Properties', () => {
    test('should access mock properties', () => {
        const mockFn = jest.fn((x) => x * 2);
        
        mockFn(1);
        mockFn(2);
        mockFn(3);
        
        // mock.calls - 所有调用参数
        expect(mockFn.mock.calls).toEqual([[1], [2], [3]]);
        
        // mock.results - 所有返回值
        expect(mockFn.mock.results).toEqual([
            { type: 'return', value: 2 },
            { type: 'return', value: 4 },
            { type: 'return', value: 6 }
        ]);
        
        // mock.instances - this值
        expect(mockFn.mock.instances).toHaveLength(3);
    });
    
    test('should clear mock', () => {
        const mockFn = jest.fn();
        
        mockFn('test');
        expect(mockFn).toHaveBeenCalledTimes(1);
        
        // 清除调用记录
        mockFn.mockClear();
        expect(mockFn).toHaveBeenCalledTimes(0);
        
        mockFn('test2');
        expect(mockFn).toHaveBeenCalledTimes(1);
    });
    
    test('should reset mock', () => {
        const mockFn = jest.fn(() => 'value');
        
        expect(mockFn()).toBe('value');
        
        // 重置mock（清除调用记录和实现）
        mockFn.mockReset();
        expect(mockFn()).toBeUndefined();
    });
});
```



---

## 6. Mock模块

### 6.1 Mock整个模块

```javascript
/**
 * Mock模块示例
 * @author erik.zhou
 */

// userService.js
const axios = require('axios');

async function getUser(id) {
    const response = await axios.get(`/api/users/${id}`);
    return response.data;
}

module.exports = { getUser };

// userService.test.js
const axios = require('axios');
const { getUser } = require('./userService');

// Mock整个axios模块
jest.mock('axios');

describe('User Service', () => {
    test('should fetch user', async () => {
        const mockUser = { id: 1, name: 'John' };
        
        // Mock axios.get返回值
        axios.get.mockResolvedValue({ data: mockUser });
        
        const user = await getUser(1);
        
        expect(user).toEqual(mockUser);
        expect(axios.get).toHaveBeenCalledWith('/api/users/1');
    });
});
```

### 6.2 部分Mock模块

```javascript
/**
 * 部分Mock模块示例
 * @author erik.zhou
 */

// utils.js
function add(a, b) {
    return a + b;
}

function subtract(a, b) {
    return a - b;
}

function multiply(a, b) {
    return a * b;
}

module.exports = { add, subtract, multiply };

// calculator.test.js
jest.mock('./utils', () => {
    const originalModule = jest.requireActual('./utils');
    
    return {
        ...originalModule,
        // 只mock multiply函数
        multiply: jest.fn((a, b) => a * b * 2)
    };
});

const { add, subtract, multiply } = require('./utils');

describe('Calculator', () => {
    test('should use real add', () => {
        expect(add(2, 3)).toBe(5);
    });
    
    test('should use real subtract', () => {
        expect(subtract(5, 3)).toBe(2);
    });
    
    test('should use mocked multiply', () => {
        expect(multiply(2, 3)).toBe(12); // 2 * 3 * 2
        expect(multiply).toHaveBeenCalledWith(2, 3);
    });
});
```

### 6.3 Mock工厂函数

```javascript
/**
 * Mock工厂函数示例
 * @author erik.zhou
 */

// config.js
module.exports = {
    apiUrl: 'https://api.example.com',
    timeout: 5000
};

// api.test.js
jest.mock('./config', () => ({
    apiUrl: 'https://test.example.com',
    timeout: 1000
}));

const config = require('./config');

describe('API Config', () => {
    test('should use mocked config', () => {
        expect(config.apiUrl).toBe('https://test.example.com');
        expect(config.timeout).toBe(1000);
    });
});
```

### 6.4 手动Mock

```javascript
/**
 * 手动Mock示例
 * @author erik.zhou
 */

// __mocks__/axios.js
module.exports = {
    get: jest.fn(() => Promise.resolve({ data: {} })),
    post: jest.fn(() => Promise.resolve({ data: {} })),
    put: jest.fn(() => Promise.resolve({ data: {} })),
    delete: jest.fn(() => Promise.resolve({ data: {} }))
};

// api.test.js
jest.mock('axios'); // 自动使用__mocks__/axios.js

const axios = require('axios');
const { fetchData } = require('./api');

describe('API', () => {
    beforeEach(() => {
        jest.clearAllMocks();
    });
    
    test('should fetch data', async () => {
        const mockData = { id: 1, name: 'Test' };
        axios.get.mockResolvedValue({ data: mockData });
        
        const result = await fetchData();
        
        expect(result).toEqual(mockData);
        expect(axios.get).toHaveBeenCalledTimes(1);
    });
});
```

### 6.5 Spy函数

```javascript
/**
 * Spy函数示例
 * @author erik.zhou
 */

const utils = {
    add: (a, b) => a + b,
    multiply: (a, b) => a * b
};

describe('Spy Functions', () => {
    test('should spy on method', () => {
        const spy = jest.spyOn(utils, 'add');
        
        const result = utils.add(2, 3);
        
        expect(result).toBe(5);
        expect(spy).toHaveBeenCalledWith(2, 3);
        
        spy.mockRestore(); // 恢复原始实现
    });
    
    test('should spy and mock implementation', () => {
        const spy = jest.spyOn(utils, 'multiply')
            .mockImplementation((a, b) => a * b * 2);
        
        expect(utils.multiply(2, 3)).toBe(12);
        expect(spy).toHaveBeenCalledWith(2, 3);
        
        spy.mockRestore();
        expect(utils.multiply(2, 3)).toBe(6); // 恢复原始实现
    });
});
```

---

## 7. 快照测试

### 7.1 基础快照测试

```javascript
/**
 * 快照测试示例
 * @author erik.zhou
 */

// Button.js
function Button({ text, onClick }) {
    return {
        type: 'button',
        props: {
            className: 'btn',
            onClick,
            children: text
        }
    };
}

// Button.test.js
describe('Button Component', () => {
    test('should match snapshot', () => {
        const button = Button({ text: 'Click me', onClick: jest.fn() });
        expect(button).toMatchSnapshot();
    });
    
    test('should match inline snapshot', () => {
        const button = Button({ text: 'Submit', onClick: jest.fn() });
        expect(button).toMatchInlineSnapshot(`
            {
              "props": {
                "children": "Submit",
                "className": "btn",
                "onClick": [MockFunction],
              },
              "type": "button",
            }
        `);
    });
});
```

### 7.2 属性匹配器

```javascript
/**
 * 属性匹配器示例
 * @author erik.zhou
 */

describe('Snapshot with Property Matchers', () => {
    test('should match snapshot with dynamic values', () => {
        const user = {
            id: 1,
            name: 'John',
            createdAt: new Date(),
            updatedAt: new Date()
        };
        
        expect(user).toMatchSnapshot({
            createdAt: expect.any(Date),
            updatedAt: expect.any(Date)
        });
    });
    
    test('should match snapshot with regex', () => {
        const data = {
            id: 'abc-123-def',
            email: 'test@example.com'
        };
        
        expect(data).toMatchSnapshot({
            id: expect.stringMatching(/^[a-z0-9-]+$/),
            email: expect.stringContaining('@')
        });
    });
});
```

### 7.3 更新快照

```bash
# 更新所有快照
npm test -- -u

# 更新特定文件的快照
npm test Button.test.js -- -u

# 交互式更新快照
npm test -- --watch
# 按 'u' 更新快照
```

---

## 8. 测试覆盖率

### 8.1 生成覆盖率报告

```bash
# 生成覆盖率报告
npm test -- --coverage

# 指定覆盖率格式
npm test -- --coverage --coverageReporters=text --coverageReporters=html

# 只收集特定文件的覆盖率
npm test -- --coverage --collectCoverageFrom='src/**/*.js'
```

### 8.2 覆盖率配置

```javascript
/**
 * 覆盖率配置
 * @author erik.zhou
 */

// jest.config.js
module.exports = {
    collectCoverage: true,
    
    // 覆盖率收集范围
    collectCoverageFrom: [
        'src/**/*.{js,jsx,ts,tsx}',
        '!src/**/*.d.ts',
        '!src/**/*.stories.{js,jsx,ts,tsx}',
        '!src/**/__tests__/**'
    ],
    
    // 覆盖率输出目录
    coverageDirectory: 'coverage',
    
    // 覆盖率报告格式
    coverageReporters: [
        'text',
        'text-summary',
        'html',
        'lcov',
        'json'
    ],
    
    // 覆盖率阈值
    coverageThreshold: {
        global: {
            branches: 80,
            functions: 80,
            lines: 80,
            statements: 80
        },
        './src/utils/': {
            branches: 90,
            functions: 90,
            lines: 90,
            statements: 90
        }
    }
};
```

### 8.3 忽略覆盖率

```javascript
/**
 * 忽略覆盖率示例
 * @author erik.zhou
 */

function processData(data) {
    // 正常代码
    const result = data.map(item => item * 2);
    
    /* istanbul ignore next */
    if (process.env.NODE_ENV === 'development') {
        console.log('Debug:', result);
    }
    
    return result;
}

/* istanbul ignore next */
function debugOnly() {
    console.log('This function is only for debugging');
}
```

---

## 9. 高级特性

### 9.1 测试并发

```javascript
/**
 * 并发测试示例
 * @author erik.zhou
 */

describe.concurrent('Concurrent Tests', () => {
    test.concurrent('test 1', async () => {
        await new Promise(resolve => setTimeout(resolve, 100));
        expect(1 + 1).toBe(2);
    });
    
    test.concurrent('test 2', async () => {
        await new Promise(resolve => setTimeout(resolve, 100));
        expect(2 + 2).toBe(4);
    });
    
    test.concurrent('test 3', async () => {
        await new Promise(resolve => setTimeout(resolve, 100));
        expect(3 + 3).toBe(6);
    });
});
```

### 9.2 测试跳过与专注

```javascript
/**
 * 测试跳过与专注示例
 * @author erik.zhou
 */

describe('Test Skip and Focus', () => {
    // 跳过测试
    test.skip('should be skipped', () => {
        expect(true).toBe(false);
    });
    
    // 只运行这个测试
    test.only('should run only this test', () => {
        expect(true).toBe(true);
    });
    
    test('should not run', () => {
        expect(true).toBe(true);
    });
});

// 跳过整个测试套件
describe.skip('Skipped Suite', () => {
    test('test 1', () => {});
    test('test 2', () => {});
});

// 只运行这个测试套件
describe.only('Focused Suite', () => {
    test('test 1', () => {});
    test('test 2', () => {});
});
```

### 9.3 参数化测试

```javascript
/**
 * 参数化测试示例
 * @author erik.zhou
 */

describe('Parameterized Tests', () => {
    // test.each - 数组形式
    test.each([
        [1, 1, 2],
        [1, 2, 3],
        [2, 1, 3]
    ])('add(%i, %i) should return %i', (a, b, expected) => {
        expect(a + b).toBe(expected);
    });
    
    // test.each - 对象形式
    test.each([
        { a: 1, b: 1, expected: 2 },
        { a: 1, b: 2, expected: 3 },
        { a: 2, b: 1, expected: 3 }
    ])('add($a, $b) should return $expected', ({ a, b, expected }) => {
        expect(a + b).toBe(expected);
    });
    
    // describe.each
    describe.each([
        { operation: 'add', fn: (a, b) => a + b },
        { operation: 'multiply', fn: (a, b) => a * b }
    ])('$operation operation', ({ operation, fn }) => {
        test(`should ${operation} numbers`, () => {
            expect(fn(2, 3)).toBeGreaterThan(0);
        });
    });
});
```

### 9.4 自定义匹配器

```javascript
/**
 * 自定义匹配器示例
 * @author erik.zhou
 */

expect.extend({
    toBeWithinRange(received, floor, ceiling) {
        const pass = received >= floor && received <= ceiling;
        if (pass) {
            return {
                message: () =>
                    `expected ${received} not to be within range ${floor} - ${ceiling}`,
                pass: true
            };
        } else {
            return {
                message: () =>
                    `expected ${received} to be within range ${floor} - ${ceiling}`,
                pass: false
            };
        }
    },
    
    toBeValidEmail(received) {
        const emailRegex = /^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/;
        const pass = emailRegex.test(received);
        
        if (pass) {
            return {
                message: () => `expected ${received} not to be a valid email`,
                pass: true
            };
        } else {
            return {
                message: () => `expected ${received} to be a valid email`,
                pass: false
            };
        }
    }
});

describe('Custom Matchers', () => {
    test('should be within range', () => {
        expect(100).toBeWithinRange(90, 110);
        expect(101).not.toBeWithinRange(0, 100);
    });
    
    test('should be valid email', () => {
        expect('test@example.com').toBeValidEmail();
        expect('invalid-email').not.toBeValidEmail();
    });
});
```

---

## 10. 最佳实践

### 10.1 测试组织

```javascript
/**
 * 测试组织最佳实践
 * @author erik.zhou
 */

// ✅ 好的实践
describe('UserService', () => {
    describe('getUser', () => {
        test('should return user when ID is valid', async () => {
            // Arrange
            const userId = 1;
            const expectedUser = { id: 1, name: 'John' };
            
            // Act
            const user = await userService.getUser(userId);
            
            // Assert
            expect(user).toEqual(expectedUser);
        });
        
        test('should throw error when ID is invalid', async () => {
            // Arrange
            const invalidId = -1;
            
            // Act & Assert
            await expect(userService.getUser(invalidId))
                .rejects
                .toThrow('Invalid user ID');
        });
    });
    
    describe('createUser', () => {
        test('should create user with valid data', async () => {
            // 测试实现
        });
    });
});

// ❌ 不好的实践
test('user service tests', async () => {
    // 多个测试混在一起
    const user1 = await userService.getUser(1);
    expect(user1).toBeDefined();
    
    const user2 = await userService.createUser({ name: 'Jane' });
    expect(user2).toBeDefined();
});
```

### 10.2 测试命名

```javascript
/**
 * 测试命名最佳实践
 * @author erik.zhou
 */

// ✅ 好的命名
describe('Calculator', () => {
    describe('add', () => {
        test('should return sum of two positive numbers', () => {
            expect(add(2, 3)).toBe(5);
        });
        
        test('should return negative sum when both numbers are negative', () => {
            expect(add(-2, -3)).toBe(-5);
        });
        
        test('should handle zero correctly', () => {
            expect(add(0, 5)).toBe(5);
            expect(add(5, 0)).toBe(5);
        });
    });
});

// ❌ 不好的命名
describe('Calculator', () => {
    test('test1', () => {
        expect(add(2, 3)).toBe(5);
    });
    
    test('add function', () => {
        expect(add(-2, -3)).toBe(-5);
    });
});
```

### 10.3 避免测试依赖

```javascript
/**
 * 避免测试依赖
 * @author erik.zhou
 */

// ❌ 不好的实践 - 测试之间有依赖
let userId;

test('should create user', async () => {
    const user = await createUser({ name: 'John' });
    userId = user.id; // 依赖于这个测试的结果
    expect(user).toBeDefined();
});

test('should get user', async () => {
    const user = await getUser(userId); // 依赖于上一个测试
    expect(user.name).toBe('John');
});

// ✅ 好的实践 - 每个测试独立
describe('User API', () => {
    let testUserId;
    
    beforeEach(async () => {
        // 每个测试前创建测试数据
        const user = await createUser({ name: 'John' });
        testUserId = user.id;
    });
    
    afterEach(async () => {
        // 每个测试后清理数据
        await deleteUser(testUserId);
    });
    
    test('should get user', async () => {
        const user = await getUser(testUserId);
        expect(user.name).toBe('John');
    });
    
    test('should update user', async () => {
        await updateUser(testUserId, { name: 'Jane' });
        const user = await getUser(testUserId);
        expect(user.name).toBe('Jane');
    });
});
```

### 10.4 测试覆盖率目标

```javascript
/**
 * 测试覆盖率最佳实践
 * @author erik.zhou
 */

// 优先测试关键路径
describe('Payment Service', () => {
    // ✅ 必须测试：核心业务逻辑
    test('should process payment successfully', async () => {
        const result = await processPayment({
            amount: 100,
            currency: 'USD'
        });
        expect(result.status).toBe('success');
    });
    
    // ✅ 必须测试：错误处理
    test('should handle insufficient funds', async () => {
        await expect(processPayment({
            amount: 1000000,
            currency: 'USD'
        })).rejects.toThrow('Insufficient funds');
    });
    
    // ✅ 必须测试：边界条件
    test('should reject negative amount', async () => {
        await expect(processPayment({
            amount: -100,
            currency: 'USD'
        })).rejects.toThrow('Invalid amount');
    });
});

// 不需要过度测试简单的getter/setter
class User {
    constructor(name) {
        this.name = name;
    }
    
    getName() {
        return this.name; // 简单的getter，不需要单独测试
    }
}
```

### 10.5 Mock使用原则

```javascript
/**
 * Mock使用最佳实践
 * @author erik.zhou
 */

// ✅ 好的实践 - Mock外部依赖
jest.mock('./api');
const api = require('./api');

describe('UserService', () => {
    test('should fetch user from API', async () => {
        api.getUser.mockResolvedValue({ id: 1, name: 'John' });
        
        const user = await userService.fetchUser(1);
        
        expect(user).toEqual({ id: 1, name: 'John' });
        expect(api.getUser).toHaveBeenCalledWith(1);
    });
});

// ❌ 不好的实践 - 过度Mock
describe('Calculator', () => {
    test('should add numbers', () => {
        const add = jest.fn((a, b) => a + b); // 不需要mock简单函数
        expect(add(2, 3)).toBe(5);
    });
});
```

---

## 总结

本教程全面介绍了Jest测试框架的核心功能和最佳实践：

1. **Jest基础**：安装配置、测试结构、生命周期钩子
2. **测试配置**：jest.config.js、TypeScript支持、Setup文件
3. **匹配器与断言**：基础匹配器、数字/字符串/数组/对象匹配器
4. **异步测试**：Promise、Async/Await、Callback、定时器测试
5. **Mock函数**：创建Mock、Mock实现、Mock属性
6. **Mock模块**：整个模块Mock、部分Mock、手动Mock、Spy函数
7. **快照测试**：基础快照、属性匹配器、快照更新
8. **测试覆盖率**：生成报告、覆盖率配置、忽略覆盖率
9. **高级特性**：并发测试、测试跳过、参数化测试、自定义匹配器
10. **最佳实践**：测试组织、命名规范、避免依赖、覆盖率目标、Mock原则

通过本教程的学习，你将能够使用Jest编写高质量的单元测试和集成测试。

---

**@author erik.zhou**
**最后更新时间：2026-03-03**
