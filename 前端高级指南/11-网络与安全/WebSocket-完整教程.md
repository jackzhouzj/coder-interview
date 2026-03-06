# WebSocket - 完整教程

## 课程简介
深入理解WebSocket协议，掌握实时双向通信、连接管理、消息处理和心跳机制。

## 学习目标
- 理解WebSocket工作原理
- 掌握WebSocket连接建立和管理
- 学会消息发送和接收
- 实现心跳检测和断线重连
- 了解WebSocket安全和性能优化

---

## 第一部分：WebSocket基础

### 1.1 WebSocket概述

#### WebSocket vs HTTP
```javascript
/**
 * WebSocket与HTTP对比
 * @author erik.zhou
 */

const protocolComparison = {
    http: {
        name: 'HTTP',
        communication: '单向',
        connection: '短连接',
        overhead: '每次请求都有HTTP头',
        realtime: '需要轮询',
        useCases: [
            '普通网页请求',
            'RESTful API',
            '文件下载'
        ],
        limitations: [
            '服务器无法主动推送',
            '轮询浪费资源',
            '延迟较高'
        ]
    },
    
    websocket: {
        name: 'WebSocket',
        communication: '双向',
        connection: '长连接',
        overhead: '握手后无额外头部',
        realtime: '真正的实时通信',
        useCases: [
            '即时聊天',
            '实时通知',
            '在线游戏',
            '协同编辑',
            '股票行情',
            '视频弹幕'
        ],
        advantages: [
            '服务器可主动推送',
            '低延迟',
            '节省带宽',
            '保持连接状态'
        ]
    }
};

// 打印对比
function printComparison() {
    console.log('=== HTTP vs WebSocket ===\n');
    
    console.log('HTTP:');
    console.log(`  通信方式: ${protocolComparison.http.communication}`);
    console.log(`  连接类型: ${protocolComparison.http.connection}`);
    console.log(`  实时性: ${protocolComparison.http.realtime}`);
    console.log(`  限制: ${protocolComparison.http.limitations.join(', ')}`);
    
    console.log('\nWebSocket:');
    console.log(`  通信方式: ${protocolComparison.websocket.communication}`);
    console.log(`  连接类型: ${protocolComparison.websocket.connection}`);
    console.log(`  实时性: ${protocolComparison.websocket.realtime}`);
    console.log(`  优势: ${protocolComparison.websocket.advantages.join(', ')}`);
}

printComparison();
```

