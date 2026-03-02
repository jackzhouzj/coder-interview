# Express - 完整教程

## 课程简介

Express是Node.js最流行的Web应用框架，提供了一套简洁而强大的API来构建Web应用和API。本教程将深入讲解Express的核心概念、中间件系统、路由管理、错误处理、安全实践等内容。

## 学习目标

- 掌握Express的核心概念和架构
- 熟练使用中间件系统
- 掌握路由管理和参数处理
- 学会模板引擎的使用
- 理解错误处理机制
- 掌握RESTful API设计
- 学会安全最佳实践
- 掌握性能优化技巧

## 目录

1. [Express基础](#第1章-express基础)
2. [中间件系统](#第2章-中间件系统)
3. [路由管理](#第3章-路由管理)
4. [请求与响应](#第4章-请求与响应)
5. [模板引擎](#第5章-模板引擎)
6. [错误处理](#第6章-错误处理)
7. [数据验证](#第7章-数据验证)
8. [文件上传](#第8章-文件上传)
9. [会话管理](#第9章-会话管理)
10. [安全实践](#第10章-安全实践)

---

## 第1章 Express基础

### 1.1 安装与初始化

```bash
# 创建项目
mkdir express-app
cd express-app
npm init -y

# 安装Express
npm install express

# 安装开发依赖
npm install --save-dev nodemon
```

```javascript
/**
 * 第一个Express应用
 * @author erik.zhou
 */

const express = require('express');

// 创建Express应用
const app = express();

// 定义路由
app.get('/', (req, res) => {
    res.send('Hello Express!');
});

// 启动服务器
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`服务器运行在 http://localhost:${PORT}`);
});
```

### 1.2 应用配置

```javascript
/**
 * Express应用配置
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 设置应用配置
app.set('port', process.env.PORT || 3000);
app.set('env', process.env.NODE_ENV || 'development');

// 获取配置
console.log('端口:', app.get('port'));
console.log('环境:', app.get('env'));

// 启用/禁用配置
app.enable('trust proxy');
app.disable('x-powered-by');

// 检查配置
console.log('trust proxy:', app.enabled('trust proxy'));
console.log('x-powered-by:', app.disabled('x-powered-by'));
```



```javascript
/**
 * 环境配置管理
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 根据环境加载配置
const config = {
    development: {
        port: 3000,
        db: 'mongodb://localhost/dev'
    },
    production: {
        port: process.env.PORT || 8080,
        db: process.env.DB_URL
    }
};

const env = app.get('env');
const currentConfig = config[env];

app.set('port', currentConfig.port);
app.set('db', currentConfig.db);

console.log('当前环境:', env);
console.log('配置:', currentConfig);
```

### 1.3 静态文件服务

```javascript
/**
 * 静态文件服务
 * @author erik.zhou
 */

const express = require('express');
const path = require('path');
const app = express();

// 提供静态文件服务
app.use(express.static('public'));

// 指定虚拟路径
app.use('/static', express.static('public'));

// 多个静态目录
app.use(express.static('public'));
app.use(express.static('files'));

// 设置缓存
app.use(express.static('public', {
    maxAge: '1d',
    etag: true,
    lastModified: true
}));

// 自定义静态文件处理
app.use('/assets', express.static(path.join(__dirname, 'public'), {
    index: false,
    redirect: false,
    setHeaders: (res, filePath) => {
        if (filePath.endsWith('.js')) {
            res.set('Content-Type', 'application/javascript');
        }
    }
}));

app.listen(3000);
```

---

## 第2章 中间件系统

### 2.1 中间件基础

```javascript
/**
 * 中间件基本概念
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 应用级中间件
app.use((req, res, next) => {
    console.log('请求时间:', new Date().toISOString());
    console.log('请求方法:', req.method);
    console.log('请求路径:', req.path);
    next(); // 调用下一个中间件
});

// 路径匹配中间件
app.use('/api', (req, res, next) => {
    console.log('API请求');
    next();
});

// 多个中间件
app.use(
    (req, res, next) => {
        console.log('中间件1');
        next();
    },
    (req, res, next) => {
        console.log('中间件2');
        next();
    }
);

// 路由处理
app.get('/', (req, res) => {
    res.send('Hello World');
});

app.listen(3000);
```

### 2.2 内置中间件

```javascript
/**
 * Express内置中间件
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 解析JSON请求体
app.use(express.json({
    limit: '10mb',
    strict: true
}));

// 解析URL编码请求体
app.use(express.urlencoded({
    extended: true,
    limit: '10mb'
}));

// 静态文件服务
app.use(express.static('public'));

// 测试路由
app.post('/api/data', (req, res) => {
    console.log('请求体:', req.body);
    res.json({
        success: true,
        data: req.body
    });
});

app.listen(3000);
```

### 2.3 第三方中间件

```javascript
/**
 * 常用第三方中间件
 * @author erik.zhou
 */

const express = require('express');
const morgan = require('morgan');
const cors = require('cors');
const helmet = require('helmet');
const compression = require('compression');
const cookieParser = require('cookie-parser');

const app = express();

// 日志中间件
app.use(morgan('combined'));

// CORS跨域
app.use(cors({
    origin: 'http://localhost:3001',
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization']
}));

// 安全中间件
app.use(helmet());

// 压缩响应
app.use(compression());

// Cookie解析
app.use(cookieParser('secret-key'));

// 请求体解析
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

app.listen(3000);
```

### 2.4 自定义中间件

```javascript
/**
 * 自定义中间件
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 日志中间件
function logger(req, res, next) {
    const start = Date.now();
    
    res.on('finish', () => {
        const duration = Date.now() - start;
        console.log(`${req.method} ${req.path} - ${res.statusCode} - ${duration}ms`);
    });
    
    next();
}

// 认证中间件
function authenticate(req, res, next) {
    const token = req.headers.authorization;
    
    if (!token) {
        return res.status(401).json({
            error: '未提供认证令牌'
        });
    }
    
    try {
        // 验证token
        const user = verifyToken(token);
        req.user = user;
        next();
    } catch (error) {
        res.status(401).json({
            error: '无效的令牌'
        });
    }
}

// 权限检查中间件
function authorize(...roles) {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({
                error: '未认证'
            });
        }
        
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({
                error: '权限不足'
            });
        }
        
        next();
    };
}

// 使用中间件
app.use(logger);

app.get('/api/public', (req, res) => {
    res.json({ message: '公开接口' });
});

app.get('/api/protected', authenticate, (req, res) => {
    res.json({ message: '受保护的接口', user: req.user });
});

app.get('/api/admin', authenticate, authorize('admin'), (req, res) => {
    res.json({ message: '管理员接口' });
});

function verifyToken(token) {
    // 简化的token验证
    return { id: 1, name: 'John', role: 'admin' };
}

app.listen(3000);
```

### 2.5 错误处理中间件

```javascript
/**
 * 错误处理中间件
 * @author erik.zhou
 */

const express = require('express');
const app = express();

app.use(express.json());

// 普通路由
app.get('/api/users/:id', (req, res, next) => {
    const userId = req.params.id;
    
    if (!userId) {
        const error = new Error('用户ID不能为空');
        error.status = 400;
        return next(error);
    }
    
    // 模拟数据库查询
    const user = findUser(userId);
    
    if (!user) {
        const error = new Error('用户不存在');
        error.status = 404;
        return next(error);
    }
    
    res.json(user);
});

// 404处理
app.use((req, res, next) => {
    const error = new Error('资源未找到');
    error.status = 404;
    next(error);
});

// 错误处理中间件（必须有4个参数）
app.use((err, req, res, next) => {
    console.error('错误:', err);
    
    const status = err.status || 500;
    const message = err.message || '服务器内部错误';
    
    res.status(status).json({
        error: {
            message: message,
            status: status,
            timestamp: new Date().toISOString(),
            path: req.path
        }
    });
});

function findUser(id) {
    // 模拟数据库查询
    return null;
}

app.listen(3000);
```

---

## 第3章 路由管理

### 3.1 基本路由

```javascript
/**
 * 基本路由定义
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// GET请求
app.get('/', (req, res) => {
    res.send('GET请求');
});

// POST请求
app.post('/api/users', (req, res) => {
    res.json({ message: 'POST请求' });
});

// PUT请求
app.put('/api/users/:id', (req, res) => {
    res.json({ message: 'PUT请求', id: req.params.id });
});

// DELETE请求
app.delete('/api/users/:id', (req, res) => {
    res.json({ message: 'DELETE请求', id: req.params.id });
});

// PATCH请求
app.patch('/api/users/:id', (req, res) => {
    res.json({ message: 'PATCH请求', id: req.params.id });
});

// 处理所有HTTP方法
app.all('/api/test', (req, res) => {
    res.json({ method: req.method });
});

app.listen(3000);
```

### 3.2 路由参数

```javascript
/**
 * 路由参数处理
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 路径参数
app.get('/users/:id', (req, res) => {
    const userId = req.params.id;
    res.json({ userId });
});

// 多个路径参数
app.get('/users/:userId/posts/:postId', (req, res) => {
    const { userId, postId } = req.params;
    res.json({ userId, postId });
});

// 可选参数
app.get('/users/:id?', (req, res) => {
    const userId = req.params.id || 'all';
    res.json({ userId });
});

// 正则表达式参数
app.get('/users/:id(\\d+)', (req, res) => {
    res.json({ userId: req.params.id });
});

// 查询参数
app.get('/search', (req, res) => {
    const { keyword, page, limit } = req.query;
    res.json({
        keyword,
        page: parseInt(page) || 1,
        limit: parseInt(limit) || 10
    });
});

// 参数验证中间件
app.param('id', (req, res, next, id) => {
    if (!/^\d+$/.test(id)) {
        return res.status(400).json({
            error: 'ID必须是数字'
        });
    }
    req.userId = parseInt(id);
    next();
});

app.get('/api/users/:id', (req, res) => {
    res.json({ userId: req.userId });
});

app.listen(3000);
```

### 3.3 路由模块化

```javascript
/**
 * 路由模块化 - routes/users.js
 * @author erik.zhou
 */

const express = require('express');
const router = express.Router();

// 中间件
router.use((req, res, next) => {
    console.log('用户路由中间件');
    next();
});

// 获取用户列表
router.get('/', (req, res) => {
    res.json({
        users: [
            { id: 1, name: 'John' },
            { id: 2, name: 'Jane' }
        ]
    });
});

// 获取单个用户
router.get('/:id', (req, res) => {
    const userId = req.params.id;
    res.json({
        user: { id: userId, name: 'John' }
    });
});

// 创建用户
router.post('/', (req, res) => {
    const user = req.body;
    res.status(201).json({
        message: '用户创建成功',
        user: user
    });
});

// 更新用户
router.put('/:id', (req, res) => {
    const userId = req.params.id;
    const user = req.body;
    res.json({
        message: '用户更新成功',
        user: { id: userId, ...user }
    });
});

// 删除用户
router.delete('/:id', (req, res) => {
    const userId = req.params.id;
    res.json({
        message: '用户删除成功',
        userId: userId
    });
});

module.exports = router;
```

```javascript
/**
 * 主应用 - app.js
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 导入路由模块
const usersRouter = require('./routes/users');
const postsRouter = require('./routes/posts');

app.use(express.json());

// 挂载路由
app.use('/api/users', usersRouter);
app.use('/api/posts', postsRouter);

// 根路由
app.get('/', (req, res) => {
    res.json({
        message: 'API服务器',
        endpoints: [
            '/api/users',
            '/api/posts'
        ]
    });
});

app.listen(3000, () => {
    console.log('服务器运行在 http://localhost:3000');
});
```

### 3.4 路由链式调用

```javascript
/**
 * 路由链式调用
 * @author erik.zhou
 */

const express = require('express');
const router = express.Router();

// 链式调用
router.route('/users/:id')
    .get((req, res) => {
        res.json({ message: '获取用户', id: req.params.id });
    })
    .put((req, res) => {
        res.json({ message: '更新用户', id: req.params.id });
    })
    .delete((req, res) => {
        res.json({ message: '删除用户', id: req.params.id });
    });

// 多个中间件
router.route('/posts')
    .all((req, res, next) => {
        console.log('所有请求都会经过这里');
        next();
    })
    .get((req, res) => {
        res.json({ message: '获取文章列表' });
    })
    .post(
        validatePost,
        (req, res) => {
            res.json({ message: '创建文章' });
        }
    );

function validatePost(req, res, next) {
    if (!req.body.title) {
        return res.status(400).json({
            error: '标题不能为空'
        });
    }
    next();
}

module.exports = router;
```

### 3.5 嵌套路由

```javascript
/**
 * 嵌套路由
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 用户路由
const usersRouter = express.Router();

usersRouter.get('/', (req, res) => {
    res.json({ message: '用户列表' });
});

usersRouter.get('/:userId', (req, res) => {
    res.json({ message: '用户详情', userId: req.params.userId });
});

// 用户的文章路由（嵌套）
const userPostsRouter = express.Router({ mergeParams: true });

userPostsRouter.get('/', (req, res) => {
    res.json({
        message: '用户的文章列表',
        userId: req.params.userId
    });
});

userPostsRouter.get('/:postId', (req, res) => {
    res.json({
        message: '用户的文章详情',
        userId: req.params.userId,
        postId: req.params.postId
    });
});

// 挂载嵌套路由
usersRouter.use('/:userId/posts', userPostsRouter);

// 挂载到应用
app.use('/api/users', usersRouter);

app.listen(3000);
```

---

## 第4章 请求与响应

### 4.1 请求对象

```javascript
/**
 * 请求对象详解
 * @author erik.zhou
 */

const express = require('express');
const app = express();

app.use(express.json());

app.all('/api/test', (req, res) => {
    // 请求基本信息
    const requestInfo = {
        // HTTP方法
        method: req.method,
        
        // URL信息
        url: req.url,
        originalUrl: req.originalUrl,
        path: req.path,
        baseUrl: req.baseUrl,
        
        // 主机信息
        hostname: req.hostname,
        ip: req.ip,
        ips: req.ips,
        protocol: req.protocol,
        secure: req.secure,
        
        // 请求头
        headers: req.headers,
        userAgent: req.get('User-Agent'),
        contentType: req.get('Content-Type'),
        
        // 参数
        params: req.params,
        query: req.query,
        body: req.body,
        
        // Cookie
        cookies: req.cookies,
        signedCookies: req.signedCookies,
        
        // 其他
        fresh: req.fresh,
        stale: req.stale,
        xhr: req.xhr
    };
    
    res.json(requestInfo);
});

app.listen(3000);
```

### 4.2 响应对象

```javascript
/**
 * 响应对象详解
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 发送文本
app.get('/text', (req, res) => {
    res.send('Hello World');
});

// 发送JSON
app.get('/json', (req, res) => {
    res.json({
        message: 'Success',
        data: { id: 1, name: 'John' }
    });
});

// 发送状态码
app.get('/status', (req, res) => {
    res.status(201).json({
        message: '创建成功'
    });
});

// 设置响应头
app.get('/headers', (req, res) => {
    res.set('X-Custom-Header', 'value');
    res.set({
        'Content-Type': 'application/json',
        'X-Powered-By': 'Express'
    });
    res.json({ message: 'Headers set' });
});

// 重定向
app.get('/redirect', (req, res) => {
    res.redirect('/new-url');
});

app.get('/redirect-301', (req, res) => {
    res.redirect(301, '/permanent-url');
});

// 发送文件
app.get('/download', (req, res) => {
    res.download('./files/document.pdf', 'my-document.pdf', (err) => {
        if (err) {
            console.error('下载失败:', err);
        }
    });
});

// 发送文件内容
app.get('/file', (req, res) => {
    res.sendFile(__dirname + '/files/index.html');
});

// 设置Cookie
app.get('/cookie', (req, res) => {
    res.cookie('name', 'value', {
        maxAge: 900000,
        httpOnly: true,
        secure: true,
        signed: true
    });
    res.json({ message: 'Cookie已设置' });
});

// 清除Cookie
app.get('/clear-cookie', (req, res) => {
    res.clearCookie('name');
    res.json({ message: 'Cookie已清除' });
});

// 链式调用
app.get('/chain', (req, res) => {
    res
        .status(200)
        .set('Content-Type', 'application/json')
        .json({ message: 'Success' });
});

app.listen(3000);
```

### 4.3 内容协商

```javascript
/**
 * 内容协商
 * @author erik.zhou
 */

const express = require('express');
const app = express();

app.get('/api/data', (req, res) => {
    const data = {
        id: 1,
        name: 'John',
        email: 'john@example.com'
    };
    
    // 根据Accept头返回不同格式
    res.format({
        'text/plain': () => {
            res.send(`ID: ${data.id}\nName: ${data.name}\nEmail: ${data.email}`);
        },
        
        'text/html': () => {
            res.send(`
                <h1>${data.name}</h1>
                <p>ID: ${data.id}</p>
                <p>Email: ${data.email}</p>
            `);
        },
        
        'application/json': () => {
            res.json(data);
        },
        
        'application/xml': () => {
            res.send(`
                <user>
                    <id>${data.id}</id>
                    <name>${data.name}</name>
                    <email>${data.email}</email>
                </user>
            `);
        },
        
        'default': () => {
            res.status(406).send('Not Acceptable');
        }
    });
});

app.listen(3000);
```

### 4.4 流式响应

```javascript
/**
 * 流式响应
 * @author erik.zhou
 */

const express = require('express');
const fs = require('fs');
const app = express();

// 发送文件流
app.get('/stream/file', (req, res) => {
    const filePath = './large-file.txt';
    const stream = fs.createReadStream(filePath);
    
    res.setHeader('Content-Type', 'text/plain');
    stream.pipe(res);
    
    stream.on('error', (err) => {
        res.status(500).json({
            error: '文件读取失败'
        });
    });
});

// Server-Sent Events (SSE)
app.get('/stream/events', (req, res) => {
    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');
    
    let count = 0;
    
    const interval = setInterval(() => {
        count++;
        res.write(`data: ${JSON.stringify({ count, time: new Date() })}\n\n`);
        
        if (count >= 10) {
            clearInterval(interval);
            res.end();
        }
    }, 1000);
    
    req.on('close', () => {
        clearInterval(interval);
    });
});

// 分块传输
app.get('/stream/chunks', (req, res) => {
    res.setHeader('Content-Type', 'text/plain');
    res.setHeader('Transfer-Encoding', 'chunked');
    
    let i = 0;
    const interval = setInterval(() => {
        res.write(`Chunk ${i}\n`);
        i++;
        
        if (i >= 5) {
            clearInterval(interval);
            res.end();
        }
    }, 500);
});

app.listen(3000);
```



---

## 第5章 模板引擎

### 5.1 EJS模板

```javascript
/**
 * EJS模板引擎
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 设置模板引擎
app.set('view engine', 'ejs');
app.set('views', './views');

// 渲染模板
app.get('/', (req, res) => {
    res.render('index', {
        title: '首页',
        user: {
            name: 'John',
            email: 'john@example.com'
        },
        items: ['Item 1', 'Item 2', 'Item 3']
    });
});

app.listen(3000);
```

```html
<!-- views/index.ejs -->
<!DOCTYPE html>
<html>
<head>
    <title><%= title %></title>
</head>
<body>
    <h1>欢迎, <%= user.name %></h1>
    <p>邮箱: <%= user.email %></p>
    
    <h2>列表</h2>
    <ul>
        <% items.forEach(item => { %>
            <li><%= item %></li>
        <% }); %>
    </ul>
    
    <!-- 包含其他模板 -->
    <%- include('partials/footer') %>
</body>
</html>
```

### 5.2 Pug模板

```javascript
/**
 * Pug模板引擎
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 设置Pug模板引擎
app.set('view engine', 'pug');
app.set('views', './views');

app.get('/', (req, res) => {
    res.render('index', {
        title: '首页',
        message: '欢迎使用Pug',
        users: [
            { name: 'John', age: 30 },
            { name: 'Jane', age: 25 }
        ]
    });
});

app.listen(3000);
```

```pug
//- views/index.pug
doctype html
html
    head
        title= title
    body
        h1= message
        
        h2 用户列表
        ul
            each user in users
                li #{user.name} - #{user.age}岁
        
        //- 包含其他模板
        include partials/footer
```

### 5.3 Handlebars模板

```javascript
/**
 * Handlebars模板引擎
 * @author erik.zhou
 */

const express = require('express');
const exphbs = require('express-handlebars');
const app = express();

// 配置Handlebars
app.engine('handlebars', exphbs.engine({
    defaultLayout: 'main',
    layoutsDir: './views/layouts',
    partialsDir: './views/partials',
    helpers: {
        uppercase: (str) => str.toUpperCase(),
        formatDate: (date) => new Date(date).toLocaleDateString()
    }
}));

app.set('view engine', 'handlebars');
app.set('views', './views');

app.get('/', (req, res) => {
    res.render('home', {
        title: '首页',
        posts: [
            { title: '文章1', date: new Date(), author: 'John' },
            { title: '文章2', date: new Date(), author: 'Jane' }
        ]
    });
});

app.listen(3000);
```

```handlebars
{{!-- views/home.handlebars --}}
<h1>{{title}}</h1>

<div class="posts">
    {{#each posts}}
        <article>
            <h2>{{uppercase title}}</h2>
            <p>作者: {{author}}</p>
            <p>日期: {{formatDate date}}</p>
        </article>
    {{/each}}
</div>

{{> footer}}
```

---

## 第6章 错误处理

### 6.1 同步错误处理

```javascript
/**
 * 同步错误处理
 * @author erik.zhou
 */

const express = require('express');
const app = express();

app.use(express.json());

// 同步错误会自动被捕获
app.get('/sync-error', (req, res) => {
    throw new Error('同步错误');
});

// 手动抛出错误
app.get('/api/users/:id', (req, res, next) => {
    const userId = parseInt(req.params.id);
    
    if (isNaN(userId)) {
        const error = new Error('无效的用户ID');
        error.status = 400;
        throw error;
    }
    
    res.json({ userId });
});

// 错误处理中间件
app.use((err, req, res, next) => {
    console.error('错误:', err);
    
    res.status(err.status || 500).json({
        error: {
            message: err.message,
            status: err.status || 500
        }
    });
});

app.listen(3000);
```

### 6.2 异步错误处理

```javascript
/**
 * 异步错误处理
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 异步错误需要传递给next
app.get('/async-error', (req, res, next) => {
    setTimeout(() => {
        try {
            throw new Error('异步错误');
        } catch (error) {
            next(error);
        }
    }, 100);
});

// Promise错误处理
app.get('/promise-error', (req, res, next) => {
    Promise.reject(new Error('Promise错误'))
        .catch(next);
});

// async/await错误处理
app.get('/async-await-error', async (req, res, next) => {
    try {
        await someAsyncOperation();
        res.json({ success: true });
    } catch (error) {
        next(error);
    }
});

// 包装async函数
function asyncHandler(fn) {
    return (req, res, next) => {
        Promise.resolve(fn(req, res, next)).catch(next);
    };
}

app.get('/wrapped', asyncHandler(async (req, res) => {
    const data = await fetchData();
    res.json(data);
}));

async function someAsyncOperation() {
    throw new Error('异步操作失败');
}

async function fetchData() {
    return { data: 'test' };
}

// 错误处理中间件
app.use((err, req, res, next) => {
    console.error('错误:', err);
    res.status(500).json({
        error: err.message
    });
});

app.listen(3000);
```

### 6.3 自定义错误类

```javascript
/**
 * 自定义错误类
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 自定义错误类
class AppError extends Error {
    constructor(message, status = 500) {
        super(message);
        this.status = status;
        this.isOperational = true;
        Error.captureStackTrace(this, this.constructor);
    }
}

class ValidationError extends AppError {
    constructor(message) {
        super(message, 400);
        this.name = 'ValidationError';
    }
}

class NotFoundError extends AppError {
    constructor(message = '资源未找到') {
        super(message, 404);
        this.name = 'NotFoundError';
    }
}

class UnauthorizedError extends AppError {
    constructor(message = '未授权') {
        super(message, 401);
        this.name = 'UnauthorizedError';
    }
}

// 使用自定义错误
app.get('/api/users/:id', (req, res, next) => {
    const userId = req.params.id;
    
    if (!userId) {
        throw new ValidationError('用户ID不能为空');
    }
    
    const user = findUser(userId);
    
    if (!user) {
        throw new NotFoundError('用户不存在');
    }
    
    res.json(user);
});

app.get('/api/protected', (req, res, next) => {
    if (!req.headers.authorization) {
        throw new UnauthorizedError();
    }
    
    res.json({ message: '受保护的资源' });
});

function findUser(id) {
    return null;
}

// 错误处理中间件
app.use((err, req, res, next) => {
    // 区分操作错误和程序错误
    if (err.isOperational) {
        res.status(err.status).json({
            error: {
                name: err.name,
                message: err.message,
                status: err.status
            }
        });
    } else {
        // 程序错误，记录日志
        console.error('程序错误:', err);
        res.status(500).json({
            error: {
                message: '服务器内部错误'
            }
        });
    }
});

app.listen(3000);
```

### 6.4 错误日志

```javascript
/**
 * 错误日志记录
 * @author erik.zhou
 */

const express = require('express');
const winston = require('winston');
const app = express();

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
app.use((req, res, next) => {
    logger.info({
        method: req.method,
        url: req.url,
        ip: req.ip,
        userAgent: req.get('User-Agent')
    });
    next();
});

// 错误处理中间件
app.use((err, req, res, next) => {
    // 记录错误日志
    logger.error({
        message: err.message,
        stack: err.stack,
        url: req.url,
        method: req.method,
        ip: req.ip,
        timestamp: new Date().toISOString()
    });
    
    res.status(err.status || 500).json({
        error: {
            message: err.message
        }
    });
});

app.listen(3000);
```

---

## 第7章 数据验证

### 7.1 使用Joi验证

```javascript
/**
 * Joi数据验证
 * @author erik.zhou
 */

const express = require('express');
const Joi = require('joi');
const app = express();

app.use(express.json());

// 定义验证模式
const userSchema = Joi.object({
    name: Joi.string().min(3).max(30).required(),
    email: Joi.string().email().required(),
    age: Joi.number().integer().min(18).max(100),
    password: Joi.string().min(8).pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
    role: Joi.string().valid('user', 'admin').default('user')
});

// 验证中间件
function validate(schema) {
    return (req, res, next) => {
        const { error, value } = schema.validate(req.body, {
            abortEarly: false,
            stripUnknown: true
        });
        
        if (error) {
            const errors = error.details.map(detail => ({
                field: detail.path.join('.'),
                message: detail.message
            }));
            
            return res.status(400).json({
                error: '验证失败',
                details: errors
            });
        }
        
        req.validatedData = value;
        next();
    };
}

// 使用验证
app.post('/api/users', validate(userSchema), (req, res) => {
    const user = req.validatedData;
    res.status(201).json({
        message: '用户创建成功',
        user: user
    });
});

// 更新验证（部分字段可选）
const updateUserSchema = Joi.object({
    name: Joi.string().min(3).max(30),
    email: Joi.string().email(),
    age: Joi.number().integer().min(18).max(100)
}).min(1);

app.put('/api/users/:id', validate(updateUserSchema), (req, res) => {
    const userId = req.params.id;
    const updates = req.validatedData;
    
    res.json({
        message: '用户更新成功',
        userId: userId,
        updates: updates
    });
});

app.listen(3000);
```

### 7.2 使用express-validator

```javascript
/**
 * express-validator数据验证
 * @author erik.zhou
 */

const express = require('express');
const { body, param, query, validationResult } = require('express-validator');
const app = express();

app.use(express.json());

// 验证规则
const userValidationRules = [
    body('name')
        .trim()
        .isLength({ min: 3, max: 30 })
        .withMessage('名字长度必须在3-30之间'),
    
    body('email')
        .isEmail()
        .normalizeEmail()
        .withMessage('邮箱格式不正确'),
    
    body('age')
        .optional()
        .isInt({ min: 18, max: 100 })
        .withMessage('年龄必须在18-100之间'),
    
    body('password')
        .isLength({ min: 8 })
        .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
        .withMessage('密码必须包含大小写字母和数字')
];

// 验证中间件
function validateRequest(req, res, next) {
    const errors = validationResult(req);
    
    if (!errors.isEmpty()) {
        return res.status(400).json({
            error: '验证失败',
            details: errors.array()
        });
    }
    
    next();
}

// 使用验证
app.post('/api/users', userValidationRules, validateRequest, (req, res) => {
    const user = req.body;
    res.status(201).json({
        message: '用户创建成功',
        user: user
    });
});

// 参数验证
app.get('/api/users/:id',
    param('id').isInt().withMessage('ID必须是整数'),
    validateRequest,
    (req, res) => {
        const userId = req.params.id;
        res.json({ userId });
    }
);

// 查询参数验证
app.get('/api/search',
    query('keyword').notEmpty().withMessage('关键词不能为空'),
    query('page').optional().isInt({ min: 1 }).withMessage('页码必须是正整数'),
    query('limit').optional().isInt({ min: 1, max: 100 }).withMessage('每页数量必须在1-100之间'),
    validateRequest,
    (req, res) => {
        const { keyword, page = 1, limit = 10 } = req.query;
        res.json({ keyword, page, limit });
    }
);

// 自定义验证
app.post('/api/register',
    body('email').custom(async (email) => {
        const user = await findUserByEmail(email);
        if (user) {
            throw new Error('邮箱已被注册');
        }
    }),
    validateRequest,
    (req, res) => {
        res.json({ message: '注册成功' });
    }
);

async function findUserByEmail(email) {
    // 模拟数据库查询
    return null;
}

app.listen(3000);
```

### 7.3 自定义验证器

```javascript
/**
 * 自定义验证器
 * @author erik.zhou
 */

const express = require('express');
const app = express();

app.use(express.json());

// 验证器工厂函数
function createValidator(rules) {
    return (req, res, next) => {
        const errors = [];
        
        for (const [field, validators] of Object.entries(rules)) {
            const value = req.body[field];
            
            for (const validator of validators) {
                const error = validator(value, field, req.body);
                if (error) {
                    errors.push(error);
                    break;
                }
            }
        }
        
        if (errors.length > 0) {
            return res.status(400).json({
                error: '验证失败',
                details: errors
            });
        }
        
        next();
    };
}

// 验证函数
const validators = {
    required: (value, field) => {
        if (!value) {
            return { field, message: `${field}不能为空` };
        }
    },
    
    minLength: (min) => (value, field) => {
        if (value && value.length < min) {
            return { field, message: `${field}长度不能小于${min}` };
        }
    },
    
    maxLength: (max) => (value, field) => {
        if (value && value.length > max) {
            return { field, message: `${field}长度不能大于${max}` };
        }
    },
    
    email: (value, field) => {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (value && !emailRegex.test(value)) {
            return { field, message: `${field}格式不正确` };
        }
    },
    
    pattern: (regex, message) => (value, field) => {
        if (value && !regex.test(value)) {
            return { field, message: message || `${field}格式不正确` };
        }
    },
    
    custom: (fn) => (value, field, data) => {
        const error = fn(value, field, data);
        if (error) {
            return { field, message: error };
        }
    }
};

// 使用验证器
const userValidator = createValidator({
    name: [
        validators.required,
        validators.minLength(3),
        validators.maxLength(30)
    ],
    email: [
        validators.required,
        validators.email
    ],
    password: [
        validators.required,
        validators.minLength(8),
        validators.pattern(
            /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
            '密码必须包含大小写字母和数字'
        )
    ],
    confirmPassword: [
        validators.required,
        validators.custom((value, field, data) => {
            if (value !== data.password) {
                return '两次密码不一致';
            }
        })
    ]
});

app.post('/api/users', userValidator, (req, res) => {
    const user = req.body;
    res.status(201).json({
        message: '用户创建成功',
        user: user
    });
});

app.listen(3000);
```



---

## 第8章 文件上传

### 8.1 使用Multer

```javascript
/**
 * Multer文件上传
 * @author erik.zhou
 */

const express = require('express');
const multer = require('multer');
const path = require('path');
const app = express();

// 配置存储
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads/');
    },
    filename: (req, file, cb) => {
        const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
        cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
    }
});

// 文件过滤
const fileFilter = (req, file, cb) => {
    const allowedTypes = /jpeg|jpg|png|gif/;
    const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
    const mimetype = allowedTypes.test(file.mimetype);
    
    if (extname && mimetype) {
        cb(null, true);
    } else {
        cb(new Error('只允许上传图片文件'));
    }
};

// 创建上传实例
const upload = multer({
    storage: storage,
    limits: {
        fileSize: 5 * 1024 * 1024 // 5MB
    },
    fileFilter: fileFilter
});

// 单文件上传
app.post('/upload/single', upload.single('avatar'), (req, res) => {
    if (!req.file) {
        return res.status(400).json({
            error: '请选择文件'
        });
    }
    
    res.json({
        message: '文件上传成功',
        file: {
            filename: req.file.filename,
            originalname: req.file.originalname,
            size: req.file.size,
            path: req.file.path
        }
    });
});

// 多文件上传
app.post('/upload/multiple', upload.array('photos', 10), (req, res) => {
    if (!req.files || req.files.length === 0) {
        return res.status(400).json({
            error: '请选择文件'
        });
    }
    
    const files = req.files.map(file => ({
        filename: file.filename,
        originalname: file.originalname,
        size: file.size
    }));
    
    res.json({
        message: '文件上传成功',
        count: files.length,
        files: files
    });
});

// 多字段上传
app.post('/upload/fields', upload.fields([
    { name: 'avatar', maxCount: 1 },
    { name: 'photos', maxCount: 5 }
]), (req, res) => {
    res.json({
        message: '文件上传成功',
        avatar: req.files['avatar'],
        photos: req.files['photos']
    });
});

// 错误处理
app.use((err, req, res, next) => {
    if (err instanceof multer.MulterError) {
        if (err.code === 'LIMIT_FILE_SIZE') {
            return res.status(400).json({
                error: '文件大小超过限制'
            });
        }
        if (err.code === 'LIMIT_FILE_COUNT') {
            return res.status(400).json({
                error: '文件数量超过限制'
            });
        }
    }
    
    res.status(500).json({
        error: err.message
    });
});

app.listen(3000);
```

### 8.2 内存存储

```javascript
/**
 * 内存存储文件上传
 * @author erik.zhou
 */

const express = require('express');
const multer = require('multer');
const sharp = require('sharp');
const app = express();

// 使用内存存储
const upload = multer({
    storage: multer.memoryStorage(),
    limits: {
        fileSize: 5 * 1024 * 1024
    }
});

// 上传并处理图片
app.post('/upload/image', upload.single('image'), async (req, res) => {
    try {
        if (!req.file) {
            return res.status(400).json({
                error: '请选择图片'
            });
        }
        
        // 使用sharp处理图片
        const processedImage = await sharp(req.file.buffer)
            .resize(800, 600, {
                fit: 'inside',
                withoutEnlargement: true
            })
            .jpeg({ quality: 80 })
            .toBuffer();
        
        // 保存处理后的图片
        const filename = `processed-${Date.now()}.jpg`;
        await sharp(processedImage).toFile(`uploads/${filename}`);
        
        res.json({
            message: '图片上传并处理成功',
            filename: filename,
            originalSize: req.file.size,
            processedSize: processedImage.length
        });
    } catch (error) {
        res.status(500).json({
            error: '图片处理失败'
        });
    }
});

app.listen(3000);
```

### 8.3 文件下载

```javascript
/**
 * 文件下载
 * @author erik.zhou
 */

const express = require('express');
const path = require('path');
const fs = require('fs');
const app = express();

// 下载文件
app.get('/download/:filename', (req, res) => {
    const filename = req.params.filename;
    const filePath = path.join(__dirname, 'uploads', filename);
    
    // 检查文件是否存在
    if (!fs.existsSync(filePath)) {
        return res.status(404).json({
            error: '文件不存在'
        });
    }
    
    // 下载文件
    res.download(filePath, filename, (err) => {
        if (err) {
            console.error('下载失败:', err);
            res.status(500).json({
                error: '下载失败'
            });
        }
    });
});

// 发送文件
app.get('/view/:filename', (req, res) => {
    const filename = req.params.filename;
    const filePath = path.join(__dirname, 'uploads', filename);
    
    if (!fs.existsSync(filePath)) {
        return res.status(404).json({
            error: '文件不存在'
        });
    }
    
    res.sendFile(filePath);
});

// 流式下载大文件
app.get('/stream/:filename', (req, res) => {
    const filename = req.params.filename;
    const filePath = path.join(__dirname, 'uploads', filename);
    
    if (!fs.existsSync(filePath)) {
        return res.status(404).json({
            error: '文件不存在'
        });
    }
    
    const stat = fs.statSync(filePath);
    
    res.setHeader('Content-Length', stat.size);
    res.setHeader('Content-Type', 'application/octet-stream');
    res.setHeader('Content-Disposition', `attachment; filename="${filename}"`);
    
    const stream = fs.createReadStream(filePath);
    stream.pipe(res);
});

app.listen(3000);
```

---

## 第9章 会话管理

### 9.1 Cookie会话

```javascript
/**
 * Cookie会话管理
 * @author erik.zhou
 */

const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();

app.use(cookieParser('secret-key'));

// 设置Cookie
app.get('/login', (req, res) => {
    // 普通Cookie
    res.cookie('username', 'john', {
        maxAge: 900000,
        httpOnly: true
    });
    
    // 签名Cookie
    res.cookie('userId', '123', {
        signed: true,
        maxAge: 900000,
        httpOnly: true,
        secure: true,
        sameSite: 'strict'
    });
    
    res.json({ message: '登录成功' });
});

// 读取Cookie
app.get('/profile', (req, res) => {
    const username = req.cookies.username;
    const userId = req.signedCookies.userId;
    
    if (!userId) {
        return res.status(401).json({
            error: '未登录'
        });
    }
    
    res.json({
        username: username,
        userId: userId
    });
});

// 清除Cookie
app.get('/logout', (req, res) => {
    res.clearCookie('username');
    res.clearCookie('userId');
    res.json({ message: '退出成功' });
});

app.listen(3000);
```

### 9.2 Session会话

```javascript
/**
 * Session会话管理
 * @author erik.zhou
 */

const express = require('express');
const session = require('express-session');
const app = express();

app.use(express.json());

// 配置Session
app.use(session({
    secret: 'secret-key',
    resave: false,
    saveUninitialized: false,
    cookie: {
        maxAge: 1000 * 60 * 60 * 24, // 24小时
        httpOnly: true,
        secure: false, // 生产环境设为true
        sameSite: 'strict'
    }
}));

// 登录
app.post('/login', (req, res) => {
    const { username, password } = req.body;
    
    // 验证用户
    if (username === 'admin' && password === 'password') {
        req.session.user = {
            id: 1,
            username: username,
            role: 'admin'
        };
        
        res.json({
            message: '登录成功',
            user: req.session.user
        });
    } else {
        res.status(401).json({
            error: '用户名或密码错误'
        });
    }
});

// 获取当前用户
app.get('/me', (req, res) => {
    if (!req.session.user) {
        return res.status(401).json({
            error: '未登录'
        });
    }
    
    res.json({
        user: req.session.user
    });
});

// 退出登录
app.post('/logout', (req, res) => {
    req.session.destroy((err) => {
        if (err) {
            return res.status(500).json({
                error: '退出失败'
            });
        }
        
        res.clearCookie('connect.sid');
        res.json({ message: '退出成功' });
    });
});

// 认证中间件
function requireAuth(req, res, next) {
    if (!req.session.user) {
        return res.status(401).json({
            error: '请先登录'
        });
    }
    next();
}

// 受保护的路由
app.get('/api/protected', requireAuth, (req, res) => {
    res.json({
        message: '受保护的资源',
        user: req.session.user
    });
});

app.listen(3000);
```

### 9.3 Redis Session

```javascript
/**
 * Redis Session存储
 * @author erik.zhou
 */

const express = require('express');
const session = require('express-session');
const RedisStore = require('connect-redis').default;
const { createClient } = require('redis');
const app = express();

// 创建Redis客户端
const redisClient = createClient({
    host: 'localhost',
    port: 6379
});

redisClient.connect().catch(console.error);

// 配置Session使用Redis存储
app.use(session({
    store: new RedisStore({ client: redisClient }),
    secret: 'secret-key',
    resave: false,
    saveUninitialized: false,
    cookie: {
        maxAge: 1000 * 60 * 60 * 24
    }
}));

app.post('/login', (req, res) => {
    req.session.user = {
        id: 1,
        username: 'john'
    };
    
    res.json({ message: '登录成功' });
});

app.get('/me', (req, res) => {
    if (!req.session.user) {
        return res.status(401).json({
            error: '未登录'
        });
    }
    
    res.json({ user: req.session.user });
});

app.listen(3000);
```

### 9.4 JWT认证

```javascript
/**
 * JWT认证
 * @author erik.zhou
 */

const express = require('express');
const jwt = require('jsonwebtoken');
const app = express();

app.use(express.json());

const SECRET_KEY = 'your-secret-key';

// 登录生成Token
app.post('/login', (req, res) => {
    const { username, password } = req.body;
    
    // 验证用户
    if (username === 'admin' && password === 'password') {
        const user = {
            id: 1,
            username: username,
            role: 'admin'
        };
        
        // 生成Token
        const token = jwt.sign(user, SECRET_KEY, {
            expiresIn: '24h'
        });
        
        res.json({
            message: '登录成功',
            token: token,
            user: user
        });
    } else {
        res.status(401).json({
            error: '用户名或密码错误'
        });
    }
});

// JWT验证中间件
function authenticateToken(req, res, next) {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    
    if (!token) {
        return res.status(401).json({
            error: '未提供认证令牌'
        });
    }
    
    jwt.verify(token, SECRET_KEY, (err, user) => {
        if (err) {
            return res.status(403).json({
                error: '无效的令牌'
            });
        }
        
        req.user = user;
        next();
    });
}

// 受保护的路由
app.get('/api/protected', authenticateToken, (req, res) => {
    res.json({
        message: '受保护的资源',
        user: req.user
    });
});

// 刷新Token
app.post('/refresh', authenticateToken, (req, res) => {
    const user = {
        id: req.user.id,
        username: req.user.username,
        role: req.user.role
    };
    
    const newToken = jwt.sign(user, SECRET_KEY, {
        expiresIn: '24h'
    });
    
    res.json({
        token: newToken
    });
});

app.listen(3000);
```

---

## 第10章 安全实践

### 10.1 安全中间件

```javascript
/**
 * 安全中间件配置
 * @author erik.zhou
 */

const express = require('express');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');
const app = express();

// Helmet安全头
app.use(helmet());

// 速率限制
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15分钟
    max: 100, // 最多100个请求
    message: '请求过于频繁，请稍后再试',
    standardHeaders: true,
    legacyHeaders: false
});

app.use('/api/', limiter);

// 登录速率限制
const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5,
    message: '登录尝试次数过多，请15分钟后再试'
});

app.use('/api/login', loginLimiter);

// 防止NoSQL注入
app.use(mongoSanitize());

// 防止XSS攻击
app.use(xss());

// 防止HTTP参数污染
app.use(hpp());

// CORS配置
app.use((req, res, next) => {
    res.setHeader('Access-Control-Allow-Origin', 'https://example.com');
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    res.setHeader('Access-Control-Allow-Credentials', 'true');
    next();
});

app.use(express.json({ limit: '10kb' }));

app.listen(3000);
```

### 10.2 输入验证与清理

```javascript
/**
 * 输入验证与清理
 * @author erik.zhou
 */

const express = require('express');
const { body, validationResult } = require('express-validator');
const app = express();

app.use(express.json());

// 输入验证和清理
app.post('/api/users',
    // 验证和清理
    body('email')
        .isEmail()
        .normalizeEmail()
        .withMessage('邮箱格式不正确'),
    
    body('username')
        .trim()
        .escape()
        .isLength({ min: 3, max: 20 })
        .matches(/^[a-zA-Z0-9_]+$/)
        .withMessage('用户名只能包含字母、数字和下划线'),
    
    body('password')
        .isLength({ min: 8 })
        .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/)
        .withMessage('密码必须包含大小写字母、数字和特殊字符'),
    
    body('age')
        .optional()
        .isInt({ min: 18, max: 100 })
        .toInt(),
    
    (req, res) => {
        const errors = validationResult(req);
        
        if (!errors.isEmpty()) {
            return res.status(400).json({
                errors: errors.array()
            });
        }
        
        res.json({
            message: '用户创建成功',
            user: req.body
        });
    }
);

app.listen(3000);
```

### 10.3 CSRF防护

```javascript
/**
 * CSRF防护
 * @author erik.zhou
 */

const express = require('express');
const csrf = require('csurf');
const cookieParser = require('cookie-parser');
const app = express();

app.use(cookieParser());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// CSRF保护
const csrfProtection = csrf({ cookie: true });

// 获取CSRF Token
app.get('/api/csrf-token', csrfProtection, (req, res) => {
    res.json({
        csrfToken: req.csrfToken()
    });
});

// 受保护的POST请求
app.post('/api/data', csrfProtection, (req, res) => {
    res.json({
        message: '数据提交成功',
        data: req.body
    });
});

// CSRF错误处理
app.use((err, req, res, next) => {
    if (err.code === 'EBADCSRFTOKEN') {
        res.status(403).json({
            error: 'CSRF令牌无效'
        });
    } else {
        next(err);
    }
});

app.listen(3000);
```

### 10.4 SQL注入防护

```javascript
/**
 * SQL注入防护
 * @author erik.zhou
 */

const express = require('express');
const mysql = require('mysql2/promise');
const app = express();

app.use(express.json());

// 创建数据库连接池
const pool = mysql.createPool({
    host: 'localhost',
    user: 'root',
    password: 'password',
    database: 'mydb',
    waitForConnections: true,
    connectionLimit: 10
});

// 错误示例：SQL注入漏洞
app.get('/users/unsafe/:id', async (req, res) => {
    try {
        const userId = req.params.id;
        // 危险：直接拼接SQL
        const sql = `SELECT * FROM users WHERE id = ${userId}`;
        const [rows] = await pool.query(sql);
        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// 正确示例：使用参数化查询
app.get('/users/safe/:id', async (req, res) => {
    try {
        const userId = req.params.id;
        // 安全：使用参数化查询
        const sql = 'SELECT * FROM users WHERE id = ?';
        const [rows] = await pool.query(sql, [userId]);
        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// 复杂查询示例
app.post('/users/search', async (req, res) => {
    try {
        const { name, email, age } = req.body;
        
        let sql = 'SELECT * FROM users WHERE 1=1';
        const params = [];
        
        if (name) {
            sql += ' AND name LIKE ?';
            params.push(`%${name}%`);
        }
        
        if (email) {
            sql += ' AND email = ?';
            params.push(email);
        }
        
        if (age) {
            sql += ' AND age = ?';
            params.push(age);
        }
        
        const [rows] = await pool.query(sql, params);
        res.json(rows);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

app.listen(3000);
```

### 10.5 密码加密

```javascript
/**
 * 密码加密
 * @author erik.zhou
 */

const express = require('express');
const bcrypt = require('bcrypt');
const app = express();

app.use(express.json());

const SALT_ROUNDS = 10;

// 用户注册
app.post('/register', async (req, res) => {
    try {
        const { username, password, email } = req.body;
        
        // 加密密码
        const hashedPassword = await bcrypt.hash(password, SALT_ROUNDS);
        
        // 保存用户（示例）
        const user = {
            username,
            email,
            password: hashedPassword
        };
        
        res.status(201).json({
            message: '注册成功',
            user: {
                username: user.username,
                email: user.email
            }
        });
    } catch (error) {
        res.status(500).json({
            error: '注册失败'
        });
    }
});

// 用户登录
app.post('/login', async (req, res) => {
    try {
        const { username, password } = req.body;
        
        // 查找用户（示例）
        const user = await findUserByUsername(username);
        
        if (!user) {
            return res.status(401).json({
                error: '用户名或密码错误'
            });
        }
        
        // 验证密码
        const isValid = await bcrypt.compare(password, user.password);
        
        if (!isValid) {
            return res.status(401).json({
                error: '用户名或密码错误'
            });
        }
        
        res.json({
            message: '登录成功',
            user: {
                username: user.username,
                email: user.email
            }
        });
    } catch (error) {
        res.status(500).json({
            error: '登录失败'
        });
    }
});

async function findUserByUsername(username) {
    // 模拟数据库查询
    return {
        username: 'admin',
        email: 'admin@example.com',
        password: '$2b$10$...' // 加密后的密码
    };
}

app.listen(3000);
```

---

## 总结

本教程全面介绍了Express框架的核心知识，包括：

1. Express基础 - 安装、配置、静态文件服务
2. 中间件系统 - 内置、第三方、自定义中间件
3. 路由管理 - 基本路由、参数、模块化、嵌套路由
4. 请求与响应 - 请求对象、响应对象、内容协商、流式响应
5. 模板引擎 - EJS、Pug、Handlebars
6. 错误处理 - 同步/异步错误、自定义错误类、错误日志
7. 数据验证 - Joi、express-validator、自定义验证器
8. 文件上传 - Multer、内存存储、文件下载
9. 会话管理 - Cookie、Session、Redis、JWT
10. 安全实践 - 安全中间件、输入验证、CSRF、SQL注入、密码加密

通过本教程的学习，你应该能够：
- 熟练使用Express构建Web应用
- 掌握中间件系统和路由管理
- 实现安全的用户认证和授权
- 处理文件上传和下载
- 遵循安全最佳实践

## 参考资源

- [Express官方文档](https://expressjs.com/)
- [Express最佳实践](https://expressjs.com/en/advanced/best-practice-security.html)
- [Node.js安全最佳实践](https://github.com/goldbergyoni/nodebestpractices#6-security-best-practices)

---

**作者**: erik.zhou  
**最后更新**: 2024年

