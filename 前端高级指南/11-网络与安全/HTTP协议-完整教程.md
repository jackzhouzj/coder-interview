# HTTP协议 - 完整教程

## 课程简介
深入理解HTTP协议的工作原理，掌握请求响应机制、状态码、Headers、HTTP/2和HTTP/3等核心概念。

## 学习目标
- 掌握HTTP协议基础知识
- 理解请求与响应结构
- 熟悉常用HTTP状态码
- 学会使用HTTP Headers
- 了解HTTP/2和HTTP/3特性
- 实践HTTP性能优化

---

## 第一部分：HTTP基础

### 1.1 HTTP协议概述

#### HTTP协议特点
```javascript
/**
 * HTTP协议特点示例
 * @author erik.zhou
 */

const httpFeatures = {
    // 1. 无状态协议
    stateless: {
        description: '每个请求都是独立的，服务器不保存客户端状态',
        solution: 'Cookie、Session、Token'
    },
    
    // 2. 基于请求-响应模型
    requestResponse: {
        description: '客户端发送请求，服务器返回响应',
        flow: ['客户端请求', '服务器处理', '服务器响应']
    },
    
    // 3. 基于TCP/IP
    transport: {
        protocol: 'TCP',
        port: {
            http: 80,
            https: 443
        }
    },
    
    // 4. 支持多种方法
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'HEAD', 'OPTIONS'],
    
    // 5. 灵活的内容类型
    contentTypes: [
        'text/html',
        'application/json',
        'application/xml',
        'multipart/form-data',
        'application/octet-stream'
    ]
};

console.log('HTTP协议特点:', httpFeatures);
```

#### HTTP请求流程
```javascript
/**
 * HTTP请求流程模拟
 * @author erik.zhou
 */

class HTTPRequestFlow {
    constructor() {
        this.steps = [];
    }
    
    // 1. DNS解析
    async dnsResolve(domain) {
        this.steps.push('DNS解析');
        console.log(`解析域名: ${domain}`);
        
        // 模拟DNS查询
        const ip = await this.mockDNSQuery(domain);
        console.log(`解析结果: ${ip}`);
        
        return ip;
    }
    
    mockDNSQuery(domain) {
        return new Promise(resolve => {
            setTimeout(() => {
                resolve('192.168.1.1');
            }, 100);
        });
    }
    
    // 2. 建立TCP连接
    async establishTCPConnection(ip, port) {
        this.steps.push('建立TCP连接');
        console.log(`连接到 ${ip}:${port}`);
        
        // 三次握手
        await this.threeWayHandshake();
        
        console.log('TCP连接已建立');
    }
    
    async threeWayHandshake() {
        console.log('1. 客户端发送SYN');
        await this.delay(50);
        
        console.log('2. 服务器返回SYN-ACK');
        await this.delay(50);
        
        console.log('3. 客户端发送ACK');
        await this.delay(50);
    }
    
    // 3. 发送HTTP请求
    async sendHTTPRequest(request) {
        this.steps.push('发送HTTP请求');
        console.log('发送请求:', request);
        
        await this.delay(100);
    }
    
    // 4. 服务器处理
    async serverProcess() {
        this.steps.push('服务器处理');
        console.log('服务器处理请求...');
        
        await this.delay(200);
    }
    
    // 5. 接收HTTP响应
    async receiveHTTPResponse() {
        this.steps.push('接收HTTP响应');
        console.log('接收响应...');
        
        await this.delay(100);
        
        return {
            status: 200,
            headers: {
                'Content-Type': 'application/json'
            },
            body: { message: 'Success' }
        };
    }
    
    // 6. 关闭连接
    async closeConnection() {
        this.steps.push('关闭连接');
        console.log('关闭TCP连接');
        
        await this.delay(50);
    }
    
    // 执行完整流程
    async execute(url) {
        console.log('=== HTTP请求流程 ===');
        console.log(`请求URL: ${url}`);
        
        const urlObj = new URL(url);
        
        // 1. DNS解析
        const ip = await this.dnsResolve(urlObj.hostname);
        
        // 2. 建立TCP连接
        await this.establishTCPConnection(ip, urlObj.port || 80);
        
        // 3. 发送HTTP请求
        await this.sendHTTPRequest({
            method: 'GET',
            path: urlObj.pathname,
            headers: {
                'Host': urlObj.hostname,
                'User-Agent': 'Mozilla/5.0'
            }
        });
        
        // 4. 服务器处理
        await this.serverProcess();
        
        // 5. 接收响应
        const response = await this.receiveHTTPResponse();
        
        // 6. 关闭连接
        await this.closeConnection();
        
        console.log('\n完成步骤:', this.steps);
        console.log('响应:', response);
        
        return response;
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
const flow = new HTTPRequestFlow();
flow.execute('http://example.com/api/data');
```

### 1.2 HTTP请求

#### 请求结构
```javascript
/**
 * HTTP请求结构
 * @author erik.zhou
 */

class HTTPRequest {
    constructor(method, url, headers = {}, body = null) {
        this.method = method;
        this.url = url;
        this.headers = headers;
        this.body = body;
    }
    
    // 构建请求行
    buildRequestLine() {
        const urlObj = new URL(this.url);
        const path = urlObj.pathname + urlObj.search;
        return `${this.method} ${path} HTTP/1.1`;
    }
    
    // 构建请求头
    buildHeaders() {
        const urlObj = new URL(this.url);
        const defaultHeaders = {
            'Host': urlObj.hostname,
            'User-Agent': 'Mozilla/5.0',
            'Accept': '*/*',
            'Connection': 'keep-alive'
        };
        
        return { ...defaultHeaders, ...this.headers };
    }
    
    // 构建完整请求
    build() {
        const requestLine = this.buildRequestLine();
        const headers = this.buildHeaders();
        
        let request = requestLine + '\r\n';
        
        // 添加请求头
        for (const [key, value] of Object.entries(headers)) {
            request += `${key}: ${value}\r\n`;
        }
        
        // 空行分隔
        request += '\r\n';
        
        // 添加请求体
        if (this.body) {
            request += this.body;
        }
        
        return request;
    }
    
    // 打印请求
    print() {
        console.log('=== HTTP请求 ===');
        console.log(this.build());
    }
}

// 使用示例
const getRequest = new HTTPRequest(
    'GET',
    'http://example.com/api/users?page=1',
    {
        'Accept': 'application/json',
        'Authorization': 'Bearer token123'
    }
);

getRequest.print();

const postRequest = new HTTPRequest(
    'POST',
    'http://example.com/api/users',
    {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer token123'
    },
    JSON.stringify({ name: 'Erik', age: 30 })
);

postRequest.print();
```

#### HTTP方法
```javascript
/**
 * HTTP方法详解
 * @author erik.zhou
 */

class HTTPMethods {
    constructor(baseURL) {
        this.baseURL = baseURL;
    }
    
    // GET - 获取资源
    async get(path, params = {}) {
        const queryString = new URLSearchParams(params).toString();
        const url = `${this.baseURL}${path}${queryString ? '?' + queryString : ''}`;
        
        try {
            const response = await fetch(url, {
                method: 'GET',
                headers: {
                    'Accept': 'application/json'
                }
            });
            
            return await response.json();
        } catch (error) {
            console.error('GET请求失败:', error);
            throw error;
        }
    }
    
    // POST - 创建资源
    async post(path, data) {
        try {
            const response = await fetch(`${this.baseURL}${path}`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Accept': 'application/json'
                },
                body: JSON.stringify(data)
            });
            
            return await response.json();
        } catch (error) {
            console.error('POST请求失败:', error);
            throw error;
        }
    }
    
    // PUT - 更新资源（完整更新）
    async put(path, data) {
        try {
            const response = await fetch(`${this.baseURL}${path}`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                    'Accept': 'application/json'
                },
                body: JSON.stringify(data)
            });
            
            return await response.json();
        } catch (error) {
            console.error('PUT请求失败:', error);
            throw error;
        }
    }
    
    // PATCH - 更新资源（部分更新）
    async patch(path, data) {
        try {
            const response = await fetch(`${this.baseURL}${path}`, {
                method: 'PATCH',
                headers: {
                    'Content-Type': 'application/json',
                    'Accept': 'application/json'
                },
                body: JSON.stringify(data)
            });
            
            return await response.json();
        } catch (error) {
            console.error('PATCH请求失败:', error);
            throw error;
        }
    }
    
    // DELETE - 删除资源
    async delete(path) {
        try {
            const response = await fetch(`${this.baseURL}${path}`, {
                method: 'DELETE',
                headers: {
                    'Accept': 'application/json'
                }
            });
            
            return await response.json();
        } catch (error) {
            console.error('DELETE请求失败:', error);
            throw error;
        }
    }
    
    // HEAD - 获取资源元信息
    async head(path) {
        try {
            const response = await fetch(`${this.baseURL}${path}`, {
                method: 'HEAD'
            });
            
            return {
                status: response.status,
                headers: Object.fromEntries(response.headers.entries())
            };
        } catch (error) {
            console.error('HEAD请求失败:', error);
            throw error;
        }
    }
    
    // OPTIONS - 获取支持的方法
    async options(path) {
        try {
            const response = await fetch(`${this.baseURL}${path}`, {
                method: 'OPTIONS'
            });
            
            return {
                status: response.status,
                allow: response.headers.get('Allow'),
                headers: Object.fromEntries(response.headers.entries())
            };
        } catch (error) {
            console.error('OPTIONS请求失败:', error);
            throw error;
        }
    }
}

// 使用示例
const api = new HTTPMethods('https://api.example.com');

// GET请求
api.get('/users', { page: 1, limit: 10 })
    .then(data => console.log('用户列表:', data));

// POST请求
api.post('/users', { name: 'Erik', email: 'erik@example.com' })
    .then(data => console.log('创建用户:', data));

// PUT请求
api.put('/users/1', { name: 'Erik Zhou', email: 'erik@example.com', age: 30 })
    .then(data => console.log('更新用户:', data));

// PATCH请求
api.patch('/users/1', { age: 31 })
    .then(data => console.log('部分更新:', data));

// DELETE请求
api.delete('/users/1')
    .then(data => console.log('删除用户:', data));

// HEAD请求
api.head('/users/1')
    .then(info => console.log('资源信息:', info));

// OPTIONS请求
api.options('/users')
    .then(info => console.log('支持的方法:', info));
```