#### WebSocket工作原理
```javascript
/**
 * WebSocket工作原理
 * @author erik.zhou
 */

class WebSocketDemo {
    constructor(url) {
        this.url = url;
        this.ws = null;
        this.reconnectAttempts = 0;
        this.maxReconnectAttempts = 5;
    }
    
    // 建立连接
    connect() {
        console.log('=== WebSocket连接建立 ===');
        console.log(`连接到: ${this.url}\n`);
        
        // 1. HTTP握手升级
        console.log('步骤1: HTTP握手升级');
        console.log('  客户端发送升级请求:');
        console.log('    GET /chat HTTP/1.1');
        console.log('    Host: example.com');
        console.log('    Upgrade: websocket');
        console.log('    Connection: Upgrade');
        console.log('    Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==');
        console.log('    Sec-WebSocket-Version: 13\n');
        
        console.log('  服务器响应:');
        console.log('    HTTP/1.1 101 Switching Protocols');
        console.log('    Upgrade: websocket');
        console.log('    Connection: Upgrade');
        console.log('    Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=\n');
        
        // 2. 建立WebSocket连接
        console.log('步骤2: WebSocket连接已建立');
        console.log('  协议升级成功，可以开始双向通信\n');
        
        // 创建WebSocket实例
        this.ws = new WebSocket(this.url);
        
        // 设置事件监听
        this.setupEventListeners();
    }
    
    // 设置事件监听
    setupEventListeners() {
        // 连接打开
        this.ws.onopen = (event) => {
            console.log('连接已打开');
            this.reconnectAttempts = 0;
        };
        
        // 接收消息
        this.ws.onmessage = (event) => {
            console.log('收到消息:', event.data);
            this.handleMessage(event.data);
        };
        
        // 连接关闭
        this.ws.onclose = (event) => {
            console.log('连接已关闭');
            console.log(`  代码: ${event.code}`);
            console.log(`  原因: ${event.reason}`);
            
            // 尝试重连
            this.reconnect();
        };
        
        // 连接错误
        this.ws.onerror = (error) => {
            console.error('连接错误:', error);
        };
    }
    
    // 发送消息
    send(data) {
        if (this.ws && this.ws.readyState === WebSocket.OPEN) {
            const message = typeof data === 'string' ? data : JSON.stringify(data);
            this.ws.send(message);
            console.log('发送消息:', message);
        } else {
            console.error('WebSocket未连接');
        }
    }
    
    // 处理消息
    handleMessage(data) {
        try {
            const message = JSON.parse(data);
            console.log('解析消息:', message);
            
            // 根据消息类型处理
            switch (message.type) {
                case 'chat':
                    console.log(`聊天消息: ${message.content}`);
                    break;
                case 'notification':
                    console.log(`通知: ${message.content}`);
                    break;
                default:
                    console.log('未知消息类型');
            }
        } catch (error) {
            console.log('文本消息:', data);
        }
    }
    
    // 重连
    reconnect() {
        if (this.reconnectAttempts < this.maxReconnectAttempts) {
            this.reconnectAttempts++;
            console.log(`尝试重连 (${this.reconnectAttempts}/${this.maxReconnectAttempts})`);
            
            setTimeout(() => {
                this.connect();
            }, 1000 * this.reconnectAttempts);
        } else {
            console.log('达到最大重连次数，停止重连');
        }
    }
    
    // 关闭连接
    close() {
        if (this.ws) {
            this.ws.close(1000, '正常关闭');
            console.log('主动关闭连接');
        }
    }
}

// 使用示例
const ws = new WebSocketDemo('ws://example.com/chat');
ws.connect();

// 发送消息
setTimeout(() => {
    ws.send({ type: 'chat', content: 'Hello, WebSocket!' });
}, 1000);
```

### 1.2 WebSocket状态

#### 连接状态管理
```javascript
/**
 * WebSocket连接状态管理
 * @author erik.zhou
 */

class WebSocketStateManager {
    constructor(url) {
        this.url = url;
        this.ws = null;
        this.state = 'DISCONNECTED';
        this.stateListeners = [];
    }
    
    // 连接状态枚举
    static STATES = {
        CONNECTING: 'CONNECTING',    // 正在连接
        OPEN: 'OPEN',                // 已连接
        CLOSING: 'CLOSING',          // 正在关闭
        CLOSED: 'CLOSED',            // 已关闭
        DISCONNECTED: 'DISCONNECTED' // 未连接
    };
    
    // WebSocket readyState映射
    static READY_STATE_MAP = {
        0: 'CONNECTING',
        1: 'OPEN',
        2: 'CLOSING',
        3: 'CLOSED'
    };
    
    // 连接
    connect() {
        this.setState('CONNECTING');
        
        this.ws = new WebSocket(this.url);
        
        this.ws.onopen = () => {
            this.setState('OPEN');
            console.log('WebSocket已连接');
        };
        
        this.ws.onclose = () => {
            this.setState('CLOSED');
            console.log('WebSocket已关闭');
        };
        
        this.ws.onerror = (error) => {
            console.error('WebSocket错误:', error);
        };
    }
    
    // 设置状态
    setState(newState) {
        const oldState = this.state;
        this.state = newState;
        
        console.log(`状态变更: ${oldState} -> ${newState}`);
        
        // 通知监听器
        this.notifyStateChange(oldState, newState);
    }
    
    // 获取当前状态
    getState() {
        if (this.ws) {
            return WebSocketStateManager.READY_STATE_MAP[this.ws.readyState];
        }
        return this.state;
    }
    
    // 检查是否可以发送消息
    canSend() {
        return this.ws && this.ws.readyState === WebSocket.OPEN;
    }
    
    // 添加状态监听器
    onStateChange(listener) {
        this.stateListeners.push(listener);
    }
    
    // 通知状态变更
    notifyStateChange(oldState, newState) {
        for (const listener of this.stateListeners) {
            listener(oldState, newState);
        }
    }
    
    // 关闭连接
    close() {
        if (this.ws) {
            this.setState('CLOSING');
            this.ws.close();
        }
    }
}

// 使用示例
const stateManager = new WebSocketStateManager('ws://example.com');

// 监听状态变化
stateManager.onStateChange((oldState, newState) => {
    console.log(`应用层监听到状态变化: ${oldState} -> ${newState}`);
    
    // 根据状态执行操作
    if (newState === 'OPEN') {
        console.log('连接成功，可以发送消息');
    } else if (newState === 'CLOSED') {
        console.log('连接关闭，需要重连');
    }
});

stateManager.connect();
```

