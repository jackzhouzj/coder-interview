# Node.js基础 - 完整教程

## 课程简介

Node.js是基于Chrome V8引擎的JavaScript运行时环境，让JavaScript可以在服务端运行。本教程将深入讲解Node.js的核心概念、模块系统、异步编程、文件系统、网络编程等基础知识。

## 学习目标

- 理解Node.js的核心概念和架构
- 掌握模块系统和包管理
- 熟练使用异步编程模式
- 掌握文件系统操作
- 学会网络编程基础
- 理解事件循环机制
- 掌握Stream和Buffer

## 目录

1. [Node.js概述](#第1章-nodejs概述)
2. [模块系统](#第2章-模块系统)
3. [包管理](#第3章-包管理)
4. [异步编程](#第4章-异步编程)
5. [事件循环](#第5章-事件循环)
6. [文件系统](#第6章-文件系统)
7. [Stream流](#第7章-stream流)
8. [Buffer缓冲区](#第8章-buffer缓冲区)
9. [网络编程](#第9章-网络编程)
10. [进程与线程](#第10章-进程与线程)

---

## 第1章 Node.js概述

### 1.1 Node.js简介

```javascript
/**
 * Node.js核心特性
 * @author erik.zhou
 */

// Node.js的主要特性：
// 1. 事件驱动
// 2. 非阻塞I/O
// 3. 单线程
// 4. 跨平台
// 5. 高性能

// 查看Node.js版本
console.log('Node.js版本:', process.version);
console.log('V8版本:', process.versions.v8);
console.log('平台:', process.platform);
console.log('架构:', process.arch);
```

### 1.2 全局对象

```javascript
/**
 * Node.js全局对象
 * @author erik.zhou
 */

// global对象（类似浏览器的window）
console.log('全局对象:', typeof global);

// process对象
console.log('进程ID:', process.pid);
console.log('当前工作目录:', process.cwd());
console.log('环境变量:', process.env.NODE_ENV);

// __dirname和__filename
console.log('当前文件目录:', __dirname);
console.log('当前文件路径:', __filename);

// console对象
console.log('普通日志');
console.error('错误日志');
console.warn('警告日志');
console.time('计时器');
console.timeEnd('计时器');

// setTimeout和setInterval
setTimeout(() => {
    console.log('延迟执行');
}, 1000);

const timer = setInterval(() => {
    console.log('定时执行');
}, 1000);

// 清除定时器
setTimeout(() => {
    clearInterval(timer);
}, 5000);
```

### 1.3 命令行参数

```javascript
/**
 * 处理命令行参数
 * @author erik.zhou
 */

// process.argv包含命令行参数
// node script.js arg1 arg2
console.log('命令行参数:', process.argv);

// 第一个参数是node可执行文件路径
// 第二个参数是脚本文件路径
// 后续是用户传入的参数
const args = process.argv.slice(2);
console.log('用户参数:', args);

// 解析命令行参数
function parseArgs(args) {
    const options = {};
    
    for (let i = 0; i < args.length; i++) {
        const arg = args[i];
        
        if (arg.startsWith('--')) {
            const key = arg.slice(2);
            const value = args[i + 1];
            options[key] = value;
            i++;
        }
    }
    
    return options;
}

const options = parseArgs(args);
console.log('解析后的参数:', options);
```

### 1.4 环境变量

```javascript
/**
 * 环境变量管理
 * @author erik.zhou
 */

// 读取环境变量
const nodeEnv = process.env.NODE_ENV || 'development';
const port = process.env.PORT || 3000;

console.log('运行环境:', nodeEnv);
console.log('端口号:', port);

// 设置环境变量
process.env.MY_VAR = 'my value';
console.log('自定义变量:', process.env.MY_VAR);

// 使用dotenv加载.env文件
// npm install dotenv
require('dotenv').config();

// .env文件内容：
// NODE_ENV=production
// PORT=8080
// DB_HOST=localhost
// DB_USER=admin
// DB_PASS=secret

console.log('数据库配置:', {
    host: process.env.DB_HOST,
    user: process.env.DB_USER,
    password: process.env.DB_PASS
});
```

---

## 第2章 模块系统

### 2.1 CommonJS模块

```javascript
/**
 * CommonJS模块导出
 * @author erik.zhou
 */

// math.js - 导出模块
function add(a, b) {
    return a + b;
}

function subtract(a, b) {
    return a - b;
}

function multiply(a, b) {
    return a * b;
}

function divide(a, b) {
    if (b === 0) {
        throw new Error('除数不能为0');
    }
    return a / b;
}

// 方式1：逐个导出
exports.add = add;
exports.subtract = subtract;

// 方式2：整体导出
module.exports = {
    add,
    subtract,
    multiply,
    divide
};

// 方式3：导出类
class Calculator {
    add(a, b) {
        return a + b;
    }
    
    subtract(a, b) {
        return a - b;
    }
}

module.exports = Calculator;
```

```javascript
/**
 * CommonJS模块导入
 * @author erik.zhou
 */

// app.js - 导入模块
const math = require('./math');

console.log('加法:', math.add(5, 3));
console.log('减法:', math.subtract(5, 3));

// 解构导入
const { add, subtract } = require('./math');
console.log('加法:', add(10, 5));

// 导入类
const Calculator = require('./calculator');
const calc = new Calculator();
console.log('计算结果:', calc.add(2, 3));

// 导入核心模块
const fs = require('fs');
const path = require('path');
const http = require('http');

// 导入第三方模块
const express = require('express');
const axios = require('axios');
```

### 2.2 ES模块

```javascript
/**
 * ES模块导出
 * @author erik.zhou
 */

// math.mjs - 命名导出
export function add(a, b) {
    return a + b;
}

export function subtract(a, b) {
    return a - b;
}

// 默认导出
export default class Calculator {
    add(a, b) {
        return a + b;
    }
    
    subtract(a, b) {
        return a - b;
    }
}

// 混合导出
export const PI = 3.14159;
export const E = 2.71828;

export default {
    add,
    subtract,
    PI,
    E
};
```

```javascript
/**
 * ES模块导入
 * @author erik.zhou
 */

// app.mjs - 导入模块
import { add, subtract } from './math.mjs';
console.log('加法:', add(5, 3));

// 导入默认导出
import Calculator from './calculator.mjs';
const calc = new Calculator();

// 导入所有
import * as math from './math.mjs';
console.log('加法:', math.add(5, 3));

// 重命名导入
import { add as sum } from './math.mjs';
console.log('求和:', sum(1, 2));

// 动态导入
async function loadModule() {
    const math = await import('./math.mjs');
    console.log('动态加载:', math.add(1, 2));
}

loadModule();
```

### 2.3 模块缓存

```javascript
/**
 * 模块缓存机制
 * @author erik.zhou
 */

// counter.js
let count = 0;

function increment() {
    count++;
    return count;
}

function getCount() {
    return count;
}

module.exports = {
    increment,
    getCount
};

// app.js
const counter1 = require('./counter');
const counter2 = require('./counter');

console.log(counter1.increment()); // 1
console.log(counter2.increment()); // 2
console.log(counter1.getCount());  // 2

// counter1和counter2是同一个实例
console.log(counter1 === counter2); // true

// 清除缓存
delete require.cache[require.resolve('./counter')];
const counter3 = require('./counter');
console.log(counter3.getCount()); // 0
```

### 2.4 循环依赖

```javascript
/**
 * 处理循环依赖
 * @author erik.zhou
 */

// a.js
console.log('a.js开始');
exports.done = false;

const b = require('./b');
console.log('在a.js中，b.done =', b.done);

exports.done = true;
console.log('a.js结束');

// b.js
console.log('b.js开始');
exports.done = false;

const a = require('./a');
console.log('在b.js中，a.done =', a.done);

exports.done = true;
console.log('b.js结束');

// main.js
console.log('main.js开始');
const a = require('./a');
const b = require('./b');

console.log('在main.js中，a.done =', a.done);
console.log('在main.js中，b.done =', b.done);

// 输出：
// main.js开始
// a.js开始
// b.js开始
// 在b.js中，a.done = false
// b.js结束
// 在a.js中，b.done = true
// a.js结束
// 在main.js中，a.done = true
// 在main.js中，b.done = true
```

---

## 第3章 包管理

### 3.1 package.json

```json
/**
 * package.json配置
 * @author erik.zhou
 */
{
    "name": "my-app",
    "version": "1.0.0",
    "description": "My Node.js application",
    "main": "index.js",
    "type": "commonjs",
    "scripts": {
        "start": "node index.js",
        "dev": "nodemon index.js",
        "test": "jest",
        "build": "webpack",
        "lint": "eslint .",
        "format": "prettier --write ."
    },
    "keywords": ["node", "app"],
    "author": "erik.zhou",
    "license": "MIT",
    "dependencies": {
        "express": "^4.18.0",
        "axios": "^1.4.0"
    },
    "devDependencies": {
        "nodemon": "^3.0.0",
        "jest": "^29.0.0",
        "eslint": "^8.0.0"
    },
    "engines": {
        "node": ">=18.0.0",
        "npm": ">=9.0.0"
    }
}
```

### 3.2 npm命令

```bash
# npm包管理命令
# @author erik.zhou

# 初始化项目
npm init
npm init -y

# 安装依赖
npm install express
npm install axios --save
npm install nodemon --save-dev
npm install -g pm2

# 安装指定版本
npm install express@4.18.0
npm install express@latest

# 更新依赖
npm update
npm update express

# 卸载依赖
npm uninstall express
npm uninstall nodemon --save-dev

# 查看依赖
npm list
npm list --depth=0
npm list express

# 查看过时的包
npm outdated

# 清理缓存
npm cache clean --force

# 审计安全漏洞
npm audit
npm audit fix
```

### 3.3 package-lock.json

```javascript
/**
 * package-lock.json作用
 * @author erik.zhou
 */

// package-lock.json的作用：
// 1. 锁定依赖版本
// 2. 提高安装速度
// 3. 确保团队一致性
// 4. 记录依赖树

// 版本号说明：
// ^1.2.3 - 兼容1.x.x的最新版本
// ~1.2.3 - 兼容1.2.x的最新版本
// 1.2.3 - 精确版本
// * - 任意版本
// >=1.2.3 - 大于等于1.2.3
```

### 3.4 npx命令

```bash
# npx命令使用
# @author erik.zhou

# 执行本地包
npx eslint .
npx jest

# 执行远程包（不安装）
npx create-react-app my-app
npx cowsay "Hello"

# 指定版本
npx create-react-app@latest my-app

# 执行GitHub仓库
npx github:user/repo
```



---

## 第4章 异步编程

### 4.1 回调函数

```javascript
/**
 * 回调函数基础
 * @author erik.zhou
 */

const fs = require('fs');

// 异步读取文件
fs.readFile('data.txt', 'utf8', (err, data) => {
    if (err) {
        console.error('读取失败:', err);
        return;
    }
    console.log('文件内容:', data);
});

// 回调地狱示例
fs.readFile('file1.txt', 'utf8', (err, data1) => {
    if (err) {
        console.error(err);
        return;
    }
    
    fs.readFile('file2.txt', 'utf8', (err, data2) => {
        if (err) {
            console.error(err);
            return;
        }
        
        fs.readFile('file3.txt', 'utf8', (err, data3) => {
            if (err) {
                console.error(err);
                return;
            }
            
            console.log('所有文件读取完成');
        });
    });
});
```

```javascript
/**
 * 错误优先回调
 * @author erik.zhou
 */

// Node.js约定：回调函数第一个参数是错误对象
function readFileAsync(filename, callback) {
    fs.readFile(filename, 'utf8', (err, data) => {
        if (err) {
            callback(err, null);
            return;
        }
        callback(null, data);
    });
}

// 使用错误优先回调
readFileAsync('data.txt', (err, data) => {
    if (err) {
        console.error('读取失败:', err.message);
        return;
    }
    console.log('读取成功:', data);
});
```

### 4.2 Promise

```javascript
/**
 * Promise基础
 * @author erik.zhou
 */

const fs = require('fs').promises;

// 创建Promise
function readFilePromise(filename) {
    return new Promise((resolve, reject) => {
        fs.readFile(filename, 'utf8')
            .then(data => resolve(data))
            .catch(err => reject(err));
    });
}

// 使用Promise
readFilePromise('data.txt')
    .then(data => {
        console.log('文件内容:', data);
        return data.toUpperCase();
    })
    .then(upperData => {
        console.log('转大写:', upperData);
    })
    .catch(err => {
        console.error('读取失败:', err);
    })
    .finally(() => {
        console.log('操作完成');
    });
```

```javascript
/**
 * Promise链式调用
 * @author erik.zhou
 */

// 解决回调地狱
fs.readFile('file1.txt', 'utf8')
    .then(data1 => {
        console.log('文件1:', data1);
        return fs.readFile('file2.txt', 'utf8');
    })
    .then(data2 => {
        console.log('文件2:', data2);
        return fs.readFile('file3.txt', 'utf8');
    })
    .then(data3 => {
        console.log('文件3:', data3);
    })
    .catch(err => {
        console.error('读取失败:', err);
    });
```

```javascript
/**
 * Promise并发控制
 * @author erik.zhou
 */

// Promise.all - 全部成功才成功
const promises = [
    fs.readFile('file1.txt', 'utf8'),
    fs.readFile('file2.txt', 'utf8'),
    fs.readFile('file3.txt', 'utf8')
];

Promise.all(promises)
    .then(results => {
        console.log('所有文件:', results);
    })
    .catch(err => {
        console.error('有文件读取失败:', err);
    });

// Promise.allSettled - 等待所有完成
Promise.allSettled(promises)
    .then(results => {
        results.forEach((result, index) => {
            if (result.status === 'fulfilled') {
                console.log(`文件${index + 1}成功:`, result.value);
            } else {
                console.error(`文件${index + 1}失败:`, result.reason);
            }
        });
    });

// Promise.race - 第一个完成的
Promise.race(promises)
    .then(result => {
        console.log('最快完成的文件:', result);
    })
    .catch(err => {
        console.error('最快失败的文件:', err);
    });

// Promise.any - 第一个成功的
Promise.any(promises)
    .then(result => {
        console.log('第一个成功的文件:', result);
    })
    .catch(err => {
        console.error('所有文件都失败了:', err);
    });
```

### 4.3 async/await

```javascript
/**
 * async/await基础
 * @author erik.zhou
 */

const fs = require('fs').promises;

// async函数返回Promise
async function readFileAsync(filename) {
    try {
        const data = await fs.readFile(filename, 'utf8');
        return data;
    } catch (err) {
        console.error('读取失败:', err);
        throw err;
    }
}

// 使用async/await
async function main() {
    try {
        const data = await readFileAsync('data.txt');
        console.log('文件内容:', data);
    } catch (err) {
        console.error('错误:', err);
    }
}

main();
```

```javascript
/**
 * async/await顺序执行
 * @author erik.zhou
 */

async function readFilesSequentially() {
    try {
        const data1 = await fs.readFile('file1.txt', 'utf8');
        console.log('文件1:', data1);
        
        const data2 = await fs.readFile('file2.txt', 'utf8');
        console.log('文件2:', data2);
        
        const data3 = await fs.readFile('file3.txt', 'utf8');
        console.log('文件3:', data3);
        
        return [data1, data2, data3];
    } catch (err) {
        console.error('读取失败:', err);
        throw err;
    }
}

readFilesSequentially();
```

```javascript
/**
 * async/await并发执行
 * @author erik.zhou
 */

async function readFilesConcurrently() {
    try {
        // 并发执行
        const [data1, data2, data3] = await Promise.all([
            fs.readFile('file1.txt', 'utf8'),
            fs.readFile('file2.txt', 'utf8'),
            fs.readFile('file3.txt', 'utf8')
        ]);
        
        console.log('文件1:', data1);
        console.log('文件2:', data2);
        console.log('文件3:', data3);
        
        return [data1, data2, data3];
    } catch (err) {
        console.error('读取失败:', err);
        throw err;
    }
}

readFilesConcurrently();
```

```javascript
/**
 * async/await错误处理
 * @author erik.zhou
 */

// 方式1：try-catch
async function readWithTryCatch() {
    try {
        const data = await fs.readFile('data.txt', 'utf8');
        return data;
    } catch (err) {
        console.error('读取失败:', err);
        return null;
    }
}

// 方式2：Promise.catch
async function readWithCatch() {
    const data = await fs.readFile('data.txt', 'utf8')
        .catch(err => {
            console.error('读取失败:', err);
            return null;
        });
    return data;
}

// 方式3：统一错误处理
async function readWithErrorHandler() {
    const data = await fs.readFile('data.txt', 'utf8');
    return data;
}

readWithErrorHandler()
    .catch(err => {
        console.error('统一错误处理:', err);
    });
```

### 4.4 异步迭代器

```javascript
/**
 * 异步迭代器
 * @author erik.zhou
 */

// 创建异步迭代器
async function* asyncGenerator() {
    yield await Promise.resolve(1);
    yield await Promise.resolve(2);
    yield await Promise.resolve(3);
}

// 使用for await...of
async function useAsyncIterator() {
    for await (const value of asyncGenerator()) {
        console.log('值:', value);
    }
}

useAsyncIterator();

// 异步迭代文件流
const fs = require('fs');
const readline = require('readline');

async function readLinesAsync(filename) {
    const fileStream = fs.createReadStream(filename);
    const rl = readline.createInterface({
        input: fileStream,
        crlfDelay: Infinity
    });
    
    for await (const line of rl) {
        console.log('行:', line);
    }
}

readLinesAsync('data.txt');
```

---

## 第5章 事件循环

### 5.1 事件循环机制

```javascript
/**
 * 事件循环基础
 * @author erik.zhou
 */

// 事件循环的6个阶段：
// 1. timers - 执行setTimeout/setInterval回调
// 2. pending callbacks - 执行延迟到下一个循环的I/O回调
// 3. idle, prepare - 内部使用
// 4. poll - 检索新的I/O事件
// 5. check - 执行setImmediate回调
// 6. close callbacks - 执行关闭回调

console.log('1. 同步代码');

setTimeout(() => {
    console.log('2. setTimeout');
}, 0);

setImmediate(() => {
    console.log('3. setImmediate');
});

process.nextTick(() => {
    console.log('4. nextTick');
});

Promise.resolve().then(() => {
    console.log('5. Promise');
});

console.log('6. 同步代码');

// 输出顺序：
// 1. 同步代码
// 6. 同步代码
// 4. nextTick
// 5. Promise
// 2. setTimeout
// 3. setImmediate
```

### 5.2 宏任务与微任务

```javascript
/**
 * 宏任务与微任务
 * @author erik.zhou
 */

// 宏任务（Macro Task）：
// - setTimeout
// - setInterval
// - setImmediate
// - I/O操作

// 微任务（Micro Task）：
// - process.nextTick
// - Promise.then/catch/finally
// - queueMicrotask

console.log('开始');

// 宏任务
setTimeout(() => {
    console.log('宏任务1: setTimeout');
    
    // 宏任务中的微任务
    Promise.resolve().then(() => {
        console.log('微任务3: Promise in setTimeout');
    });
}, 0);

// 微任务
Promise.resolve().then(() => {
    console.log('微任务1: Promise');
    
    // 微任务中的微任务
    Promise.resolve().then(() => {
        console.log('微任务2: Promise in Promise');
    });
});

// nextTick优先级最高
process.nextTick(() => {
    console.log('微任务0: nextTick');
});

console.log('结束');

// 输出顺序：
// 开始
// 结束
// 微任务0: nextTick
// 微任务1: Promise
// 微任务2: Promise in Promise
// 宏任务1: setTimeout
// 微任务3: Promise in setTimeout
```

### 5.3 定时器详解

```javascript
/**
 * setTimeout与setImmediate
 * @author erik.zhou
 */

// setTimeout vs setImmediate
setTimeout(() => {
    console.log('setTimeout');
}, 0);

setImmediate(() => {
    console.log('setImmediate');
});

// 在I/O回调中，setImmediate总是先执行
const fs = require('fs');

fs.readFile(__filename, () => {
    setTimeout(() => {
        console.log('setTimeout in I/O');
    }, 0);
    
    setImmediate(() => {
        console.log('setImmediate in I/O');
    });
});

// 输出：
// setImmediate in I/O
// setTimeout in I/O
```

```javascript
/**
 * process.nextTick详解
 * @author erik.zhou
 */

// nextTick在当前操作完成后立即执行
console.log('开始');

process.nextTick(() => {
    console.log('nextTick 1');
    
    process.nextTick(() => {
        console.log('nextTick 2');
    });
});

Promise.resolve().then(() => {
    console.log('Promise 1');
});

console.log('结束');

// 输出：
// 开始
// 结束
// nextTick 1
// nextTick 2
// Promise 1

// 注意：过度使用nextTick可能导致I/O饥饿
```

### 5.4 事件循环实战

```javascript
/**
 * 事件循环综合示例
 * @author erik.zhou
 */

async function complexEventLoop() {
    console.log('1. 同步开始');
    
    setTimeout(() => {
        console.log('2. setTimeout 0');
        
        process.nextTick(() => {
            console.log('3. nextTick in setTimeout');
        });
    }, 0);
    
    setTimeout(() => {
        console.log('4. setTimeout 10');
    }, 10);
    
    setImmediate(() => {
        console.log('5. setImmediate');
        
        Promise.resolve().then(() => {
            console.log('6. Promise in setImmediate');
        });
    });
    
    process.nextTick(() => {
        console.log('7. nextTick 1');
        
        process.nextTick(() => {
            console.log('8. nextTick 2');
        });
    });
    
    await Promise.resolve().then(() => {
        console.log('9. Promise 1');
        
        return Promise.resolve();
    }).then(() => {
        console.log('10. Promise 2');
    });
    
    console.log('11. 同步结束');
}

complexEventLoop();
```

```javascript
/**
 * 避免事件循环阻塞
 * @author erik.zhou
 */

// 错误示例：阻塞事件循环
function blockingOperation() {
    const start = Date.now();
    while (Date.now() - start < 5000) {
        // 阻塞5秒
    }
    console.log('阻塞操作完成');
}

// 正确示例：使用setImmediate分片
function nonBlockingOperation(data, callback) {
    const chunkSize = 1000;
    let index = 0;
    
    function processChunk() {
        const end = Math.min(index + chunkSize, data.length);
        
        for (let i = index; i < end; i++) {
            // 处理数据
            data[i] = data[i] * 2;
        }
        
        index = end;
        
        if (index < data.length) {
            setImmediate(processChunk);
        } else {
            callback();
        }
    }
    
    processChunk();
}

// 使用Worker Threads处理CPU密集型任务
const { Worker } = require('worker_threads');

function runWorker(data) {
    return new Promise((resolve, reject) => {
        const worker = new Worker('./worker.js', {
            workerData: data
        });
        
        worker.on('message', resolve);
        worker.on('error', reject);
        worker.on('exit', (code) => {
            if (code !== 0) {
                reject(new Error(`Worker stopped with exit code ${code}`));
            }
        });
    });
}
```



---

## 第6章 文件系统

### 6.1 文件读取

```javascript
/**
 * 同步读取文件
 * @author erik.zhou
 */

const fs = require('fs');
const path = require('path');

// 同步读取
try {
    const data = fs.readFileSync('data.txt', 'utf8');
    console.log('文件内容:', data);
} catch (err) {
    console.error('读取失败:', err);
}

// 读取二进制文件
const buffer = fs.readFileSync('image.png');
console.log('文件大小:', buffer.length);

// 读取JSON文件
const jsonData = JSON.parse(fs.readFileSync('config.json', 'utf8'));
console.log('配置:', jsonData);
```

```javascript
/**
 * 异步读取文件
 * @author erik.zhou
 */

// 回调方式
fs.readFile('data.txt', 'utf8', (err, data) => {
    if (err) {
        console.error('读取失败:', err);
        return;
    }
    console.log('文件内容:', data);
});

// Promise方式
const fsPromises = require('fs').promises;

async function readFileAsync() {
    try {
        const data = await fsPromises.readFile('data.txt', 'utf8');
        console.log('文件内容:', data);
    } catch (err) {
        console.error('读取失败:', err);
    }
}

readFileAsync();
```

```javascript
/**
 * 流式读取大文件
 * @author erik.zhou
 */

const fs = require('fs');

// 创建可读流
const readStream = fs.createReadStream('large-file.txt', {
    encoding: 'utf8',
    highWaterMark: 64 * 1024 // 64KB缓冲区
});

let content = '';

readStream.on('data', (chunk) => {
    console.log('读取数据块:', chunk.length);
    content += chunk;
});

readStream.on('end', () => {
    console.log('读取完成，总大小:', content.length);
});

readStream.on('error', (err) => {
    console.error('读取失败:', err);
});

// 暂停和恢复
readStream.pause();
setTimeout(() => {
    readStream.resume();
}, 1000);
```

### 6.2 文件写入

```javascript
/**
 * 同步写入文件
 * @author erik.zhou
 */

const fs = require('fs');

// 覆盖写入
try {
    fs.writeFileSync('output.txt', 'Hello World', 'utf8');
    console.log('写入成功');
} catch (err) {
    console.error('写入失败:', err);
}

// 追加写入
try {
    fs.appendFileSync('output.txt', '\nNew Line', 'utf8');
    console.log('追加成功');
} catch (err) {
    console.error('追加失败:', err);
}

// 写入JSON
const data = { name: 'John', age: 30 };
fs.writeFileSync('data.json', JSON.stringify(data, null, 4));
```

```javascript
/**
 * 异步写入文件
 * @author erik.zhou
 */

// 回调方式
fs.writeFile('output.txt', 'Hello World', 'utf8', (err) => {
    if (err) {
        console.error('写入失败:', err);
        return;
    }
    console.log('写入成功');
});

// Promise方式
const fsPromises = require('fs').promises;

async function writeFileAsync() {
    try {
        await fsPromises.writeFile('output.txt', 'Hello World', 'utf8');
        console.log('写入成功');
        
        await fsPromises.appendFile('output.txt', '\nNew Line', 'utf8');
        console.log('追加成功');
    } catch (err) {
        console.error('操作失败:', err);
    }
}

writeFileAsync();
```

```javascript
/**
 * 流式写入大文件
 * @author erik.zhou
 */

const fs = require('fs');

// 创建可写流
const writeStream = fs.createWriteStream('output.txt', {
    encoding: 'utf8',
    flags: 'w' // 'w'覆盖，'a'追加
});

// 写入数据
for (let i = 0; i < 1000000; i++) {
    const canWrite = writeStream.write(`Line ${i}\n`);
    
    // 如果缓冲区满了，等待drain事件
    if (!canWrite) {
        await new Promise(resolve => {
            writeStream.once('drain', resolve);
        });
    }
}

// 结束写入
writeStream.end(() => {
    console.log('写入完成');
});

writeStream.on('error', (err) => {
    console.error('写入失败:', err);
});
```

### 6.3 目录操作

```javascript
/**
 * 目录基本操作
 * @author erik.zhou
 */

const fs = require('fs');
const path = require('path');

// 创建目录
fs.mkdirSync('new-dir');

// 递归创建目录
fs.mkdirSync('parent/child/grandchild', { recursive: true });

// 读取目录
const files = fs.readdirSync('.');
console.log('文件列表:', files);

// 读取目录详细信息
const entries = fs.readdirSync('.', { withFileTypes: true });
entries.forEach(entry => {
    console.log(entry.name, entry.isDirectory() ? '目录' : '文件');
});

// 删除目录
fs.rmdirSync('new-dir');

// 递归删除目录
fs.rmSync('parent', { recursive: true, force: true });
```

```javascript
/**
 * 异步目录操作
 * @author erik.zhou
 */

const fsPromises = require('fs').promises;

async function directoryOperations() {
    try {
        // 创建目录
        await fsPromises.mkdir('new-dir', { recursive: true });
        
        // 读取目录
        const files = await fsPromises.readdir('.');
        console.log('文件列表:', files);
        
        // 读取目录详细信息
        const entries = await fsPromises.readdir('.', { withFileTypes: true });
        for (const entry of entries) {
            const stats = await fsPromises.stat(entry.name);
            console.log(entry.name, {
                isFile: stats.isFile(),
                isDirectory: stats.isDirectory(),
                size: stats.size,
                created: stats.birthtime,
                modified: stats.mtime
            });
        }
        
        // 删除目录
        await fsPromises.rmdir('new-dir');
    } catch (err) {
        console.error('操作失败:', err);
    }
}

directoryOperations();
```

```javascript
/**
 * 递归遍历目录
 * @author erik.zhou
 */

const fs = require('fs');
const path = require('path');

function walkDirectory(dir, callback) {
    const files = fs.readdirSync(dir);
    
    files.forEach(file => {
        const filePath = path.join(dir, file);
        const stats = fs.statSync(filePath);
        
        if (stats.isDirectory()) {
            walkDirectory(filePath, callback);
        } else {
            callback(filePath, stats);
        }
    });
}

// 使用示例
walkDirectory('.', (filePath, stats) => {
    console.log(filePath, stats.size);
});

// 异步版本
async function walkDirectoryAsync(dir, callback) {
    const files = await fsPromises.readdir(dir);
    
    for (const file of files) {
        const filePath = path.join(dir, file);
        const stats = await fsPromises.stat(filePath);
        
        if (stats.isDirectory()) {
            await walkDirectoryAsync(filePath, callback);
        } else {
            await callback(filePath, stats);
        }
    }
}
```

### 6.4 文件信息

```javascript
/**
 * 获取文件信息
 * @author erik.zhou
 */

const fs = require('fs');

// 同步获取
const stats = fs.statSync('data.txt');

console.log('文件信息:', {
    size: stats.size,
    isFile: stats.isFile(),
    isDirectory: stats.isDirectory(),
    isSymbolicLink: stats.isSymbolicLink(),
    created: stats.birthtime,
    modified: stats.mtime,
    accessed: stats.atime,
    mode: stats.mode.toString(8),
    uid: stats.uid,
    gid: stats.gid
});

// 异步获取
fs.stat('data.txt', (err, stats) => {
    if (err) {
        console.error('获取失败:', err);
        return;
    }
    console.log('文件大小:', stats.size);
});

// 检查文件是否存在
if (fs.existsSync('data.txt')) {
    console.log('文件存在');
}

// 使用access检查权限
fs.access('data.txt', fs.constants.R_OK | fs.constants.W_OK, (err) => {
    if (err) {
        console.log('文件不可读写');
    } else {
        console.log('文件可读写');
    }
});
```

### 6.5 文件监听

```javascript
/**
 * 监听文件变化
 * @author erik.zhou
 */

const fs = require('fs');

// watch方法
const watcher = fs.watch('data.txt', (eventType, filename) => {
    console.log('事件类型:', eventType);
    console.log('文件名:', filename);
});

// 停止监听
setTimeout(() => {
    watcher.close();
}, 60000);

// watchFile方法（轮询）
fs.watchFile('data.txt', { interval: 1000 }, (curr, prev) => {
    console.log('当前修改时间:', curr.mtime);
    console.log('之前修改时间:', prev.mtime);
});

// 停止监听
setTimeout(() => {
    fs.unwatchFile('data.txt');
}, 60000);
```

```javascript
/**
 * 使用chokidar监听文件
 * @author erik.zhou
 */

// npm install chokidar
const chokidar = require('chokidar');

// 监听文件和目录
const watcher = chokidar.watch('src/**/*.js', {
    ignored: /(^|[\/\\])\../, // 忽略隐藏文件
    persistent: true,
    ignoreInitial: true
});

watcher
    .on('add', path => console.log(`文件添加: ${path}`))
    .on('change', path => console.log(`文件修改: ${path}`))
    .on('unlink', path => console.log(`文件删除: ${path}`))
    .on('addDir', path => console.log(`目录添加: ${path}`))
    .on('unlinkDir', path => console.log(`目录删除: ${path}`))
    .on('error', error => console.error(`监听错误: ${error}`))
    .on('ready', () => console.log('初始扫描完成'));

// 停止监听
setTimeout(() => {
    watcher.close();
}, 60000);
```

### 6.6 文件操作实战

```javascript
/**
 * 复制文件
 * @author erik.zhou
 */

const fs = require('fs');
const fsPromises = require('fs').promises;

// 同步复制
fs.copyFileSync('source.txt', 'dest.txt');

// 异步复制
async function copyFile(src, dest) {
    try {
        await fsPromises.copyFile(src, dest);
        console.log('复制成功');
    } catch (err) {
        console.error('复制失败:', err);
    }
}

// 流式复制（大文件）
function copyFileStream(src, dest) {
    return new Promise((resolve, reject) => {
        const readStream = fs.createReadStream(src);
        const writeStream = fs.createWriteStream(dest);
        
        readStream.pipe(writeStream);
        
        writeStream.on('finish', resolve);
        writeStream.on('error', reject);
        readStream.on('error', reject);
    });
}
```

```javascript
/**
 * 移动和重命名文件
 * @author erik.zhou
 */

// 重命名文件
fs.renameSync('old.txt', 'new.txt');

// 移动文件
fs.renameSync('file.txt', 'dir/file.txt');

// 异步版本
async function moveFile(src, dest) {
    try {
        await fsPromises.rename(src, dest);
        console.log('移动成功');
    } catch (err) {
        // 跨分区移动需要复制后删除
        if (err.code === 'EXDEV') {
            await fsPromises.copyFile(src, dest);
            await fsPromises.unlink(src);
            console.log('跨分区移动成功');
        } else {
            throw err;
        }
    }
}
```

```javascript
/**
 * 删除文件
 * @author erik.zhou
 */

// 删除文件
fs.unlinkSync('file.txt');

// 异步删除
async function deleteFile(filePath) {
    try {
        await fsPromises.unlink(filePath);
        console.log('删除成功');
    } catch (err) {
        if (err.code === 'ENOENT') {
            console.log('文件不存在');
        } else {
            throw err;
        }
    }
}

// 批量删除
async function deleteFiles(pattern) {
    const glob = require('glob');
    const files = glob.sync(pattern);
    
    for (const file of files) {
        await fsPromises.unlink(file);
        console.log('删除:', file);
    }
}

// 使用示例
deleteFiles('temp/*.tmp');
```

```javascript
/**
 * 文件权限管理
 * @author erik.zhou
 */

// 修改文件权限
fs.chmodSync('file.txt', 0o644); // rw-r--r--

// 修改文件所有者
fs.chownSync('file.txt', 1000, 1000);

// 异步版本
async function changePermissions(filePath, mode) {
    try {
        await fsPromises.chmod(filePath, mode);
        console.log('权限修改成功');
    } catch (err) {
        console.error('权限修改失败:', err);
    }
}

// 检查权限
fs.access('file.txt', fs.constants.R_OK, (err) => {
    if (err) {
        console.log('文件不可读');
    } else {
        console.log('文件可读');
    }
});
```

---

## 第7章 Stream流

### 7.1 可读流

```javascript
/**
 * 创建可读流
 * @author erik.zhou
 */

const fs = require('fs');
const { Readable } = require('stream');

// 从文件创建可读流
const readStream = fs.createReadStream('data.txt', {
    encoding: 'utf8',
    highWaterMark: 16 * 1024 // 16KB缓冲区
});

readStream.on('data', (chunk) => {
    console.log('接收数据:', chunk.length);
});

readStream.on('end', () => {
    console.log('读取完成');
});

readStream.on('error', (err) => {
    console.error('读取错误:', err);
});

// 自定义可读流
class MyReadable extends Readable {
    constructor(options) {
        super(options);
        this.index = 0;
        this.max = 10;
    }
    
    _read() {
        if (this.index < this.max) {
            this.push(`数据 ${this.index}\n`);
            this.index++;
        } else {
            this.push(null); // 结束流
        }
    }
}

const myStream = new MyReadable();
myStream.on('data', (chunk) => {
    console.log('数据:', chunk.toString());
});
```

```javascript
/**
 * 可读流模式
 * @author erik.zhou
 */

const fs = require('fs');

// 流动模式（flowing mode）
const readStream1 = fs.createReadStream('data.txt');

readStream1.on('data', (chunk) => {
    console.log('流动模式:', chunk.length);
});

// 暂停模式（paused mode）
const readStream2 = fs.createReadStream('data.txt');

readStream2.on('readable', () => {
    let chunk;
    while ((chunk = readStream2.read()) !== null) {
        console.log('暂停模式:', chunk.length);
    }
});

// 切换模式
const readStream3 = fs.createReadStream('data.txt');

readStream3.on('data', (chunk) => {
    console.log('接收数据');
    readStream3.pause(); // 切换到暂停模式
    
    setTimeout(() => {
        readStream3.resume(); // 切换回流动模式
    }, 1000);
});
```

### 7.2 可写流

```javascript
/**
 * 创建可写流
 * @author erik.zhou
 */

const fs = require('fs');
const { Writable } = require('stream');

// 从文件创建可写流
const writeStream = fs.createWriteStream('output.txt', {
    encoding: 'utf8',
    flags: 'w'
});

writeStream.write('第一行\n');
writeStream.write('第二行\n');
writeStream.end('最后一行\n');

writeStream.on('finish', () => {
    console.log('写入完成');
});

writeStream.on('error', (err) => {
    console.error('写入错误:', err);
});

// 自定义可写流
class MyWritable extends Writable {
    _write(chunk, encoding, callback) {
        console.log('写入数据:', chunk.toString());
        callback();
    }
}

const myStream = new MyWritable();
myStream.write('Hello');
myStream.write('World');
myStream.end();
```

```javascript
/**
 * 背压处理
 * @author erik.zhou
 */

const fs = require('fs');

const writeStream = fs.createWriteStream('output.txt');

function writeMillionLines() {
    let i = 0;
    
    function write() {
        let ok = true;
        
        while (i < 1000000 && ok) {
            ok = writeStream.write(`行 ${i}\n`);
            i++;
        }
        
        if (i < 1000000) {
            // 缓冲区满了，等待drain事件
            writeStream.once('drain', write);
        } else {
            writeStream.end();
        }
    }
    
    write();
}

writeMillionLines();

writeStream.on('finish', () => {
    console.log('写入完成');
});
```

### 7.3 双工流

```javascript
/**
 * 双工流（Duplex）
 * @author erik.zhou
 */

const { Duplex } = require('stream');

// 自定义双工流
class MyDuplex extends Duplex {
    constructor(options) {
        super(options);
        this.data = [];
    }
    
    _read() {
        if (this.data.length > 0) {
            this.push(this.data.shift());
        } else {
            this.push(null);
        }
    }
    
    _write(chunk, encoding, callback) {
        this.data.push(chunk);
        callback();
    }
}

const duplex = new MyDuplex();

// 写入数据
duplex.write('Hello');
duplex.write('World');
duplex.end();

// 读取数据
duplex.on('data', (chunk) => {
    console.log('读取:', chunk.toString());
});
```

### 7.4 转换流

```javascript
/**
 * 转换流（Transform）
 * @author erik.zhou
 */

const { Transform } = require('stream');

// 转大写转换流
class UpperCaseTransform extends Transform {
    _transform(chunk, encoding, callback) {
        this.push(chunk.toString().toUpperCase());
        callback();
    }
}

const upperCase = new UpperCaseTransform();

process.stdin
    .pipe(upperCase)
    .pipe(process.stdout);

// CSV解析转换流
class CSVParser extends Transform {
    constructor(options) {
        super(options);
        this.headers = null;
    }
    
    _transform(chunk, encoding, callback) {
        const lines = chunk.toString().split('\n');
        
        lines.forEach((line, index) => {
            if (index === 0 && !this.headers) {
                this.headers = line.split(',');
            } else if (line) {
                const values = line.split(',');
                const obj = {};
                
                this.headers.forEach((header, i) => {
                    obj[header] = values[i];
                });
                
                this.push(JSON.stringify(obj) + '\n');
            }
        });
        
        callback();
    }
}

const csvParser = new CSVParser();

fs.createReadStream('data.csv')
    .pipe(csvParser)
    .pipe(fs.createWriteStream('data.json'));
```

### 7.5 管道操作

```javascript
/**
 * 管道（Pipe）
 * @author erik.zhou
 */

const fs = require('fs');
const zlib = require('zlib');

// 简单管道
fs.createReadStream('input.txt')
    .pipe(fs.createWriteStream('output.txt'));

// 链式管道
fs.createReadStream('input.txt')
    .pipe(zlib.createGzip())
    .pipe(fs.createWriteStream('input.txt.gz'));

// 解压缩
fs.createReadStream('input.txt.gz')
    .pipe(zlib.createGunzip())
    .pipe(fs.createWriteStream('output.txt'));

// 错误处理
const readStream = fs.createReadStream('input.txt');
const writeStream = fs.createWriteStream('output.txt');

readStream
    .on('error', (err) => {
        console.error('读取错误:', err);
    })
    .pipe(writeStream)
    .on('error', (err) => {
        console.error('写入错误:', err);
    })
    .on('finish', () => {
        console.log('复制完成');
    });
```

```javascript
/**
 * pipeline函数
 * @author erik.zhou
 */

const { pipeline } = require('stream');
const fs = require('fs');
const zlib = require('zlib');

// 使用pipeline处理错误
pipeline(
    fs.createReadStream('input.txt'),
    zlib.createGzip(),
    fs.createWriteStream('input.txt.gz'),
    (err) => {
        if (err) {
            console.error('管道错误:', err);
        } else {
            console.log('管道完成');
        }
    }
);

// Promise版本
const { pipeline: pipelinePromise } = require('stream/promises');

async function compressFile(input, output) {
    try {
        await pipelinePromise(
            fs.createReadStream(input),
            zlib.createGzip(),
            fs.createWriteStream(output)
        );
        console.log('压缩完成');
    } catch (err) {
        console.error('压缩失败:', err);
    }
}

compressFile('input.txt', 'input.txt.gz');
```



### 7.6 Stream实战应用

```javascript
/**
 * 大文件处理
 * @author erik.zhou
 */

const fs = require('fs');
const readline = require('readline');

// 逐行读取大文件
async function processLargeFile(filename) {
    const fileStream = fs.createReadStream(filename);
    
    const rl = readline.createInterface({
        input: fileStream,
        crlfDelay: Infinity
    });
    
    let lineCount = 0;
    
    for await (const line of rl) {
        lineCount++;
        // 处理每一行
        if (line.includes('ERROR')) {
            console.log(`错误行 ${lineCount}:`, line);
        }
    }
    
    console.log(`总行数: ${lineCount}`);
}

processLargeFile('large-log.txt');
```

```javascript
/**
 * 文件分片上传
 * @author erik.zhou
 */

const fs = require('fs');
const path = require('path');

async function uploadFileInChunks(filePath, chunkSize = 1024 * 1024) {
    const stats = fs.statSync(filePath);
    const totalChunks = Math.ceil(stats.size / chunkSize);
    
    for (let i = 0; i < totalChunks; i++) {
        const start = i * chunkSize;
        const end = Math.min(start + chunkSize, stats.size);
        
        const chunk = await new Promise((resolve, reject) => {
            const stream = fs.createReadStream(filePath, { start, end: end - 1 });
            const chunks = [];
            
            stream.on('data', (chunk) => chunks.push(chunk));
            stream.on('end', () => resolve(Buffer.concat(chunks)));
            stream.on('error', reject);
        });
        
        // 上传分片
        await uploadChunk(chunk, i, totalChunks);
        console.log(`上传进度: ${((i + 1) / totalChunks * 100).toFixed(2)}%`);
    }
}

async function uploadChunk(chunk, index, total) {
    // 模拟上传
    return new Promise(resolve => setTimeout(resolve, 100));
}
```

```javascript
/**
 * 实时日志处理
 * @author erik.zhou
 */

const fs = require('fs');
const { Transform } = require('stream');

// 日志过滤转换流
class LogFilter extends Transform {
    constructor(level) {
        super();
        this.level = level;
    }
    
    _transform(chunk, encoding, callback) {
        const lines = chunk.toString().split('\n');
        
        lines.forEach(line => {
            if (line.includes(this.level)) {
                this.push(line + '\n');
            }
        });
        
        callback();
    }
}

// 监听日志文件并过滤
const logStream = fs.createReadStream('app.log');
const errorFilter = new LogFilter('ERROR');
const errorLog = fs.createWriteStream('error.log');

logStream
    .pipe(errorFilter)
    .pipe(errorLog);

console.log('开始过滤错误日志...');
```

---

## 第8章 Buffer缓冲区

### 8.1 Buffer基础

```javascript
/**
 * 创建Buffer
 * @author erik.zhou
 */

// 创建指定大小的Buffer
const buf1 = Buffer.alloc(10); // 初始化为0
console.log('buf1:', buf1);

const buf2 = Buffer.allocUnsafe(10); // 未初始化，可能包含旧数据
console.log('buf2:', buf2);

// 从字符串创建
const buf3 = Buffer.from('Hello World');
console.log('buf3:', buf3);
console.log('字符串:', buf3.toString());

// 从数组创建
const buf4 = Buffer.from([0x48, 0x65, 0x6c, 0x6c, 0x6f]);
console.log('buf4:', buf4.toString()); // Hello

// 从另一个Buffer创建
const buf5 = Buffer.from(buf3);
console.log('buf5:', buf5.toString());
```

### 8.2 Buffer操作

```javascript
/**
 * Buffer读写操作
 * @author erik.zhou
 */

const buf = Buffer.alloc(10);

// 写入数据
buf.write('Hello');
console.log('写入后:', buf.toString());

// 写入指定位置
buf.write('World', 5);
console.log('写入后:', buf.toString());

// 读取数据
const str = buf.toString('utf8', 0, 5);
console.log('读取:', str);

// 填充Buffer
buf.fill(0);
console.log('填充后:', buf);

buf.fill('ab');
console.log('填充后:', buf.toString());

// 复制Buffer
const buf1 = Buffer.from('Hello');
const buf2 = Buffer.alloc(5);
buf1.copy(buf2);
console.log('复制后:', buf2.toString());

// 切片Buffer
const buf3 = Buffer.from('Hello World');
const slice = buf3.slice(0, 5);
console.log('切片:', slice.toString());
```

```javascript
/**
 * Buffer比较和拼接
 * @author erik.zhou
 */

// 比较Buffer
const buf1 = Buffer.from('ABC');
const buf2 = Buffer.from('ABD');
const buf3 = Buffer.from('ABC');

console.log('buf1 vs buf2:', buf1.compare(buf2)); // -1
console.log('buf1 vs buf3:', buf1.compare(buf3)); // 0
console.log('buf1 equals buf3:', buf1.equals(buf3)); // true

// 拼接Buffer
const buf4 = Buffer.from('Hello ');
const buf5 = Buffer.from('World');
const buf6 = Buffer.concat([buf4, buf5]);
console.log('拼接:', buf6.toString());

// 拼接多个Buffer
const buffers = [
    Buffer.from('Hello'),
    Buffer.from(' '),
    Buffer.from('World')
];
const result = Buffer.concat(buffers);
console.log('拼接:', result.toString());
```

### 8.3 编码转换

```javascript
/**
 * Buffer编码
 * @author erik.zhou
 */

const text = 'Hello 世界';

// 不同编码
const utf8 = Buffer.from(text, 'utf8');
const utf16le = Buffer.from(text, 'utf16le');
const base64 = Buffer.from(text, 'utf8').toString('base64');
const hex = Buffer.from(text, 'utf8').toString('hex');

console.log('UTF-8:', utf8);
console.log('UTF-16LE:', utf16le);
console.log('Base64:', base64);
console.log('Hex:', hex);

// 解码
console.log('解码Base64:', Buffer.from(base64, 'base64').toString('utf8'));
console.log('解码Hex:', Buffer.from(hex, 'hex').toString('utf8'));

// 支持的编码
console.log('支持的编码:', Buffer.isEncoding('utf8')); // true
console.log('支持的编码:', Buffer.isEncoding('gbk')); // false
```

```javascript
/**
 * 二进制数据处理
 * @author erik.zhou
 */

const buf = Buffer.alloc(8);

// 写入整数
buf.writeInt8(127, 0);
buf.writeInt16BE(32767, 1);
buf.writeInt32BE(2147483647, 3);

console.log('Buffer:', buf);

// 读取整数
console.log('Int8:', buf.readInt8(0));
console.log('Int16BE:', buf.readInt16BE(1));
console.log('Int32BE:', buf.readInt32BE(3));

// 写入浮点数
const floatBuf = Buffer.alloc(8);
floatBuf.writeFloatBE(3.14, 0);
floatBuf.writeDoubleBE(3.141592653589793, 4);

console.log('Float:', floatBuf.readFloatBE(0));
console.log('Double:', floatBuf.readDoubleBE(4));
```

### 8.4 Buffer与Stream

```javascript
/**
 * Buffer在Stream中的应用
 * @author erik.zhou
 */

const fs = require('fs');
const { Transform } = require('stream');

// Buffer转换流
class BufferTransform extends Transform {
    _transform(chunk, encoding, callback) {
        // chunk是Buffer对象
        console.log('接收Buffer:', chunk.length);
        
        // 处理Buffer
        const processed = Buffer.from(chunk.toString().toUpperCase());
        
        this.push(processed);
        callback();
    }
}

fs.createReadStream('input.txt')
    .pipe(new BufferTransform())
    .pipe(fs.createWriteStream('output.txt'));
```

```javascript
/**
 * Buffer池化
 * @author erik.zhou
 */

// Buffer池化可以减少内存分配
class BufferPool {
    constructor(size) {
        this.pool = Buffer.allocUnsafe(size);
        this.offset = 0;
    }
    
    alloc(size) {
        if (this.offset + size > this.pool.length) {
            throw new Error('Buffer池已满');
        }
        
        const buf = this.pool.slice(this.offset, this.offset + size);
        this.offset += size;
        
        return buf;
    }
    
    reset() {
        this.offset = 0;
    }
}

const pool = new BufferPool(1024);

const buf1 = pool.alloc(100);
const buf2 = pool.alloc(200);

console.log('已分配:', pool.offset);

pool.reset();
console.log('重置后:', pool.offset);
```

### 8.5 Buffer性能优化

```javascript
/**
 * Buffer性能优化
 * @author erik.zhou
 */

// 避免频繁创建Buffer
function inefficient() {
    const start = Date.now();
    
    for (let i = 0; i < 100000; i++) {
        const buf = Buffer.from('Hello World');
    }
    
    console.log('耗时:', Date.now() - start, 'ms');
}

// 使用Buffer池
function efficient() {
    const start = Date.now();
    const buf = Buffer.alloc(11);
    
    for (let i = 0; i < 100000; i++) {
        buf.write('Hello World');
    }
    
    console.log('耗时:', Date.now() - start, 'ms');
}

inefficient();
efficient();

// 使用allocUnsafe提高性能（注意安全性）
function fastAlloc() {
    const start = Date.now();
    
    for (let i = 0; i < 100000; i++) {
        const buf = Buffer.allocUnsafe(1024);
        buf.fill(0); // 手动清零
    }
    
    console.log('耗时:', Date.now() - start, 'ms');
}

fastAlloc();
```

---

## 第9章 网络编程

### 9.1 HTTP服务器

```javascript
/**
 * 创建HTTP服务器
 * @author erik.zhou
 */

const http = require('http');

// 创建服务器
const server = http.createServer((req, res) => {
    console.log('请求方法:', req.method);
    console.log('请求URL:', req.url);
    console.log('请求头:', req.headers);
    
    // 设置响应头
    res.setHeader('Content-Type', 'text/plain; charset=utf-8');
    res.statusCode = 200;
    
    // 发送响应
    res.end('Hello World');
});

// 监听端口
server.listen(3000, () => {
    console.log('服务器运行在 http://localhost:3000');
});

// 错误处理
server.on('error', (err) => {
    console.error('服务器错误:', err);
});
```

```javascript
/**
 * 处理不同路由
 * @author erik.zhou
 */

const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
    const parsedUrl = url.parse(req.url, true);
    const pathname = parsedUrl.pathname;
    const query = parsedUrl.query;
    
    res.setHeader('Content-Type', 'application/json; charset=utf-8');
    
    if (pathname === '/') {
        res.statusCode = 200;
        res.end(JSON.stringify({ message: '首页' }));
    } else if (pathname === '/api/users') {
        res.statusCode = 200;
        res.end(JSON.stringify({
            users: [
                { id: 1, name: 'John' },
                { id: 2, name: 'Jane' }
            ]
        }));
    } else if (pathname === '/api/user') {
        const userId = query.id;
        res.statusCode = 200;
        res.end(JSON.stringify({
            user: { id: userId, name: 'John' }
        }));
    } else {
        res.statusCode = 404;
        res.end(JSON.stringify({ error: '未找到' }));
    }
});

server.listen(3000);
```

```javascript
/**
 * 处理POST请求
 * @author erik.zhou
 */

const http = require('http');

const server = http.createServer((req, res) => {
    if (req.method === 'POST') {
        let body = '';
        
        // 接收数据
        req.on('data', (chunk) => {
            body += chunk.toString();
        });
        
        // 数据接收完成
        req.on('end', () => {
            try {
                const data = JSON.parse(body);
                console.log('接收数据:', data);
                
                res.setHeader('Content-Type', 'application/json');
                res.statusCode = 200;
                res.end(JSON.stringify({
                    success: true,
                    data: data
                }));
            } catch (err) {
                res.statusCode = 400;
                res.end(JSON.stringify({
                    error: '无效的JSON'
                }));
            }
        });
    } else {
        res.statusCode = 405;
        res.end('Method Not Allowed');
    }
});

server.listen(3000);
```

### 9.2 HTTP客户端

```javascript
/**
 * 发送HTTP请求
 * @author erik.zhou
 */

const http = require('http');

// GET请求
const options = {
    hostname: 'api.example.com',
    port: 80,
    path: '/api/users',
    method: 'GET',
    headers: {
        'Content-Type': 'application/json'
    }
};

const req = http.request(options, (res) => {
    console.log('状态码:', res.statusCode);
    console.log('响应头:', res.headers);
    
    let data = '';
    
    res.on('data', (chunk) => {
        data += chunk;
    });
    
    res.on('end', () => {
        console.log('响应数据:', data);
    });
});

req.on('error', (err) => {
    console.error('请求错误:', err);
});

req.end();

// 简化的GET请求
http.get('http://api.example.com/api/users', (res) => {
    let data = '';
    
    res.on('data', (chunk) => {
        data += chunk;
    });
    
    res.on('end', () => {
        console.log('响应:', data);
    });
});
```

```javascript
/**
 * 发送POST请求
 * @author erik.zhou
 */

const http = require('http');

const postData = JSON.stringify({
    name: 'John',
    age: 30
});

const options = {
    hostname: 'api.example.com',
    port: 80,
    path: '/api/users',
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Content-Length': Buffer.byteLength(postData)
    }
};

const req = http.request(options, (res) => {
    let data = '';
    
    res.on('data', (chunk) => {
        data += chunk;
    });
    
    res.on('end', () => {
        console.log('响应:', data);
    });
});

req.on('error', (err) => {
    console.error('请求错误:', err);
});

req.write(postData);
req.end();
```

### 9.3 HTTPS

```javascript
/**
 * HTTPS服务器
 * @author erik.zhou
 */

const https = require('https');
const fs = require('fs');

// 读取证书
const options = {
    key: fs.readFileSync('private-key.pem'),
    cert: fs.readFileSync('certificate.pem')
};

// 创建HTTPS服务器
const server = https.createServer(options, (req, res) => {
    res.writeHead(200);
    res.end('Hello HTTPS');
});

server.listen(443, () => {
    console.log('HTTPS服务器运行在 https://localhost:443');
});
```

```javascript
/**
 * HTTPS客户端
 * @author erik.zhou
 */

const https = require('https');

// GET请求
https.get('https://api.example.com/data', (res) => {
    let data = '';
    
    res.on('data', (chunk) => {
        data += chunk;
    });
    
    res.on('end', () => {
        console.log('响应:', data);
    });
}).on('error', (err) => {
    console.error('请求错误:', err);
});

// 忽略证书验证（仅用于开发）
const options = {
    hostname: 'api.example.com',
    path: '/data',
    rejectUnauthorized: false
};

https.get(options, (res) => {
    // 处理响应
});
```

### 9.4 TCP服务器

```javascript
/**
 * TCP服务器
 * @author erik.zhou
 */

const net = require('net');

// 创建TCP服务器
const server = net.createServer((socket) => {
    console.log('客户端连接:', socket.remoteAddress, socket.remotePort);
    
    // 接收数据
    socket.on('data', (data) => {
        console.log('接收数据:', data.toString());
        
        // 回复数据
        socket.write('服务器收到: ' + data);
    });
    
    // 连接关闭
    socket.on('end', () => {
        console.log('客户端断开连接');
    });
    
    // 错误处理
    socket.on('error', (err) => {
        console.error('Socket错误:', err);
    });
});

server.listen(8080, () => {
    console.log('TCP服务器运行在端口 8080');
});

server.on('error', (err) => {
    console.error('服务器错误:', err);
});
```

```javascript
/**
 * TCP客户端
 * @author erik.zhou
 */

const net = require('net');

// 创建TCP客户端
const client = net.createConnection({
    host: 'localhost',
    port: 8080
}, () => {
    console.log('连接到服务器');
    
    // 发送数据
    client.write('Hello Server');
});

// 接收数据
client.on('data', (data) => {
    console.log('接收数据:', data.toString());
    client.end();
});

// 连接关闭
client.on('end', () => {
    console.log('断开连接');
});

// 错误处理
client.on('error', (err) => {
    console.error('客户端错误:', err);
});
```

### 9.5 UDP通信

```javascript
/**
 * UDP服务器
 * @author erik.zhou
 */

const dgram = require('dgram');

// 创建UDP服务器
const server = dgram.createSocket('udp4');

server.on('message', (msg, rinfo) => {
    console.log(`接收消息: ${msg} 来自 ${rinfo.address}:${rinfo.port}`);
    
    // 回复消息
    const reply = Buffer.from('服务器收到: ' + msg);
    server.send(reply, rinfo.port, rinfo.address, (err) => {
        if (err) {
            console.error('发送失败:', err);
        }
    });
});

server.on('listening', () => {
    const address = server.address();
    console.log(`UDP服务器运行在 ${address.address}:${address.port}`);
});

server.on('error', (err) => {
    console.error('服务器错误:', err);
    server.close();
});

server.bind(8080);
```

```javascript
/**
 * UDP客户端
 * @author erik.zhou
 */

const dgram = require('dgram');

// 创建UDP客户端
const client = dgram.createSocket('udp4');

const message = Buffer.from('Hello Server');

// 发送消息
client.send(message, 8080, 'localhost', (err) => {
    if (err) {
        console.error('发送失败:', err);
        client.close();
    } else {
        console.log('消息已发送');
    }
});

// 接收消息
client.on('message', (msg, rinfo) => {
    console.log(`接收消息: ${msg} 来自 ${rinfo.address}:${rinfo.port}`);
    client.close();
});

// 错误处理
client.on('error', (err) => {
    console.error('客户端错误:', err);
    client.close();
});
```

### 9.6 WebSocket

```javascript
/**
 * WebSocket服务器
 * @author erik.zhou
 */

// npm install ws
const WebSocket = require('ws');

// 创建WebSocket服务器
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
    console.log('客户端连接');
    
    // 接收消息
    ws.on('message', (message) => {
        console.log('接收消息:', message.toString());
        
        // 回复消息
        ws.send('服务器收到: ' + message);
    });
    
    // 连接关闭
    ws.on('close', () => {
        console.log('客户端断开连接');
    });
    
    // 错误处理
    ws.on('error', (err) => {
        console.error('WebSocket错误:', err);
    });
    
    // 发送欢迎消息
    ws.send('欢迎连接WebSocket服务器');
});

console.log('WebSocket服务器运行在 ws://localhost:8080');
```

```javascript
/**
 * WebSocket客户端
 * @author erik.zhou
 */

const WebSocket = require('ws');

// 创建WebSocket客户端
const ws = new WebSocket('ws://localhost:8080');

ws.on('open', () => {
    console.log('连接到服务器');
    
    // 发送消息
    ws.send('Hello Server');
});

ws.on('message', (data) => {
    console.log('接收消息:', data.toString());
});

ws.on('close', () => {
    console.log('断开连接');
});

ws.on('error', (err) => {
    console.error('WebSocket错误:', err);
});
```



---

## 第10章 进程与线程

### 10.1 进程信息

```javascript
/**
 * 进程基本信息
 * @author erik.zhou
 */

// 进程ID
console.log('进程ID:', process.pid);
console.log('父进程ID:', process.ppid);

// 进程标题
console.log('进程标题:', process.title);
process.title = 'my-node-app';

// 当前工作目录
console.log('当前目录:', process.cwd());
process.chdir('/tmp');
console.log('切换后:', process.cwd());

// 环境变量
console.log('环境变量:', process.env);
console.log('NODE_ENV:', process.env.NODE_ENV);

// 平台信息
console.log('平台:', process.platform);
console.log('架构:', process.arch);
console.log('版本:', process.version);
console.log('Node版本:', process.versions);

// 内存使用
const memUsage = process.memoryUsage();
console.log('内存使用:', {
    rss: `${Math.round(memUsage.rss / 1024 / 1024)}MB`,
    heapTotal: `${Math.round(memUsage.heapTotal / 1024 / 1024)}MB`,
    heapUsed: `${Math.round(memUsage.heapUsed / 1024 / 1024)}MB`,
    external: `${Math.round(memUsage.external / 1024 / 1024)}MB`
});

// CPU使用
const cpuUsage = process.cpuUsage();
console.log('CPU使用:', cpuUsage);

// 运行时间
console.log('运行时间:', process.uptime(), '秒');
```

### 10.2 子进程

```javascript
/**
 * 创建子进程 - exec
 * @author erik.zhou
 */

const { exec } = require('child_process');

// exec - 执行shell命令
exec('ls -la', (error, stdout, stderr) => {
    if (error) {
        console.error('执行错误:', error);
        return;
    }
    
    if (stderr) {
        console.error('错误输出:', stderr);
        return;
    }
    
    console.log('标准输出:', stdout);
});

// 带选项的exec
exec('ls -la', {
    cwd: '/tmp',
    env: { ...process.env, MY_VAR: 'value' },
    maxBuffer: 1024 * 1024
}, (error, stdout, stderr) => {
    if (error) {
        console.error('执行错误:', error);
        return;
    }
    console.log('输出:', stdout);
});

// Promise版本
const { promisify } = require('util');
const execPromise = promisify(exec);

async function runCommand() {
    try {
        const { stdout, stderr } = await execPromise('ls -la');
        console.log('输出:', stdout);
    } catch (error) {
        console.error('错误:', error);
    }
}

runCommand();
```

```javascript
/**
 * 创建子进程 - spawn
 * @author erik.zhou
 */

const { spawn } = require('child_process');

// spawn - 流式执行命令
const ls = spawn('ls', ['-la']);

ls.stdout.on('data', (data) => {
    console.log(`标准输出: ${data}`);
});

ls.stderr.on('data', (data) => {
    console.error(`错误输出: ${data}`);
});

ls.on('close', (code) => {
    console.log(`子进程退出，退出码: ${code}`);
});

ls.on('error', (error) => {
    console.error('启动失败:', error);
});

// 带选项的spawn
const child = spawn('node', ['script.js'], {
    cwd: '/tmp',
    env: { ...process.env, MY_VAR: 'value' },
    stdio: 'inherit' // 继承父进程的stdio
});

// 管道操作
const grep = spawn('grep', ['error']);
const cat = spawn('cat', ['log.txt']);

cat.stdout.pipe(grep.stdin);

grep.stdout.on('data', (data) => {
    console.log('匹配行:', data.toString());
});
```

```javascript
/**
 * 创建子进程 - fork
 * @author erik.zhou
 */

const { fork } = require('child_process');

// fork - 创建Node.js子进程
const child = fork('worker.js');

// 发送消息给子进程
child.send({ type: 'start', data: 'Hello' });

// 接收子进程消息
child.on('message', (message) => {
    console.log('收到消息:', message);
});

// 子进程退出
child.on('exit', (code) => {
    console.log('子进程退出:', code);
});

// worker.js
process.on('message', (message) => {
    console.log('收到父进程消息:', message);
    
    // 处理任务
    const result = processTask(message.data);
    
    // 发送结果给父进程
    process.send({ type: 'result', data: result });
});

function processTask(data) {
    // 处理任务
    return data.toUpperCase();
}
```

```javascript
/**
 * 创建子进程 - execFile
 * @author erik.zhou
 */

const { execFile } = require('child_process');

// execFile - 执行可执行文件
execFile('node', ['--version'], (error, stdout, stderr) => {
    if (error) {
        console.error('执行错误:', error);
        return;
    }
    console.log('Node版本:', stdout);
});

// 执行自定义脚本
execFile('./script.sh', ['arg1', 'arg2'], (error, stdout, stderr) => {
    if (error) {
        console.error('执行错误:', error);
        return;
    }
    console.log('输出:', stdout);
});
```

### 10.3 进程通信

```javascript
/**
 * 父子进程通信
 * @author erik.zhou
 */

// parent.js
const { fork } = require('child_process');

const child = fork('child.js');

// 发送消息
child.send({ type: 'task', data: { id: 1, value: 100 } });

// 接收消息
child.on('message', (message) => {
    console.log('收到结果:', message);
    
    if (message.type === 'complete') {
        child.kill();
    }
});

// child.js
process.on('message', (message) => {
    if (message.type === 'task') {
        const result = message.data.value * 2;
        
        process.send({
            type: 'result',
            data: { id: message.data.id, result }
        });
        
        process.send({ type: 'complete' });
    }
});
```

```javascript
/**
 * 进程池
 * @author erik.zhou
 */

const { fork } = require('child_process');

class ProcessPool {
    constructor(size) {
        this.size = size;
        this.workers = [];
        this.queue = [];
        
        // 创建工作进程
        for (let i = 0; i < size; i++) {
            this.createWorker();
        }
    }
    
    createWorker() {
        const worker = fork('worker.js');
        worker.busy = false;
        
        worker.on('message', (message) => {
            worker.busy = false;
            
            if (worker.callback) {
                worker.callback(null, message);
                worker.callback = null;
            }
            
            this.processQueue();
        });
        
        worker.on('error', (error) => {
            console.error('Worker错误:', error);
        });
        
        this.workers.push(worker);
    }
    
    exec(task, callback) {
        const worker = this.workers.find(w => !w.busy);
        
        if (worker) {
            worker.busy = true;
            worker.callback = callback;
            worker.send(task);
        } else {
            this.queue.push({ task, callback });
        }
    }
    
    processQueue() {
        if (this.queue.length > 0) {
            const { task, callback } = this.queue.shift();
            this.exec(task, callback);
        }
    }
    
    close() {
        this.workers.forEach(worker => worker.kill());
    }
}

// 使用进程池
const pool = new ProcessPool(4);

for (let i = 0; i < 10; i++) {
    pool.exec({ id: i, value: i * 10 }, (err, result) => {
        if (err) {
            console.error('任务失败:', err);
        } else {
            console.log('任务完成:', result);
        }
    });
}
```

### 10.4 Cluster集群

```javascript
/**
 * Cluster基础
 * @author erik.zhou
 */

const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
    console.log(`主进程 ${process.pid} 正在运行`);
    
    // 创建工作进程
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    
    // 工作进程退出
    cluster.on('exit', (worker, code, signal) => {
        console.log(`工作进程 ${worker.process.pid} 已退出`);
        
        // 重启工作进程
        cluster.fork();
    });
    
    // 工作进程在线
    cluster.on('online', (worker) => {
        console.log(`工作进程 ${worker.process.pid} 已上线`);
    });
    
} else {
    // 工作进程创建HTTP服务器
    http.createServer((req, res) => {
        res.writeHead(200);
        res.end(`由进程 ${process.pid} 处理\n`);
    }).listen(8000);
    
    console.log(`工作进程 ${process.pid} 已启动`);
}
```

```javascript
/**
 * Cluster进程通信
 * @author erik.zhou
 */

const cluster = require('cluster');

if (cluster.isMaster) {
    const worker = cluster.fork();
    
    // 发送消息给工作进程
    worker.send({ type: 'task', data: 'Hello' });
    
    // 接收工作进程消息
    worker.on('message', (message) => {
        console.log('主进程收到消息:', message);
    });
    
} else {
    // 接收主进程消息
    process.on('message', (message) => {
        console.log('工作进程收到消息:', message);
        
        // 发送消息给主进程
        process.send({ type: 'result', data: 'World' });
    });
}
```

```javascript
/**
 * Cluster负载均衡
 * @author erik.zhou
 */

const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
    console.log(`主进程 ${process.pid} 正在运行`);
    
    // 统计信息
    const stats = {};
    
    // 创建工作进程
    for (let i = 0; i < numCPUs; i++) {
        const worker = cluster.fork();
        stats[worker.id] = 0;
        
        // 接收工作进程消息
        worker.on('message', (message) => {
            if (message.type === 'request') {
                stats[worker.id]++;
            }
        });
    }
    
    // 定期输出统计
    setInterval(() => {
        console.log('请求统计:', stats);
    }, 5000);
    
} else {
    http.createServer((req, res) => {
        // 通知主进程
        process.send({ type: 'request' });
        
        res.writeHead(200);
        res.end(`由进程 ${process.pid} 处理\n`);
    }).listen(8000);
}
```

### 10.5 Worker Threads

```javascript
/**
 * Worker Threads基础
 * @author erik.zhou
 */

const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

if (isMainThread) {
    // 主线程
    console.log('主线程:', process.pid);
    
    // 创建工作线程
    const worker = new Worker(__filename, {
        workerData: { value: 10 }
    });
    
    // 接收消息
    worker.on('message', (message) => {
        console.log('收到消息:', message);
    });
    
    // 线程退出
    worker.on('exit', (code) => {
        console.log('线程退出:', code);
    });
    
    // 错误处理
    worker.on('error', (error) => {
        console.error('线程错误:', error);
    });
    
} else {
    // 工作线程
    console.log('工作线程:', workerData);
    
    // 执行计算
    const result = workerData.value * 2;
    
    // 发送消息
    parentPort.postMessage({ result });
}
```

```javascript
/**
 * Worker Threads通信
 * @author erik.zhou
 */

// main.js
const { Worker } = require('worker_threads');

function runWorker(data) {
    return new Promise((resolve, reject) => {
        const worker = new Worker('./worker.js', {
            workerData: data
        });
        
        worker.on('message', resolve);
        worker.on('error', reject);
        worker.on('exit', (code) => {
            if (code !== 0) {
                reject(new Error(`Worker stopped with exit code ${code}`));
            }
        });
    });
}

async function main() {
    try {
        const result = await runWorker({ value: 100 });
        console.log('结果:', result);
    } catch (error) {
        console.error('错误:', error);
    }
}

main();

// worker.js
const { parentPort, workerData } = require('worker_threads');

// 执行CPU密集型任务
function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

const result = fibonacci(workerData.value);
parentPort.postMessage(result);
```

```javascript
/**
 * Worker Threads线程池
 * @author erik.zhou
 */

const { Worker } = require('worker_threads');

class WorkerPool {
    constructor(workerScript, poolSize) {
        this.workerScript = workerScript;
        this.poolSize = poolSize;
        this.workers = [];
        this.queue = [];
        
        // 创建工作线程
        for (let i = 0; i < poolSize; i++) {
            this.createWorker();
        }
    }
    
    createWorker() {
        const worker = new Worker(this.workerScript);
        worker.busy = false;
        
        worker.on('message', (message) => {
            worker.busy = false;
            
            if (worker.callback) {
                worker.callback(null, message);
                worker.callback = null;
            }
            
            this.processQueue();
        });
        
        worker.on('error', (error) => {
            if (worker.callback) {
                worker.callback(error);
                worker.callback = null;
            }
        });
        
        this.workers.push(worker);
    }
    
    exec(data) {
        return new Promise((resolve, reject) => {
            const worker = this.workers.find(w => !w.busy);
            
            if (worker) {
                worker.busy = true;
                worker.callback = (err, result) => {
                    if (err) reject(err);
                    else resolve(result);
                };
                worker.postMessage(data);
            } else {
                this.queue.push({ data, resolve, reject });
            }
        });
    }
    
    processQueue() {
        if (this.queue.length > 0) {
            const { data, resolve, reject } = this.queue.shift();
            this.exec(data).then(resolve).catch(reject);
        }
    }
    
    close() {
        this.workers.forEach(worker => worker.terminate());
    }
}

// 使用线程池
const pool = new WorkerPool('./worker.js', 4);

async function main() {
    const tasks = [];
    
    for (let i = 0; i < 10; i++) {
        tasks.push(pool.exec({ value: i }));
    }
    
    const results = await Promise.all(tasks);
    console.log('所有任务完成:', results);
    
    pool.close();
}

main();
```

### 10.6 进程管理

```javascript
/**
 * 进程信号处理
 * @author erik.zhou
 */

// 监听信号
process.on('SIGINT', () => {
    console.log('收到 SIGINT 信号');
    
    // 清理资源
    cleanup();
    
    // 退出进程
    process.exit(0);
});

process.on('SIGTERM', () => {
    console.log('收到 SIGTERM 信号');
    cleanup();
    process.exit(0);
});

process.on('uncaughtException', (error) => {
    console.error('未捕获的异常:', error);
    cleanup();
    process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
    console.error('未处理的Promise拒绝:', reason);
});

function cleanup() {
    console.log('清理资源...');
    // 关闭数据库连接
    // 保存状态
    // 等待请求完成
}

// 优雅退出
let isShuttingDown = false;

process.on('SIGTERM', async () => {
    if (isShuttingDown) return;
    isShuttingDown = true;
    
    console.log('开始优雅退出...');
    
    // 停止接收新请求
    server.close(() => {
        console.log('服务器已关闭');
    });
    
    // 等待现有请求完成
    await waitForRequests();
    
    // 关闭数据库连接
    await closeDatabase();
    
    console.log('优雅退出完成');
    process.exit(0);
});
```

```javascript
/**
 * 使用PM2管理进程
 * @author erik.zhou
 */

// ecosystem.config.js
module.exports = {
    apps: [{
        name: 'my-app',
        script: './app.js',
        instances: 4,
        exec_mode: 'cluster',
        watch: true,
        ignore_watch: ['node_modules', 'logs'],
        max_memory_restart: '1G',
        env: {
            NODE_ENV: 'development'
        },
        env_production: {
            NODE_ENV: 'production'
        },
        error_file: './logs/error.log',
        out_file: './logs/out.log',
        log_date_format: 'YYYY-MM-DD HH:mm:ss',
        merge_logs: true,
        autorestart: true,
        max_restarts: 10,
        min_uptime: '10s'
    }]
};

// PM2命令
// pm2 start ecosystem.config.js
// pm2 start app.js -i max
// pm2 list
// pm2 stop my-app
// pm2 restart my-app
// pm2 reload my-app
// pm2 delete my-app
// pm2 logs my-app
// pm2 monit
```

---

## 总结

本教程全面介绍了Node.js的核心基础知识，包括：

1. Node.js概述 - 了解Node.js的特性和全局对象
2. 模块系统 - 掌握CommonJS和ES模块的使用
3. 包管理 - 学会使用npm管理项目依赖
4. 异步编程 - 理解回调、Promise和async/await
5. 事件循环 - 深入理解Node.js的事件循环机制
6. 文件系统 - 掌握文件和目录的操作方法
7. Stream流 - 学会使用流处理大数据
8. Buffer缓冲区 - 理解二进制数据处理
9. 网络编程 - 掌握HTTP、TCP、UDP等网络协议
10. 进程与线程 - 学会使用子进程和Worker Threads

通过本教程的学习，你应该能够：
- 理解Node.js的核心概念和工作原理
- 熟练使用异步编程模式
- 掌握文件系统和网络编程
- 能够处理大数据和高并发场景
- 了解进程管理和性能优化

## 附录

### A. 常用核心模块

```javascript
/**
 * Node.js核心模块列表
 * @author erik.zhou
 */

// 文件系统
const fs = require('fs');
const path = require('path');

// 网络
const http = require('http');
const https = require('https');
const net = require('net');
const dgram = require('dgram');

// 进程
const process = require('process');
const child_process = require('child_process');
const cluster = require('cluster');
const worker_threads = require('worker_threads');

// 流
const stream = require('stream');
const readline = require('readline');

// 工具
const util = require('util');
const url = require('url');
const querystring = require('querystring');
const crypto = require('crypto');
const zlib = require('zlib');

// 事件
const events = require('events');

// 操作系统
const os = require('os');

// 定时器
const timers = require('timers');
```

### B. 最佳实践

```javascript
/**
 * Node.js最佳实践
 * @author erik.zhou
 */

// 1. 使用async/await处理异步
async function goodPractice() {
    try {
        const data = await readFile('data.txt');
        return data;
    } catch (error) {
        console.error('错误:', error);
        throw error;
    }
}

// 2. 使用Promise.all并发执行
async function concurrentExecution() {
    const [data1, data2, data3] = await Promise.all([
        readFile('file1.txt'),
        readFile('file2.txt'),
        readFile('file3.txt')
    ]);
}

// 3. 使用Stream处理大文件
function processLargeFile() {
    fs.createReadStream('large.txt')
        .pipe(transform)
        .pipe(fs.createWriteStream('output.txt'));
}

// 4. 正确处理错误
process.on('uncaughtException', (error) => {
    console.error('未捕获的异常:', error);
    process.exit(1);
});

process.on('unhandledRejection', (reason) => {
    console.error('未处理的Promise拒绝:', reason);
});

// 5. 使用环境变量配置
const config = {
    port: process.env.PORT || 3000,
    env: process.env.NODE_ENV || 'development'
};

// 6. 优雅退出
process.on('SIGTERM', async () => {
    await cleanup();
    process.exit(0);
});
```

### C. 常见问题

**Q1: 如何避免回调地狱？**
```javascript
// 使用async/await
async function solution() {
    const data1 = await readFile('file1.txt');
    const data2 = await readFile('file2.txt');
    const data3 = await readFile('file3.txt');
}
```

**Q2: 如何处理大文件？**
```javascript
// 使用Stream
fs.createReadStream('large.txt')
    .pipe(transform)
    .pipe(fs.createWriteStream('output.txt'));
```

**Q3: 如何提高性能？**
```javascript
// 使用Cluster
if (cluster.isMaster) {
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
} else {
    // 创建服务器
}
```

**Q4: 如何处理CPU密集型任务？**
```javascript
// 使用Worker Threads
const worker = new Worker('./worker.js', {
    workerData: data
});
```

## 参考资源

- [Node.js官方文档](https://nodejs.org/docs/)
- [Node.js最佳实践](https://github.com/goldbergyoni/nodebestpractices)
- [Node.js设计模式](https://www.nodejsdesignpatterns.com/)

---

**作者**: erik.zhou  
**最后更新**: 2024年