### 1.3 HTTP响应

#### 响应结构
```javascript
/**
 * HTTP响应结构
 * @author erik.zhou
 */

class HTTPResponse {
    constructor(status, headers = {}, body = null) {
        this.status = status;
        this.headers = headers;
        this.body = body;
    }
    
    // 构建状态行
    buildStatusLine() {
        const statusTexts = {
            200: 'OK',
            201: 'Created',
            204: 'No Content',
            301: 'Moved Permanently',
            302: 'Found',
            304: 'Not Modified',
            400: 'Bad Request',
            401: 'Unauthorized',
            403: 'Forbidden',
            404: 'Not Found',
            500: 'Internal Server Error',
            502: 'Bad Gateway',
            503: 'Service Unavailable'
        };
        
        const statusText = statusTexts[this.status] || 'Unknown';
        return `HTTP/1.1 ${this.status} ${statusText}`;
    }
    
    // 构建响应头
    buildHeaders() {
        const defaultHeaders = {
            'Date': new Date().toUTCString(),
            'Server': 'Node.js',
            'Connection': 'keep-alive'
        };
        
        return { ...defaultHeaders, ...this.headers };
    }
    
    // 构建完整响应
    build() {
        const statusLine = this.buildStatusLine();
        const headers = this.buildHeaders();
        
        let response = statusLine + '\r\n';
        
        // 添加响应头
        for (const [key, value] of Object.entries(headers)) {
            response += `${key}: ${value}\r\n`;
        }
        
        // 空行分隔
        response += '\r\n';
        
        // 添加响应体
        if (this.body) {
            response += this.body;
        }
        
        return response;
    }
    
    // 打印响应
    print() {
        console.log('=== HTTP响应 ===');
        console.log(this.build());
    }
}

// 使用示例
const successResponse = new HTTPResponse(
    200,
    {
        'Content-Type': 'application/json',
        'Content-Length': '45'
    },
    JSON.stringify({ message: 'Success', data: { id: 1 } })
);

successResponse.print();

const errorResponse = new HTTPResponse(
    404,
    {
        'Content-Type': 'application/json'
    },
    JSON.stringify({ error: 'Not Found', message: '资源不存在' })
);

errorResponse.print();
```


## 第二部分：HTTP状态码

### 2.1 状态码分类

#### 状态码概览
```javascript
/**
 * HTTP状态码分类
 * @author erik.zhou
 */

const httpStatusCodes = {
    // 1xx - 信息性状态码
    informational: {
        100: {
            name: 'Continue',
            description: '客户端应继续请求',
            usage: '用于大文件上传前的确认'
        },
        101: {
            name: 'Switching Protocols',
            description: '服务器切换协议',
            usage: 'WebSocket握手'
        },
        102: {
            name: 'Processing',
            description: '服务器正在处理请求',
            usage: 'WebDAV'
        }
    },
    
    // 2xx - 成功状态码
    success: {
        200: {
            name: 'OK',
            description: '请求成功',
            usage: '最常见的成功响应'
        },
        201: {
            name: 'Created',
            description: '资源已创建',
            usage: 'POST请求成功创建资源'
        },
        202: {
            name: 'Accepted',
            description: '请求已接受，但未完成处理',
            usage: '异步任务'
        },
        204: {
            name: 'No Content',
            description: '请求成功，但无返回内容',
            usage: 'DELETE请求成功'
        },
        206: {
            name: 'Partial Content',
            description: '部分内容',
            usage: '断点续传、视频流'
        }
    },
    
    // 3xx - 重定向状态码
    redirection: {
        301: {
            name: 'Moved Permanently',
            description: '永久重定向',
            usage: 'URL永久变更，SEO友好'
        },
        302: {
            name: 'Found',
            description: '临时重定向',
            usage: '临时跳转'
        },
        303: {
            name: 'See Other',
            description: '查看其他位置',
            usage: 'POST后重定向到GET'
        },
        304: {
            name: 'Not Modified',
            description: '资源未修改',
            usage: '缓存验证'
        },
        307: {
            name: 'Temporary Redirect',
            description: '临时重定向（保持方法）',
            usage: '临时跳转，不改变请求方法'
        },
        308: {
            name: 'Permanent Redirect',
            description: '永久重定向（保持方法）',
            usage: '永久跳转，不改变请求方法'
        }
    },
    
    // 4xx - 客户端错误
    clientError: {
        400: {
            name: 'Bad Request',
            description: '请求语法错误',
            usage: '参数验证失败'
        },
        401: {
            name: 'Unauthorized',
            description: '未授权',
            usage: '需要身份认证'
        },
        403: {
            name: 'Forbidden',
            description: '禁止访问',
            usage: '权限不足'
        },
        404: {
            name: 'Not Found',
            description: '资源不存在',
            usage: '路由不存在'
        },
        405: {
            name: 'Method Not Allowed',
            description: '方法不允许',
            usage: '请求方法不支持'
        },
        408: {
            name: 'Request Timeout',
            description: '请求超时',
            usage: '客户端请求超时'
        },
        409: {
            name: 'Conflict',
            description: '请求冲突',
            usage: '资源状态冲突'
        },
        410: {
            name: 'Gone',
            description: '资源已永久删除',
            usage: '资源不再可用'
        },
        413: {
            name: 'Payload Too Large',
            description: '请求体过大',
            usage: '文件上传超限'
        },
        414: {
            name: 'URI Too Long',
            description: 'URI过长',
            usage: 'URL长度超限'
        },
        415: {
            name: 'Unsupported Media Type',
            description: '不支持的媒体类型',
            usage: 'Content-Type不支持'
        },
        429: {
            name: 'Too Many Requests',
            description: '请求过多',
            usage: '限流'
        }
    },
    
    // 5xx - 服务器错误
    serverError: {
        500: {
            name: 'Internal Server Error',
            description: '服务器内部错误',
            usage: '未捕获的异常'
        },
        501: {
            name: 'Not Implemented',
            description: '功能未实现',
            usage: '不支持的功能'
        },
        502: {
            name: 'Bad Gateway',
            description: '网关错误',
            usage: '上游服务器错误'
        },
        503: {
            name: 'Service Unavailable',
            description: '服务不可用',
            usage: '服务器过载或维护'
        },
        504: {
            name: 'Gateway Timeout',
            description: '网关超时',
            usage: '上游服务器超时'
        }
    }
};

// 打印状态码信息
function printStatusCode(code) {
    for (const category in httpStatusCodes) {
        if (httpStatusCodes[category][code]) {
            const info = httpStatusCodes[category][code];
            console.log(`状态码 ${code}:`);
            console.log(`  名称: ${info.name}`);
            console.log(`  描述: ${info.description}`);
            console.log(`  用途: ${info.usage}`);
            return;
        }
    }
    console.log(`未知状态码: ${code}`);
}

printStatusCode(200);
printStatusCode(404);
printStatusCode(500);
```

### 2.2 状态码处理