## 第二部分：消息处理

### 2.1 消息格式

#### 消息封装
```javascript
/**
 * WebSocket消息封装
 * @author erik.zhou
 */

class WebSocketMessage {
    constructor() {
        this.messageId = 0;
    }
    
    // 创建消息
    create(type, data, options = {}) {
        this.messageId++;
        
        const message = {
            id: this.messageId,
            type: type,
            data: data,
            timestamp: Date.now(),
            ...options
        };
        
        return message;
    }
    
    // 序列化消息
    serialize(message) {
        return JSON.stringify(message);
    }
    
    // 反序列化消息
    deserialize(data) {
        try {
            return JSON.parse(data);
        } catch (error) {
            console.error('消息解析失败:', error);
            return null;
        }
    }
    
    // 创建聊天消息
    createChatMessage(content, userId) {
        return this.create('chat', {
            content: content,
            userId: userId
        });
    }
    
    // 创建通知消息
    createNotification(title, content) {
        return this.create('notification', {
            title: title,
            content: content
        });
    }
    
    // 创建心跳消息
    createHeartbeat() {
        return this.create('heartbeat', {
            timestamp: Date.now()
        });
    }
    
    // 创建响应消息
    createResponse(requestId, success, data) {
        return this.create('response', {
            requestId: requestId,
            success: success,
            data: data
        });
    }
}

// 使用示例
const messageFactory = new WebSocketMessage();

// 创建各种类型的消息
const chatMsg = messageFactory.createChatMessage('Hello!', 'user123');
console.log('聊天消息:', messageFactory.serialize(chatMsg));

const notification = messageFactory.createNotification('新消息', '您有一条新消息');
console.log('通知消息:', messageFactory.serialize(notification));

const heartbeat = messageFactory.createHeartbeat();
console.log('心跳消息:', messageFactory.serialize(heartbeat));
```

### 2.2 消息队列

#### 消息队列管理
```javascript
/**
 * WebSocket消息队列
 * @author erik.zhou
 */

class MessageQueue {
    constructor(maxSize = 100) {
        this.queue = [];
        this.maxSize = maxSize;
        this.processing = false;
    }
    
    // 添加消息到队列
    enqueue(message) {
        if (this.queue.length >= this.maxSize) {
            console.warn('队列已满，移除最旧的消息');
            this.queue.shift();
        }
        
        this.queue.push({
            message: message,
            timestamp: Date.now(),
            retries: 0
        });
        
        console.log(`消息入队，当前队列长度: ${this.queue.length}`);
    }
    
    // 从队列取出消息
    dequeue() {
        if (this.queue.length === 0) {
            return null;
        }
        
        const item = this.queue.shift();
        console.log(`消息出队，剩余队列长度: ${this.queue.length}`);
        
        return item;
    }
    
    // 处理队列
    async process(sendFunction) {
        if (this.processing) {
            console.log('队列正在处理中');
            return;
        }
        
        this.processing = true;
        console.log('开始处理消息队列');
        
        while (this.queue.length > 0) {
            const item = this.dequeue();
            
            try {
                await sendFunction(item.message);
                console.log('消息发送成功');
            } catch (error) {
                console.error('消息发送失败:', error);
                
                // 重试逻辑
                if (item.retries < 3) {
                    item.retries++;
                    this.queue.unshift(item); // 放回队列头部
                    console.log(`消息重试 ${item.retries}/3`);
                    await this.delay(1000);
                } else {
                    console.error('消息发送失败，已达最大重试次数');
                }
            }
        }
        
        this.processing = false;
        console.log('消息队列处理完成');
    }
    
    // 清空队列
    clear() {
        this.queue = [];
        console.log('队列已清空');
    }
    
    // 获取队列长度
    size() {
        return this.queue.length;
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
const queue = new MessageQueue(50);

// 模拟发送函数
async function mockSend(message) {
    console.log('发送消息:', message);
    await new Promise(resolve => setTimeout(resolve, 100));
    
    // 模拟随机失败
    if (Math.random() < 0.2) {
        throw new Error('发送失败');
    }
}

// 添加消息到队列
for (let i = 1; i <= 5; i++) {
    queue.enqueue({ id: i, content: `Message ${i}` });
}

// 处理队列
queue.process(mockSend);
```

