# Koa - 完整教程

## 课程简介

Koa是由Express原班人马打造的新一代Web框架，采用async/await语法，提供了更优雅的异步流程控制。本教程将深入讲解Koa的核心概念、中间件机制、路由管理、错误处理等内容。

## 学习目标

- 理解Koa的设计理念和架构
- 掌握Koa中间件机制
- 熟练使用async/await处理异步
- 掌握路由和请求处理
- 学会错误处理和日志记录
- 理解上下文对象的使用
- 掌握常用中间件的使用
- 学会构建RESTful API

## 目录

1. [Koa基础](#第1章-koa基础)
2. [中间件机制](#第2章-中间件机制)
3. [上下文对象](#第3章-上下文对象)
4. [路由管理](#第4章-路由管理)
5. [请求处理](#第5章-请求处理)
6. [响应处理](#第6章-响应处理)
7. [错误处理](#第7章-错误处理)
8. [常用中间件](#第8章-常用中间件)
9. [数据库集成](#第9章-数据库集成)
10. [实战应用](#第10章-实战应用)

---

## 第1章 Koa基础

### 1.1 安装与初始化

```bash
# 创建项目
mkdir koa-app
cd koa-app
npm init -y

# 安装Koa
npm install koa

# 安装开发依赖
npm install --save-dev nodemon
```

```javascript
/**
 * 第一个Koa应用
 * @author erik.zhou
 */

const Koa = require('koa');

// 创建Koa应用
const app = new Koa();

// 中间件
app.use(async (ctx) => {
    ctx.body = 'Hello Koa!';
});

// 启动服务器
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`服务器运行在 http://localhost:${PORT}`);
});
```

### 1.2 Koa vs Express

```javascript
/**
 * Koa与Express对比
 * @author erik.zhou
 */

// Express风格
const express = require('express');
const expressApp = express();

expressApp.get('/', (req, res) => {
    res.send('Hello Express');
});

// Koa风格
const Koa = require('koa');
const koaApp = new Koa();

koaApp.use(async (ctx) => {
    ctx.body = 'Hello Koa';
});

// 主要区别：
// 1. Koa使用async/await，Express使用回调
// 2. Koa没有内置路由，需要使用koa-router
// 3. Koa的中间件是洋葱模型
// 4. Koa更轻量，核心功能更少
// 5. Koa的错误处理更优雅
```

### 1.3 应用配置

```javascript
/**
 * Koa应用配置
 * @author erik.zhou
 */

const Koa = require('koa');
const app = new Koa();

// 应用级配置
app.env = process.env.NODE_ENV || 'development';
app.proxy = true; // 信任代理头
app.subdomainOffset = 2; // 子域名偏移

// 自定义属性
app.context.db = {
    // 数据库连接
};

app.context.config = {
    apiPrefix: '/api',
    port: 3000
};

// 错误处理
app.on('error', (err, ctx) => {
    console.error('服务器错误:', err);
    console.error('请求URL:', ctx.url);
});

// 使用配置
app.use(async (ctx) => {
    ctx.body = {
        env: app.env,
        config: ctx.config
    };
});

app.listen(3000);
```

### 1.4 生命周期

```javascript
/**
 * Koa应用生命周期
 * @author erik.zhou
 */

const Koa = require('koa');
const app = new Koa();

// 应用启动前
console.log('1. 应用初始化');

// 注册中间件
app.use(async (ctx, next) => {
    console.log('2. 请求开始');
    const start = Date.now();
    
    await next();
    
    const duration = Date.now() - start;
    console.log('5. 请求结束');
    console.log(`耗时: ${duration}ms`);
});

app.use(async (ctx, next) => {
    console.log('3. 中间件1');
    await next();
    console.log('4. 中间件1结束');
});

app.use(async (ctx) => {
    ctx.body = 'Hello';
});

// 应用启动
app.listen(3000, () => {
    console.log('应用启动完成');
});
```

---

## 第2章 中间件机制

### 2.1 洋葱模型

```javascript
/**
 * Koa洋葱模型
 * @author erik.zhou
 */

const Koa = require('koa');
const app = new Koa();

// 中间件1
app.use(async (ctx, next) => {
    console.log('1. 中间件1开始');
    await next();
    console.log('6. 中间件1结束');
});

// 中间件2
app.use(async (ctx, next) => {
    console.log('2. 中间件2开始');
    await next();
    console.log('5. 中间件2结束');
});

// 中间件3
app.use(async (ctx, next) => {
    console.log('3. 中间件3开始');
    await next();
    console.log('4. 中间件3结束');
});

// 最终处理
app.use(async (ctx) => {
    ctx.body = 'Hello';
});

// 执行顺序：1 -> 2 -> 3 -> 4 -> 5 -> 6
app.listen(3000);
```

### 2.2 中间件组合

```javascript
/**
 * 中间件组合
 * @author erik.zhou
 */

const Koa = require('koa');
const compose = require('koa-compose');
const app = new Koa();

// 定义多个中间件
const middleware1 = async (ctx, next) => {
    console.log('中间件1');
    await next();
};

const middleware2 = async (ctx, next) => {
    console.log('中间件2');
    await next();
};

const middleware3 = async (ctx, next) => {
    console.log('中间件3');
    await next();
};

// 组合中间件
const combined = compose([
    middleware1,
    middleware2,
    middleware3
]);

app.use(combined);

app.use(async (ctx) => {
    ctx.body = 'Hello';
});

app.listen(3000);
```

### 2.3 异步中间件

```javascript
/**
 * 异步中间件
 * @author erik.zhou
 */

const Koa = require('koa');
const app = new Koa();

// 异步操作中间件
app.use(async (ctx, next) => {
    console.log('开始异步操作');
    
    // 模拟异步操作
    await new Promise(resolve => setTimeout(resolve, 1000));
    
    console.log('异步操作完成');
    await next();
});

// 数据库查询中间件
app.use(async (ctx, next) => {
    try {
        const data = await fetchDataFromDB();
        ctx.state.data = data;
        await next();
    } catch (error) {
        ctx.throw(500, '数据库查询失败');
    }
});

// 响应处理
app.use(async (ctx) => {
    ctx.body = {
        message: '成功',
        data: ctx.state.data
    };
});

async function fetchDataFromDB() {
    // 模拟数据库查询
    return new Promise(resolve => {
        setTimeout(() => {
            resolve({ id: 1, name: 'John' });
        }, 500);
    });
}

app.listen(3000);
```

### 2.4 条件中间件

```javascript
/**
 * 条件中间件
 * @author erik.zhou
 */

const Koa = require('koa');
const app = new Koa();

// 条件执行中间件
function conditionalMiddleware(condition, middleware) {
    return async (ctx, next) => {
        if (condition(ctx)) {
            await middleware(ctx, next);
        } else {
            await next();
        }
    };
}

// 仅对API路径执行
const apiOnly = conditionalMiddleware(
    (ctx) => ctx.path.startsWith('/api'),
    async (ctx, next) => {
        console.log('API请求');
        await next();
    }
);

app.use(apiOnly);

// 仅对POST请求执行
const postOnly = conditionalMiddleware(
    (ctx) => ctx.method === 'POST',
    async (ctx, next) => {
        console.log('POST请求');
        await next();
    }
);

app.use(postOnly);

// 认证中间件（仅对受保护路由）
const authMiddleware = conditionalMiddleware(
    (ctx) => ctx.path.startsWith('/protected'),
    async (ctx, next) => {
        if (!ctx.headers.authorization) {
            ctx.throw(401, '未授权');
        }
        await next();
    }
);

app.use(authMiddleware);

app.use(async (ctx) => {
    ctx.body = { path: ctx.path, method: ctx.method };
});

app.listen(3000);
```

### 2.5 中间件最佳实践

```javascript
/**
 * 中间件最佳实践
 * @author erik.zhou
 */

const Koa = require('koa');
const app = new Koa();

// 1. 日志中间件（最外层）
app.use(async (ctx, next) => {
    const start = Date.now();
    await next();
    const duration = Date.now() - start;
    console.log(`${ctx.method} ${ctx.url} - ${duration}ms`);
});

// 2. 错误处理中间件
app.use(async (ctx, next) => {
    try {
        await next();
    } catch (err) {
        ctx.status = err.status || 500;
        ctx.body = {
            error: err.message
        };
        ctx.app.emit('error', err, ctx);
    }
});

// 3. 响应时间中间件
app.use(async (ctx, next) => {
    const start = Date.now();
    await next();
    const duration = Date.now() - start;
    ctx.set('X-Response-Time', `${duration}ms`);
});

// 4. 请求ID中间件
app.use(async (ctx, next) => {
    ctx.state.requestId = generateRequestId();
    ctx.set('X-Request-ID', ctx.state.requestId);
    await next();
});

// 5. 业务逻辑
app.use(async (ctx) => {
    ctx.body = {
        message: 'Hello',
        requestId: ctx.state.requestId
    };
});

function generateRequestId() {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
}

app.listen(3000);
```

---

## 第3章 上下文对象

### 3.1 Context对象

```javascript
/**
 * Context上下文对象
 * @author erik.zhou
 */

const Koa = require('koa');
const app = new Koa();

app.use(async (ctx) => {
    // ctx包含request和response
    const contextInfo = {
        // 请求信息
        method: ctx.method,
        url: ctx.url,
        path: ctx.path,
        query: ctx.query,
        headers: ctx.headers,
        
        // 响应信息
        status: ctx.status,
        message: ctx.message,
        
        // 应用信息
        app: ctx.app.constructor.name,
        
        // 请求对象
        request: {
            method: ctx.request.method,
            url: ctx.request.url,
            header: ctx.request.header
        },
        
        // 响应对象
        response: {
            status: ctx.response.status,
            message: ctx.response.message,
            header: ctx.response.header
        },
        
        // 其他
        ip: ctx.ip,
        ips: ctx.ips,
        protocol: ctx.protocol,
        secure: ctx.secure,
        hostname: ctx.hostname
    };
    
    ctx.body = contextInfo;
});

app.listen(3000);
```

### 3.2 Request对象

```javascript
/**
 * Request请求对象
 * @author erik.zhou
 */

const Koa = require('koa');
const app = new Koa();

app.use(async (ctx) => {
    const requestInfo = {
        // URL信息
        url: ctx.request.url,
        origin: ctx.request.origin,
        href: ctx.request.href,
        path: ctx.request.path,
        querystring: ctx.request.querystring,
        search: ctx.request.search,
        
        // 请求头
        headers: ctx.request.headers,
        header: ctx.request.header,
        
        // 请求方法
        method: ctx.request.method,
        
        // 主机信息
        host: ctx.request.host,
        hostname: ctx.request.hostname,
        
        // 协议
        protocol: ctx.request.protocol,
        secure: ctx.request.secure,
        
        // IP
        ip: ctx.request.ip,
        ips: ctx.request.ips,
        
        // 类型
        type: ctx.request.type,
        charset: ctx.request.charset,
        
        // 长度
        length: ctx.request.length,
        
        // 其他
        fresh: ctx.request.fresh,
        stale: ctx.request.stale,
        idempotent: ctx.request.idempotent
    };
    
    ctx.body = requestInfo;
});

app.listen(3000);
```

### 3.3 Response对象

```javascript
/**
 * Response响应对象
 * @author erik.zhou
 */

const Koa = require('koa');
const app = new Koa();

app.use(async (ctx) => {
    // 设置状态码
    ctx.response.status = 200;
    
    // 设置消息
    ctx.response.message = 'OK';
    
    // 设置响应头
    ctx.response.set('X-Custom-Header', 'value');
    ctx.response.set({
        'Content-Type': 'application/json',
        'X-Powered-By': 'Koa'
    });
    
    // 设置响应体
    ctx.response.body = {
        message: 'Success',
        data: { id: 1, name: 'John' }
    };
    
    // 设置类型
    ctx.response.type = 'json';
    
    // 设置长度
    ctx.response.length = 100;
    
    // 设置Last-Modified
    ctx.response.lastModified = new Date();
    
    // 设置ETag
    ctx.response.etag = 'abc123';
});

app.listen(3000);
```

### 3.4 State状态对象

```javascript
/**
 * State状态对象
 * @author erik.zhou
 */

const Koa = require('koa');
const app = new Koa();

// 中间件1：设置用户信息
app.use(async (ctx, next) => {
    ctx.state.user = {
        id: 1,
        name: 'John',
        role: 'admin'
    };
    await next();
});

// 中间件2：设置请求信息
app.use(async (ctx, next) => {
    ctx.state.requestInfo = {
        timestamp: Date.now(),
        ip: ctx.ip,
        userAgent: ctx.get('User-Agent')
    };
    await next();
});

// 中间件3：使用状态
app.use(async (ctx) => {
    ctx.body = {
        user: ctx.state.user,
        requestInfo: ctx.state.requestInfo
    };
});

app.listen(3000);
```

### 3.5 自定义Context

```javascript
/**
 * 自定义Context方法
 * @author erik.zhou
 */

const Koa = require('koa');
const app = new Koa();

// 扩展Context
app.context.success = function(data, message = '成功') {
    this.body = {
        code: 0,
        message: message,
        data: data
    };
};

app.context.error = function(message = '失败', code = 1) {
    this.body = {
        code: code,
        message: message,
        data: null
    };
};

app.context.paginate = function(data, page, limit, total) {
    this.body = {
        code: 0,
        message: '成功',
        data: data,
        pagination: {
            page: page,
            limit: limit,
            total: total,
            pages: Math.ceil(total / limit)
        }
    };
};

// 使用自定义方法
app.use(async (ctx) => {
    const { type } = ctx.query;
    
    if (type === 'success') {
        ctx.success({ id: 1, name: 'John' });
    } else if (type === 'error') {
        ctx.error('操作失败');
    } else if (type === 'paginate') {
        const data = [1, 2, 3, 4, 5];
        ctx.paginate(data, 1, 5, 100);
    } else {
        ctx.body = 'Hello';
    }
});

app.listen(3000);
```



---

## 第4章 路由管理

### 4.1 koa-router基础

```javascript
/**
 * koa-router基础使用
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

// 基本路由
router.get('/', async (ctx) => {
    ctx.body = '首页';
});

router.get('/about', async (ctx) => {
    ctx.body = '关于';
});

// HTTP方法
router.post('/users', async (ctx) => {
    ctx.body = '创建用户';
});

router.put('/users/:id', async (ctx) => {
    ctx.body = `更新用户 ${ctx.params.id}`;
});

router.delete('/users/:id', async (ctx) => {
    ctx.body = `删除用户 ${ctx.params.id}`;
});

// 注册路由
app.use(router.routes());
app.use(router.allowedMethods());

app.listen(3000);
```

### 4.2 路由参数

```javascript
/**
 * 路由参数处理
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

// 路径参数
router.get('/users/:id', async (ctx) => {
    const userId = ctx.params.id;
    ctx.body = { userId };
});

// 多个参数
router.get('/users/:userId/posts/:postId', async (ctx) => {
    const { userId, postId } = ctx.params;
    ctx.body = { userId, postId };
});

// 正则表达式参数
router.get('/users/:id(\\d+)', async (ctx) => {
    ctx.body = { userId: ctx.params.id };
});

// 查询参数
router.get('/search', async (ctx) => {
    const { keyword, page = 1, limit = 10 } = ctx.query;
    ctx.body = {
        keyword,
        page: parseInt(page),
        limit: parseInt(limit)
    };
});

// 参数验证
router.param('id', async (id, ctx, next) => {
    if (!/^\d+$/.test(id)) {
        ctx.throw(400, '无效的ID');
    }
    ctx.state.userId = parseInt(id);
    await next();
});

router.get('/api/users/:id', async (ctx) => {
    ctx.body = { userId: ctx.state.userId };
});

app.use(router.routes());
app.use(router.allowedMethods());

app.listen(3000);
```

### 4.3 路由前缀

```javascript
/**
 * 路由前缀
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');

const app = new Koa();

// API路由
const apiRouter = new Router({
    prefix: '/api'
});

apiRouter.get('/users', async (ctx) => {
    ctx.body = { users: [] };
});

apiRouter.get('/posts', async (ctx) => {
    ctx.body = { posts: [] };
});

// 管理路由
const adminRouter = new Router({
    prefix: '/admin'
});

adminRouter.get('/dashboard', async (ctx) => {
    ctx.body = { dashboard: 'data' };
});

adminRouter.get('/users', async (ctx) => {
    ctx.body = { adminUsers: [] };
});

// 注册路由
app.use(apiRouter.routes());
app.use(adminRouter.routes());

app.listen(3000);
```

### 4.4 嵌套路由

```javascript
/**
 * 嵌套路由
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');

const app = new Koa();

// 用户路由
const usersRouter = new Router({
    prefix: '/users'
});

usersRouter.get('/', async (ctx) => {
    ctx.body = { users: [] };
});

usersRouter.get('/:id', async (ctx) => {
    ctx.body = { user: { id: ctx.params.id } };
});

// 用户的文章路由（嵌套）
const userPostsRouter = new Router({
    prefix: '/:userId/posts'
});

userPostsRouter.get('/', async (ctx) => {
    ctx.body = {
        userId: ctx.params.userId,
        posts: []
    };
});

userPostsRouter.get('/:postId', async (ctx) => {
    ctx.body = {
        userId: ctx.params.userId,
        post: { id: ctx.params.postId }
    };
});

// 嵌套路由
usersRouter.use(userPostsRouter.routes());

// 主路由
const router = new Router({
    prefix: '/api'
});

router.use(usersRouter.routes());

app.use(router.routes());
app.use(router.allowedMethods());

app.listen(3000);
```

### 4.5 路由模块化

```javascript
/**
 * 路由模块化 - routes/users.js
 * @author erik.zhou
 */

const Router = require('@koa/router');
const router = new Router({
    prefix: '/users'
});

// 获取用户列表
router.get('/', async (ctx) => {
    ctx.body = {
        users: [
            { id: 1, name: 'John' },
            { id: 2, name: 'Jane' }
        ]
    };
});

// 获取单个用户
router.get('/:id', async (ctx) => {
    const userId = ctx.params.id;
    ctx.body = {
        user: { id: userId, name: 'John' }
    };
});

// 创建用户
router.post('/', async (ctx) => {
    const user = ctx.request.body;
    ctx.status = 201;
    ctx.body = {
        message: '用户创建成功',
        user: user
    };
});

// 更新用户
router.put('/:id', async (ctx) => {
    const userId = ctx.params.id;
    const user = ctx.request.body;
    ctx.body = {
        message: '用户更新成功',
        user: { id: userId, ...user }
    };
});

// 删除用户
router.delete('/:id', async (ctx) => {
    const userId = ctx.params.id;
    ctx.body = {
        message: '用户删除成功',
        userId: userId
    };
});

module.exports = router;
```

```javascript
/**
 * 主应用 - app.js
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');
const bodyParser = require('koa-bodyparser');

const app = new Koa();
const router = new Router({
    prefix: '/api'
});

// 导入路由模块
const usersRouter = require('./routes/users');
const postsRouter = require('./routes/posts');

// 中间件
app.use(bodyParser());

// 注册路由
router.use(usersRouter.routes());
router.use(postsRouter.routes());

app.use(router.routes());
app.use(router.allowedMethods());

// 根路由
app.use(async (ctx) => {
    ctx.body = {
        message: 'API服务器',
        endpoints: [
            '/api/users',
            '/api/posts'
        ]
    };
});

app.listen(3000, () => {
    console.log('服务器运行在 http://localhost:3000');
});
```

---

## 第5章 请求处理

### 5.1 请求体解析

```javascript
/**
 * 请求体解析
 * @author erik.zhou
 */

const Koa = require('koa');
const bodyParser = require('koa-bodyparser');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

// 使用bodyParser中间件
app.use(bodyParser({
    enableTypes: ['json', 'form', 'text'],
    jsonLimit: '10mb',
    formLimit: '10mb',
    textLimit: '10mb'
}));

// JSON请求
router.post('/api/json', async (ctx) => {
    const data = ctx.request.body;
    ctx.body = {
        message: '接收JSON数据',
        data: data
    };
});

// 表单请求
router.post('/api/form', async (ctx) => {
    const data = ctx.request.body;
    ctx.body = {
        message: '接收表单数据',
        data: data
    };
});

// 文本请求
router.post('/api/text', async (ctx) => {
    const text = ctx.request.body;
    ctx.body = {
        message: '接收文本数据',
        text: text
    };
});

app.use(router.routes());

app.listen(3000);
```

### 5.2 文件上传

```javascript
/**
 * 文件上传处理
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');
const multer = require('@koa/multer');
const path = require('path');

const app = new Koa();
const router = new Router();

// 配置multer
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads/');
    },
    filename: (req, file, cb) => {
        const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
        cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
    }
});

const upload = multer({
    storage: storage,
    limits: {
        fileSize: 5 * 1024 * 1024 // 5MB
    },
    fileFilter: (req, file, cb) => {
        const allowedTypes = /jpeg|jpg|png|gif/;
        const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
        const mimetype = allowedTypes.test(file.mimetype);
        
        if (extname && mimetype) {
            cb(null, true);
        } else {
            cb(new Error('只允许上传图片文件'));
        }
    }
});

// 单文件上传
router.post('/upload/single', upload.single('avatar'), async (ctx) => {
    ctx.body = {
        message: '文件上传成功',
        file: {
            filename: ctx.file.filename,
            originalname: ctx.file.originalname,
            size: ctx.file.size,
            path: ctx.file.path
        }
    };
});

// 多文件上传
router.post('/upload/multiple', upload.array('photos', 10), async (ctx) => {
    const files = ctx.files.map(file => ({
        filename: file.filename,
        originalname: file.originalname,
        size: file.size
    }));
    
    ctx.body = {
        message: '文件上传成功',
        count: files.length,
        files: files
    };
});

app.use(router.routes());

app.listen(3000);
```

### 5.3 Cookie处理

```javascript
/**
 * Cookie处理
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

// 设置Cookie密钥
app.keys = ['secret-key-1', 'secret-key-2'];

// 设置Cookie
router.get('/set-cookie', async (ctx) => {
    // 普通Cookie
    ctx.cookies.set('username', 'john', {
        maxAge: 900000,
        httpOnly: true,
        overwrite: true
    });
    
    // 签名Cookie
    ctx.cookies.set('userId', '123', {
        signed: true,
        maxAge: 900000,
        httpOnly: true,
        secure: false
    });
    
    ctx.body = { message: 'Cookie已设置' };
});

// 读取Cookie
router.get('/get-cookie', async (ctx) => {
    const username = ctx.cookies.get('username');
    const userId = ctx.cookies.get('userId', { signed: true });
    
    ctx.body = {
        username: username,
        userId: userId
    };
});

// 删除Cookie
router.get('/delete-cookie', async (ctx) => {
    ctx.cookies.set('username', null);
    ctx.cookies.set('userId', null);
    
    ctx.body = { message: 'Cookie已删除' };
});

app.use(router.routes());

app.listen(3000);
```

### 5.4 Session处理

```javascript
/**
 * Session处理
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');
const session = require('koa-session');

const app = new Koa();
const router = new Router();

// Session配置
app.keys = ['secret-key'];

const CONFIG = {
    key: 'koa.sess',
    maxAge: 86400000, // 24小时
    autoCommit: true,
    overwrite: true,
    httpOnly: true,
    signed: true,
    rolling: false,
    renew: false
};

app.use(session(CONFIG, app));

// 登录
router.post('/login', async (ctx) => {
    const { username, password } = ctx.request.body;
    
    if (username === 'admin' && password === 'password') {
        ctx.session.user = {
            id: 1,
            username: username,
            role: 'admin'
        };
        
        ctx.body = {
            message: '登录成功',
            user: ctx.session.user
        };
    } else {
        ctx.status = 401;
        ctx.body = {
            error: '用户名或密码错误'
        };
    }
});

// 获取当前用户
router.get('/me', async (ctx) => {
    if (!ctx.session.user) {
        ctx.status = 401;
        ctx.body = {
            error: '未登录'
        };
        return;
    }
    
    ctx.body = {
        user: ctx.session.user
    };
});

// 退出登录
router.post('/logout', async (ctx) => {
    ctx.session = null;
    ctx.body = { message: '退出成功' };
});

app.use(router.routes());

app.listen(3000);
```

### 5.5 请求验证

```javascript
/**
 * 请求验证
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');
const bodyParser = require('koa-bodyparser');
const Joi = require('joi');

const app = new Koa();
const router = new Router();

app.use(bodyParser());

// 验证中间件
function validate(schema) {
    return async (ctx, next) => {
        const { error, value } = schema.validate(ctx.request.body, {
            abortEarly: false,
            stripUnknown: true
        });
        
        if (error) {
            ctx.status = 400;
            ctx.body = {
                error: '验证失败',
                details: error.details.map(detail => ({
                    field: detail.path.join('.'),
                    message: detail.message
                }))
            };
            return;
        }
        
        ctx.state.validatedData = value;
        await next();
    };
}

// 用户验证模式
const userSchema = Joi.object({
    name: Joi.string().min(3).max(30).required(),
    email: Joi.string().email().required(),
    age: Joi.number().integer().min(18).max(100),
    password: Joi.string().min(8).pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
});

// 使用验证
router.post('/api/users', validate(userSchema), async (ctx) => {
    const user = ctx.state.validatedData;
    ctx.status = 201;
    ctx.body = {
        message: '用户创建成功',
        user: user
    };
});

app.use(router.routes());

app.listen(3000);
```

---

## 第6章 响应处理

### 6.1 响应类型

```javascript
/**
 * 不同响应类型
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');
const fs = require('fs');

const app = new Koa();
const router = new Router();

// JSON响应
router.get('/json', async (ctx) => {
    ctx.body = {
        message: 'Success',
        data: { id: 1, name: 'John' }
    };
});

// 文本响应
router.get('/text', async (ctx) => {
    ctx.type = 'text/plain';
    ctx.body = 'Hello World';
});

// HTML响应
router.get('/html', async (ctx) => {
    ctx.type = 'text/html';
    ctx.body = '<h1>Hello World</h1>';
});

// Buffer响应
router.get('/buffer', async (ctx) => {
    ctx.body = Buffer.from('Hello World');
});

// Stream响应
router.get('/stream', async (ctx) => {
    ctx.type = 'text/plain';
    ctx.body = fs.createReadStream('./file.txt');
});

// 空响应
router.get('/empty', async (ctx) => {
    ctx.status = 204;
});

app.use(router.routes());

app.listen(3000);
```

### 6.2 状态码

```javascript
/**
 * HTTP状态码
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

// 成功响应
router.get('/success', async (ctx) => {
    ctx.status = 200;
    ctx.body = { message: '成功' };
});

// 创建成功
router.post('/created', async (ctx) => {
    ctx.status = 201;
    ctx.body = { message: '创建成功' };
});

// 无内容
router.delete('/no-content', async (ctx) => {
    ctx.status = 204;
});

// 重定向
router.get('/redirect', async (ctx) => {
    ctx.redirect('/new-url');
    ctx.status = 302;
});

// 客户端错误
router.get('/bad-request', async (ctx) => {
    ctx.status = 400;
    ctx.body = { error: '请求参数错误' };
});

router.get('/unauthorized', async (ctx) => {
    ctx.status = 401;
    ctx.body = { error: '未授权' };
});

router.get('/forbidden', async (ctx) => {
    ctx.status = 403;
    ctx.body = { error: '禁止访问' };
});

router.get('/not-found', async (ctx) => {
    ctx.status = 404;
    ctx.body = { error: '资源未找到' };
});

// 服务器错误
router.get('/server-error', async (ctx) => {
    ctx.status = 500;
    ctx.body = { error: '服务器内部错误' };
});

app.use(router.routes());

app.listen(3000);
```

### 6.3 响应头

```javascript
/**
 * 响应头设置
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

router.get('/headers', async (ctx) => {
    // 设置单个响应头
    ctx.set('X-Custom-Header', 'value');
    
    // 设置多个响应头
    ctx.set({
        'Content-Type': 'application/json',
        'X-Powered-By': 'Koa',
        'X-Request-ID': generateRequestId()
    });
    
    // 设置缓存
    ctx.set('Cache-Control', 'public, max-age=3600');
    
    // 设置CORS
    ctx.set('Access-Control-Allow-Origin', '*');
    ctx.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    
    ctx.body = { message: '响应头已设置' };
});

// 条件响应
router.get('/conditional', async (ctx) => {
    const etag = 'abc123';
    const lastModified = new Date('2024-01-01');
    
    ctx.set('ETag', etag);
    ctx.set('Last-Modified', lastModified.toUTCString());
    
    // 检查If-None-Match
    if (ctx.get('If-None-Match') === etag) {
        ctx.status = 304;
        return;
    }
    
    // 检查If-Modified-Since
    const ifModifiedSince = ctx.get('If-Modified-Since');
    if (ifModifiedSince && new Date(ifModifiedSince) >= lastModified) {
        ctx.status = 304;
        return;
    }
    
    ctx.body = { data: 'content' };
});

function generateRequestId() {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
}

app.use(router.routes());

app.listen(3000);
```

### 6.4 内容协商

```javascript
/**
 * 内容协商
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

router.get('/data', async (ctx) => {
    const data = {
        id: 1,
        name: 'John',
        email: 'john@example.com'
    };
    
    // 根据Accept头返回不同格式
    const accepts = ctx.accepts('json', 'xml', 'html', 'text');
    
    switch (accepts) {
        case 'json':
            ctx.type = 'application/json';
            ctx.body = data;
            break;
            
        case 'xml':
            ctx.type = 'application/xml';
            ctx.body = `
                <user>
                    <id>${data.id}</id>
                    <name>${data.name}</name>
                    <email>${data.email}</email>
                </user>
            `;
            break;
            
        case 'html':
            ctx.type = 'text/html';
            ctx.body = `
                <h1>${data.name}</h1>
                <p>ID: ${data.id}</p>
                <p>Email: ${data.email}</p>
            `;
            break;
            
        case 'text':
            ctx.type = 'text/plain';
            ctx.body = `ID: ${data.id}\nName: ${data.name}\nEmail: ${data.email}`;
            break;
            
        default:
            ctx.status = 406;
            ctx.body = { error: 'Not Acceptable' };
    }
});

app.use(router.routes());

app.listen(3000);
```

### 6.5 流式响应

```javascript
/**
 * 流式响应
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');
const fs = require('fs');
const { PassThrough } = require('stream');

const app = new Koa();
const router = new Router();

// 文件流
router.get('/stream/file', async (ctx) => {
    ctx.type = 'text/plain';
    ctx.body = fs.createReadStream('./large-file.txt');
});

// Server-Sent Events
router.get('/stream/events', async (ctx) => {
    ctx.set({
        'Content-Type': 'text/event-stream',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive'
    });
    
    const stream = new PassThrough();
    ctx.body = stream;
    
    let count = 0;
    const interval = setInterval(() => {
        count++;
        stream.write(`data: ${JSON.stringify({ count, time: new Date() })}\n\n`);
        
        if (count >= 10) {
            clearInterval(interval);
            stream.end();
        }
    }, 1000);
    
    ctx.req.on('close', () => {
        clearInterval(interval);
        stream.end();
    });
});

// 分块传输
router.get('/stream/chunks', async (ctx) => {
    ctx.set('Transfer-Encoding', 'chunked');
    
    const stream = new PassThrough();
    ctx.body = stream;
    
    let i = 0;
    const interval = setInterval(() => {
        stream.write(`Chunk ${i}\n`);
        i++;
        
        if (i >= 5) {
            clearInterval(interval);
            stream.end();
        }
    }, 500);
});

app.use(router.routes());

app.listen(3000);
```



---

## 第7章 错误处理

### 7.1 错误捕获

```javascript
/**
 * 错误捕获
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

// 全局错误处理
app.use(async (ctx, next) => {
    try {
        await next();
    } catch (err) {
        ctx.status = err.status || 500;
        ctx.body = {
            error: err.message,
            status: ctx.status
        };
        
        // 触发error事件
        ctx.app.emit('error', err, ctx);
    }
});

// 错误事件监听
app.on('error', (err, ctx) => {
    console.error('服务器错误:', err);
    console.error('请求URL:', ctx.url);
    console.error('请求方法:', ctx.method);
});

// 同步错误
router.get('/sync-error', async (ctx) => {
    throw new Error('同步错误');
});

// 异步错误
router.get('/async-error', async (ctx) => {
    await Promise.reject(new Error('异步错误'));
});

// 手动抛出错误
router.get('/manual-error', async (ctx) => {
    ctx.throw(400, '手动抛出的错误');
});

app.use(router.routes());

app.listen(3000);
```

### 7.2 自定义错误类

```javascript
/**
 * 自定义错误类
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

// 自定义错误类
class AppError extends Error {
    constructor(message, status = 500, code = 'INTERNAL_ERROR') {
        super(message);
        this.status = status;
        this.code = code;
        this.isOperational = true;
    }
}

class ValidationError extends AppError {
    constructor(message, details = []) {
        super(message, 400, 'VALIDATION_ERROR');
        this.details = details;
    }
}

class NotFoundError extends AppError {
    constructor(resource = '资源') {
        super(`${resource}未找到`, 404, 'NOT_FOUND');
    }
}

class UnauthorizedError extends AppError {
    constructor(message = '未授权') {
        super(message, 401, 'UNAUTHORIZED');
    }
}

class ForbiddenError extends AppError {
    constructor(message = '禁止访问') {
        super(message, 403, 'FORBIDDEN');
    }
}

// 错误处理中间件
app.use(async (ctx, next) => {
    try {
        await next();
    } catch (err) {
        if (err.isOperational) {
            ctx.status = err.status;
            ctx.body = {
                error: {
                    code: err.code,
                    message: err.message,
                    details: err.details || undefined
                }
            };
        } else {
            // 程序错误
            console.error('程序错误:', err);
            ctx.status = 500;
            ctx.body = {
                error: {
                    code: 'INTERNAL_ERROR',
                    message: '服务器内部错误'
                }
            };
        }
        
        ctx.app.emit('error', err, ctx);
    }
});

// 使用自定义错误
router.get('/users/:id', async (ctx) => {
    const userId = ctx.params.id;
    
    if (!userId) {
        throw new ValidationError('用户ID不能为空');
    }
    
    const user = await findUser(userId);
    
    if (!user) {
        throw new NotFoundError('用户');
    }
    
    ctx.body = { user };
});

router.get('/protected', async (ctx) => {
    if (!ctx.headers.authorization) {
        throw new UnauthorizedError();
    }
    
    ctx.body = { message: '受保护的资源' };
});

async function findUser(id) {
    return null; // 模拟未找到
}

app.use(router.routes());

app.listen(3000);
```

### 7.3 错误日志

```javascript
/**
 * 错误日志记录
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');
const winston = require('winston');

const app = new Koa();
const router = new Router();

// 配置Winston日志
const logger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json()
    ),
    transports: [
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
        new winston.transports.File({ filename: 'combined.log' })
    ]
});

if (process.env.NODE_ENV !== 'production') {
    logger.add(new winston.transports.Console({
        format: winston.format.simple()
    }));
}

// 请求日志中间件
app.use(async (ctx, next) => {
    const start = Date.now();
    
    try {
        await next();
        
        const duration = Date.now() - start;
        logger.info({
            method: ctx.method,
            url: ctx.url,
            status: ctx.status,
            duration: `${duration}ms`,
            ip: ctx.ip
        });
    } catch (err) {
        const duration = Date.now() - start;
        logger.error({
            message: err.message,
            stack: err.stack,
            method: ctx.method,
            url: ctx.url,
            status: err.status || 500,
            duration: `${duration}ms`,
            ip: ctx.ip,
            userAgent: ctx.get('User-Agent')
        });
        
        throw err;
    }
});

// 错误处理
app.use(async (ctx, next) => {
    try {
        await next();
    } catch (err) {
        ctx.status = err.status || 500;
        ctx.body = {
            error: err.message
        };
    }
});

router.get('/error', async (ctx) => {
    throw new Error('测试错误');
});

app.use(router.routes());

app.listen(3000);
```

---

## 第8章 常用中间件

### 8.1 静态文件服务

```javascript
/**
 * 静态文件服务
 * @author erik.zhou
 */

const Koa = require('koa');
const serve = require('koa-static');
const mount = require('koa-mount');

const app = new Koa();

// 提供静态文件服务
app.use(serve('public'));

// 挂载到特定路径
app.use(mount('/static', serve('public')));

// 多个静态目录
app.use(serve('public'));
app.use(serve('assets'));

// 配置选项
app.use(serve('public', {
    maxage: 1000 * 60 * 60 * 24, // 1天
    hidden: false,
    index: 'index.html',
    defer: false,
    gzip: true
}));

app.listen(3000);
```

### 8.2 CORS跨域

```javascript
/**
 * CORS跨域处理
 * @author erik.zhou
 */

const Koa = require('koa');
const cors = require('@koa/cors');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

// 简单CORS配置
app.use(cors());

// 详细CORS配置
app.use(cors({
    origin: (ctx) => {
        const allowedOrigins = [
            'http://localhost:3001',
            'https://example.com'
        ];
        
        const origin = ctx.get('Origin');
        if (allowedOrigins.includes(origin)) {
            return origin;
        }
        return allowedOrigins[0];
    },
    credentials: true,
    allowMethods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
    allowHeaders: ['Content-Type', 'Authorization', 'Accept'],
    exposeHeaders: ['X-Request-ID'],
    maxAge: 86400
}));

router.get('/api/data', async (ctx) => {
    ctx.body = { data: 'test' };
});

app.use(router.routes());

app.listen(3000);
```

### 8.3 压缩

```javascript
/**
 * 响应压缩
 * @author erik.zhou
 */

const Koa = require('koa');
const compress = require('koa-compress');
const Router = require('@koa/router');

const app = new Koa();
const router = new Router();

// 压缩中间件
app.use(compress({
    filter: (contentType) => {
        return /text|json|javascript|css/.test(contentType);
    },
    threshold: 2048, // 2KB以上才压缩
    gzip: {
        flush: require('zlib').constants.Z_SYNC_FLUSH
    },
    deflate: {
        flush: require('zlib').constants.Z_SYNC_FLUSH
    },
    br: false // 禁用Brotli
}));

router.get('/large-data', async (ctx) => {
    const largeData = {
        items: Array(1000).fill(null).map((_, i) => ({
            id: i,
            name: `Item ${i}`,
            description: 'A'.repeat(100)
        }))
    };
    
    ctx.body = largeData;
});

app.use(router.routes());

app.listen(3000);
```

### 8.4 限流

```javascript
/**
 * 请求限流
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');
const ratelimit = require('koa-ratelimit');
const Redis = require('ioredis');

const app = new Koa();
const router = new Router();

// Redis客户端
const redis = new Redis();

// 全局限流
app.use(ratelimit({
    driver: 'redis',
    db: redis,
    duration: 60000, // 1分钟
    errorMessage: '请求过于频繁，请稍后再试',
    id: (ctx) => ctx.ip,
    headers: {
        remaining: 'Rate-Limit-Remaining',
        reset: 'Rate-Limit-Reset',
        total: 'Rate-Limit-Total'
    },
    max: 100,
    disableHeader: false
}));

// API限流
const apiLimiter = ratelimit({
    driver: 'redis',
    db: redis,
    duration: 60000,
    max: 30,
    id: (ctx) => ctx.ip
});

router.get('/api/data', apiLimiter, async (ctx) => {
    ctx.body = { data: 'test' };
});

// 登录限流
const loginLimiter = ratelimit({
    driver: 'redis',
    db: redis,
    duration: 900000, // 15分钟
    max: 5,
    id: (ctx) => ctx.ip
});

router.post('/login', loginLimiter, async (ctx) => {
    ctx.body = { message: '登录成功' };
});

app.use(router.routes());

app.listen(3000);
```

### 8.5 JWT认证

```javascript
/**
 * JWT认证
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');
const bodyParser = require('koa-bodyparser');
const jwt = require('koa-jwt');
const jsonwebtoken = require('jsonwebtoken');

const app = new Koa();
const router = new Router();

const SECRET_KEY = 'your-secret-key';

app.use(bodyParser());

// 登录生成Token
router.post('/login', async (ctx) => {
    const { username, password } = ctx.request.body;
    
    if (username === 'admin' && password === 'password') {
        const token = jsonwebtoken.sign(
            {
                id: 1,
                username: username,
                role: 'admin'
            },
            SECRET_KEY,
            { expiresIn: '24h' }
        );
        
        ctx.body = {
            message: '登录成功',
            token: token
        };
    } else {
        ctx.status = 401;
        ctx.body = {
            error: '用户名或密码错误'
        };
    }
});

// JWT验证中间件
app.use(jwt({ secret: SECRET_KEY }).unless({
    path: [/^\/login/, /^\/public/]
}));

// 受保护的路由
router.get('/protected', async (ctx) => {
    ctx.body = {
        message: '受保护的资源',
        user: ctx.state.user
    };
});

// 错误处理
app.use(async (ctx, next) => {
    try {
        await next();
    } catch (err) {
        if (err.status === 401) {
            ctx.status = 401;
            ctx.body = {
                error: '无效的令牌'
            };
        } else {
            throw err;
        }
    }
});

app.use(router.routes());

app.listen(3000);
```

---

## 第9章 数据库集成

### 9.1 MongoDB集成

```javascript
/**
 * MongoDB集成
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');
const bodyParser = require('koa-bodyparser');
const mongoose = require('mongoose');

const app = new Koa();
const router = new Router();

app.use(bodyParser());

// 连接MongoDB
mongoose.connect('mongodb://localhost/mydb', {
    useNewUrlParser: true,
    useUnifiedTopology: true
});

// 定义模型
const UserSchema = new mongoose.Schema({
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    age: Number,
    createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', UserSchema);

// 获取用户列表
router.get('/api/users', async (ctx) => {
    try {
        const users = await User.find();
        ctx.body = { users };
    } catch (error) {
        ctx.throw(500, '查询失败');
    }
});

// 获取单个用户
router.get('/api/users/:id', async (ctx) => {
    try {
        const user = await User.findById(ctx.params.id);
        
        if (!user) {
            ctx.throw(404, '用户不存在');
        }
        
        ctx.body = { user };
    } catch (error) {
        ctx.throw(500, '查询失败');
    }
});

// 创建用户
router.post('/api/users', async (ctx) => {
    try {
        const user = new User(ctx.request.body);
        await user.save();
        
        ctx.status = 201;
        ctx.body = {
            message: '用户创建成功',
            user: user
        };
    } catch (error) {
        ctx.throw(400, '创建失败');
    }
});

// 更新用户
router.put('/api/users/:id', async (ctx) => {
    try {
        const user = await User.findByIdAndUpdate(
            ctx.params.id,
            ctx.request.body,
            { new: true, runValidators: true }
        );
        
        if (!user) {
            ctx.throw(404, '用户不存在');
        }
        
        ctx.body = {
            message: '用户更新成功',
            user: user
        };
    } catch (error) {
        ctx.throw(400, '更新失败');
    }
});

// 删除用户
router.delete('/api/users/:id', async (ctx) => {
    try {
        const user = await User.findByIdAndDelete(ctx.params.id);
        
        if (!user) {
            ctx.throw(404, '用户不存在');
        }
        
        ctx.body = {
            message: '用户删除成功'
        };
    } catch (error) {
        ctx.throw(500, '删除失败');
    }
});

app.use(router.routes());

app.listen(3000);
```

### 9.2 MySQL集成

```javascript
/**
 * MySQL集成
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');
const bodyParser = require('koa-bodyparser');
const mysql = require('mysql2/promise');

const app = new Koa();
const router = new Router();

app.use(bodyParser());

// 创建连接池
const pool = mysql.createPool({
    host: 'localhost',
    user: 'root',
    password: 'password',
    database: 'mydb',
    waitForConnections: true,
    connectionLimit: 10,
    queueLimit: 0
});

// 获取用户列表
router.get('/api/users', async (ctx) => {
    try {
        const [rows] = await pool.query('SELECT * FROM users');
        ctx.body = { users: rows };
    } catch (error) {
        ctx.throw(500, '查询失败');
    }
});

// 获取单个用户
router.get('/api/users/:id', async (ctx) => {
    try {
        const [rows] = await pool.query(
            'SELECT * FROM users WHERE id = ?',
            [ctx.params.id]
        );
        
        if (rows.length === 0) {
            ctx.throw(404, '用户不存在');
        }
        
        ctx.body = { user: rows[0] };
    } catch (error) {
        ctx.throw(500, '查询失败');
    }
});

// 创建用户
router.post('/api/users', async (ctx) => {
    try {
        const { name, email, age } = ctx.request.body;
        
        const [result] = await pool.query(
            'INSERT INTO users (name, email, age) VALUES (?, ?, ?)',
            [name, email, age]
        );
        
        ctx.status = 201;
        ctx.body = {
            message: '用户创建成功',
            userId: result.insertId
        };
    } catch (error) {
        ctx.throw(400, '创建失败');
    }
});

// 更新用户
router.put('/api/users/:id', async (ctx) => {
    try {
        const { name, email, age } = ctx.request.body;
        
        const [result] = await pool.query(
            'UPDATE users SET name = ?, email = ?, age = ? WHERE id = ?',
            [name, email, age, ctx.params.id]
        );
        
        if (result.affectedRows === 0) {
            ctx.throw(404, '用户不存在');
        }
        
        ctx.body = {
            message: '用户更新成功'
        };
    } catch (error) {
        ctx.throw(400, '更新失败');
    }
});

// 删除用户
router.delete('/api/users/:id', async (ctx) => {
    try {
        const [result] = await pool.query(
            'DELETE FROM users WHERE id = ?',
            [ctx.params.id]
        );
        
        if (result.affectedRows === 0) {
            ctx.throw(404, '用户不存在');
        }
        
        ctx.body = {
            message: '用户删除成功'
        };
    } catch (error) {
        ctx.throw(500, '删除失败');
    }
});

app.use(router.routes());

app.listen(3000);
```

---

## 第10章 实战应用

### 10.1 完整RESTful API

```javascript
/**
 * 完整的RESTful API示例
 * @author erik.zhou
 */

const Koa = require('koa');
const Router = require('@koa/router');
const bodyParser = require('koa-bodyparser');
const cors = require('@koa/cors');
const helmet = require('koa-helmet');
const compress = require('koa-compress');
const winston = require('winston');

const app = new Koa();
const router = new Router({ prefix: '/api' });

// 日志配置
const logger = winston.createLogger({
    level: 'info',
    format: winston.format.json(),
    transports: [
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
        new winston.transports.File({ filename: 'combined.log' })
    ]
});

// 中间件
app.use(helmet());
app.use(compress());
app.use(cors());
app.use(bodyParser());

// 请求日志
app.use(async (ctx, next) => {
    const start = Date.now();
    await next();
    const duration = Date.now() - start;
    
    logger.info({
        method: ctx.method,
        url: ctx.url,
        status: ctx.status,
        duration: `${duration}ms`
    });
});

// 错误处理
app.use(async (ctx, next) => {
    try {
        await next();
    } catch (err) {
        ctx.status = err.status || 500;
        ctx.body = {
            error: {
                message: err.message,
                status: ctx.status
            }
        };
        
        logger.error({
            error: err.message,
            stack: err.stack,
            url: ctx.url
        });
    }
});

// 模拟数据库
let users = [
    { id: 1, name: 'John', email: 'john@example.com' },
    { id: 2, name: 'Jane', email: 'jane@example.com' }
];

let nextId = 3;

// 路由
router.get('/users', async (ctx) => {
    const { page = 1, limit = 10 } = ctx.query;
    const start = (page - 1) * limit;
    const end = start + parseInt(limit);
    
    ctx.body = {
        users: users.slice(start, end),
        pagination: {
            page: parseInt(page),
            limit: parseInt(limit),
            total: users.length
        }
    };
});

router.get('/users/:id', async (ctx) => {
    const user = users.find(u => u.id === parseInt(ctx.params.id));
    
    if (!user) {
        ctx.throw(404, '用户不存在');
    }
    
    ctx.body = { user };
});

router.post('/users', async (ctx) => {
    const { name, email } = ctx.request.body;
    
    if (!name || !email) {
        ctx.throw(400, '名字和邮箱不能为空');
    }
    
    const user = {
        id: nextId++,
        name,
        email
    };
    
    users.push(user);
    
    ctx.status = 201;
    ctx.body = {
        message: '用户创建成功',
        user
    };
});

router.put('/users/:id', async (ctx) => {
    const userId = parseInt(ctx.params.id);
    const index = users.findIndex(u => u.id === userId);
    
    if (index === -1) {
        ctx.throw(404, '用户不存在');
    }
    
    const { name, email } = ctx.request.body;
    users[index] = { ...users[index], name, email };
    
    ctx.body = {
        message: '用户更新成功',
        user: users[index]
    };
});

router.delete('/users/:id', async (ctx) => {
    const userId = parseInt(ctx.params.id);
    const index = users.findIndex(u => u.id === userId);
    
    if (index === -1) {
        ctx.throw(404, '用户不存在');
    }
    
    users.splice(index, 1);
    
    ctx.body = {
        message: '用户删除成功'
    };
});

app.use(router.routes());
app.use(router.allowedMethods());

app.listen(3000, () => {
    console.log('服务器运行在 http://localhost:3000');
});
```

---

## 总结

本教程全面介绍了Koa框架的核心知识，包括：

1. Koa基础 - 安装、配置、生命周期
2. 中间件机制 - 洋葱模型、组合、异步处理
3. 上下文对象 - Context、Request、Response、State
4. 路由管理 - koa-router、参数、前缀、嵌套、模块化
5. 请求处理 - 请求体解析、文件上传、Cookie、Session、验证
6. 响应处理 - 响应类型、状态码、响应头、内容协商、流式响应
7. 错误处理 - 错误捕获、自定义错误类、错误日志
8. 常用中间件 - 静态文件、CORS、压缩、限流、JWT
9. 数据库集成 - MongoDB、MySQL
10. 实战应用 - 完整RESTful API

通过本教程的学习，你应该能够：
- 理解Koa的设计理念和洋葱模型
- 熟练使用async/await处理异步
- 掌握中间件的编写和使用
- 构建完整的RESTful API
- 实现安全的认证和授权
- 集成数据库进行CRUD操作

## 参考资源

- [Koa官方文档](https://koajs.com/)
- [Koa中文文档](https://koa.bootcss.com/)
- [Awesome Koa](https://github.com/ellerbrock/awesome-koa)

---

**作者**: erik.zhou  
**最后更新**: 2024年