#### 状态码处理器
```javascript
/**
 * HTTP状态码处理器
 * @author erik.zhou
 */

class StatusCodeHandler {
    constructor() {
        this.handlers = new Map();
        this.setupDefaultHandlers();
    }
    
    // 设置默认处理器
    setupDefaultHandlers() {
        // 2xx 成功
        this.register(200, (response) => {
            console.log('请求成功');
            return response.data;
        });
        
        this.register(201, (response) => {
            console.log('资源创建成功');
            return response.data;
        });
        
        this.register(204, (response) => {
            console.log('操作成功，无返回内容');
            return null;
        });
        
        // 3xx 重定向
        this.register(301, (response) => {
            console.log('永久重定向:', response.headers.location);
            return this.redirect(response.headers.location);
        });
        
        this.register(302, (response) => {
            console.log('临时重定向:', response.headers.location);
            return this.redirect(response.headers.location);
        });
        
        this.register(304, (response) => {
            console.log('使用缓存');
            return this.getFromCache(response.url);
        });
        
        // 4xx 客户端错误
        this.register(400, (response) => {
            throw new Error('请求参数错误: ' + response.data.message);
        });
        
        this.register(401, (response) => {
            console.log('未授权，跳转到登录页');
            this.redirectToLogin();
            throw new Error('未授权');
        });
        
        this.register(403, (response) => {
            throw new Error('权限不足');
        });
        
        this.register(404, (response) => {
            throw new Error('资源不存在');
        });
        
        this.register(429, (response) => {
            const retryAfter = response.headers['retry-after'];
            console.log(`请求过多，${retryAfter}秒后重试`);
            throw new Error('请求过多，请稍后重试');
        });
        
        // 5xx 服务器错误
        this.register(500, (response) => {
            console.error('服务器内部错误');
            throw new Error('服务器错误，请稍后重试');
        });
        
        this.register(502, (response) => {
            console.error('网关错误');
            throw new Error('服务暂时不可用');
        });
        
        this.register(503, (response) => {
            console.error('服务不可用');
            throw new Error('服务维护中，请稍后重试');
        });
    }
    
    // 注册处理器
    register(statusCode, handler) {
        this.handlers.set(statusCode, handler);
    }
    
    // 处理响应
    handle(response) {
        const handler = this.handlers.get(response.status);
        
        if (handler) {
            return handler(response);
        }
        
        // 默认处理
        if (response.status >= 200 && response.status < 300) {
            return response.data;
        } else if (response.status >= 400) {
            throw new Error(`请求失败: ${response.status}`);
        }
        
        return response;
    }
    
    // 重定向
    redirect(url) {
        console.log('重定向到:', url);
        // 实际应用中会发起新请求
        return fetch(url);
    }
    
    // 从缓存获取
    getFromCache(url) {
        console.log('从缓存获取:', url);
        // 实际应用中会从缓存读取
        return null;
    }
    
    // 跳转到登录页
    redirectToLogin() {
        console.log('跳转到登录页');
        // 实际应用中会跳转
        // window.location.href = '/login';
    }
}

// 使用示例
const handler = new StatusCodeHandler();

// 模拟不同状态码的响应
const responses = [
    { status: 200, data: { id: 1, name: 'Erik' } },
    { status: 404, data: { message: '用户不存在' } },
    { status: 500, data: { message: '服务器错误' } }
];

responses.forEach(response => {
    try {
        const result = handler.handle(response);
        console.log('处理结果:', result);
    } catch (error) {
        console.error('处理失败:', error.message);
    }
});
```

#### 自定义状态码响应
```javascript
/**
 * 自定义状态码响应
 * @author erik.zhou
 */

class CustomStatusResponse {
    // 成功响应
    static success(data, message = '操作成功') {
        return {
            status: 200,
            headers: {
                'Content-Type': 'application/json'
            },
            body: {
                code: 0,
                message: message,
                data: data,
                timestamp: Date.now()
            }
        };
    }
    
    // 创建成功
    static created(data, message = '创建成功') {
        return {
            status: 201,
            headers: {
                'Content-Type': 'application/json',
                'Location': `/api/resource/${data.id}`
            },
            body: {
                code: 0,
                message: message,
                data: data,
                timestamp: Date.now()
            }
        };
    }
    
    // 无内容
    static noContent() {
        return {
            status: 204,
            headers: {},
            body: null
        };
    }
    
    // 参数错误
    static badRequest(message = '请求参数错误', errors = []) {
        return {
            status: 400,
            headers: {
                'Content-Type': 'application/json'
            },
            body: {
                code: 400,
                message: message,
                errors: errors,
                timestamp: Date.now()
            }
        };
    }
    
    // 未授权
    static unauthorized(message = '未授权，请先登录') {
        return {
            status: 401,
            headers: {
                'Content-Type': 'application/json',
                'WWW-Authenticate': 'Bearer realm="api"'
            },
            body: {
                code: 401,
                message: message,
                timestamp: Date.now()
            }
        };
    }
    
    // 禁止访问
    static forbidden(message = '权限不足') {
        return {
            status: 403,
            headers: {
                'Content-Type': 'application/json'
            },
            body: {
                code: 403,
                message: message,
                timestamp: Date.now()
            }
        };
    }
    
    // 资源不存在
    static notFound(message = '资源不存在') {
        return {
            status: 404,
            headers: {
                'Content-Type': 'application/json'
            },
            body: {
                code: 404,
                message: message,
                timestamp: Date.now()
            }
        };
    }
    
    // 请求过多
    static tooManyRequests(retryAfter = 60) {
        return {
            status: 429,
            headers: {
                'Content-Type': 'application/json',
                'Retry-After': retryAfter.toString()
            },
            body: {
                code: 429,
                message: '请求过多，请稍后重试',
                retryAfter: retryAfter,
                timestamp: Date.now()
            }
        };
    }
    
    // 服务器错误
    static internalError(message = '服务器内部错误') {
        return {
            status: 500,
            headers: {
                'Content-Type': 'application/json'
            },
            body: {
                code: 500,
                message: message,
                timestamp: Date.now()
            }
        };
    }
    
    // 服务不可用
    static serviceUnavailable(message = '服务暂时不可用') {
        return {
            status: 503,
            headers: {
                'Content-Type': 'application/json',
                'Retry-After': '300'
            },
            body: {
                code: 503,
                message: message,
                timestamp: Date.now()
            }
        };
    }
}

// 使用示例
console.log('成功响应:', CustomStatusResponse.success({ id: 1, name: 'Erik' }));
console.log('创建成功:', CustomStatusResponse.created({ id: 1 }));
console.log('参数错误:', CustomStatusResponse.badRequest('用户名不能为空', [
    { field: 'username', message: '必填字段' }
]));
console.log('未授权:', CustomStatusResponse.unauthorized());
console.log('资源不存在:', CustomStatusResponse.notFound());
```

## 第三部分：HTTP Headers

### 3.1 请求头

#### 常用请求头
```javascript
/**
 * HTTP请求头详解
 * @author erik.zhou
 */

class RequestHeaders {
    constructor() {
        this.headers = {};
    }
    
    // Accept - 可接受的内容类型
    setAccept(type) {
        const acceptTypes = {
            json: 'application/json',
            xml: 'application/xml',
            html: 'text/html',
            text: 'text/plain',
            any: '*/*'
        };
        
        this.headers['Accept'] = acceptTypes[type] || type;
        return this;
    }
    
    // Accept-Encoding - 可接受的编码
    setAcceptEncoding(encodings = ['gzip', 'deflate', 'br']) {
        this.headers['Accept-Encoding'] = encodings.join(', ');
        return this;
    }
    
    // Accept-Language - 可接受的语言
    setAcceptLanguage(languages = ['zh-CN', 'zh', 'en']) {
        this.headers['Accept-Language'] = languages.join(', ');
        return this;
    }
    
    // Authorization - 认证信息
    setAuthorization(type, token) {
        const authTypes = {
            bearer: `Bearer ${token}`,
            basic: `Basic ${btoa(token)}`,
            digest: `Digest ${token}`
        };
        
        this.headers['Authorization'] = authTypes[type] || token;
        return this;
    }
    
    // Content-Type - 请求体类型
    setContentType(type) {
        const contentTypes = {
            json: 'application/json',
            form: 'application/x-www-form-urlencoded',
            multipart: 'multipart/form-data',
            text: 'text/plain'
        };
        
        this.headers['Content-Type'] = contentTypes[type] || type;
        return this;
    }
    
    // Cookie - Cookie信息
    setCookie(cookies) {
        if (typeof cookies === 'object') {
            this.headers['Cookie'] = Object.entries(cookies)
                .map(([key, value]) => `${key}=${value}`)
                .join('; ');
        } else {
            this.headers['Cookie'] = cookies;
        }
        return this;
    }
    
    // User-Agent - 用户代理
    setUserAgent(ua) {
        this.headers['User-Agent'] = ua || 
            'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36';
        return this;
    }
    
    // Referer - 来源页面
    setReferer(url) {
        this.headers['Referer'] = url;
        return this;
    }
    
    // Cache-Control - 缓存控制
    setCacheControl(directive) {
        this.headers['Cache-Control'] = directive;
        return this;
    }
    
    // If-None-Match - 条件请求（ETag）
    setIfNoneMatch(etag) {
        this.headers['If-None-Match'] = etag;
        return this;
    }
    
    // If-Modified-Since - 条件请求（时间）
    setIfModifiedSince(date) {
        this.headers['If-Modified-Since'] = 
            date instanceof Date ? date.toUTCString() : date;
        return this;
    }
    
    // Range - 范围请求
    setRange(start, end) {
        this.headers['Range'] = `bytes=${start}-${end || ''}`;
        return this;
    }
    
    // 自定义头
    setCustom(key, value) {
        this.headers[key] = value;
        return this;
    }
    
    // 获取所有头
    getAll() {
        return { ...this.headers };
    }
    
    // 构建fetch选项
    buildFetchOptions(method = 'GET', body = null) {
        const options = {
            method: method,
            headers: this.headers
        };
        
        if (body) {
            options.body = typeof body === 'string' ? body : JSON.stringify(body);
        }
        
        return options;
    }
}

// 使用示例
const headers = new RequestHeaders()
    .setAccept('json')
    .setAcceptEncoding()
    .setAcceptLanguage(['zh-CN', 'en'])
    .setAuthorization('bearer', 'your-token-here')
    .setContentType('json')
    .setUserAgent()
    .setCacheControl('no-cache');

console.log('请求头:', headers.getAll());

// 发送请求
const fetchOptions = headers.buildFetchOptions('POST', { name: 'Erik' });
console.log('Fetch选项:', fetchOptions);
```