## 第三部分：心跳机制

### 3.1 心跳检测

#### 心跳实现
```javascript
/**
 * WebSocket心跳机制
 * @author erik.zhou
 */

class WebSocketHeartbeat {
    constructor(ws, options = {}) {
        this.ws = ws;
        this.heartbeatInterval = options.interval || 30000; // 30秒
        this.heartbeatTimeout = options.timeout || 5000;    // 5秒
        this.reconnectDelay = options.reconnectDelay || 3000; // 3秒
        
        this.heartbeatTimer = null;
        this.heartbeatTimeoutTimer = null;
        this.missedHeartbeats = 0;
        this.maxMissedHeartbeats = 3;
    }
    
    // 启动心跳
    start() {
        console.log('启动心跳检测');
        this.stop(); // 先停止之前的定时器
        
        this.heartbeatTimer = setInterval(() => {
            this.sendHeartbeat();
        }, this.heartbeatInterval);
    }
    
    // 停止心跳
    stop() {
        if (this.heartbeatTimer) {
            clearInterval(this.heartbeatTimer);
            this.heartbeatTimer = null;
            console.log('停止心跳检测');
        }
        
        if (this.heartbeatTimeoutTimer) {
            clearTimeout(this.heartbeatTimeoutTimer);
            this.heartbeatTimeoutTimer = null;
        }
    }
    
    // 发送心跳
    sendHeartbeat() {
        if (this.ws.readyState !== WebSocket.OPEN) {
            console.log('连接未打开，跳过心跳');
            return;
        }
        
        console.log('发送心跳 ping');
        
        const heartbeat = {
            type: 'heartbeat',
            timestamp: Date.now()
        };
        
        this.ws.send(JSON.stringify(heartbeat));
        
        // 设置超时检测
        this.heartbeatTimeoutTimer = setTimeout(() => {
            this.onHeartbeatTimeout();
        }, this.heartbeatTimeout);
    }
    
    // 接收心跳响应
    onHeartbeatResponse() {
        console.log('收到心跳 pong');
        
        // 清除超时定时器
        if (this.heartbeatTimeoutTimer) {
            clearTimeout(this.heartbeatTimeoutTimer);
            this.heartbeatTimeoutTimer = null;
        }
        
        // 重置计数器
        this.missedHeartbeats = 0;
    }
    
    // 心跳超时
    onHeartbeatTimeout() {
        this.missedHeartbeats++;
        console.log(`心跳超时 (${this.missedHeartbeats}/${this.maxMissedHeartbeats})`);
        
        if (this.missedHeartbeats >= this.maxMissedHeartbeats) {
            console.log('连续心跳超时，判定连接断开');
            this.stop();
            this.ws.close();
            
            // 触发重连
            this.onConnectionLost();
        }
    }
    
    // 连接丢失
    onConnectionLost() {
        console.log('连接丢失，准备重连');
        
        setTimeout(() => {
            console.log('尝试重新连接...');
            // 这里应该调用重连逻辑
        }, this.reconnectDelay);
    }
    
    // 重置心跳
    reset() {
        this.missedHeartbeats = 0;
        this.start();
    }
}

// 使用示例
const ws = new WebSocket('ws://example.com');

ws.onopen = () => {
    console.log('WebSocket已连接');
    
    // 启动心跳
    const heartbeat = new WebSocketHeartbeat(ws, {
        interval: 30000,
        timeout: 5000
    });
    
    heartbeat.start();
    
    // 监听消息
    ws.onmessage = (event) => {
        const message = JSON.parse(event.data);
        
        if (message.type === 'heartbeat') {
            heartbeat.onHeartbeatResponse();
        }
    };
};
```

## 第四部分：断线重连

### 4.1 重连策略

