# RESTful API设计 - 完整教程

## 目录
1. [RESTful API基础](#1-restful-api基础)
2. [API设计原则](#2-api设计原则)
3. [资源与路由设计](#3-资源与路由设计)
4. [HTTP方法与状态码](#4-http方法与状态码)
5. [请求与响应格式](#5-请求与响应格式)
6. [版本控制](#6-版本控制)
7. [认证与授权](#7-认证与授权)
8. [错误处理](#8-错误处理)
9. [API文档生成](#9-api文档生成)
10. [最佳实践与安全](#10-最佳实践与安全)

---

## 1. RESTful API基础

### 1.1 REST架构风格

REST（Representational State Transfer）是一种软件架构风格。

```javascript
/**
 * REST核心概念
 * @author erik.zhou
 */

// 1. 资源（Resource）- 使用名词表示
// ✅ 正确
GET /api/users
GET /api/products
GET /api/orders

// ❌ 错误
GET /api/getUsers
GET /api/getAllProducts

// 2. 统一接口（Uniform Interface）
// 使用标准HTTP方法操作资源
GET    /api/users      // 获取用户列表
POST   /api/users      // 创建用户
GET    /api/users/:id  // 获取单个用户
PUT    /api/users/:id  // 更新用户
DELETE /api/users/:id  // 删除用户

// 3. 无状态（Stateless）
// 每个请求包含所有必要信息
const request = {
    method: 'GET',
    url: '/api/users/123',
    headers: {
        'Authorization': 'Bearer token123',
        'Content-Type': 'application/json'
    }
};

// 4. 可缓存（Cacheable）
// 响应明确标识是否可缓存
const response = {
    headers: {
        'Cache-Control': 'max-age=3600',
        'ETag': '"33a64df551425fcc55e4d42a148795d9f25f89d4"'
    }
};
```

### 1.2 RESTful API优势

```javascript
/**
 * RESTful API优势示例
 * @author erik.zhou
 */

// 1. 简单易懂
// URL即文档，一目了然
GET /api/users/123/orders/456

// 2. 可扩展性
// 易于添加新资源
GET /api/users/123/addresses
GET /api/users/123/payments

// 3. 平台无关
// 任何支持HTTP的客户端都可以使用
fetch('https://api.example.com/users')
    .then(response => response.json())
    .then(data => console.log(data));

// 4. 利用HTTP特性
// 缓存、认证、内容协商等
const headers = {
    'Accept': 'application/json',
    'Accept-Language': 'zh-CN',
    'If-None-Match': '"33a64df551425fcc55e4d42a148795d9f25f89d4"'
};
```

---

## 2. API设计原则

### 2.1 使用名词而非动词

```javascript
/**
 * 资源命名规范
 * @author erik.zhou
 */

// ✅ 正确 - 使用名词
GET    /api/users
POST   /api/users
GET    /api/products
DELETE /api/orders/123

// ❌ 错误 - 使用动词
GET    /api/getUsers
POST   /api/createUser
GET    /api/fetchProducts
DELETE /api/deleteOrder/123

// ✅ 正确 - 资源层级关系
GET /api/users/123/orders          // 获取用户的订单
GET /api/orders/456/items          // 获取订单的商品
GET /api/categories/789/products   // 获取分类下的产品

// ❌ 错误 - 过深的嵌套
GET /api/users/123/orders/456/items/789/details/abc
```

### 2.2 使用复数形式

```javascript
/**
 * 资源复数命名
 * @author erik.zhou
 */

// ✅ 正确 - 统一使用复数
GET /api/users
GET /api/users/123
GET /api/products
GET /api/products/456

// ❌ 错误 - 单复数混用
GET /api/user
GET /api/user/123
GET /api/product
GET /api/products/456

// 特殊情况：不可数名词
GET /api/information
GET /api/data
GET /api/metadata
```

### 2.3 使用连字符分隔

```javascript
/**
 * URL命名规范
 * @author erik.zhou
 */

// ✅ 正确 - 使用连字符
GET /api/user-profiles
GET /api/order-items
GET /api/product-categories

// ❌ 错误 - 使用下划线或驼峰
GET /api/user_profiles
GET /api/userProfiles
GET /api/OrderItems

// ✅ 正确 - 查询参数使用驼峰或下划线
GET /api/users?sortBy=createdAt&orderBy=desc
GET /api/products?category_id=123&min_price=100
```

### 2.4 版本化API

```javascript
/**
 * API版本控制
 * @author erik.zhou
 */

// 方式1：URL路径版本
GET /api/v1/users
GET /api/v2/users

// 方式2：请求头版本
GET /api/users
Headers: {
    'Accept': 'application/vnd.example.v1+json'
}

// 方式3：查询参数版本（不推荐）
GET /api/users?version=1

// Express实现路径版本
const express = require('express');
const app = express();

// V1 API
app.get('/api/v1/users', (req, res) => {
    res.json({
        version: 'v1',
        users: []
    });
});

// V2 API
app.get('/api/v2/users', (req, res) => {
    res.json({
        version: 'v2',
        users: [],
        metadata: {
            total: 0,
            page: 1
        }
    });
});
```

---

## 3. 资源与路由设计

### 3.1 基础CRUD路由

```javascript
/**
 * 标准CRUD路由设计
 * @author erik.zhou
 */

const express = require('express');
const router = express.Router();

// 1. 获取资源列表
router.get('/users', async (req, res) => {
    try {
        const { page = 1, limit = 10, sort = 'createdAt' } = req.query;
        
        const users = await User.find()
            .limit(limit * 1)
            .skip((page - 1) * limit)
            .sort({ [sort]: -1 });
        
        const count = await User.countDocuments();
        
        res.json({
            success: true,
            data: users,
            pagination: {
                total: count,
                page: parseInt(page),
                pages: Math.ceil(count / limit)
            }
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: error.message
        });
    }
});

// 2. 获取单个资源
router.get('/users/:id', async (req, res) => {
    try {
        const user = await User.findById(req.params.id);
        
        if (!user) {
            return res.status(404).json({
                success: false,
                message: 'User not found'
            });
        }
        
        res.json({
            success: true,
            data: user
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: error.message
        });
    }
});

// 3. 创建资源
router.post('/users', async (req, res) => {
    try {
        const { username, email, password } = req.body;
        
        // 验证必填字段
        if (!username || !email || !password) {
            return res.status(400).json({
                success: false,
                message: 'Missing required fields'
            });
        }
        
        const user = new User({
            username,
            email,
            password
        });
        
        await user.save();
        
        res.status(201).json({
            success: true,
            data: user,
            message: 'User created successfully'
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: error.message
        });
    }
});

// 4. 更新资源（完整更新）
router.put('/users/:id', async (req, res) => {
    try {
        const { username, email } = req.body;
        
        const user = await User.findByIdAndUpdate(
            req.params.id,
            { username, email },
            { new: true, runValidators: true }
        );
        
        if (!user) {
            return res.status(404).json({
                success: false,
                message: 'User not found'
            });
        }
        
        res.json({
            success: true,
            data: user,
            message: 'User updated successfully'
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: error.message
        });
    }
});

// 5. 部分更新资源
router.patch('/users/:id', async (req, res) => {
    try {
        const updates = req.body;
        
        const user = await User.findByIdAndUpdate(
            req.params.id,
            { $set: updates },
            { new: true, runValidators: true }
        );
        
        if (!user) {
            return res.status(404).json({
                success: false,
                message: 'User not found'
            });
        }
        
        res.json({
            success: true,
            data: user,
            message: 'User updated successfully'
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: error.message
        });
    }
});

// 6. 删除资源
router.delete('/users/:id', async (req, res) => {
    try {
        const user = await User.findByIdAndDelete(req.params.id);
        
        if (!user) {
            return res.status(404).json({
                success: false,
                message: 'User not found'
            });
        }
        
        res.json({
            success: true,
            message: 'User deleted successfully'
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: error.message
        });
    }
});

module.exports = router;
```

### 3.2 嵌套资源路由

```javascript
/**
 * 嵌套资源路由设计
 * @author erik.zhou
 */

const express = require('express');
const router = express.Router();

// 1. 用户的订单
router.get('/users/:userId/orders', async (req, res) => {
    try {
        const orders = await Order.find({ userId: req.params.userId });
        
        res.json({
            success: true,
            data: orders
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: error.message
        });
    }
});

// 2. 用户的特定订单
router.get('/users/:userId/orders/:orderId', async (req, res) => {
    try {
        const order = await Order.findOne({
            _id: req.params.orderId,
            userId: req.params.userId
        });
        
        if (!order) {
            return res.status(404).json({
                success: false,
                message: 'Order not found'
            });
        }
        
        res.json({
            success: true,
            data: order
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: error.message
        });
    }
});

// 3. 创建用户订单
router.post('/users/:userId/orders', async (req, res) => {
    try {
        const order = new Order({
            userId: req.params.userId,
            ...req.body
        });
        
        await order.save();
        
        res.status(201).json({
            success: true,
            data: order
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: error.message
        });
    }
});

// 4. 订单的商品
router.get('/orders/:orderId/items', async (req, res) => {
    try {
        const items = await OrderItem.find({ orderId: req.params.orderId });
        
        res.json({
            success: true,
            data: items
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: error.message
        });
    }
});

module.exports = router;
```

### 3.3 过滤、排序、分页

```javascript
/**
 * 查询参数处理
 * @author erik.zhou
 */

const express = require('express');
const router = express.Router();

// 高级查询接口
router.get('/products', async (req, res) => {
    try {
        const {
            // 分页参数
            page = 1,
            limit = 10,
            
            // 排序参数
            sortBy = 'createdAt',
            order = 'desc',
            
            // 过滤参数
            category,
            minPrice,
            maxPrice,
            inStock,
            
            // 搜索参数
            search,
            
            // 字段选择
            fields
        } = req.query;
        
        // 构建查询条件
        const query = {};
        
        if (category) {
            query.category = category;
        }
        
        if (minPrice || maxPrice) {
            query.price = {};
            if (minPrice) query.price.$gte = parseFloat(minPrice);
            if (maxPrice) query.price.$lte = parseFloat(maxPrice);
        }
        
        if (inStock !== undefined) {
            query.inStock = inStock === 'true';
        }
        
        if (search) {
            query.$or = [
                { name: { $regex: search, $options: 'i' } },
                { description: { $regex: search, $options: 'i' } }
            ];
        }
        
        // 构建排序
        const sortOptions = {};
        sortOptions[sortBy] = order === 'asc' ? 1 : -1;
        
        // 构建字段选择
        const selectFields = fields ? fields.split(',').join(' ') : '';
        
        // 执行查询
        const products = await Product.find(query)
            .select(selectFields)
            .sort(sortOptions)
            .limit(limit * 1)
            .skip((page - 1) * limit)
            .lean();
        
        const count = await Product.countDocuments(query);
        
        res.json({
            success: true,
            data: products,
            pagination: {
                total: count,
                page: parseInt(page),
                limit: parseInt(limit),
                pages: Math.ceil(count / limit)
            }
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: error.message
        });
    }
});

// 使用示例
// GET /api/products?category=electronics&minPrice=100&maxPrice=1000&sortBy=price&order=asc&page=1&limit=20&fields=name,price,image

module.exports = router;
```



---

## 4. HTTP方法与状态码

### 4.1 HTTP方法详解

```javascript
/**
 * HTTP方法使用规范
 * @author erik.zhou
 */

// 1. GET - 获取资源（幂等、安全）
// 特点：可缓存、可书签、参数在URL中
router.get('/users/:id', async (req, res) => {
    const user = await User.findById(req.params.id);
    res.json({ data: user });
});

// 2. POST - 创建资源（非幂等）
// 特点：不可缓存、参数在请求体中
router.post('/users', async (req, res) => {
    const user = await User.create(req.body);
    res.status(201).json({ data: user });
});

// 3. PUT - 完整更新资源（幂等）
// 特点：需要提供完整的资源数据
router.put('/users/:id', async (req, res) => {
    const user = await User.findByIdAndUpdate(
        req.params.id,
        req.body,
        { new: true, overwrite: true }
    );
    res.json({ data: user });
});

// 4. PATCH - 部分更新资源（幂等）
// 特点：只需提供要更新的字段
router.patch('/users/:id', async (req, res) => {
    const user = await User.findByIdAndUpdate(
        req.params.id,
        { $set: req.body },
        { new: true }
    );
    res.json({ data: user });
});

// 5. DELETE - 删除资源（幂等）
router.delete('/users/:id', async (req, res) => {
    await User.findByIdAndDelete(req.params.id);
    res.status(204).send();
});

// 6. HEAD - 获取资源元信息
router.head('/users/:id', async (req, res) => {
    const exists = await User.exists({ _id: req.params.id });
    res.status(exists ? 200 : 404).end();
});

// 7. OPTIONS - 获取资源支持的方法
router.options('/users', (req, res) => {
    res.set('Allow', 'GET, POST, PUT, PATCH, DELETE, OPTIONS');
    res.status(200).end();
});
```

### 4.2 HTTP状态码规范

```javascript
/**
 * HTTP状态码使用规范
 * @author erik.zhou
 */

const express = require('express');
const router = express.Router();

// 2xx 成功
router.get('/success-examples', (req, res) => {
    // 200 OK - 请求成功
    res.status(200).json({ message: 'Success' });
    
    // 201 Created - 资源创建成功
    res.status(201).json({ message: 'Resource created' });
    
    // 204 No Content - 请求成功但无返回内容
    res.status(204).send();
});

// 3xx 重定向
router.get('/redirect-examples', (req, res) => {
    // 301 Moved Permanently - 永久重定向
    res.redirect(301, '/new-url');
    
    // 302 Found - 临时重定向
    res.redirect(302, '/temp-url');
    
    // 304 Not Modified - 资源未修改（缓存）
    res.status(304).end();
});

// 4xx 客户端错误
router.get('/client-error-examples', (req, res) => {
    // 400 Bad Request - 请求参数错误
    if (!req.query.id) {
        return res.status(400).json({
            error: 'Bad Request',
            message: 'Missing required parameter: id'
        });
    }
    
    // 401 Unauthorized - 未认证
    if (!req.headers.authorization) {
        return res.status(401).json({
            error: 'Unauthorized',
            message: 'Authentication required'
        });
    }
    
    // 403 Forbidden - 无权限
    if (!req.user.isAdmin) {
        return res.status(403).json({
            error: 'Forbidden',
            message: 'Insufficient permissions'
        });
    }
    
    // 404 Not Found - 资源不存在
    const user = await User.findById(req.params.id);
    if (!user) {
        return res.status(404).json({
            error: 'Not Found',
            message: 'User not found'
        });
    }
    
    // 409 Conflict - 资源冲突
    const existingUser = await User.findOne({ email: req.body.email });
    if (existingUser) {
        return res.status(409).json({
            error: 'Conflict',
            message: 'Email already exists'
        });
    }
    
    // 422 Unprocessable Entity - 验证失败
    if (!isValidEmail(req.body.email)) {
        return res.status(422).json({
            error: 'Unprocessable Entity',
            message: 'Invalid email format',
            errors: {
                email: 'Must be a valid email address'
            }
        });
    }
    
    // 429 Too Many Requests - 请求过多
    if (rateLimitExceeded) {
        return res.status(429).json({
            error: 'Too Many Requests',
            message: 'Rate limit exceeded',
            retryAfter: 60
        });
    }
});

// 5xx 服务器错误
router.get('/server-error-examples', (req, res) => {
    // 500 Internal Server Error - 服务器内部错误
    try {
        // 业务逻辑
    } catch (error) {
        return res.status(500).json({
            error: 'Internal Server Error',
            message: 'An unexpected error occurred'
        });
    }
    
    // 502 Bad Gateway - 网关错误
    // 503 Service Unavailable - 服务不可用
    if (isMaintenanceMode) {
        return res.status(503).json({
            error: 'Service Unavailable',
            message: 'Service is under maintenance'
        });
    }
});

module.exports = router;
```

### 4.3 状态码最佳实践

```javascript
/**
 * 状态码使用最佳实践
 * @author erik.zhou
 */

const express = require('express');
const router = express.Router();

// 统一的响应格式
class ApiResponse {
    static success(data, message = 'Success', statusCode = 200) {
        return {
            success: true,
            statusCode,
            message,
            data,
            timestamp: new Date().toISOString()
        };
    }
    
    static error(message, statusCode = 500, errors = null) {
        return {
            success: false,
            statusCode,
            message,
            errors,
            timestamp: new Date().toISOString()
        };
    }
}

// 使用示例
router.get('/users/:id', async (req, res) => {
    try {
        const user = await User.findById(req.params.id);
        
        if (!user) {
            return res.status(404).json(
                ApiResponse.error('User not found', 404)
            );
        }
        
        res.status(200).json(
            ApiResponse.success(user, 'User retrieved successfully')
        );
    } catch (error) {
        res.status(500).json(
            ApiResponse.error('Internal server error', 500)
        );
    }
});

router.post('/users', async (req, res) => {
    try {
        const { username, email, password } = req.body;
        
        // 验证
        const errors = {};
        if (!username) errors.username = 'Username is required';
        if (!email) errors.email = 'Email is required';
        if (!password) errors.password = 'Password is required';
        
        if (Object.keys(errors).length > 0) {
            return res.status(422).json(
                ApiResponse.error('Validation failed', 422, errors)
            );
        }
        
        // 检查冲突
        const existingUser = await User.findOne({ email });
        if (existingUser) {
            return res.status(409).json(
                ApiResponse.error('Email already exists', 409)
            );
        }
        
        // 创建用户
        const user = await User.create({ username, email, password });
        
        res.status(201).json(
            ApiResponse.success(user, 'User created successfully', 201)
        );
    } catch (error) {
        res.status(500).json(
            ApiResponse.error('Internal server error', 500)
        );
    }
});

module.exports = router;
```

---

## 5. 请求与响应格式

### 5.1 请求格式规范

```javascript
/**
 * 请求格式规范
 * @author erik.zhou
 */

// 1. JSON请求体
const createUserRequest = {
    method: 'POST',
    url: '/api/users',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({
        username: 'john_doe',
        email: 'john@example.com',
        password: 'securePassword123',
        profile: {
            firstName: 'John',
            lastName: 'Doe',
            age: 30
        }
    })
};

// 2. 查询参数
const getUsersRequest = {
    method: 'GET',
    url: '/api/users?page=1&limit=10&sortBy=createdAt&order=desc&status=active'
};

// 3. 路径参数
const getUserRequest = {
    method: 'GET',
    url: '/api/users/123'
};

// 4. 请求头
const authenticatedRequest = {
    method: 'GET',
    url: '/api/users/profile',
    headers: {
        'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',
        'Accept': 'application/json',
        'Accept-Language': 'zh-CN',
        'User-Agent': 'MyApp/1.0.0'
    }
};

// 5. 文件上传
const uploadFileRequest = {
    method: 'POST',
    url: '/api/users/123/avatar',
    headers: {
        'Content-Type': 'multipart/form-data'
    },
    body: formData // FormData对象
};
```

### 5.2 响应格式规范

```javascript
/**
 * 响应格式规范
 * @author erik.zhou
 */

const express = require('express');
const router = express.Router();

// 1. 成功响应 - 单个资源
router.get('/users/:id', async (req, res) => {
    const user = await User.findById(req.params.id);
    
    res.json({
        success: true,
        data: {
            id: user._id,
            username: user.username,
            email: user.email,
            profile: {
                firstName: user.firstName,
                lastName: user.lastName,
                avatar: user.avatar
            },
            createdAt: user.createdAt,
            updatedAt: user.updatedAt
        },
        message: 'User retrieved successfully',
        timestamp: new Date().toISOString()
    });
});

// 2. 成功响应 - 资源列表
router.get('/users', async (req, res) => {
    const { page = 1, limit = 10 } = req.query;
    
    const users = await User.find()
        .limit(limit * 1)
        .skip((page - 1) * limit);
    
    const count = await User.countDocuments();
    
    res.json({
        success: true,
        data: users,
        pagination: {
            total: count,
            page: parseInt(page),
            limit: parseInt(limit),
            pages: Math.ceil(count / limit),
            hasNext: page * limit < count,
            hasPrev: page > 1
        },
        message: 'Users retrieved successfully',
        timestamp: new Date().toISOString()
    });
});

// 3. 错误响应
router.post('/users', async (req, res) => {
    try {
        // 验证失败
        const errors = validateUser(req.body);
        if (errors.length > 0) {
            return res.status(422).json({
                success: false,
                error: {
                    code: 'VALIDATION_ERROR',
                    message: 'Validation failed',
                    details: errors
                },
                timestamp: new Date().toISOString()
            });
        }
        
        // 业务逻辑错误
        const existingUser = await User.findOne({ email: req.body.email });
        if (existingUser) {
            return res.status(409).json({
                success: false,
                error: {
                    code: 'DUPLICATE_EMAIL',
                    message: 'Email already exists',
                    field: 'email'
                },
                timestamp: new Date().toISOString()
            });
        }
        
        const user = await User.create(req.body);
        res.status(201).json({
            success: true,
            data: user,
            message: 'User created successfully',
            timestamp: new Date().toISOString()
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            error: {
                code: 'INTERNAL_ERROR',
                message: 'An unexpected error occurred',
                details: process.env.NODE_ENV === 'development' ? error.message : undefined
            },
            timestamp: new Date().toISOString()
        });
    }
});

// 4. 空响应
router.delete('/users/:id', async (req, res) => {
    await User.findByIdAndDelete(req.params.id);
    res.status(204).send();
});

module.exports = router;
```

### 5.3 HATEOAS（超媒体）

```javascript
/**
 * HATEOAS实现
 * @author erik.zhou
 */

const express = require('express');
const router = express.Router();

// 带链接的响应
router.get('/users/:id', async (req, res) => {
    const user = await User.findById(req.params.id);
    
    res.json({
        success: true,
        data: {
            id: user._id,
            username: user.username,
            email: user.email,
            // HATEOAS链接
            _links: {
                self: {
                    href: `/api/users/${user._id}`,
                    method: 'GET'
                },
                update: {
                    href: `/api/users/${user._id}`,
                    method: 'PUT'
                },
                delete: {
                    href: `/api/users/${user._id}`,
                    method: 'DELETE'
                },
                orders: {
                    href: `/api/users/${user._id}/orders`,
                    method: 'GET'
                },
                addresses: {
                    href: `/api/users/${user._id}/addresses`,
                    method: 'GET'
                }
            }
        }
    });
});

// 列表响应带链接
router.get('/users', async (req, res) => {
    const { page = 1, limit = 10 } = req.query;
    
    const users = await User.find()
        .limit(limit * 1)
        .skip((page - 1) * limit);
    
    const count = await User.countDocuments();
    const totalPages = Math.ceil(count / limit);
    
    res.json({
        success: true,
        data: users.map(user => ({
            id: user._id,
            username: user.username,
            email: user.email,
            _links: {
                self: {
                    href: `/api/users/${user._id}`,
                    method: 'GET'
                }
            }
        })),
        pagination: {
            total: count,
            page: parseInt(page),
            limit: parseInt(limit),
            pages: totalPages
        },
        _links: {
            self: {
                href: `/api/users?page=${page}&limit=${limit}`,
                method: 'GET'
            },
            first: {
                href: `/api/users?page=1&limit=${limit}`,
                method: 'GET'
            },
            last: {
                href: `/api/users?page=${totalPages}&limit=${limit}`,
                method: 'GET'
            },
            ...(page > 1 && {
                prev: {
                    href: `/api/users?page=${page - 1}&limit=${limit}`,
                    method: 'GET'
                }
            }),
            ...(page < totalPages && {
                next: {
                    href: `/api/users?page=${parseInt(page) + 1}&limit=${limit}`,
                    method: 'GET'
                }
            })
        }
    });
});

module.exports = router;
```



---

## 6. 版本控制

### 6.1 URL路径版本控制

```javascript
/**
 * URL路径版本控制
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// V1 API
const v1Router = express.Router();

v1Router.get('/users', (req, res) => {
    res.json({
        version: 'v1',
        data: [
            { id: 1, name: 'John' }
        ]
    });
});

v1Router.get('/users/:id', (req, res) => {
    res.json({
        version: 'v1',
        data: { id: req.params.id, name: 'John' }
    });
});

app.use('/api/v1', v1Router);

// V2 API - 增强版本
const v2Router = express.Router();

v2Router.get('/users', (req, res) => {
    res.json({
        version: 'v2',
        data: [
            {
                id: 1,
                username: 'john_doe',
                email: 'john@example.com',
                profile: {
                    firstName: 'John',
                    lastName: 'Doe'
                }
            }
        ],
        metadata: {
            total: 1,
            page: 1,
            limit: 10
        }
    });
});

v2Router.get('/users/:id', (req, res) => {
    res.json({
        version: 'v2',
        data: {
            id: req.params.id,
            username: 'john_doe',
            email: 'john@example.com',
            profile: {
                firstName: 'John',
                lastName: 'Doe'
            }
        }
    });
});

app.use('/api/v2', v2Router);

app.listen(3000);
```

### 6.2 请求头版本控制

```javascript
/**
 * 请求头版本控制
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 版本解析中间件
const versionMiddleware = (req, res, next) => {
    const acceptHeader = req.headers.accept || '';
    
    // 解析版本号
    // Accept: application/vnd.example.v1+json
    const versionMatch = acceptHeader.match(/vnd\.example\.v(\d+)\+json/);
    
    if (versionMatch) {
        req.apiVersion = parseInt(versionMatch[1]);
    } else {
        req.apiVersion = 1; // 默认版本
    }
    
    next();
};

app.use(versionMiddleware);

// 根据版本返回不同数据
app.get('/api/users', (req, res) => {
    if (req.apiVersion === 1) {
        // V1响应
        res.json({
            version: 'v1',
            data: [
                { id: 1, name: 'John' }
            ]
        });
    } else if (req.apiVersion === 2) {
        // V2响应
        res.json({
            version: 'v2',
            data: [
                {
                    id: 1,
                    username: 'john_doe',
                    email: 'john@example.com'
                }
            ],
            metadata: {
                total: 1
            }
        });
    } else {
        res.status(400).json({
            error: 'Unsupported API version'
        });
    }
});

app.listen(3000);
```

### 6.3 版本迁移策略

```javascript
/**
 * 版本迁移策略
 * @author erik.zhou
 */

const express = require('express');
const app = express();

// 版本配置
const API_VERSIONS = {
    v1: {
        deprecated: true,
        sunsetDate: '2024-12-31',
        message: 'API v1 is deprecated. Please migrate to v2.'
    },
    v2: {
        deprecated: false,
        current: true
    },
    v3: {
        beta: true,
        message: 'API v3 is in beta. Use with caution.'
    }
};

// 版本检查中间件
const checkVersion = (version) => {
    return (req, res, next) => {
        const versionInfo = API_VERSIONS[version];
        
        if (!versionInfo) {
            return res.status(400).json({
                error: 'Invalid API version'
            });
        }
        
        // 添加版本信息到响应头
        if (versionInfo.deprecated) {
            res.set('X-API-Deprecated', 'true');
            res.set('X-API-Sunset-Date', versionInfo.sunsetDate);
            res.set('X-API-Deprecation-Message', versionInfo.message);
        }
        
        if (versionInfo.beta) {
            res.set('X-API-Beta', 'true');
            res.set('X-API-Beta-Message', versionInfo.message);
        }
        
        next();
    };
};

// V1 API（已废弃）
app.use('/api/v1', checkVersion('v1'), (req, res) => {
    res.json({
        version: 'v1',
        warning: 'This API version is deprecated',
        data: []
    });
});

// V2 API（当前版本）
app.use('/api/v2', checkVersion('v2'), (req, res) => {
    res.json({
        version: 'v2',
        data: []
    });
});

// V3 API（测试版）
app.use('/api/v3', checkVersion('v3'), (req, res) => {
    res.json({
        version: 'v3',
        beta: true,
        data: []
    });
});

app.listen(3000);
```

### 6.4 向后兼容处理

```javascript
/**
 * 向后兼容处理
 * @author erik.zhou
 */

const express = require('express');
const router = express.Router();

// 数据转换器
class DataTransformer {
    // V1格式转V2格式
    static v1ToV2(v1Data) {
        return {
            id: v1Data.id,
            username: v1Data.name,
            email: v1Data.email || `${v1Data.name}@example.com`,
            profile: {
                firstName: v1Data.name.split(' ')[0],
                lastName: v1Data.name.split(' ')[1] || ''
            },
            createdAt: v1Data.created_at,
            updatedAt: v1Data.updated_at
        };
    }
    
    // V2格式转V1格式
    static v2ToV1(v2Data) {
        return {
            id: v2Data.id,
            name: `${v2Data.profile.firstName} ${v2Data.profile.lastName}`,
            email: v2Data.email,
            created_at: v2Data.createdAt,
            updated_at: v2Data.updatedAt
        };
    }
}

// V1 API - 使用转换器
router.get('/v1/users/:id', async (req, res) => {
    // 从数据库获取V2格式数据
    const user = await User.findById(req.params.id);
    
    // 转换为V1格式
    const v1User = DataTransformer.v2ToV1(user);
    
    res.json({
        version: 'v1',
        data: v1User
    });
});

// V2 API - 原生格式
router.get('/v2/users/:id', async (req, res) => {
    const user = await User.findById(req.params.id);
    
    res.json({
        version: 'v2',
        data: user
    });
});

// 接受V1格式创建，存储为V2格式
router.post('/v1/users', async (req, res) => {
    // 转换V1请求为V2格式
    const v2Data = DataTransformer.v1ToV2(req.body);
    
    // 存储V2格式
    const user = await User.create(v2Data);
    
    // 返回V1格式
    const v1User = DataTransformer.v2ToV1(user);
    
    res.status(201).json({
        version: 'v1',
        data: v1User
    });
});

module.exports = router;
```

---

## 7. 认证与授权

### 7.1 JWT认证

```javascript
/**
 * JWT认证实现
 * @author erik.zhou
 */

const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');
const router = express.Router();

const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key';
const JWT_EXPIRES_IN = '7d';

// 1. 用户注册
router.post('/auth/register', async (req, res) => {
    try {
        const { username, email, password } = req.body;
        
        // 验证
        if (!username || !email || !password) {
            return res.status(400).json({
                success: false,
                message: 'Missing required fields'
            });
        }
        
        // 检查用户是否存在
        const existingUser = await User.findOne({ email });
        if (existingUser) {
            return res.status(409).json({
                success: false,
                message: 'Email already exists'
            });
        }
        
        // 加密密码
        const hashedPassword = await bcrypt.hash(password, 10);
        
        // 创建用户
        const user = await User.create({
            username,
            email,
            password: hashedPassword
        });
        
        // 生成JWT
        const token = jwt.sign(
            {
                userId: user._id,
                email: user.email,
                role: user.role
            },
            JWT_SECRET,
            { expiresIn: JWT_EXPIRES_IN }
        );
        
        res.status(201).json({
            success: true,
            data: {
                user: {
                    id: user._id,
                    username: user.username,
                    email: user.email
                },
                token,
                expiresIn: JWT_EXPIRES_IN
            },
            message: 'User registered successfully'
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: error.message
        });
    }
});

// 2. 用户登录
router.post('/auth/login', async (req, res) => {
    try {
        const { email, password } = req.body;
        
        // 验证
        if (!email || !password) {
            return res.status(400).json({
                success: false,
                message: 'Email and password are required'
            });
        }
        
        // 查找用户
        const user = await User.findOne({ email }).select('+password');
        if (!user) {
            return res.status(401).json({
                success: false,
                message: 'Invalid credentials'
            });
        }
        
        // 验证密码
        const isPasswordValid = await bcrypt.compare(password, user.password);
        if (!isPasswordValid) {
            return res.status(401).json({
                success: false,
                message: 'Invalid credentials'
            });
        }
        
        // 生成JWT
        const token = jwt.sign(
            {
                userId: user._id,
                email: user.email,
                role: user.role
            },
            JWT_SECRET,
            { expiresIn: JWT_EXPIRES_IN }
        );
        
        res.json({
            success: true,
            data: {
                user: {
                    id: user._id,
                    username: user.username,
                    email: user.email,
                    role: user.role
                },
                token,
                expiresIn: JWT_EXPIRES_IN
            },
            message: 'Login successful'
        });
    } catch (error) {
        res.status(500).json({
            success: false,
            message: error.message
        });
    }
});

// 3. 刷新Token
router.post('/auth/refresh', async (req, res) => {
    try {
        const { token } = req.body;
        
        if (!token) {
            return res.status(400).json({
                success: false,
                message: 'Token is required'
            });
        }
        
        // 验证旧token
        const decoded = jwt.verify(token, JWT_SECRET, {
            ignoreExpiration: true
        });
        
        // 生成新token
        const newToken = jwt.sign(
            {
                userId: decoded.userId,
                email: decoded.email,
                role: decoded.role
            },
            JWT_SECRET,
            { expiresIn: JWT_EXPIRES_IN }
        );
        
        res.json({
            success: true,
            data: {
                token: newToken,
                expiresIn: JWT_EXPIRES_IN
            },
            message: 'Token refreshed successfully'
        });
    } catch (error) {
        res.status(401).json({
            success: false,
            message: 'Invalid token'
        });
    }
});

module.exports = router;
```

### 7.2 认证中间件

```javascript
/**
 * 认证中间件
 * @author erik.zhou
 */

const jwt = require('jsonwebtoken');

const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key';

// 认证中间件
const authenticate = async (req, res, next) => {
    try {
        // 获取token
        const authHeader = req.headers.authorization;
        
        if (!authHeader || !authHeader.startsWith('Bearer ')) {
            return res.status(401).json({
                success: false,
                message: 'Authentication required'
            });
        }
        
        const token = authHeader.substring(7);
        
        // 验证token
        const decoded = jwt.verify(token, JWT_SECRET);
        
        // 查找用户
        const user = await User.findById(decoded.userId);
        
        if (!user) {
            return res.status(401).json({
                success: false,
                message: 'User not found'
            });
        }
        
        // 将用户信息附加到请求对象
        req.user = user;
        req.userId = user._id;
        
        next();
    } catch (error) {
        if (error.name === 'TokenExpiredError') {
            return res.status(401).json({
                success: false,
                message: 'Token expired'
            });
        }
        
        if (error.name === 'JsonWebTokenError') {
            return res.status(401).json({
                success: false,
                message: 'Invalid token'
            });
        }
        
        res.status(500).json({
            success: false,
            message: 'Authentication failed'
        });
    }
};

// 可选认证中间件
const optionalAuthenticate = async (req, res, next) => {
    try {
        const authHeader = req.headers.authorization;
        
        if (authHeader && authHeader.startsWith('Bearer ')) {
            const token = authHeader.substring(7);
            const decoded = jwt.verify(token, JWT_SECRET);
            const user = await User.findById(decoded.userId);
            
            if (user) {
                req.user = user;
                req.userId = user._id;
            }
        }
        
        next();
    } catch (error) {
        // 可选认证失败不阻止请求
        next();
    }
};

module.exports = {
    authenticate,
    optionalAuthenticate
};
```

### 7.3 授权中间件

```javascript
/**
 * 授权中间件
 * @author erik.zhou
 */

// 角色检查中间件
const authorize = (...roles) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({
                success: false,
                message: 'Authentication required'
            });
        }
        
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({
                success: false,
                message: 'Insufficient permissions'
            });
        }
        
        next();
    };
};

// 权限检查中间件
const checkPermission = (permission) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({
                success: false,
                message: 'Authentication required'
            });
        }
        
        if (!req.user.permissions || !req.user.permissions.includes(permission)) {
            return res.status(403).json({
                success: false,
                message: `Permission denied: ${permission}`
            });
        }
        
        next();
    };
};

// 资源所有权检查
const checkOwnership = (resourceModel, resourceIdParam = 'id') => {
    return async (req, res, next) => {
        try {
            const resourceId = req.params[resourceIdParam];
            const resource = await resourceModel.findById(resourceId);
            
            if (!resource) {
                return res.status(404).json({
                    success: false,
                    message: 'Resource not found'
                });
            }
            
            // 检查是否是资源所有者或管理员
            if (
                resource.userId.toString() !== req.userId.toString() &&
                req.user.role !== 'admin'
            ) {
                return res.status(403).json({
                    success: false,
                    message: 'Access denied'
                });
            }
            
            req.resource = resource;
            next();
        } catch (error) {
            res.status(500).json({
                success: false,
                message: error.message
            });
        }
    };
};

// 使用示例
const express = require('express');
const router = express.Router();
const { authenticate } = require('./auth.middleware');

// 只有管理员可以访问
router.get('/admin/users',
    authenticate,
    authorize('admin'),
    async (req, res) => {
        const users = await User.find();
        res.json({ data: users });
    }
);

// 管理员和编辑可以访问
router.post('/posts',
    authenticate,
    authorize('admin', 'editor'),
    async (req, res) => {
        const post = await Post.create(req.body);
        res.status(201).json({ data: post });
    }
);

// 需要特定权限
router.delete('/posts/:id',
    authenticate,
    checkPermission('posts:delete'),
    async (req, res) => {
        await Post.findByIdAndDelete(req.params.id);
        res.status(204).send();
    }
);

// 只有资源所有者可以修改
router.put('/posts/:id',
    authenticate,
    checkOwnership(Post, 'id'),
    async (req, res) => {
        const post = await Post.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true }
        );
        res.json({ data: post });
    }
);

module.exports = {
    authorize,
    checkPermission,
    checkOwnership
};
```



---

## 8. 错误处理

### 8.1 统一错误处理

```javascript
/**
 * 统一错误处理
 * @author erik.zhou
 */

// 自定义错误类
class ApiError extends Error {
    constructor(statusCode, message, errors = null) {
        super(message);
        this.statusCode = statusCode;
        this.errors = errors;
        this.isOperational = true;
        Error.captureStackTrace(this, this.constructor);
    }
}

// 错误类型
class BadRequestError extends ApiError {
    constructor(message = 'Bad Request', errors = null) {
        super(400, message, errors);
    }
}

class UnauthorizedError extends ApiError {
    constructor(message = 'Unauthorized') {
        super(401, message);
    }
}

class ForbiddenError extends ApiError {
    constructor(message = 'Forbidden') {
        super(403, message);
    }
}

class NotFoundError extends ApiError {
    constructor(message = 'Resource not found') {
        super(404, message);
    }
}

class ConflictError extends ApiError {
    constructor(message = 'Resource conflict') {
        super(409, message);
    }
}

class ValidationError extends ApiError {
    constructor(errors) {
        super(422, 'Validation failed', errors);
    }
}

class InternalServerError extends ApiError {
    constructor(message = 'Internal server error') {
        super(500, message);
    }
}

// 错误处理中间件
const errorHandler = (err, req, res, next) => {
    // 默认错误
    let error = err;
    
    // Mongoose验证错误
    if (err.name === 'ValidationError') {
        const errors = Object.values(err.errors).map(e => ({
            field: e.path,
            message: e.message
        }));
        error = new ValidationError(errors);
    }
    
    // Mongoose重复键错误
    if (err.code === 11000) {
        const field = Object.keys(err.keyPattern)[0];
        error = new ConflictError(`${field} already exists`);
    }
    
    // Mongoose CastError
    if (err.name === 'CastError') {
        error = new BadRequestError(`Invalid ${err.path}: ${err.value}`);
    }
    
    // JWT错误
    if (err.name === 'JsonWebTokenError') {
        error = new UnauthorizedError('Invalid token');
    }
    
    if (err.name === 'TokenExpiredError') {
        error = new UnauthorizedError('Token expired');
    }
    
    // 响应错误
    const statusCode = error.statusCode || 500;
    const message = error.message || 'Internal server error';
    
    res.status(statusCode).json({
        success: false,
        error: {
            statusCode,
            message,
            ...(error.errors && { errors: error.errors }),
            ...(process.env.NODE_ENV === 'development' && {
                stack: error.stack
            })
        },
        timestamp: new Date().toISOString()
    });
    
    // 记录错误日志
    if (!error.isOperational) {
        console.error('UNHANDLED ERROR:', error);
    }
};

// 404处理
const notFoundHandler = (req, res, next) => {
    const error = new NotFoundError(`Route ${req.originalUrl} not found`);
    next(error);
};

module.exports = {
    ApiError,
    BadRequestError,
    UnauthorizedError,
    ForbiddenError,
    NotFoundError,
    ConflictError,
    ValidationError,
    InternalServerError,
    errorHandler,
    notFoundHandler
};
```

### 8.2 异步错误处理

```javascript
/**
 * 异步错误处理
 * @author erik.zhou
 */

// 异步错误包装器
const asyncHandler = (fn) => {
    return (req, res, next) => {
        Promise.resolve(fn(req, res, next)).catch(next);
    };
};

// 使用示例
const express = require('express');
const router = express.Router();
const { NotFoundError, ValidationError } = require('./errors');

router.get('/users/:id', asyncHandler(async (req, res) => {
    const user = await User.findById(req.params.id);
    
    if (!user) {
        throw new NotFoundError('User not found');
    }
    
    res.json({
        success: true,
        data: user
    });
}));

router.post('/users', asyncHandler(async (req, res) => {
    const { username, email, password } = req.body;
    
    // 验证
    const errors = [];
    if (!username) errors.push({ field: 'username', message: 'Username is required' });
    if (!email) errors.push({ field: 'email', message: 'Email is required' });
    if (!password) errors.push({ field: 'password', message: 'Password is required' });
    
    if (errors.length > 0) {
        throw new ValidationError(errors);
    }
    
    const user = await User.create({ username, email, password });
    
    res.status(201).json({
        success: true,
        data: user
    });
}));

module.exports = { asyncHandler };
```

### 8.3 错误响应格式

```javascript
/**
 * 错误响应格式
 * @author erik.zhou
 */

// 1. 简单错误响应
const simpleErrorResponse = {
    success: false,
    error: {
        statusCode: 404,
        message: 'User not found'
    },
    timestamp: '2024-03-02T10:00:00.000Z'
};

// 2. 验证错误响应
const validationErrorResponse = {
    success: false,
    error: {
        statusCode: 422,
        message: 'Validation failed',
        errors: [
            {
                field: 'email',
                message: 'Email is required'
            },
            {
                field: 'password',
                message: 'Password must be at least 8 characters'
            }
        ]
    },
    timestamp: '2024-03-02T10:00:00.000Z'
};

// 3. 详细错误响应（开发环境）
const detailedErrorResponse = {
    success: false,
    error: {
        statusCode: 500,
        message: 'Internal server error',
        stack: 'Error: Something went wrong\n    at ...',
        details: {
            query: 'SELECT * FROM users',
            params: { id: 123 }
        }
    },
    timestamp: '2024-03-02T10:00:00.000Z'
};

// 4. 多语言错误响应
const i18nErrorResponse = {
    success: false,
    error: {
        statusCode: 400,
        code: 'INVALID_EMAIL',
        message: 'Invalid email format',
        localizedMessage: {
            'en': 'Invalid email format',
            'zh-CN': '邮箱格式不正确',
            'ja': 'メールアドレスの形式が無効です'
        }
    },
    timestamp: '2024-03-02T10:00:00.000Z'
};
```

### 8.4 错误日志记录

```javascript
/**
 * 错误日志记录
 * @author erik.zhou
 */

const winston = require('winston');

// 配置日志
const logger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json()
    ),
    transports: [
        // 错误日志
        new winston.transports.File({
            filename: 'logs/error.log',
            level: 'error'
        }),
        // 所有日志
        new winston.transports.File({
            filename: 'logs/combined.log'
        })
    ]
});

// 开发环境添加控制台输出
if (process.env.NODE_ENV !== 'production') {
    logger.add(new winston.transports.Console({
        format: winston.format.combine(
            winston.format.colorize(),
            winston.format.simple()
        )
    }));
}

// 错误日志中间件
const errorLogger = (err, req, res, next) => {
    // 记录错误信息
    logger.error({
        message: err.message,
        statusCode: err.statusCode || 500,
        stack: err.stack,
        url: req.originalUrl,
        method: req.method,
        ip: req.ip,
        userId: req.userId,
        timestamp: new Date().toISOString()
    });
    
    next(err);
};

// 请求日志中间件
const requestLogger = (req, res, next) => {
    const start = Date.now();
    
    res.on('finish', () => {
        const duration = Date.now() - start;
        
        logger.info({
            method: req.method,
            url: req.originalUrl,
            statusCode: res.statusCode,
            duration: `${duration}ms`,
            ip: req.ip,
            userId: req.userId,
            timestamp: new Date().toISOString()
        });
    });
    
    next();
};

module.exports = {
    logger,
    errorLogger,
    requestLogger
};
```

---

## 9. API文档生成

### 9.1 Swagger/OpenAPI文档

```javascript
/**
 * Swagger文档配置
 * @author erik.zhou
 */

const swaggerJsdoc = require('swagger-jsdoc');
const swaggerUi = require('swagger-ui-express');

// Swagger配置
const swaggerOptions = {
    definition: {
        openapi: '3.0.0',
        info: {
            title: 'RESTful API Documentation',
            version: '1.0.0',
            description: 'Complete API documentation for the application',
            contact: {
                name: 'API Support',
                email: 'support@example.com'
            },
            license: {
                name: 'MIT',
                url: 'https://opensource.org/licenses/MIT'
            }
        },
        servers: [
            {
                url: 'http://localhost:3000',
                description: 'Development server'
            },
            {
                url: 'https://api.example.com',
                description: 'Production server'
            }
        ],
        components: {
            securitySchemes: {
                bearerAuth: {
                    type: 'http',
                    scheme: 'bearer',
                    bearerFormat: 'JWT'
                }
            },
            schemas: {
                User: {
                    type: 'object',
                    required: ['username', 'email', 'password'],
                    properties: {
                        id: {
                            type: 'string',
                            description: 'User ID'
                        },
                        username: {
                            type: 'string',
                            description: 'Username'
                        },
                        email: {
                            type: 'string',
                            format: 'email',
                            description: 'Email address'
                        },
                        password: {
                            type: 'string',
                            format: 'password',
                            description: 'Password (min 8 characters)'
                        },
                        role: {
                            type: 'string',
                            enum: ['user', 'admin'],
                            description: 'User role'
                        },
                        createdAt: {
                            type: 'string',
                            format: 'date-time',
                            description: 'Creation timestamp'
                        }
                    }
                },
                Error: {
                    type: 'object',
                    properties: {
                        success: {
                            type: 'boolean',
                            example: false
                        },
                        error: {
                            type: 'object',
                            properties: {
                                statusCode: {
                                    type: 'integer'
                                },
                                message: {
                                    type: 'string'
                                }
                            }
                        }
                    }
                }
            }
        },
        security: [
            {
                bearerAuth: []
            }
        ]
    },
    apis: ['./routes/*.js']
};

const swaggerSpec = swaggerJsdoc(swaggerOptions);

// 使用Swagger UI
const setupSwagger = (app) => {
    app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec, {
        customCss: '.swagger-ui .topbar { display: none }',
        customSiteTitle: 'API Documentation'
    }));
    
    // JSON格式文档
    app.get('/api-docs.json', (req, res) => {
        res.setHeader('Content-Type', 'application/json');
        res.send(swaggerSpec);
    });
};

module.exports = setupSwagger;
```

### 9.2 API注释示例

```javascript
/**
 * API路由注释
 * @author erik.zhou
 */

const express = require('express');
const router = express.Router();

/**
 * @swagger
 * /api/users:
 *   get:
 *     summary: Get all users
 *     description: Retrieve a list of all users with pagination
 *     tags: [Users]
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           default: 1
 *         description: Page number
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           default: 10
 *         description: Number of items per page
 *       - in: query
 *         name: sortBy
 *         schema:
 *           type: string
 *           default: createdAt
 *         description: Sort field
 *       - in: query
 *         name: order
 *         schema:
 *           type: string
 *           enum: [asc, desc]
 *           default: desc
 *         description: Sort order
 *     responses:
 *       200:
 *         description: Success
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                   example: true
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 *                 pagination:
 *                   type: object
 *                   properties:
 *                     total:
 *                       type: integer
 *                     page:
 *                       type: integer
 *                     pages:
 *                       type: integer
 *       500:
 *         description: Server error
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Error'
 */
router.get('/users', async (req, res) => {
    // 实现代码
});

/**
 * @swagger
 * /api/users/{id}:
 *   get:
 *     summary: Get user by ID
 *     description: Retrieve a single user by their ID
 *     tags: [Users]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 *         description: User ID
 *     responses:
 *       200:
 *         description: Success
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   $ref: '#/components/schemas/User'
 *       404:
 *         description: User not found
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Error'
 */
router.get('/users/:id', async (req, res) => {
    // 实现代码
});

/**
 * @swagger
 * /api/users:
 *   post:
 *     summary: Create a new user
 *     description: Create a new user account
 *     tags: [Users]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - username
 *               - email
 *               - password
 *             properties:
 *               username:
 *                 type: string
 *                 example: john_doe
 *               email:
 *                 type: string
 *                 format: email
 *                 example: john@example.com
 *               password:
 *                 type: string
 *                 format: password
 *                 example: securePassword123
 *     responses:
 *       201:
 *         description: User created successfully
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   $ref: '#/components/schemas/User'
 *       422:
 *         description: Validation error
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Error'
 *     security:
 *       - bearerAuth: []
 */
router.post('/users', async (req, res) => {
    // 实现代码
});

module.exports = router;
```

### 9.3 Postman集合生成

```javascript
/**
 * Postman集合生成
 * @author erik.zhou
 */

const postmanCollection = {
    info: {
        name: 'RESTful API Collection',
        description: 'Complete API collection for testing',
        schema: 'https://schema.getpostman.com/json/collection/v2.1.0/collection.json'
    },
    auth: {
        type: 'bearer',
        bearer: [
            {
                key: 'token',
                value: '{{token}}',
                type: 'string'
            }
        ]
    },
    variable: [
        {
            key: 'baseUrl',
            value: 'http://localhost:3000/api',
            type: 'string'
        },
        {
            key: 'token',
            value: '',
            type: 'string'
        }
    ],
    item: [
        {
            name: 'Authentication',
            item: [
                {
                    name: 'Register',
                    request: {
                        method: 'POST',
                        header: [
                            {
                                key: 'Content-Type',
                                value: 'application/json'
                            }
                        ],
                        body: {
                            mode: 'raw',
                            raw: JSON.stringify({
                                username: 'john_doe',
                                email: 'john@example.com',
                                password: 'securePassword123'
                            }, null, 2)
                        },
                        url: {
                            raw: '{{baseUrl}}/auth/register',
                            host: ['{{baseUrl}}'],
                            path: ['auth', 'register']
                        }
                    }
                },
                {
                    name: 'Login',
                    event: [
                        {
                            listen: 'test',
                            script: {
                                exec: [
                                    'const response = pm.response.json();',
                                    'if (response.data && response.data.token) {',
                                    '    pm.environment.set("token", response.data.token);',
                                    '}'
                                ]
                            }
                        }
                    ],
                    request: {
                        method: 'POST',
                        header: [
                            {
                                key: 'Content-Type',
                                value: 'application/json'
                            }
                        ],
                        body: {
                            mode: 'raw',
                            raw: JSON.stringify({
                                email: 'john@example.com',
                                password: 'securePassword123'
                            }, null, 2)
                        },
                        url: {
                            raw: '{{baseUrl}}/auth/login',
                            host: ['{{baseUrl}}'],
                            path: ['auth', 'login']
                        }
                    }
                }
            ]
        },
        {
            name: 'Users',
            item: [
                {
                    name: 'Get All Users',
                    request: {
                        method: 'GET',
                        header: [],
                        url: {
                            raw: '{{baseUrl}}/users?page=1&limit=10',
                            host: ['{{baseUrl}}'],
                            path: ['users'],
                            query: [
                                {
                                    key: 'page',
                                    value: '1'
                                },
                                {
                                    key: 'limit',
                                    value: '10'
                                }
                            ]
                        }
                    }
                },
                {
                    name: 'Get User by ID',
                    request: {
                        method: 'GET',
                        header: [],
                        url: {
                            raw: '{{baseUrl}}/users/:id',
                            host: ['{{baseUrl}}'],
                            path: ['users', ':id'],
                            variable: [
                                {
                                    key: 'id',
                                    value: '123'
                                }
                            ]
                        }
                    }
                },
                {
                    name: 'Create User',
                    request: {
                        auth: {
                            type: 'bearer',
                            bearer: [
                                {
                                    key: 'token',
                                    value: '{{token}}',
                                    type: 'string'
                                }
                            ]
                        },
                        method: 'POST',
                        header: [
                            {
                                key: 'Content-Type',
                                value: 'application/json'
                            }
                        ],
                        body: {
                            mode: 'raw',
                            raw: JSON.stringify({
                                username: 'new_user',
                                email: 'newuser@example.com',
                                password: 'password123'
                            }, null, 2)
                        },
                        url: {
                            raw: '{{baseUrl}}/users',
                            host: ['{{baseUrl}}'],
                            path: ['users']
                        }
                    }
                }
            ]
        }
    ]
};

// 导出Postman集合
const fs = require('fs');
fs.writeFileSync(
    'postman_collection.json',
    JSON.stringify(postmanCollection, null, 2)
);
```



---

## 10. 最佳实践与安全

### 10.1 API安全最佳实践

```javascript
/**
 * API安全最佳实践
 * @author erik.zhou
 */

const express = require('express');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');
const cors = require('cors');

const app = express();

// 1. 设置安全HTTP头
app.use(helmet());

// 2. 限流
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15分钟
    max: 100, // 限制100个请求
    message: 'Too many requests from this IP, please try again later',
    standardHeaders: true,
    legacyHeaders: false
});

app.use('/api', limiter);

// 登录接口更严格的限流
const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5,
    message: 'Too many login attempts, please try again later'
});

app.use('/api/auth/login', loginLimiter);

// 3. 防止NoSQL注入
app.use(mongoSanitize());

// 4. 防止XSS攻击
app.use(xss());

// 5. 防止HTTP参数污染
app.use(hpp({
    whitelist: ['page', 'limit', 'sort'] // 允许重复的参数
}));

// 6. CORS配置
const corsOptions = {
    origin: (origin, callback) => {
        const whitelist = [
            'http://localhost:3000',
            'https://example.com'
        ];
        
        if (!origin || whitelist.indexOf(origin) !== -1) {
            callback(null, true);
        } else {
            callback(new Error('Not allowed by CORS'));
        }
    },
    credentials: true,
    optionsSuccessStatus: 200
};

app.use(cors(corsOptions));

// 7. 请求体大小限制
app.use(express.json({ limit: '10kb' }));
app.use(express.urlencoded({ extended: true, limit: '10kb' }));

// 8. 隐藏技术栈信息
app.disable('x-powered-by');

// 9. 输入验证
const { body, validationResult } = require('express-validator');

app.post('/api/users',
    [
        body('email').isEmail().normalizeEmail(),
        body('password').isLength({ min: 8 }),
        body('username').trim().isLength({ min: 3, max: 20 })
    ],
    (req, res) => {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(422).json({
                success: false,
                errors: errors.array()
            });
        }
        
        // 处理请求
    }
);

// 10. SQL注入防护（使用参数化查询）
const mysql = require('mysql2/promise');

const pool = mysql.createPool({
    host: 'localhost',
    user: 'root',
    password: 'password',
    database: 'mydb'
});

// ✅ 正确 - 使用参数化查询
app.get('/api/users/:id', async (req, res) => {
    const [rows] = await pool.execute(
        'SELECT * FROM users WHERE id = ?',
        [req.params.id]
    );
    res.json({ data: rows });
});

// ❌ 错误 - 直接拼接SQL
// const query = `SELECT * FROM users WHERE id = ${req.params.id}`;
```

### 10.2 数据验证与清理

```javascript
/**
 * 数据验证与清理
 * @author erik.zhou
 */

const { body, param, query, validationResult } = require('express-validator');

// 验证规则
const userValidationRules = {
    create: [
        body('username')
            .trim()
            .isLength({ min: 3, max: 20 })
            .withMessage('Username must be between 3 and 20 characters')
            .matches(/^[a-zA-Z0-9_]+$/)
            .withMessage('Username can only contain letters, numbers and underscores'),
        
        body('email')
            .trim()
            .isEmail()
            .withMessage('Must be a valid email')
            .normalizeEmail(),
        
        body('password')
            .isLength({ min: 8 })
            .withMessage('Password must be at least 8 characters')
            .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
            .withMessage('Password must contain uppercase, lowercase and number'),
        
        body('age')
            .optional()
            .isInt({ min: 18, max: 120 })
            .withMessage('Age must be between 18 and 120'),
        
        body('website')
            .optional()
            .isURL()
            .withMessage('Must be a valid URL')
    ],
    
    update: [
        param('id')
            .isMongoId()
            .withMessage('Invalid user ID'),
        
        body('username')
            .optional()
            .trim()
            .isLength({ min: 3, max: 20 })
            .withMessage('Username must be between 3 and 20 characters'),
        
        body('email')
            .optional()
            .trim()
            .isEmail()
            .withMessage('Must be a valid email')
            .normalizeEmail()
    ],
    
    list: [
        query('page')
            .optional()
            .isInt({ min: 1 })
            .withMessage('Page must be a positive integer'),
        
        query('limit')
            .optional()
            .isInt({ min: 1, max: 100 })
            .withMessage('Limit must be between 1 and 100'),
        
        query('sortBy')
            .optional()
            .isIn(['createdAt', 'username', 'email'])
            .withMessage('Invalid sort field'),
        
        query('order')
            .optional()
            .isIn(['asc', 'desc'])
            .withMessage('Order must be asc or desc')
    ]
};

// 验证中间件
const validate = (req, res, next) => {
    const errors = validationResult(req);
    
    if (!errors.isEmpty()) {
        return res.status(422).json({
            success: false,
            error: {
                statusCode: 422,
                message: 'Validation failed',
                errors: errors.array().map(err => ({
                    field: err.param,
                    message: err.msg,
                    value: err.value
                }))
            }
        });
    }
    
    next();
};

// 使用示例
const express = require('express');
const router = express.Router();

router.post('/users',
    userValidationRules.create,
    validate,
    async (req, res) => {
        // 数据已验证和清理
        const user = await User.create(req.body);
        res.status(201).json({ data: user });
    }
);

router.put('/users/:id',
    userValidationRules.update,
    validate,
    async (req, res) => {
        const user = await User.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true }
        );
        res.json({ data: user });
    }
);

router.get('/users',
    userValidationRules.list,
    validate,
    async (req, res) => {
        const { page = 1, limit = 10, sortBy = 'createdAt', order = 'desc' } = req.query;
        
        const users = await User.find()
            .sort({ [sortBy]: order === 'asc' ? 1 : -1 })
            .limit(limit * 1)
            .skip((page - 1) * limit);
        
        res.json({ data: users });
    }
);

module.exports = router;
```

### 10.3 性能优化

```javascript
/**
 * API性能优化
 * @author erik.zhou
 */

const express = require('express');
const compression = require('compression');
const redis = require('redis');
const { promisify } = require('util');

const app = express();

// 1. 启用Gzip压缩
app.use(compression());

// 2. Redis缓存
const redisClient = redis.createClient({
    host: 'localhost',
    port: 6379
});

const getAsync = promisify(redisClient.get).bind(redisClient);
const setAsync = promisify(redisClient.set).bind(redisClient);

// 缓存中间件
const cache = (duration) => {
    return async (req, res, next) => {
        const key = `cache:${req.originalUrl}`;
        
        try {
            const cachedData = await getAsync(key);
            
            if (cachedData) {
                return res.json(JSON.parse(cachedData));
            }
            
            // 保存原始的res.json方法
            const originalJson = res.json.bind(res);
            
            // 重写res.json方法
            res.json = (data) => {
                // 缓存数据
                setAsync(key, JSON.stringify(data), 'EX', duration);
                // 调用原始方法
                return originalJson(data);
            };
            
            next();
        } catch (error) {
            next();
        }
    };
};

// 使用缓存
app.get('/api/users', cache(300), async (req, res) => {
    const users = await User.find();
    res.json({ data: users });
});

// 3. 数据库查询优化
const router = express.Router();

// 使用索引
router.get('/users', async (req, res) => {
    const { email } = req.query;
    
    // 确保email字段有索引
    const users = await User.find({ email }).lean();
    res.json({ data: users });
});

// 字段选择
router.get('/users/:id', async (req, res) => {
    // 只选择需要的字段
    const user = await User.findById(req.params.id)
        .select('username email profile')
        .lean();
    
    res.json({ data: user });
});

// 分页优化
router.get('/posts', async (req, res) => {
    const { page = 1, limit = 10 } = req.query;
    
    // 使用游标分页（更高效）
    const lastId = req.query.lastId;
    
    const query = lastId ? { _id: { $gt: lastId } } : {};
    
    const posts = await Post.find(query)
        .sort({ _id: 1 })
        .limit(limit * 1)
        .lean();
    
    res.json({
        data: posts,
        nextCursor: posts.length > 0 ? posts[posts.length - 1]._id : null
    });
});

// 4. 批量操作
router.post('/users/batch', async (req, res) => {
    const { users } = req.body;
    
    // 使用bulkWrite批量插入
    const operations = users.map(user => ({
        insertOne: {
            document: user
        }
    }));
    
    const result = await User.bulkWrite(operations);
    
    res.status(201).json({
        success: true,
        inserted: result.insertedCount
    });
});

// 5. 响应时间监控
app.use((req, res, next) => {
    const start = Date.now();
    
    res.on('finish', () => {
        const duration = Date.now() - start;
        
        // 记录慢查询
        if (duration > 1000) {
            console.warn(`Slow request: ${req.method} ${req.originalUrl} - ${duration}ms`);
        }
    });
    
    next();
});

module.exports = router;
```

### 10.4 API测试

```javascript
/**
 * API测试示例
 * @author erik.zhou
 */

const request = require('supertest');
const app = require('../app');
const User = require('../models/User');

describe('User API', () => {
    let token;
    let userId;
    
    // 测试前清理数据库
    beforeEach(async () => {
        await User.deleteMany({});
    });
    
    // 测试用户注册
    describe('POST /api/auth/register', () => {
        it('should register a new user', async () => {
            const res = await request(app)
                .post('/api/auth/register')
                .send({
                    username: 'testuser',
                    email: 'test@example.com',
                    password: 'Password123'
                })
                .expect(201);
            
            expect(res.body.success).toBe(true);
            expect(res.body.data.user).toHaveProperty('id');
            expect(res.body.data).toHaveProperty('token');
            
            token = res.body.data.token;
        });
        
        it('should return 422 for invalid email', async () => {
            const res = await request(app)
                .post('/api/auth/register')
                .send({
                    username: 'testuser',
                    email: 'invalid-email',
                    password: 'Password123'
                })
                .expect(422);
            
            expect(res.body.success).toBe(false);
            expect(res.body.error.errors).toBeDefined();
        });
        
        it('should return 409 for duplicate email', async () => {
            await User.create({
                username: 'existing',
                email: 'test@example.com',
                password: 'hashedpassword'
            });
            
            const res = await request(app)
                .post('/api/auth/register')
                .send({
                    username: 'testuser',
                    email: 'test@example.com',
                    password: 'Password123'
                })
                .expect(409);
            
            expect(res.body.success).toBe(false);
        });
    });
    
    // 测试用户登录
    describe('POST /api/auth/login', () => {
        beforeEach(async () => {
            await request(app)
                .post('/api/auth/register')
                .send({
                    username: 'testuser',
                    email: 'test@example.com',
                    password: 'Password123'
                });
        });
        
        it('should login with valid credentials', async () => {
            const res = await request(app)
                .post('/api/auth/login')
                .send({
                    email: 'test@example.com',
                    password: 'Password123'
                })
                .expect(200);
            
            expect(res.body.success).toBe(true);
            expect(res.body.data).toHaveProperty('token');
        });
        
        it('should return 401 for invalid credentials', async () => {
            const res = await request(app)
                .post('/api/auth/login')
                .send({
                    email: 'test@example.com',
                    password: 'WrongPassword'
                })
                .expect(401);
            
            expect(res.body.success).toBe(false);
        });
    });
    
    // 测试获取用户列表
    describe('GET /api/users', () => {
        beforeEach(async () => {
            // 创建测试用户
            await User.create([
                { username: 'user1', email: 'user1@example.com', password: 'hash1' },
                { username: 'user2', email: 'user2@example.com', password: 'hash2' },
                { username: 'user3', email: 'user3@example.com', password: 'hash3' }
            ]);
        });
        
        it('should get all users with pagination', async () => {
            const res = await request(app)
                .get('/api/users?page=1&limit=2')
                .expect(200);
            
            expect(res.body.success).toBe(true);
            expect(res.body.data).toHaveLength(2);
            expect(res.body.pagination.total).toBe(3);
        });
        
        it('should filter users by query', async () => {
            const res = await request(app)
                .get('/api/users?search=user1')
                .expect(200);
            
            expect(res.body.success).toBe(true);
            expect(res.body.data).toHaveLength(1);
            expect(res.body.data[0].username).toBe('user1');
        });
    });
    
    // 测试获取单个用户
    describe('GET /api/users/:id', () => {
        beforeEach(async () => {
            const user = await User.create({
                username: 'testuser',
                email: 'test@example.com',
                password: 'hashedpassword'
            });
            userId = user._id;
        });
        
        it('should get user by id', async () => {
            const res = await request(app)
                .get(`/api/users/${userId}`)
                .expect(200);
            
            expect(res.body.success).toBe(true);
            expect(res.body.data.username).toBe('testuser');
        });
        
        it('should return 404 for non-existent user', async () => {
            const fakeId = '507f1f77bcf86cd799439011';
            
            const res = await request(app)
                .get(`/api/users/${fakeId}`)
                .expect(404);
            
            expect(res.body.success).toBe(false);
        });
    });
    
    // 测试更新用户
    describe('PUT /api/users/:id', () => {
        beforeEach(async () => {
            const registerRes = await request(app)
                .post('/api/auth/register')
                .send({
                    username: 'testuser',
                    email: 'test@example.com',
                    password: 'Password123'
                });
            
            token = registerRes.body.data.token;
            userId = registerRes.body.data.user.id;
        });
        
        it('should update user with valid token', async () => {
            const res = await request(app)
                .put(`/api/users/${userId}`)
                .set('Authorization', `Bearer ${token}`)
                .send({
                    username: 'updateduser'
                })
                .expect(200);
            
            expect(res.body.success).toBe(true);
            expect(res.body.data.username).toBe('updateduser');
        });
        
        it('should return 401 without token', async () => {
            const res = await request(app)
                .put(`/api/users/${userId}`)
                .send({
                    username: 'updateduser'
                })
                .expect(401);
            
            expect(res.body.success).toBe(false);
        });
    });
    
    // 测试删除用户
    describe('DELETE /api/users/:id', () => {
        beforeEach(async () => {
            const registerRes = await request(app)
                .post('/api/auth/register')
                .send({
                    username: 'testuser',
                    email: 'test@example.com',
                    password: 'Password123'
                });
            
            token = registerRes.body.data.token;
            userId = registerRes.body.data.user.id;
        });
        
        it('should delete user with valid token', async () => {
            await request(app)
                .delete(`/api/users/${userId}`)
                .set('Authorization', `Bearer ${token}`)
                .expect(204);
            
            const user = await User.findById(userId);
            expect(user).toBeNull();
        });
    });
});
```

### 10.5 API监控与日志

```javascript
/**
 * API监控与日志
 * @author erik.zhou
 */

const express = require('express');
const morgan = require('morgan');
const winston = require('winston');
const prometheus = require('prom-client');

const app = express();

// 1. 请求日志
const logger = winston.createLogger({
    level: 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json()
    ),
    transports: [
        new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
        new winston.transports.File({ filename: 'logs/combined.log' })
    ]
});

// HTTP请求日志
app.use(morgan('combined', {
    stream: {
        write: (message) => logger.info(message.trim())
    }
}));

// 2. Prometheus指标
const register = new prometheus.Registry();

// 默认指标
prometheus.collectDefaultMetrics({ register });

// 自定义指标
const httpRequestDuration = new prometheus.Histogram({
    name: 'http_request_duration_seconds',
    help: 'Duration of HTTP requests in seconds',
    labelNames: ['method', 'route', 'status_code'],
    buckets: [0.1, 0.5, 1, 2, 5]
});

const httpRequestTotal = new prometheus.Counter({
    name: 'http_requests_total',
    help: 'Total number of HTTP requests',
    labelNames: ['method', 'route', 'status_code']
});

register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);

// 指标收集中间件
app.use((req, res, next) => {
    const start = Date.now();
    
    res.on('finish', () => {
        const duration = (Date.now() - start) / 1000;
        const route = req.route ? req.route.path : req.path;
        
        httpRequestDuration
            .labels(req.method, route, res.statusCode)
            .observe(duration);
        
        httpRequestTotal
            .labels(req.method, route, res.statusCode)
            .inc();
    });
    
    next();
});

// 指标端点
app.get('/metrics', async (req, res) => {
    res.set('Content-Type', register.contentType);
    res.end(await register.metrics());
});

// 3. 健康检查
app.get('/health', async (req, res) => {
    const health = {
        uptime: process.uptime(),
        timestamp: Date.now(),
        status: 'OK',
        checks: {
            database: 'OK',
            redis: 'OK',
            memory: {
                used: process.memoryUsage().heapUsed,
                total: process.memoryUsage().heapTotal
            }
        }
    };
    
    try {
        // 检查数据库连接
        await mongoose.connection.db.admin().ping();
    } catch (error) {
        health.checks.database = 'ERROR';
        health.status = 'DEGRADED';
    }
    
    const statusCode = health.status === 'OK' ? 200 : 503;
    res.status(statusCode).json(health);
});

// 4. API状态端点
app.get('/api/status', (req, res) => {
    res.json({
        version: '1.0.0',
        environment: process.env.NODE_ENV,
        uptime: process.uptime(),
        timestamp: new Date().toISOString()
    });
});

module.exports = app;
```

---

## 总结

本教程全面介绍了RESTful API设计的核心概念和最佳实践：

1. **RESTful基础**：理解REST架构风格和核心原则
2. **设计原则**：遵循统一的命名和版本控制规范
3. **资源路由**：设计清晰的CRUD路由和嵌套资源
4. **HTTP规范**：正确使用HTTP方法和状态码
5. **数据格式**：统一的请求响应格式和HATEOAS
6. **版本控制**：实现API版本管理和向后兼容
7. **认证授权**：JWT认证和基于角色的访问控制
8. **错误处理**：统一的错误处理和日志记录
9. **API文档**：使用Swagger生成交互式文档
10. **安全实践**：防护常见安全威胁和性能优化

通过本教程的学习，你将能够设计和实现高质量、安全、可维护的RESTful API。

---

**@author erik.zhou**
**最后更新时间：2026-03-03**