### 3.2 响应头

#### 常用响应头
```javascript
/**
 * HTTP响应头详解
 * @author erik.zhou
 */

class ResponseHeaders {
    constructor() {
        this.headers = {};
    }
    
    // Content-Type - 响应内容类型
    setContentType(type, charset = 'utf-8') {
        const contentTypes = {
            json: 'application/json',
            html: 'text/html',
            xml: 'application/xml',
            text: 'text/plain',
            stream: 'application/octet-stream'
        };
        
        const contentType = contentTypes[type] || type;
        this.headers['Content-Type'] = `${contentType}; charset=${charset}`;
        return this;
    }
    
    // Content-Length - 响应体长度
    setContentLength(length) {
        this.headers['Content-Length'] = length.toString();
        return this;
    }
    
    // Content-Encoding - 内容编码
    setContentEncoding(encoding) {
        this.headers['Content-Encoding'] = encoding;
        return this;
    }
    
    // Cache-Control - 缓存控制
    setCacheControl(directive) {
        this.headers['Cache-Control'] = directive;
        return this;
    }
    
    // ETag - 资源标识
    setETag(etag) {
        this.headers['ETag'] = `"${etag}"`;
        return this;
    }
    
    // Last-Modified - 最后修改时间
    setLastModified(date) {
        this.headers['Last-Modified'] = 
            date instanceof Date ? date.toUTCString() : date;
        return this;
    }
    
    // Expires - 过期时间
    setExpires(date) {
        this.headers['Expires'] = 
            date instanceof Date ? date.toUTCString() : date;
        return this;
    }
    
    // Location - 重定向地址
    setLocation(url) {
        this.headers['Location'] = url;
        return this;
    }
    
    // Set-Cookie - 设置Cookie
    setCookie(name, value, options = {}) {
        let cookie = `${name}=${value}`;
        
        if (options.maxAge) {
            cookie += `; Max-Age=${options.maxAge}`;
        }
        
        if (options.expires) {
            const expires = options.expires instanceof Date 
                ? options.expires.toUTCString() 
                : options.expires;
            cookie += `; Expires=${expires}`;
        }
        
        if (options.domain) {
            cookie += `; Domain=${options.domain}`;
        }
        
        if (options.path) {
            cookie += `; Path=${options.path}`;
        }
        
        if (options.secure) {
            cookie += '; Secure';
        }
        
        if (options.httpOnly) {
            cookie += '; HttpOnly';
        }
        
        if (options.sameSite) {
            cookie += `; SameSite=${options.sameSite}`;
        }
        
        this.headers['Set-Cookie'] = cookie;
        return this;
    }
    
    // Access-Control-Allow-Origin - CORS
    setAccessControlAllowOrigin(origin = '*') {
        this.headers['Access-Control-Allow-Origin'] = origin;
        return this;
    }
    
    // Access-Control-Allow-Methods - 允许的方法
    setAccessControlAllowMethods(methods) {
        this.headers['Access-Control-Allow-Methods'] = 
            Array.isArray(methods) ? methods.join(', ') : methods;
        return this;
    }
    
    // Access-Control-Allow-Headers - 允许的头
    setAccessControlAllowHeaders(headers) {
        this.headers['Access-Control-Allow-Headers'] = 
            Array.isArray(headers) ? headers.join(', ') : headers;
        return this;
    }
    
    // Access-Control-Max-Age - 预检缓存时间
    setAccessControlMaxAge(seconds) {
        this.headers['Access-Control-Max-Age'] = seconds.toString();
        return this;
    }
    
    // X-Content-Type-Options - 防止MIME嗅探
    setXContentTypeOptions() {
        this.headers['X-Content-Type-Options'] = 'nosniff';
        return this;
    }
    
    // X-Frame-Options - 防止点击劫持
    setXFrameOptions(option = 'DENY') {
        this.headers['X-Frame-Options'] = option;
        return this;
    }
    
    // X-XSS-Protection - XSS防护
    setXXSSProtection() {
        this.headers['X-XSS-Protection'] = '1; mode=block';
        return this;
    }
    
    // Strict-Transport-Security - HSTS
    setStrictTransportSecurity(maxAge = 31536000) {
        this.headers['Strict-Transport-Security'] = 
            `max-age=${maxAge}; includeSubDomains`;
        return this;
    }
    
    // Content-Security-Policy - CSP
    setContentSecurityPolicy(policy) {
        this.headers['Content-Security-Policy'] = policy;
        return this;
    }
    
    // 自定义头
    setCustom(key, value) {
        this.headers[key] = value;
        return this;
    }
    
    // 获取所有头
    getAll() {
        return { ...this.headers };
    }
}

// 使用示例
const responseHeaders = new ResponseHeaders()
    .setContentType('json')
    .setCacheControl('public, max-age=3600')
    .setETag('abc123')
    .setAccessControlAllowOrigin('https://example.com')
    .setAccessControlAllowMethods(['GET', 'POST', 'PUT', 'DELETE'])
    .setXContentTypeOptions()
    .setXFrameOptions('SAMEORIGIN')
    .setStrictTransportSecurity();

console.log('响应头:', responseHeaders.getAll());

// 设置Cookie
const cookieHeaders = new ResponseHeaders()
    .setCookie('session_id', 'abc123', {
        maxAge: 3600,
        path: '/',
        httpOnly: true,
        secure: true,
        sameSite: 'Strict'
    });

console.log('Cookie响应头:', cookieHeaders.getAll());
```

### 3.3 自定义Headers

#### Headers管理器
```javascript
/**
 * HTTP Headers管理器
 * @author erik.zhou
 */

class HeadersManager {
    constructor() {
        this.defaultHeaders = new Map();
        this.interceptors = [];
    }
    
    // 设置默认头
    setDefault(key, value) {
        this.defaultHeaders.set(key, value);
    }
    
    // 删除默认头
    removeDefault(key) {
        this.defaultHeaders.delete(key);
    }
    
    // 添加拦截器
    addInterceptor(interceptor) {
        this.interceptors.push(interceptor);
    }
    
    // 构建最终headers
    build(customHeaders = {}) {
        // 合并默认headers
        let headers = {
            ...Object.fromEntries(this.defaultHeaders),
            ...customHeaders
        };
        
        // 执行拦截器
        for (const interceptor of this.interceptors) {
            headers = interceptor(headers);
        }
        
        return headers;
    }
    
    // 创建请求配置
    createRequestConfig(url, options = {}) {
        const headers = this.build(options.headers);
        
        return {
            ...options,
            headers: headers
        };
    }
}

// 使用示例
const headersManager = new HeadersManager();

// 设置默认headers
headersManager.setDefault('Content-Type', 'application/json');
headersManager.setDefault('Accept', 'application/json');

// 添加认证拦截器
headersManager.addInterceptor((headers) => {
    const token = localStorage.getItem('token');
    if (token) {
        headers['Authorization'] = `Bearer ${token}`;
    }
    return headers;
});

// 添加时间戳拦截器
headersManager.addInterceptor((headers) => {
    headers['X-Request-Time'] = Date.now().toString();
    return headers;
});

// 添加请求ID拦截器
headersManager.addInterceptor((headers) => {
    headers['X-Request-ID'] = Math.random().toString(36).substring(7);
    return headers;
});

// 构建请求配置
const config = headersManager.createRequestConfig('https://api.example.com/users', {
    method: 'GET',
    headers: {
        'X-Custom-Header': 'custom-value'
    }
});

console.log('请求配置:', config);
```

## 第四部分：HTTP/2

### 4.1 HTTP/2特性