#### 智能重连
```javascript
/**
 * WebSocket智能重连
 * @author erik.zhou
 */

class WebSocketReconnect {
    constructor(url, options = {}) {
        this.url = url;
        this.ws = null;
        
        // 重连配置
        this.reconnectAttempts = 0;
        this.maxReconnectAttempts = options.maxAttempts || 10;
        this.reconnectDelay = options.initialDelay || 1000;
        this.maxReconnectDelay = options.maxDelay || 30000;
        this.reconnectDecay = options.decay || 1.5;
        
        // 状态
        this.shouldReconnect = true;
        this.reconnectTimer = null;
        
        // 消息队列
        this.messageQueue = [];
    }
    
    // 连接
    connect() {
        console.log(`连接WebSocket: ${this.url}`);
        
        try {
            this.ws = new WebSocket(this.url);
            this.setupEventHandlers();
        } catch (error) {
            console.error('连接失败:', error);
            this.scheduleReconnect();
        }
    }
    
    // 设置事件处理
    setupEventHandlers() {
        this.ws.onopen = () => {
            console.log('WebSocket已连接');
            this.onConnected();
        };
        
        this.ws.onclose = (event) => {
            console.log('WebSocket已关闭');
            this.onDisconnected(event);
        };
        
        this.ws.onerror = (error) => {
            console.error('WebSocket错误:', error);
        };
        
        this.ws.onmessage = (event) => {
            this.onMessage(event.data);
        };
    }
    
    // 连接成功
    onConnected() {
        // 重置重连计数
        this.reconnectAttempts = 0;
        this.reconnectDelay = 1000;
        
        // 发送队列中的消息
        this.flushMessageQueue();
    }
    
    // 连接断开
    onDisconnected(event) {
        console.log(`断开原因: ${event.reason} (代码: ${event.code})`);
        
        // 判断是否需要重连
        if (this.shouldReconnect && !this.isNormalClosure(event.code)) {
            this.scheduleReconnect();
        }
    }
    
    // 判断是否正常关闭
    isNormalClosure(code) {
        return code === 1000 || code === 1001;
    }
    
    // 安排重连
    scheduleReconnect() {
        if (this.reconnectAttempts >= this.maxReconnectAttempts) {
            console.log('达到最大重连次数，停止重连');
            return;
        }
        
        this.reconnectAttempts++;
        
        // 计算延迟（指数退避）
        const delay = Math.min(
            this.reconnectDelay * Math.pow(this.reconnectDecay, this.reconnectAttempts - 1),
            this.maxReconnectDelay
        );
        
        console.log(`${delay}ms后尝试第${this.reconnectAttempts}次重连`);
        
        this.reconnectTimer = setTimeout(() => {
            this.connect();
        }, delay);
    }
    
    // 发送消息
    send(data) {
        if (this.ws && this.ws.readyState === WebSocket.OPEN) {
            this.ws.send(data);
        } else {
            console.log('连接未就绪，消息加入队列');
            this.messageQueue.push(data);
        }
    }
    
    // 发送队列中的消息
    flushMessageQueue() {
        console.log(`发送队列中的${this.messageQueue.length}条消息`);
        
        while (this.messageQueue.length > 0) {
            const message = this.messageQueue.shift();
            this.ws.send(message);
        }
    }
    
    // 接收消息
    onMessage(data) {
        console.log('收到消息:', data);
    }
    
    // 关闭连接
    close() {
        this.shouldReconnect = false;
        
        if (this.reconnectTimer) {
            clearTimeout(this.reconnectTimer);
        }
        
        if (this.ws) {
            this.ws.close(1000, '正常关闭');
        }
    }
}

// 使用示例
const wsReconnect = new WebSocketReconnect('ws://example.com', {
    maxAttempts: 10,
    initialDelay: 1000,
    maxDelay: 30000,
    decay: 1.5
});

wsReconnect.connect();

// 发送消息
setTimeout(() => {
    wsReconnect.send(JSON.stringify({ type: 'chat', content: 'Hello!' }));
}, 2000);
```

## 第五部分：安全性

### 5.1 WebSocket安全

#### 安全最佳实践
```javascript
/**
 * WebSocket安全实践
 * @author erik.zhou
 */

class SecureWebSocket {
    constructor(url, token) {
        this.url = url;
        this.token = token;
        this.ws = null;
    }
    
    // 使用WSS协议
    ensureSecureProtocol() {
        if (!this.url.startsWith('wss://')) {
            console.warn('建议使用WSS协议（加密连接）');
            
            // 自动转换为WSS
            if (this.url.startsWith('ws://')) {
                this.url = this.url.replace('ws://', 'wss://');
                console.log(`已转换为WSS: ${this.url}`);
            }
        }
    }
    
    // 连接时进行身份验证
    connect() {
        this.ensureSecureProtocol();
        
        // 在URL中添加token（不推荐，仅作示例）
        // const urlWithToken = `${this.url}?token=${this.token}`;
        
        this.ws = new WebSocket(this.url);
        
        this.ws.onopen = () => {
            // 连接后立即发送认证消息（推荐）
            this.authenticate();
        };
        
        this.ws.onmessage = (event) => {
            this.handleMessage(event.data);
        };
    }
    
    // 身份验证
    authenticate() {
        const authMessage = {
            type: 'auth',
            token: this.token,
            timestamp: Date.now()
        };
        
        this.ws.send(JSON.stringify(authMessage));
        console.log('发送认证消息');
    }
    
    // 消息验证
    validateMessage(message) {
        // 检查消息格式
        if (!message.type || !message.data) {
            console.error('消息格式无效');
            return false;
        }
        
        // 检查时间戳（防止重放攻击）
        if (message.timestamp) {
            const now = Date.now();
            const diff = Math.abs(now - message.timestamp);
            
            if (diff > 60000) { // 1分钟
                console.error('消息时间戳过期');
                return false;
            }
        }
        
        return true;
    }
    
    // 处理消息
    handleMessage(data) {
        try {
            const message = JSON.parse(data);
            
            // 验证消息
            if (!this.validateMessage(message)) {
                return;
            }
            
            // 处理不同类型的消息
            switch (message.type) {
                case 'auth_success':
                    console.log('认证成功');
                    break;
                case 'auth_failed':
                    console.error('认证失败');
                    this.ws.close();
                    break;
                default:
                    console.log('收到消息:', message);
            }
        } catch (error) {
            console.error('消息解析失败:', error);
        }
    }
    
    // 发送加密消息
    sendEncrypted(data) {
        // 实际应用中应该使用真正的加密算法
        const encrypted = {
            type: 'encrypted',
            data: btoa(JSON.stringify(data)), // Base64编码（仅作示例）
            timestamp: Date.now()
        };
        
        this.ws.send(JSON.stringify(encrypted));
    }
}

// 使用示例
const secureWs = new SecureWebSocket('wss://example.com/chat', 'user-token-123');
secureWs.connect();
```

## 总结

### WebSocket核心要点

1. WebSocket基础
   - 全双工通信
   - 长连接
   - 低延迟
   - 节省带宽

2. 连接管理
   - 握手升级
   - 状态管理
   - 事件监听
   - 错误处理

3. 消息处理
   - 消息格式化
   - 消息队列
   - 消息验证
   - 消息加密

4. 心跳机制
   - 定时发送心跳
   - 超时检测
   - 连接保活
   - 异常检测

5. 断线重连
   - 指数退避
   - 最大重试次数
   - 消息队列
   - 状态恢复

6. 安全性
   - 使用WSS协议
   - 身份认证
   - 消息验证
   - 防重放攻击

### 最佳实践

1. 始终使用WSS协议
2. 实现心跳机制
3. 实现断线重连
4. 使用消息队列
5. 进行身份验证
6. 验证消息格式
7. 处理异常情况
8. 监控连接状态

### 常见问题

1. 连接频繁断开
   - 实现心跳机制
   - 检查网络稳定性
   - 优化服务器配置

2. 消息丢失
   - 使用消息队列
   - 实现消息确认机制
   - 记录消息日志

3. 性能问题
   - 控制消息频率
   - 压缩消息内容
   - 使用二进制格式

4. 安全问题
   - 使用WSS协议
   - 实现认证机制
   - 验证消息来源

### 学习资源

- [MDN WebSocket文档](https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket)
- [WebSocket协议规范](https://tools.ietf.org/html/rfc6455)
- [Socket.IO文档](https://socket.io/docs/)
- [WebSocket安全指南](https://owasp.org/www-community/vulnerabilities/WebSocket_Security)

---

**@author erik.zhou**
**最后更新时间：** 2026-03-06