#### 多路复用
```javascript
/**
 * HTTP/2多路复用示例
 * @author erik.zhou
 */

class HTTP2Multiplexing {
    constructor() {
        this.streams = new Map();
        this.nextStreamId = 1;
    }
    
    // 创建流
    createStream(request) {
        const streamId = this.nextStreamId;
        this.nextStreamId += 2; // 客户端流ID为奇数
        
        const stream = {
            id: streamId,
            request: request,
            state: 'idle',
            priority: request.priority || 0,
            dependencies: request.dependencies || [],
            startTime: Date.now()
        };
        
        this.streams.set(streamId, stream);
        return stream;
    }
    
    // 发送多个请求
    async sendMultipleRequests(requests) {
        console.log(`=== HTTP/2多路复用：同时发送${requests.length}个请求 ===`);
        
        // 创建所有流
        const streams = requests.map(req => this.createStream(req));
        
        // 并行发送所有请求
        const promises = streams.map(stream => this.sendStream(stream));
        
        // 等待所有响应
        const responses = await Promise.all(promises);
        
        console.log('所有请求完成');
        return responses;
    }
    
    // 发送单个流
    async sendStream(stream) {
        console.log(`流 ${stream.id}: 发送请求 ${stream.request.url}`);
        stream.state = 'open';
        
        // 模拟网络延迟
        await this.delay(Math.random() * 1000);
        
        const response = {
            streamId: stream.id,
            status: 200,
            headers: {
                ':status': '200',
                'content-type': 'application/json'
            },
            data: { message: `Response for stream ${stream.id}` },
            duration: Date.now() - stream.startTime
        };
        
        stream.state = 'closed';
        console.log(`流 ${stream.id}: 接收响应 (${response.duration}ms)`);
        
        return response;
    }
    
    // HTTP/1.1对比（串行请求）
    async sendSequentialRequests(requests) {
        console.log(`=== HTTP/1.1串行：依次发送${requests.length}个请求 ===`);
        
        const responses = [];
        const startTime = Date.now();
        
        for (const request of requests) {
            console.log(`发送请求: ${request.url}`);
            await this.delay(Math.random() * 1000);
            
            responses.push({
                url: request.url,
                status: 200,
                data: { message: 'Response' }
            });
            
            console.log(`接收响应: ${request.url}`);
        }
        
        const totalTime = Date.now() - startTime;
        console.log(`总耗时: ${totalTime}ms`);
        
        return responses;
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
const http2 = new HTTP2Multiplexing();

const requests = [
    { url: '/api/user', priority: 1 },
    { url: '/api/posts', priority: 2 },
    { url: '/api/comments', priority: 3 },
    { url: '/api/likes', priority: 4 }
];

// HTTP/2多路复用
http2.sendMultipleRequests(requests)
    .then(responses => {
        console.log('HTTP/2响应:', responses);
    });

// HTTP/1.1串行请求（对比）
// http2.sendSequentialRequests(requests);
```

#### 服务器推送
```javascript
/**
 * HTTP/2服务器推送示例
 * @author erik.zhou
 */

class HTTP2ServerPush {
    constructor() {
        this.pushCache = new Map();
    }
    
    // 服务器推送资源
    async pushResources(mainResource, relatedResources) {
        console.log('=== HTTP/2服务器推送 ===');
        console.log(`主资源: ${mainResource}`);
        
        // 推送相关资源
        const pushPromises = relatedResources.map(resource => {
            return this.push(resource);
        });
        
        // 并行推送
        await Promise.all(pushPromises);
        
        console.log('所有资源推送完成');
    }
    
    // 推送单个资源
    async push(resource) {
        console.log(`推送资源: ${resource.url}`);
        
        // 模拟推送延迟
        await this.delay(100);
        
        // 缓存推送的资源
        this.pushCache.set(resource.url, {
            url: resource.url,
            type: resource.type,
            data: `Content of ${resource.url}`,
            pushedAt: Date.now()
        });
        
        console.log(`资源已推送并缓存: ${resource.url}`);
    }
    
    // 客户端请求资源
    async requestResource(url) {
        // 检查推送缓存
        if (this.pushCache.has(url)) {
            console.log(`使用推送缓存: ${url}`);
            return this.pushCache.get(url);
        }
        
        // 正常请求
        console.log(`发起请求: ${url}`);
        await this.delay(500);
        
        return {
            url: url,
            data: `Content of ${url}`,
            requestedAt: Date.now()
        };
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
const serverPush = new HTTP2ServerPush();

// 服务器推送场景
async function demonstrateServerPush() {
    // 请求HTML页面时，服务器主动推送CSS和JS
    await serverPush.pushResources('/index.html', [
        { url: '/styles/main.css', type: 'text/css' },
        { url: '/scripts/app.js', type: 'application/javascript' },
        { url: '/images/logo.png', type: 'image/png' }
    ]);
    
    // 客户端后续请求这些资源时，直接使用缓存
    console.log('\n=== 客户端请求资源 ===');
    await serverPush.requestResource('/styles/main.css');
    await serverPush.requestResource('/scripts/app.js');
    await serverPush.requestResource('/images/logo.png');
}

demonstrateServerPush();
```

#### 头部压缩（HPACK）
```javascript
/**
 * HTTP/2头部压缩（HPACK）示例
 * @author erik.zhou
 */

class HPACKCompression {
    constructor() {
        // 静态表（部分常用头部）
        this.staticTable = [
            [':authority', ''],
            [':method', 'GET'],
            [':method', 'POST'],
            [':path', '/'],
            [':scheme', 'https'],
            ['accept', '*/*'],
            ['accept-encoding', 'gzip, deflate'],
            ['content-type', 'application/json'],
            ['user-agent', '']
        ];
        
        // 动态表
        this.dynamicTable = [];
        this.maxDynamicTableSize = 4096;
    }
    
    // 编码头部
    encode(headers) {
        console.log('=== HPACK编码 ===');
        console.log('原始头部:', headers);
        
        const encoded = [];
        let originalSize = 0;
        let compressedSize = 0;
        
        for (const [name, value] of Object.entries(headers)) {
            originalSize += name.length + value.length + 2; // +2 for ": "
            
            // 查找静态表
            const staticIndex = this.findInStaticTable(name, value);
            if (staticIndex !== -1) {
                encoded.push({ type: 'indexed', index: staticIndex });
                compressedSize += 1; // 索引编码只需1字节
                console.log(`  ${name}: ${value} -> 静态表索引 ${staticIndex}`);
                continue;
            }
            
            // 查找动态表
            const dynamicIndex = this.findInDynamicTable(name, value);
            if (dynamicIndex !== -1) {
                encoded.push({ type: 'indexed', index: dynamicIndex + this.staticTable.length });
                compressedSize += 1;
                console.log(`  ${name}: ${value} -> 动态表索引 ${dynamicIndex}`);
                continue;
            }
            
            // 新增到动态表
            this.addToDynamicTable(name, value);
            encoded.push({ type: 'literal', name: name, value: value });
            compressedSize += Math.ceil((name.length + value.length) / 2); // 简化计算
            console.log(`  ${name}: ${value} -> 字面量（已加入动态表）`);
        }
        
        const compressionRatio = ((1 - compressedSize / originalSize) * 100).toFixed(2);
        console.log(`\n压缩前: ${originalSize} 字节`);
        console.log(`压缩后: ${compressedSize} 字节`);
        console.log(`压缩率: ${compressionRatio}%`);
        
        return encoded;
    }
    
    // 解码头部
    decode(encoded) {
        console.log('\n=== HPACK解码 ===');
        
        const headers = {};
        
        for (const entry of encoded) {
            if (entry.type === 'indexed') {
                const [name, value] = this.getFromTable(entry.index);
                headers[name] = value;
                console.log(`  索引 ${entry.index} -> ${name}: ${value}`);
            } else if (entry.type === 'literal') {
                headers[entry.name] = entry.value;
                console.log(`  字面量 -> ${entry.name}: ${entry.value}`);
            }
        }
        
        console.log('解码后的头部:', headers);
        return headers;
    }
    
    // 在静态表中查找
    findInStaticTable(name, value) {
        return this.staticTable.findIndex(
            ([n, v]) => n === name && (v === value || v === '')
        );
    }
    
    // 在动态表中查找
    findInDynamicTable(name, value) {
        return this.dynamicTable.findIndex(
            ([n, v]) => n === name && v === value
        );
    }
    
    // 添加到动态表
    addToDynamicTable(name, value) {
        this.dynamicTable.unshift([name, value]);
        
        // 限制动态表大小
        let size = 0;
        for (let i = 0; i < this.dynamicTable.length; i++) {
            size += this.dynamicTable[i][0].length + this.dynamicTable[i][1].length + 32;
            if (size > this.maxDynamicTableSize) {
                this.dynamicTable = this.dynamicTable.slice(0, i);
                break;
            }
        }
    }
    
    // 从表中获取
    getFromTable(index) {
        if (index < this.staticTable.length) {
            return this.staticTable[index];
        }
        return this.dynamicTable[index - this.staticTable.length];
    }
}

// 使用示例
const hpack = new HPACKCompression();

// 第一个请求
const headers1 = {
    ':method': 'GET',
    ':path': '/api/users',
    ':scheme': 'https',
    ':authority': 'api.example.com',
    'accept': '*/*',
    'accept-encoding': 'gzip, deflate',
    'user-agent': 'Mozilla/5.0',
    'authorization': 'Bearer token123'
};

const encoded1 = hpack.encode(headers1);
const decoded1 = hpack.decode(encoded1);

// 第二个请求（复用动态表）
console.log('\n\n=== 第二个请求（复用动态表）===');
const headers2 = {
    ':method': 'POST',
    ':path': '/api/users',
    ':scheme': 'https',
    ':authority': 'api.example.com',
    'accept': '*/*',
    'accept-encoding': 'gzip, deflate',
    'content-type': 'application/json',
    'authorization': 'Bearer token123' // 复用动态表
};

const encoded2 = hpack.encode(headers2);
const decoded2 = hpack.decode(encoded2);
```

### 4.2 HTTP/2优化

#### 流优先级
```javascript
/**
 * HTTP/2流优先级管理
 * @author erik.zhou
 */

class StreamPriority {
    constructor() {
        this.streams = new Map();
        this.priorityQueue = [];
    }
    
    // 创建流
    createStream(id, request, priority = 0, dependency = null) {
        const stream = {
            id: id,
            request: request,
            priority: priority,
            dependency: dependency,
            weight: this.calculateWeight(priority),
            state: 'idle'
        };
        
        this.streams.set(id, stream);
        this.addToPriorityQueue(stream);
        
        return stream;
    }
    
    // 计算权重
    calculateWeight(priority) {
        // 优先级越高，权重越大
        return Math.max(1, Math.min(256, priority * 10));
    }
    
    // 添加到优先级队列
    addToPriorityQueue(stream) {
        this.priorityQueue.push(stream);
        this.priorityQueue.sort((a, b) => {
            // 先按依赖关系排序
            if (a.dependency === b.id) return 1;
            if (b.dependency === a.id) return -1;
            
            // 再按优先级排序
            return b.priority - a.priority;
        });
    }
    
    // 获取下一个要处理的流
    getNextStream() {
        for (const stream of this.priorityQueue) {
            if (stream.state === 'idle') {
                // 检查依赖是否完成
                if (stream.dependency) {
                    const depStream = this.streams.get(stream.dependency);
                    if (depStream && depStream.state !== 'closed') {
                        continue;
                    }
                }
                
                return stream;
            }
        }
        
        return null;
    }
    
    // 处理流
    async processStreams() {
        console.log('=== 按优先级处理流 ===');
        
        while (true) {
            const stream = this.getNextStream();
            if (!stream) break;
            
            console.log(`处理流 ${stream.id} (优先级: ${stream.priority}, 权重: ${stream.weight})`);
            stream.state = 'open';
            
            // 模拟处理
            await this.delay(100);
            
            stream.state = 'closed';
            console.log(`流 ${stream.id} 完成`);
        }
        
        console.log('所有流处理完成');
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
const priority = new StreamPriority();

// 创建不同优先级的流
priority.createStream(1, { url: '/index.html' }, 10); // 最高优先级
priority.createStream(3, { url: '/styles/main.css' }, 8, 1); // 依赖HTML
priority.createStream(5, { url: '/scripts/app.js' }, 8, 1); // 依赖HTML
priority.createStream(7, { url: '/api/data' }, 5); // 中等优先级
priority.createStream(9, { url: '/images/bg.jpg' }, 2); // 低优先级

// 处理流
priority.processStreams();
```

## 第五部分：HTTP/3

### 5.1 HTTP/3特性

#### QUIC协议
```javascript
/**
 * HTTP/3 QUIC协议特性示例
 * @author erik.zhou
 */

class QUICProtocol {
    constructor() {
        this.connections = new Map();
        this.packets = [];
    }
    
    // 建立QUIC连接
    async establishConnection(host) {
        console.log('=== QUIC连接建立 ===');
        console.log(`连接到: ${host}`);
        
        const connectionId = this.generateConnectionId();
        
        // 0-RTT或1-RTT握手
        const hasSessionTicket = this.hasSessionTicket(host);
        
        if (hasSessionTicket) {
            console.log('使用0-RTT握手（复用会话）');
            await this.zeroRTTHandshake(connectionId, host);
        } else {
            console.log('使用1-RTT握手（首次连接）');
            await this.oneRTTHandshake(connectionId, host);
        }
        
        const connection = {
            id: connectionId,
            host: host,
            state: 'established',
            streams: new Map(),
            createdAt: Date.now()
        };
        
        this.connections.set(connectionId, connection);
        console.log(`连接已建立: ${connectionId}`);
        
        return connection;
    }
    
    // 1-RTT握手
    async oneRTTHandshake(connectionId, host) {
        console.log('1. 客户端发送Initial包（包含TLS ClientHello）');
        await this.delay(50);
        
        console.log('2. 服务器返回Initial包（包含TLS ServerHello）');
        await this.delay(50);
        
        console.log('3. 完成TLS握手');
        await this.delay(50);
        
        // 保存会话票据
        this.saveSessionTicket(host);
    }
    
    // 0-RTT握手
    async zeroRTTHandshake(connectionId, host) {
        console.log('1. 客户端发送0-RTT数据（包含应用数据）');
        await this.delay(25);
        
        console.log('2. 服务器确认并返回数据');
        await this.delay(25);
    }
    
    // 连接迁移
    async migrateConnection(connectionId, newPath) {
        console.log('\n=== QUIC连接迁移 ===');
        const connection = this.connections.get(connectionId);
        
        if (!connection) {
            throw new Error('连接不存在');
        }
        
        console.log(`从 ${connection.path || 'default'} 迁移到 ${newPath}`);
        console.log('场景：WiFi切换到4G，或IP地址变更');
        
        // 发送PATH_CHALLENGE
        console.log('1. 发送PATH_CHALLENGE验证新路径');
        await this.delay(50);
        
        // 接收PATH_RESPONSE
        console.log('2. 接收PATH_RESPONSE确认');
        await this.delay(50);
        
        connection.path = newPath;
        console.log('连接迁移完成，无需重新握手');
    }
    
    // 丢包恢复
    async handlePacketLoss(connectionId, lostPackets) {
        console.log('\n=== QUIC丢包恢复 ===');
        console.log(`检测到丢包: ${lostPackets.length} 个包`);
        
        for (const packet of lostPackets) {
            console.log(`重传包 ${packet.id} (流 ${packet.streamId})`);
            await this.retransmitPacket(packet);
        }
        
        console.log('丢包恢复完成');
    }
    
    // 重传数据包
    async retransmitPacket(packet) {
        // QUIC使用新的包号重传，避免重传歧义
        const newPacket = {
            ...packet,
            id: this.generatePacketId(),
            retransmission: true,
            originalId: packet.id
        };
        
        this.packets.push(newPacket);
        await this.delay(20);
    }
    
    // 生成连接ID
    generateConnectionId() {
        return Math.random().toString(36).substring(2, 10);
    }
    
    // 生成包ID
    generatePacketId() {
        return this.packets.length + 1;
    }
    
    // 检查会话票据
    hasSessionTicket(host) {
        // 模拟检查
        return Math.random() > 0.5;
    }
    
    // 保存会话票据
    saveSessionTicket(host) {
        console.log(`保存会话票据: ${host}`);
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
const quic = new QUICProtocol();

async function demonstrateQUIC() {
    // 建立连接
    const conn = await quic.establishConnection('api.example.com');
    
    // 模拟连接迁移（网络切换）
    await quic.migrateConnection(conn.id, '4G');
    
    // 模拟丢包恢复
    const lostPackets = [
        { id: 1, streamId: 1, data: 'packet1' },
        { id: 3, streamId: 2, data: 'packet3' }
    ];
    await quic.handlePacketLoss(conn.id, lostPackets);
}

demonstrateQUIC();
```

### 5.2 HTTP/3优势

#### 性能对比
```javascript
/**
 * HTTP/1.1 vs HTTP/2 vs HTTP/3 性能对比
 * @author erik.zhou
 */

class HTTPVersionComparison {
    constructor() {
        this.results = {
            http1: [],
            http2: [],
            http3: []
        };
    }
    
    // HTTP/1.1 性能测试
    async testHTTP1(requests) {
        console.log('=== HTTP/1.1 测试 ===');
        const startTime = Date.now();
        
        // 串行请求（或最多6个并发连接）
        const maxConnections = 6;
        const results = [];
        
        for (let i = 0; i < requests.length; i += maxConnections) {
            const batch = requests.slice(i, i + maxConnections);
            const batchResults = await Promise.all(
                batch.map(req => this.simulateRequest(req, 'HTTP/1.1'))
            );
            results.push(...batchResults);
        }
        
        const totalTime = Date.now() - startTime;
        console.log(`总耗时: ${totalTime}ms`);
        console.log(`平均延迟: ${(totalTime / requests.length).toFixed(2)}ms`);
        
        this.results.http1 = { totalTime, results };
        return this.results.http1;
    }
    
    // HTTP/2 性能测试
    async testHTTP2(requests) {
        console.log('\n=== HTTP/2 测试 ===');
        const startTime = Date.now();
        
        // 多路复用，所有请求并行
        const results = await Promise.all(
            requests.map(req => this.simulateRequest(req, 'HTTP/2'))
        );
        
        const totalTime = Date.now() - startTime;
        console.log(`总耗时: ${totalTime}ms`);
        console.log(`平均延迟: ${(totalTime / requests.length).toFixed(2)}ms`);
        console.log(`性能提升: ${((1 - totalTime / this.results.http1.totalTime) * 100).toFixed(2)}%`);
        
        this.results.http2 = { totalTime, results };
        return this.results.http2;
    }
    
    // HTTP/3 性能测试
    async testHTTP3(requests) {
        console.log('\n=== HTTP/3 测试 ===');
        const startTime = Date.now();
        
        // QUIC协议，更快的连接建立和丢包恢复
        const results = await Promise.all(
            requests.map(req => this.simulateRequest(req, 'HTTP/3', true))
        );
        
        const totalTime = Date.now() - startTime;
        console.log(`总耗时: ${totalTime}ms`);
        console.log(`平均延迟: ${(totalTime / requests.length).toFixed(2)}ms`);
        console.log(`相比HTTP/1.1提升: ${((1 - totalTime / this.results.http1.totalTime) * 100).toFixed(2)}%`);
        console.log(`相比HTTP/2提升: ${((1 - totalTime / this.results.http2.totalTime) * 100).toFixed(2)}%`);
        
        this.results.http3 = { totalTime, results };
        return this.results.http3;
    }
    
    // 模拟请求
    async simulateRequest(request, protocol, fastRecovery = false) {
        const baseDelay = 100;
        const protocolDelay = {
            'HTTP/1.1': 1.0,
            'HTTP/2': 0.7,
            'HTTP/3': 0.5
        };
        
        const delay = baseDelay * protocolDelay[protocol];
        
        // 模拟丢包
        if (Math.random() < 0.1 && !fastRecovery) {
            await this.delay(delay * 2); // 丢包重传
        } else {
            await this.delay(delay);
        }
        
        return {
            url: request.url,
            protocol: protocol,
            time: delay
        };
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
const comparison = new HTTPVersionComparison();

const requests = [
    { url: '/api/user' },
    { url: '/api/posts' },
    { url: '/api/comments' },
    { url: '/api/likes' },
    { url: '/styles/main.css' },
    { url: '/scripts/app.js' },
    { url: '/images/logo.png' },
    { url: '/images/banner.jpg' }
];

async function runComparison() {
    await comparison.testHTTP1(requests);
    await comparison.testHTTP2(requests);
    await comparison.testHTTP3(requests);
}

runComparison();
```

## 第六部分：HTTP性能优化

### 6.1 连接优化

#### 连接复用
```javascript
/**
 * HTTP连接复用
 * @author erik.zhou
 */

class ConnectionPool {
    constructor(maxConnections = 6) {
        this.maxConnections = maxConnections;
        this.connections = new Map();
        this.queue = [];
    }
    
    // 获取连接
    async getConnection(host) {
        const key = this.getConnectionKey(host);
        
        // 检查是否有可用连接
        if (this.connections.has(key)) {
            const conn = this.connections.get(key);
            if (conn.isAvailable()) {
                console.log(`复用连接: ${key}`);
                return conn;
            }
        }
        
        // 检查连接数限制
        if (this.connections.size >= this.maxConnections) {
            console.log('连接池已满，等待可用连接');
            return await this.waitForConnection(key);
        }
        
        // 创建新连接
        console.log(`创建新连接: ${key}`);
        const conn = await this.createConnection(host);
        this.connections.set(key, conn);
        
        return conn;
    }
    
    // 创建连接
    async createConnection(host) {
        // 模拟TCP握手
        await this.delay(100);
        
        return {
            host: host,
            createdAt: Date.now(),
            requests: 0,
            maxRequests: 100, // HTTP/2可以更多
            
            isAvailable() {
                return this.requests < this.maxRequests;
            },
            
            async send(request) {
                this.requests++;
                console.log(`发送请求 #${this.requests}: ${request.url}`);
                await new Promise(resolve => setTimeout(resolve, 50));
                return { status: 200, data: 'Response' };
            },
            
            close() {
                console.log(`关闭连接: ${host}`);
            }
        };
    }
    
    // 等待可用连接
    async waitForConnection(key) {
        return new Promise((resolve) => {
            this.queue.push({ key, resolve });
        });
    }
    
    // 释放连接
    releaseConnection(host) {
        const key = this.getConnectionKey(host);
        
        // 检查队列中是否有等待的请求
        const waiting = this.queue.find(item => item.key === key);
        if (waiting) {
            const conn = this.connections.get(key);
            waiting.resolve(conn);
            this.queue = this.queue.filter(item => item !== waiting);
        }
    }
    
    // 获取连接键
    getConnectionKey(host) {
        return host;
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
const pool = new ConnectionPool(3);

async function sendRequests() {
    const requests = [
        { url: 'https://api.example.com/users' },
        { url: 'https://api.example.com/posts' },
        { url: 'https://api.example.com/comments' },
        { url: 'https://api.example.com/likes' }
    ];
    
    for (const request of requests) {
        const conn = await pool.getConnection('api.example.com');
        await conn.send(request);
        pool.releaseConnection('api.example.com');
    }
}

sendRequests();
```

### 6.2 缓存优化

#### HTTP缓存策略
```javascript
/**
 * HTTP缓存策略实现
 * @author erik.zhou
 */

class HTTPCache {
    constructor() {
        this.cache = new Map();
    }
    
    // 强缓存检查
    checkStrongCache(url) {
        const cached = this.cache.get(url);
        
        if (!cached) {
            return null;
        }
        
        // 检查Cache-Control
        if (cached.cacheControl) {
            const maxAge = this.parseMaxAge(cached.cacheControl);
            const age = (Date.now() - cached.cachedAt) / 1000;
            
            if (age < maxAge) {
                console.log(`强缓存命中: ${url} (剩余 ${(maxAge - age).toFixed(0)}s)`);
                return cached.response;
            }
        }
        
        // 检查Expires
        if (cached.expires) {
            const expiresTime = new Date(cached.expires).getTime();
            if (Date.now() < expiresTime) {
                console.log(`Expires缓存命中: ${url}`);
                return cached.response;
            }
        }
        
        return null;
    }
    
    // 协商缓存检查
    async checkNegotiatedCache(url, request) {
        const cached = this.cache.get(url);
        
        if (!cached) {
            return null;
        }
        
        // 添加条件请求头
        if (cached.etag) {
            request.headers['If-None-Match'] = cached.etag;
        }
        
        if (cached.lastModified) {
            request.headers['If-Modified-Since'] = cached.lastModified;
        }
        
        // 发送条件请求
        const response = await this.sendRequest(url, request);
        
        if (response.status === 304) {
            console.log(`协商缓存命中: ${url} (304 Not Modified)`);
            return cached.response;
        }
        
        return response;
    }
    
    // 缓存响应
    cacheResponse(url, response) {
        const cacheEntry = {
            url: url,
            response: response,
            cachedAt: Date.now(),
            cacheControl: response.headers['Cache-Control'],
            expires: response.headers['Expires'],
            etag: response.headers['ETag'],
            lastModified: response.headers['Last-Modified']
        };
        
        this.cache.set(url, cacheEntry);
        console.log(`缓存响应: ${url}`);
    }
    
    // 解析max-age
    parseMaxAge(cacheControl) {
        const match = cacheControl.match(/max-age=(\d+)/);
        return match ? parseInt(match[1]) : 0;
    }
    
    // 发送请求
    async sendRequest(url, request) {
        console.log(`发送请求: ${url}`);
        
        // 模拟网络请求
        await this.delay(100);
        
        return {
            status: 200,
            headers: {
                'Cache-Control': 'public, max-age=3600',
                'ETag': '"abc123"',
                'Last-Modified': new Date().toUTCString()
            },
            data: { message: 'Response data' }
        };
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
const cache = new HTTPCache();

async function demonstrateCache() {
    const url = 'https://api.example.com/data';
    
    // 第一次请求
    console.log('=== 第一次请求 ===');
    let response = await cache.sendRequest(url, {});
    cache.cacheResponse(url, response);
    
    // 第二次请求（强缓存）
    console.log('\n=== 第二次请求（强缓存）===');
    let cached = cache.checkStrongCache(url);
    if (!cached) {
        response = await cache.sendRequest(url, {});
    }
    
    // 模拟缓存过期
    console.log('\n=== 缓存过期后请求（协商缓存）===');
    const cacheEntry = cache.cache.get(url);
    cacheEntry.cachedAt = Date.now() - 3700000; // 超过1小时
    
    cached = cache.checkStrongCache(url);
    if (!cached) {
        response = await cache.checkNegotiatedCache(url, { headers: {} });
    }
}

demonstrateCache();
```

#### 缓存策略配置
```javascript
/**
 * 缓存策略配置
 * @author erik.zhou
 */

class CacheStrategy {
    // 静态资源缓存策略
    static staticAssets() {
        return {
            'Cache-Control': 'public, max-age=31536000, immutable',
            description: '静态资源（带hash的文件）- 永久缓存'
        };
    }
    
    // HTML缓存策略
    static html() {
        return {
            'Cache-Control': 'no-cache',
            'ETag': 'W/"abc123"',
            description: 'HTML文件 - 协商缓存'
        };
    }
    
    // API缓存策略
    static api(maxAge = 60) {
        return {
            'Cache-Control': `private, max-age=${maxAge}`,
            'ETag': '"api-etag"',
            description: `API响应 - 私有缓存${maxAge}秒`
        };
    }
    
    // 不缓存策略
    static noCache() {
        return {
            'Cache-Control': 'no-store, no-cache, must-revalidate',
            'Pragma': 'no-cache',
            'Expires': '0',
            description: '敏感数据 - 不缓存'
        };
    }
    
    // CDN缓存策略
    static cdn(maxAge = 3600, sMaxAge = 86400) {
        return {
            'Cache-Control': `public, max-age=${maxAge}, s-maxage=${sMaxAge}`,
            description: `CDN缓存 - 浏览器${maxAge}秒，CDN${sMaxAge}秒`
        };
    }
    
    // 打印所有策略
    static printAll() {
        console.log('=== HTTP缓存策略 ===\n');
        
        console.log('1. 静态资源:');
        console.log(this.staticAssets());
        
        console.log('\n2. HTML文件:');
        console.log(this.html());
        
        console.log('\n3. API响应:');
        console.log(this.api(300));
        
        console.log('\n4. 不缓存:');
        console.log(this.noCache());
        
        console.log('\n5. CDN缓存:');
        console.log(this.cdn());
    }
}

// 使用示例
CacheStrategy.printAll();
```

### 6.3 压缩优化

#### 内容压缩
```javascript
/**
 * HTTP内容压缩
 * @author erik.zhou
 */

class ContentCompression {
    constructor() {
        this.compressionRatios = {
            gzip: 0.3,
            deflate: 0.35,
            br: 0.25
        };
    }
    
    // 选择压缩算法
    selectCompression(acceptEncoding, contentType) {
        console.log('=== 选择压缩算法 ===');
        console.log(`Accept-Encoding: ${acceptEncoding}`);
        console.log(`Content-Type: ${contentType}`);
        
        // 检查是否支持压缩
        if (!this.shouldCompress(contentType)) {
            console.log('该内容类型不适合压缩');
            return null;
        }
        
        // 按优先级选择
        const encodings = acceptEncoding.split(',').map(e => e.trim());
        
        if (encodings.includes('br')) {
            console.log('选择: Brotli (最佳压缩率)');
            return 'br';
        }
        
        if (encodings.includes('gzip')) {
            console.log('选择: Gzip (广泛支持)');
            return 'gzip';
        }
        
        if (encodings.includes('deflate')) {
            console.log('选择: Deflate');
            return 'deflate';
        }
        
        console.log('不压缩');
        return null;
    }
    
    // 判断是否应该压缩
    shouldCompress(contentType) {
        const compressibleTypes = [
            'text/html',
            'text/css',
            'text/javascript',
            'application/javascript',
            'application/json',
            'application/xml',
            'text/xml',
            'image/svg+xml'
        ];
        
        return compressibleTypes.some(type => contentType.includes(type));
    }
    
    // 压缩内容
    compress(content, algorithm) {
        const originalSize = content.length;
        const ratio = this.compressionRatios[algorithm] || 1;
        const compressedSize = Math.floor(originalSize * ratio);
        
        console.log('\n=== 压缩结果 ===');
        console.log(`算法: ${algorithm}`);
        console.log(`原始大小: ${originalSize} 字节`);
        console.log(`压缩后: ${compressedSize} 字节`);
        console.log(`压缩率: ${((1 - ratio) * 100).toFixed(2)}%`);
        
        return {
            content: content.substring(0, compressedSize), // 模拟压缩
            size: compressedSize,
            encoding: algorithm
        };
    }
    
    // 构建响应头
    buildResponseHeaders(compressed) {
        return {
            'Content-Encoding': compressed.encoding,
            'Content-Length': compressed.size.toString(),
            'Vary': 'Accept-Encoding'
        };
    }
}

// 使用示例
const compression = new ContentCompression();

// 模拟请求
const request = {
    headers: {
        'Accept-Encoding': 'gzip, deflate, br'
    }
};

const response = {
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ data: 'x'.repeat(10000) })
};

// 选择压缩算法
const algorithm = compression.selectCompression(
    request.headers['Accept-Encoding'],
    response.headers['Content-Type']
);

// 压缩内容
if (algorithm) {
    const compressed = compression.compress(response.body, algorithm);
    const headers = compression.buildResponseHeaders(compressed);
    console.log('\n响应头:', headers);
}
```

### 6.4 请求优化

#### 请求合并
```javascript
/**
 * HTTP请求合并
 * @author erik.zhou
 */

class RequestBatcher {
    constructor(batchDelay = 50) {
        this.batchDelay = batchDelay;
        this.queue = [];
        this.timer = null;
    }
    
    // 添加请求到批次
    add(request) {
        return new Promise((resolve, reject) => {
            this.queue.push({ request, resolve, reject });
            
            // 设置批次处理定时器
            if (!this.timer) {
                this.timer = setTimeout(() => {
                    this.flush();
                }, this.batchDelay);
            }
        });
    }
    
    // 执行批次请求
    async flush() {
        if (this.queue.length === 0) {
            return;
        }
        
        console.log(`=== 批量处理 ${this.queue.length} 个请求 ===`);
        
        const batch = this.queue.splice(0);
        this.timer = null;
        
        try {
            // 合并请求
            const batchRequest = {
                method: 'POST',
                url: '/api/batch',
                body: batch.map(item => item.request)
            };
            
            console.log('发送批量请求:', batchRequest);
            
            // 发送批量请求
            const response = await this.sendBatchRequest(batchRequest);
            
            // 分发响应
            batch.forEach((item, index) => {
                item.resolve(response.results[index]);
            });
            
        } catch (error) {
            // 批量失败
            batch.forEach(item => {
                item.reject(error);
            });
        }
    }
    
    // 发送批量请求
    async sendBatchRequest(batchRequest) {
        await this.delay(100);
        
        return {
            results: batchRequest.body.map(req => ({
                status: 200,
                data: { message: `Response for ${req.url}` }
            }))
        };
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
const batcher = new RequestBatcher(100);

async function sendMultipleRequests() {
    console.log('添加多个请求到批次...\n');
    
    const requests = [
        { url: '/api/user/1' },
        { url: '/api/user/2' },
        { url: '/api/user/3' },
        { url: '/api/user/4' }
    ];
    
    // 添加请求（会自动合并）
    const promises = requests.map(req => batcher.add(req));
    
    // 等待所有响应
    const responses = await Promise.all(promises);
    
    console.log('\n所有响应:', responses);
}

sendMultipleRequests();
```

## 总结

### HTTP协议核心要点

1. HTTP基础
   - 无状态协议
   - 请求-响应模型
   - 基于TCP/IP
   - 支持多种方法和内容类型

2. HTTP状态码
   - 1xx: 信息性状态码
   - 2xx: 成功状态码
   - 3xx: 重定向状态码
   - 4xx: 客户端错误
   - 5xx: 服务器错误

3. HTTP Headers
   - 请求头：Accept、Authorization、Content-Type等
   - 响应头：Cache-Control、ETag、Set-Cookie等
   - 安全头：CSP、HSTS、X-Frame-Options等

4. HTTP/2特性
   - 多路复用
   - 服务器推送
   - 头部压缩（HPACK）
   - 流优先级

5. HTTP/3特性
   - QUIC协议
   - 0-RTT握手
   - 连接迁移
   - 更好的丢包恢复

6. 性能优化
   - 连接复用
   - 缓存策略
   - 内容压缩
   - 请求合并

### 最佳实践

1. 使用HTTPS保证安全
2. 合理设置缓存策略
3. 启用内容压缩
4. 使用HTTP/2或HTTP/3
5. 实现请求合并和批处理
6. 正确处理状态码
7. 设置安全响应头
8. 监控和优化性能

### 学习资源

- [MDN HTTP文档](https://developer.mozilla.org/zh-CN/docs/Web/HTTP)
- [HTTP/2规范](https://http2.github.io/)
- [HTTP/3规范](https://quicwg.org/)
- [Web性能优化](https://web.dev/performance/)

---

**@author erik.zhou**
**最后更新时间：** 2026-03-06
