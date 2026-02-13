# Python API 安全完整教程

> @author erik.zhou

## 📋 目录

1. [API 安全概述](#api-安全概述)
2. [接口限流](#接口限流)
3. [签名验证](#签名验证)
4. [HTTPS 配置](#https-配置)
5. [API 密钥管理](#api-密钥管理)
6. [防重放攻击](#防重放攻击)
7. [实战案例](#实战案例)

---

## API 安全概述

### API 安全威胁

| 威胁类型 | 描述 | 防护措施 |
|---------|------|---------|
| 未授权访问 | 无认证访问 API | JWT、API Key |
| 暴力破解 | 大量请求尝试 | 限流、验证码 |
| 重放攻击 | 重复发送请求 | Nonce、时间戳 |
| 中间人攻击 | 窃听通信 | HTTPS |
| DDoS 攻击 | 大量请求导致服务不可用 | 限流、CDN |
| 数据泄露 | 敏感数据暴露 | 加密、脱敏 |

### API 安全检查清单

```python
"""
API 安全检查清单：

1. [ ] 使用 HTTPS 传输
2. [ ] 实施认证机制（JWT/API Key）
3. [ ] 实施授权检查（RBAC）
4. [ ] 接口限流
5. [ ] 签名验证
6. [ ] 防重放攻击
7. [ ] 输入验证
8. [ ] 输出脱敏
9. [ ] 错误处理（不泄露敏感信息）
10. [ ] 审计日志
"""
```

---

## 接口限流

### 固定窗口限流

```python
from flask import Flask, request, jsonify
from functools import wraps
import time
from collections import defaultdict

app = Flask(__name__)

class FixedWindowRateLimiter:
    """固定窗口限流器"""
    
    def __init__(self, max_requests: int, window_size: int):
        """
        Args:
            max_requests: 窗口内最大请求数
            window_size: 窗口大小（秒）
        """
        self.max_requests = max_requests
        self.window_size = window_size
        self.requests = defaultdict(list)
    
    def is_allowed(self, key: str) -> bool:
        """检查是否允许请求"""
        now = time.time()
        window_start = now - self.window_size
        
        # 清理过期请求
        self.requests[key] = [
            req_time for req_time in self.requests[key]
            if req_time > window_start
        ]
        
        # 检查请求数
        if len(self.requests[key]) < self.max_requests:
            self.requests[key].append(now)
            return True
        
        return False

# 创建限流器：每分钟最多 10 个请求
rate_limiter = FixedWindowRateLimiter(max_requests=10, window_size=60)

def rate_limit(f):
    """限流装饰器"""
    @wraps(f)
    def decorated(*args, **kwargs):
        # 使用 IP 作为限流 key
        client_ip = request.remote_addr
        
        if not rate_limiter.is_allowed(client_ip):
            return jsonify({
                'error': '请求过于频繁，请稍后再试'
            }), 429
        
        return f(*args, **kwargs)
    
    return decorated

@app.route('/api/data')
@rate_limit
def get_data():
    """受限流保护的 API"""
    return jsonify({'data': 'some data'})
```

### 滑动窗口限流

```python
import time
from collections import deque

class SlidingWindowRateLimiter:
    """滑动窗口限流器"""
    
    def __init__(self, max_requests: int, window_size: int):
        self.max_requests = max_requests
        self.window_size = window_size
        self.requests = {}
    
    def is_allowed(self, key: str) -> bool:
        """检查是否允许请求"""
        now = time.time()
        
        if key not in self.requests:
            self.requests[key] = deque()
        
        # 移除窗口外的请求
        while self.requests[key] and self.requests[key][0] <= now - self.window_size:
            self.requests[key].popleft()
        
        # 检查请求数
        if len(self.requests[key]) < self.max_requests:
            self.requests[key].append(now)
            return True
        
        return False
```

### 令牌桶限流

```python
import time
import threading

class TokenBucketRateLimiter:
    """令牌桶限流器"""
    
    def __init__(self, capacity: int, refill_rate: float):
        """
        Args:
            capacity: 桶容量
            refill_rate: 每秒填充速率
        """
        self.capacity = capacity
        self.refill_rate = refill_rate
        self.tokens = capacity
        self.last_refill = time.time()
        self.lock = threading.Lock()
    
    def is_allowed(self) -> bool:
        """检查是否允许请求"""
        with self.lock:
            now = time.time()
            
            # 填充令牌
            elapsed = now - self.last_refill
            self.tokens = min(
                self.capacity,
                self.tokens + elapsed * self.refill_rate
            )
            self.last_refill = now
            
            # 消耗令牌
            if self.tokens >= 1:
                self.tokens -= 1
                return True
            
            return False
```

### 使用 Redis 实现分布式限流

```bash
pip install redis
```

```python
import redis
import time

class RedisRateLimiter:
    """基于 Redis 的分布式限流器"""
    
    def __init__(self, redis_client: redis.Redis, max_requests: int, window_size: int):
        self.redis = redis_client
        self.max_requests = max_requests
        self.window_size = window_size
    
    def is_allowed(self, key: str) -> bool:
        """检查是否允许请求"""
        now = time.time()
        window_start = now - self.window_size
        
        # Redis key
        redis_key = f"rate_limit:{key}"
        
        # 使用 pipeline 提高性能
        pipe = self.redis.pipeline()
        
        # 移除过期请求
        pipe.zremrangebyscore(redis_key, 0, window_start)
        
        # 获取当前请求数
        pipe.zcard(redis_key)
        
        # 添加当前请求
        pipe.zadd(redis_key, {str(now): now})
        
        # 设置过期时间
        pipe.expire(redis_key, self.window_size)
        
        # 执行
        results = pipe.execute()
        request_count = results[1]
        
        return request_count < self.max_requests

# 使用示例
redis_client = redis.Redis(host='localhost', port=6379, db=0)
rate_limiter = RedisRateLimiter(redis_client, max_requests=100, window_size=60)

def rate_limit_redis(f):
    """Redis 限流装饰器"""
    @wraps(f)
    def decorated(*args, **kwargs):
        client_ip = request.remote_addr
        
        if not rate_limiter.is_allowed(client_ip):
            return jsonify({'error': '请求过于频繁'}), 429
        
        return f(*args, **kwargs)
    
    return decorated
```

---

## 签名验证

### HMAC 签名

```python
import hmac
import hashlib
import time
from flask import Flask, request, jsonify
from functools import wraps

app = Flask(__name__)

class SignatureVerifier:
    """签名验证器"""
    
    def __init__(self, secret_key: str):
        self.secret_key = secret_key.encode()
    
    def generate_signature(self, data: str, timestamp: str) -> str:
        """生成签名"""
        message = f"{data}{timestamp}".encode()
        signature = hmac.new(
            self.secret_key,
            message,
            hashlib.sha256
        ).hexdigest()
        return signature
    
    def verify_signature(self, data: str, timestamp: str, signature: str) -> bool:
        """验证签名"""
        expected_signature = self.generate_signature(data, timestamp)
        return hmac.compare_digest(expected_signature, signature)

# 创建验证器
verifier = SignatureVerifier(secret_key="your-secret-key")

def require_signature(f):
    """签名验证装饰器"""
    @wraps(f)
    def decorated(*args, **kwargs):
        # 获取签名相关参数
        timestamp = request.headers.get('X-Timestamp')
        signature = request.headers.get('X-Signature')
        
        if not timestamp or not signature:
            return jsonify({'error': '缺少签名参数'}), 401
        
        # 检查时间戳（防重放）
        now = int(time.time())
        if abs(now - int(timestamp)) > 300:  # 5 分钟有效期
            return jsonify({'error': '请求已过期'}), 401
        
        # 获取请求数据
        if request.method == 'GET':
            data = request.query_string.decode()
        else:
            data = request.get_data().decode()
        
        # 验证签名
        if not verifier.verify_signature(data, timestamp, signature):
            return jsonify({'error': '签名验证失败'}), 401
        
        return f(*args, **kwargs)
    
    return decorated

@app.route('/api/secure-data', methods=['POST'])
@require_signature
def secure_data():
    """需要签名验证的 API"""
    return jsonify({'message': '数据已接收'})

# 客户端示例
def make_signed_request(url: str, data: dict):
    """发送带签名的请求"""
    import requests
    import json
    
    timestamp = str(int(time.time()))
    data_str = json.dumps(data)
    
    # 生成签名
    signature = verifier.generate_signature(data_str, timestamp)
    
    # 发送请求
    headers = {
        'X-Timestamp': timestamp,
        'X-Signature': signature,
        'Content-Type': 'application/json'
    }
    
    response = requests.post(url, data=data_str, headers=headers)
    return response.json()
```

### 完整的签名方案

```python
import hmac
import hashlib
import json
from typing import Dict
from urllib.parse import urlencode

class APISignature:
    """API 签名工具"""
    
    def __init__(self, app_id: str, app_secret: str):
        self.app_id = app_id
        self.app_secret = app_secret
    
    def sign(self, method: str, path: str, params: Dict, body: Dict = None) -> str:
        """生成签名
        
        签名算法：
        1. 按字母顺序排序所有参数
        2. 拼接成字符串：key1=value1&key2=value2
        3. 添加 app_secret
        4. HMAC-SHA256 签名
        """
        # 1. 收集所有参数
        all_params = params.copy()
        all_params['app_id'] = self.app_id
        all_params['timestamp'] = str(int(time.time()))
        
        if body:
            all_params['body'] = json.dumps(body, sort_keys=True)
        
        # 2. 按字母顺序排序
        sorted_params = sorted(all_params.items())
        
        # 3. 拼接字符串
        param_str = urlencode(sorted_params)
        
        # 4. 添加方法和路径
        sign_str = f"{method.upper()}&{path}&{param_str}"
        
        # 5. HMAC-SHA256 签名
        signature = hmac.new(
            self.app_secret.encode(),
            sign_str.encode(),
            hashlib.sha256
        ).hexdigest()
        
        return signature
    
    def verify(self, method: str, path: str, params: Dict, body: Dict, signature: str) -> bool:
        """验证签名"""
        expected_signature = self.sign(method, path, params, body)
        return hmac.compare_digest(expected_signature, signature)

# 使用示例
api_sig = APISignature(app_id="app123", app_secret="secret456")

# 客户端：生成签名
method = "POST"
path = "/api/users"
params = {"page": "1", "size": "10"}
body = {"name": "John", "email": "john@example.com"}

signature = api_sig.sign(method, path, params, body)
print(f"签名: {signature}")

# 服务端：验证签名
is_valid = api_sig.verify(method, path, params, body, signature)
print(f"验证结果: {is_valid}")
```


---

## HTTPS 配置

### 为什么需要 HTTPS

| 威胁 | HTTP | HTTPS |
|------|------|-------|
| 数据窃听 | ❌ 明文传输 | ✅ 加密传输 |
| 数据篡改 | ❌ 可被修改 | ✅ 完整性校验 |
| 身份伪造 | ❌ 无法验证 | ✅ 证书验证 |

### Flask 配置 HTTPS

```python
from flask import Flask

app = Flask(__name__)

if __name__ == '__main__':
    # 开发环境：使用自签名证书
    app.run(
        host='0.0.0.0',
        port=443,
        ssl_context='adhoc'  # 自动生成临时证书
    )
    
    # 生产环境：使用正式证书
    # app.run(
    #     host='0.0.0.0',
    #     port=443,
    #     ssl_context=('cert.pem', 'key.pem')
    # )
```

### 生成自签名证书

```bash
# 安装 pyOpenSSL
pip install pyopenssl

# 生成证书
openssl req -x509 -newkey rsa:4096 -nodes \
  -out cert.pem -keyout key.pem -days 365
```

### FastAPI 配置 HTTPS

```python
import uvicorn
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello HTTPS"}

if __name__ == "__main__":
    uvicorn.run(
        app,
        host="0.0.0.0",
        port=443,
        ssl_keyfile="./key.pem",
        ssl_certfile="./cert.pem"
    )
```

### 强制 HTTPS 重定向

```python
from flask import Flask, redirect, request

app = Flask(__name__)

@app.before_request
def force_https():
    """强制使用 HTTPS"""
    if not request.is_secure:
        url = request.url.replace('http://', 'https://', 1)
        return redirect(url, code=301)

@app.route('/')
def index():
    return "Secure connection"
```

### Nginx 反向代理配置

```nginx
server {
    listen 80;
    server_name api.example.com;
    
    # 强制跳转到 HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;
    
    # SSL 证书配置
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    
    # SSL 安全配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    # HSTS（强制 HTTPS）
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## API 密钥管理

### 密钥生成

```python
import secrets
import hashlib

class APIKeyManager:
    """API 密钥管理器"""
    
    @staticmethod
    def generate_api_key() -> str:
        """生成 API 密钥"""
        return secrets.token_urlsafe(32)
    
    @staticmethod
    def hash_api_key(api_key: str) -> str:
        """哈希 API 密钥（存储用）"""
        return hashlib.sha256(api_key.encode()).hexdigest()
    
    @staticmethod
    def verify_api_key(api_key: str, hashed_key: str) -> bool:
        """验证 API 密钥"""
        return hashlib.sha256(api_key.encode()).hexdigest() == hashed_key

# 使用示例
manager = APIKeyManager()

# 生成密钥
api_key = manager.generate_api_key()
print(f"API Key: {api_key}")

# 哈希存储
hashed = manager.hash_api_key(api_key)
print(f"Hashed: {hashed}")

# 验证
is_valid = manager.verify_api_key(api_key, hashed)
print(f"Valid: {is_valid}")
```

### API 密钥认证中间件

```python
from flask import Flask, request, jsonify
from functools import wraps

app = Flask(__name__)

# 模拟数据库中的 API 密钥（实际应从数据库读取）
VALID_API_KEYS = {
    "hashed_key_1": {"user_id": 1, "name": "User 1"},
    "hashed_key_2": {"user_id": 2, "name": "User 2"}
}

def require_api_key(f):
    """API 密钥认证装饰器"""
    @wraps(f)
    def decorated(*args, **kwargs):
        # 从请求头获取 API 密钥
        api_key = request.headers.get('X-API-Key')
        
        if not api_key:
            return jsonify({'error': '缺少 API 密钥'}), 401
        
        # 哈希并验证
        hashed = APIKeyManager.hash_api_key(api_key)
        
        if hashed not in VALID_API_KEYS:
            return jsonify({'error': 'API 密钥无效'}), 401
        
        # 将用户信息添加到请求上下文
        request.user = VALID_API_KEYS[hashed]
        
        return f(*args, **kwargs)
    
    return decorated

@app.route('/api/protected')
@require_api_key
def protected_endpoint():
    """受保护的 API"""
    return jsonify({
        'message': f'Hello, {request.user["name"]}!',
        'user_id': request.user['user_id']
    })
```

### 密钥轮换策略

```python
from datetime import datetime, timedelta
from typing import Optional

class APIKeyRotation:
    """API 密钥轮换管理"""
    
    def __init__(self, rotation_days: int = 90):
        self.rotation_days = rotation_days
        self.keys = {}  # {key_id: {key, created_at, expires_at}}
    
    def create_key(self, user_id: int) -> dict:
        """创建新密钥"""
        key = APIKeyManager.generate_api_key()
        created_at = datetime.now()
        expires_at = created_at + timedelta(days=self.rotation_days)
        
        key_info = {
            'key': key,
            'user_id': user_id,
            'created_at': created_at,
            'expires_at': expires_at,
            'is_active': True
        }
        
        self.keys[key] = key_info
        return key_info
    
    def is_key_valid(self, key: str) -> bool:
        """检查密钥是否有效"""
        if key not in self.keys:
            return False
        
        key_info = self.keys[key]
        
        # 检查是否过期
        if datetime.now() > key_info['expires_at']:
            return False
        
        # 检查是否被禁用
        if not key_info['is_active']:
            return False
        
        return True
    
    def rotate_key(self, old_key: str) -> Optional[str]:
        """轮换密钥"""
        if old_key not in self.keys:
            return None
        
        user_id = self.keys[old_key]['user_id']
        
        # 禁用旧密钥
        self.keys[old_key]['is_active'] = False
        
        # 创建新密钥
        new_key_info = self.create_key(user_id)
        
        return new_key_info['key']
```

---

## 防重放攻击

### 什么是重放攻击

重放攻击是指攻击者截获合法的请求，然后重复发送该请求，以达到欺骗服务器的目的。

### 使用 Nonce 防重放

```python
import time
import uuid
from flask import Flask, request, jsonify
from functools import wraps

app = Flask(__name__)

class NonceManager:
    """Nonce 管理器"""
    
    def __init__(self, ttl: int = 300):
        """
        Args:
            ttl: Nonce 有效期（秒）
        """
        self.ttl = ttl
        self.used_nonces = {}  # {nonce: timestamp}
    
    def generate_nonce(self) -> str:
        """生成 Nonce"""
        return str(uuid.uuid4())
    
    def is_valid(self, nonce: str) -> bool:
        """验证 Nonce"""
        now = time.time()
        
        # 清理过期的 Nonce
        self.used_nonces = {
            n: t for n, t in self.used_nonces.items()
            if now - t < self.ttl
        }
        
        # 检查是否已使用
        if nonce in self.used_nonces:
            return False
        
        # 标记为已使用
        self.used_nonces[nonce] = now
        return True

nonce_manager = NonceManager()

def prevent_replay(f):
    """防重放装饰器"""
    @wraps(f)
    def decorated(*args, **kwargs):
        # 获取 Nonce 和时间戳
        nonce = request.headers.get('X-Nonce')
        timestamp = request.headers.get('X-Timestamp')
        
        if not nonce or not timestamp:
            return jsonify({'error': '缺少防重放参数'}), 400
        
        # 检查时间戳（5分钟有效期）
        now = int(time.time())
        if abs(now - int(timestamp)) > 300:
            return jsonify({'error': '请求已过期'}), 400
        
        # 验证 Nonce
        if not nonce_manager.is_valid(nonce):
            return jsonify({'error': '请求已被使用或无效'}), 400
        
        return f(*args, **kwargs)
    
    return decorated

@app.route('/api/transfer', methods=['POST'])
@prevent_replay
def transfer():
    """转账接口（防重放）"""
    data = request.get_json()
    return jsonify({
        'message': '转账成功',
        'amount': data.get('amount')
    })
```

### 使用 Redis 存储 Nonce

```python
import redis
import uuid
import time

class RedisNonceManager:
    """基于 Redis 的 Nonce 管理器"""
    
    def __init__(self, redis_client: redis.Redis, ttl: int = 300):
        self.redis = redis_client
        self.ttl = ttl
    
    def generate_nonce(self) -> str:
        """生成 Nonce"""
        return str(uuid.uuid4())
    
    def is_valid(self, nonce: str) -> bool:
        """验证 Nonce"""
        key = f"nonce:{nonce}"
        
        # 尝试设置 Nonce（NX：不存在时才设置）
        result = self.redis.set(key, '1', ex=self.ttl, nx=True)
        
        # 如果设置成功，说明 Nonce 未被使用
        return result is not None

# 使用示例
redis_client = redis.Redis(host='localhost', port=6379, db=0)
nonce_manager = RedisNonceManager(redis_client)

# 客户端生成请求
def make_replay_safe_request(url: str, data: dict):
    """发送防重放请求"""
    import requests
    import json
    
    nonce = nonce_manager.generate_nonce()
    timestamp = str(int(time.time()))
    
    headers = {
        'X-Nonce': nonce,
        'X-Timestamp': timestamp,
        'Content-Type': 'application/json'
    }
    
    response = requests.post(url, data=json.dumps(data), headers=headers)
    return response.json()
```

### 组合防护：签名 + Nonce + 时间戳

```python
from flask import Flask, request, jsonify
from functools import wraps
import hmac
import hashlib
import time

app = Flask(__name__)

class SecureAPIValidator:
    """安全 API 验证器（组合防护）"""
    
    def __init__(self, secret_key: str, nonce_manager: NonceManager):
        self.secret_key = secret_key.encode()
        self.nonce_manager = nonce_manager
    
    def validate_request(self) -> tuple[bool, str]:
        """验证请求
        
        Returns:
            (is_valid, error_message)
        """
        # 1. 获取参数
        nonce = request.headers.get('X-Nonce')
        timestamp = request.headers.get('X-Timestamp')
        signature = request.headers.get('X-Signature')
        
        if not all([nonce, timestamp, signature]):
            return False, '缺少安全参数'
        
        # 2. 验证时间戳
        now = int(time.time())
        if abs(now - int(timestamp)) > 300:
            return False, '请求已过期'
        
        # 3. 验证 Nonce
        if not self.nonce_manager.is_valid(nonce):
            return False, '请求已被使用'
        
        # 4. 验证签名
        data = request.get_data().decode()
        expected_signature = self._generate_signature(data, timestamp, nonce)
        
        if not hmac.compare_digest(expected_signature, signature):
            return False, '签名验证失败'
        
        return True, ''
    
    def _generate_signature(self, data: str, timestamp: str, nonce: str) -> str:
        """生成签名"""
        message = f"{data}{timestamp}{nonce}".encode()
        return hmac.new(self.secret_key, message, hashlib.sha256).hexdigest()

# 创建验证器
validator = SecureAPIValidator(
    secret_key="your-secret-key",
    nonce_manager=nonce_manager
)

def secure_api(f):
    """安全 API 装饰器"""
    @wraps(f)
    def decorated(*args, **kwargs):
        is_valid, error = validator.validate_request()
        
        if not is_valid:
            return jsonify({'error': error}), 401
        
        return f(*args, **kwargs)
    
    return decorated

@app.route('/api/secure-transfer', methods=['POST'])
@secure_api
def secure_transfer():
    """安全转账接口"""
    data = request.get_json()
    return jsonify({
        'message': '转账成功',
        'amount': data.get('amount')
    })
```

---

## 实战案例

### 案例 1：构建安全的支付 API

```python
from flask import Flask, request, jsonify
from functools import wraps
import hmac
import hashlib
import time
import redis
from decimal import Decimal

app = Flask(__name__)
redis_client = redis.Redis(host='localhost', port=6379, db=0)

class PaymentAPI:
    """安全的支付 API"""
    
    def __init__(self, secret_key: str, redis_client: redis.Redis):
        self.secret_key = secret_key.encode()
        self.redis = redis_client
        self.rate_limiter = RedisRateLimiter(redis_client, max_requests=10, window_size=60)
        self.nonce_manager = RedisNonceManager(redis_client)
    
    def validate_payment_request(self) -> tuple[bool, str, dict]:
        """验证支付请求
        
        Returns:
            (is_valid, error_message, payment_data)
        """
        # 1. 限流检查
        client_ip = request.remote_addr
        if not self.rate_limiter.is_allowed(client_ip):
            return False, '请求过于频繁', {}
        
        # 2. 获取参数
        nonce = request.headers.get('X-Nonce')
        timestamp = request.headers.get('X-Timestamp')
        signature = request.headers.get('X-Signature')
        
        if not all([nonce, timestamp, signature]):
            return False, '缺少安全参数', {}
        
        # 3. 验证时间戳
        now = int(time.time())
        if abs(now - int(timestamp)) > 300:
            return False, '请求已过期', {}
        
        # 4. 验证 Nonce
        if not self.nonce_manager.is_valid(nonce):
            return False, '请求已被使用', {}
        
        # 5. 获取支付数据
        payment_data = request.get_json()
        
        # 6. 验证签名
        data_str = str(payment_data)
        expected_signature = self._generate_signature(data_str, timestamp, nonce)
        
        if not hmac.compare_digest(expected_signature, signature):
            return False, '签名验证失败', {}
        
        # 7. 验证支付金额
        amount = payment_data.get('amount')
        if not amount or Decimal(amount) <= 0:
            return False, '支付金额无效', {}
        
        return True, '', payment_data
    
    def _generate_signature(self, data: str, timestamp: str, nonce: str) -> str:
        """生成签名"""
        message = f"{data}{timestamp}{nonce}".encode()
        return hmac.new(self.secret_key, message, hashlib.sha256).hexdigest()

payment_api = PaymentAPI(secret_key="payment-secret-key", redis_client=redis_client)

@app.route('/api/payment', methods=['POST'])
def create_payment():
    """创建支付订单"""
    is_valid, error, payment_data = payment_api.validate_payment_request()
    
    if not is_valid:
        return jsonify({'error': error}), 400
    
    # 处理支付逻辑
    order_id = f"ORDER_{int(time.time())}"
    
    return jsonify({
        'order_id': order_id,
        'amount': payment_data['amount'],
        'status': 'pending'
    })
```

### 案例 2：OAuth2 + API 密钥双重认证

```python
from flask import Flask, request, jsonify
from functools import wraps
import jwt
from datetime import datetime, timedelta

app = Flask(__name__)

class DualAuthAPI:
    """双重认证 API（OAuth2 + API Key）"""
    
    def __init__(self, jwt_secret: str, api_keys: dict):
        self.jwt_secret = jwt_secret
        self.api_keys = api_keys  # {api_key: user_info}
    
    def verify_jwt_token(self, token: str) -> tuple[bool, dict]:
        """验证 JWT Token"""
        try:
            payload = jwt.decode(token, self.jwt_secret, algorithms=['HS256'])
            return True, payload
        except jwt.ExpiredSignatureError:
            return False, {'error': 'Token 已过期'}
        except jwt.InvalidTokenError:
            return False, {'error': 'Token 无效'}
    
    def verify_api_key(self, api_key: str) -> tuple[bool, dict]:
        """验证 API Key"""
        if api_key in self.api_keys:
            return True, self.api_keys[api_key]
        return False, {'error': 'API Key 无效'}

dual_auth = DualAuthAPI(
    jwt_secret="jwt-secret-key",
    api_keys={
        "key123": {"app_id": "app1", "name": "App 1"},
        "key456": {"app_id": "app2", "name": "App 2"}
    }
)

def require_dual_auth(f):
    """双重认证装饰器"""
    @wraps(f)
    def decorated(*args, **kwargs):
        # 1. 验证 API Key
        api_key = request.headers.get('X-API-Key')
        if not api_key:
            return jsonify({'error': '缺少 API Key'}), 401
        
        is_valid, app_info = dual_auth.verify_api_key(api_key)
        if not is_valid:
            return jsonify(app_info), 401
        
        # 2. 验证 JWT Token
        auth_header = request.headers.get('Authorization')
        if not auth_header or not auth_header.startswith('Bearer '):
            return jsonify({'error': '缺少 JWT Token'}), 401
        
        token = auth_header.split(' ')[1]
        is_valid, user_info = dual_auth.verify_jwt_token(token)
        if not is_valid:
            return jsonify(user_info), 401
        
        # 将认证信息添加到请求上下文
        request.app_info = app_info
        request.user_info = user_info
        
        return f(*args, **kwargs)
    
    return decorated

@app.route('/api/sensitive-data')
@require_dual_auth
def get_sensitive_data():
    """需要双重认证的敏感数据接口"""
    return jsonify({
        'app': request.app_info['name'],
        'user': request.user_info['username'],
        'data': 'sensitive information'
    })
```

### 案例 3：完整的 API 安全方案

```python
from flask import Flask, request, jsonify
from functools import wraps
import hmac
import hashlib
import time
import redis
import logging

app = Flask(__name__)
redis_client = redis.Redis(host='localhost', port=6379, db=0)

# 配置日志
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class ComprehensiveAPISecurityclass ComprehensiveAPISecurity:
    """综合 API 安全方案"""
    
    def __init__(self, config: dict):
        self.secret_key = config['secret_key'].encode()
        self.redis = config['redis_client']
        
        # 初始化各个安全组件
        self.rate_limiter = RedisRateLimiter(
            self.redis,
            max_requests=config.get('max_requests', 100),
            window_size=config.get('window_size', 60)
        )
        self.nonce_manager = RedisNonceManager(
            self.redis,
            ttl=config.get('nonce_ttl', 300)
        )
    
    def validate_request(self) -> tuple[bool, str]:
        """综合验证请求"""
        client_ip = request.remote_addr
        
        # 1. IP 限流
        if not self.rate_limiter.is_allowed(client_ip):
            logger.warning(f"Rate limit exceeded for IP: {client_ip}")
            return False, '请求过于频繁，请稍后再试'
        
        # 2. 获取安全参数
        api_key = request.headers.get('X-API-Key')
        nonce = request.headers.get('X-Nonce')
        timestamp = request.headers.get('X-Timestamp')
        signature = request.headers.get('X-Signature')
        
        if not all([api_key, nonce, timestamp, signature]):
            logger.warning(f"Missing security parameters from IP: {client_ip}")
            return False, '缺少安全参数'
        
        # 3. 验证时间戳（防过期）
        now = int(time.time())
        try:
            req_time = int(timestamp)
        except ValueError:
            return False, '时间戳格式错误'
        
        if abs(now - req_time) > 300:  # 5分钟有效期
            logger.warning(f"Expired request from IP: {client_ip}")
            return False, '请求已过期'
        
        # 4. 验证 Nonce（防重放）
        if not self.nonce_manager.is_valid(nonce):
            logger.warning(f"Replay attack detected from IP: {client_ip}")
            return False, '请求已被使用或无效'
        
        # 5. 验证签名
        data = request.get_data().decode()
        expected_signature = self._generate_signature(
            data, timestamp, nonce, api_key
        )
        
        if not hmac.compare_digest(expected_signature, signature):
            logger.error(f"Signature verification failed from IP: {client_ip}")
            return False, '签名验证失败'
        
        # 6. 记录审计日志
        self._log_audit(client_ip, api_key, request.path)
        
        return True, ''
    
    def _generate_signature(self, data: str, timestamp: str, 
                          nonce: str, api_key: str) -> str:
        """生成签名"""
        message = f"{data}{timestamp}{nonce}{api_key}".encode()
        return hmac.new(self.secret_key, message, hashlib.sha256).hexdigest()
    
    def _log_audit(self, ip: str, api_key: str, path: str):
        """记录审计日志"""
        log_entry = {
            'timestamp': datetime.now().isoformat(),
            'ip': ip,
            'api_key': api_key[:8] + '***',  # 脱敏
            'path': path,
            'method': request.method
        }
        logger.info(f"API Access: {log_entry}")

# 创建安全实例
security = ComprehensiveAPISecurity({
    'secret_key': 'your-secret-key',
    'redis_client': redis_client,
    'max_requests': 100,
    'window_size': 60,
    'nonce_ttl': 300
})

def secure_endpoint(f):
    """安全端点装饰器"""
    @wraps(f)
    def decorated(*args, **kwargs):
        is_valid, error = security.validate_request()
        
        if not is_valid:
            return jsonify({'error': error}), 401
        
        return f(*args, **kwargs)
    
    return decorated

@app.route('/api/v1/users', methods=['GET'])
@secure_endpoint
def get_users():
    """获取用户列表（安全端点）"""
    return jsonify({
        'users': [
            {'id': 1, 'name': 'User 1'},
            {'id': 2, 'name': 'User 2'}
        ]
    })

@app.route('/api/v1/orders', methods=['POST'])
@secure_endpoint
def create_order():
    """创建订单（安全端点）"""
    data = request.get_json()
    return jsonify({
        'order_id': f"ORD_{int(time.time())}",
        'status': 'created',
        'data': data
    })
```

---

## 总结

### API 安全最佳实践

| 安全措施 | 重要性 | 实施难度 | 推荐场景 |
|---------|--------|---------|---------|
| HTTPS | ⭐⭐⭐⭐⭐ | ⭐⭐ | 所有生产环境 |
| 接口限流 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | 所有公开 API |
| 签名验证 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | 支付、敏感操作 |
| API 密钥 | ⭐⭐⭐⭐ | ⭐⭐ | 第三方集成 |
| 防重放 | ⭐⭐⭐⭐ | ⭐⭐⭐ | 金融、支付接口 |
| 审计日志 | ⭐⭐⭐⭐⭐ | ⭐⭐ | 所有 API |

### 安全检查清单

```python
"""
API 安全检查清单：

传输安全
- [ ] 强制使用 HTTPS
- [ ] 配置 HSTS 头
- [ ] 使用 TLS 1.2+

认证授权
- [ ] 实施 API 密钥认证
- [ ] 使用 JWT Token
- [ ] 实施权限控制（RBAC）

防护措施
- [ ] 接口限流（固定窗口/滑动窗口/令牌桶）
- [ ] 签名验证（HMAC-SHA256）
- [ ] 防重放攻击（Nonce + 时间戳）
- [ ] 输入验证和过滤

监控审计
- [ ] 记录所有 API 访问日志
- [ ] 监控异常请求
- [ ] 设置告警规则
- [ ] 定期安全审计

密钥管理
- [ ] 密钥定期轮换
- [ ] 密钥加密存储
- [ ] 使用环境变量
- [ ] 实施密钥过期策略
"""
```

### 常见安全漏洞

| 漏洞类型 | 危害 | 防护方法 |
|---------|------|---------|
| 未加密传输 | 数据泄露 | 强制 HTTPS |
| 无限流保护 | DDoS 攻击 | 实施限流 |
| 缺少签名验证 | 数据篡改 | HMAC 签名 |
| 重放攻击 | 重复操作 | Nonce + 时间戳 |
| API 密钥泄露 | 未授权访问 | 密钥轮换 + 加密存储 |
| 日志泄露敏感信息 | 信息泄露 | 日志脱敏 |

### 性能优化建议

1. **使用 Redis 存储限流和 Nonce 数据**
   - 内存存储性能更好
   - 支持分布式部署
   - 自动过期清理

2. **签名验证优化**
   - 使用 HMAC 而非 RSA（性能更好）
   - 缓存验证结果
   - 异步记录审计日志

3. **限流策略选择**
   - 固定窗口：实现简单，适合低并发
   - 滑动窗口：更精确，适合中等并发
   - 令牌桶：平滑限流，适合高并发

### 学习路径

1. **基础阶段**（1-2周）
   - HTTPS 配置和证书管理
   - 基本的 API 密钥认证
   - 简单的固定窗口限流

2. **进阶阶段**（2-3周）
   - HMAC 签名验证
   - 滑动窗口和令牌桶限流
   - Nonce 防重放攻击

3. **高级阶段**（3-4周）
   - 分布式限流（Redis）
   - 双重认证（OAuth2 + API Key）
   - 完整的安全审计系统

4. **实战阶段**（4-6周）
   - 构建生产级安全 API
   - 性能优化和压力测试
   - 安全漏洞扫描和修复

### 推荐工具和库

```bash
# 限流
pip install flask-limiter

# 签名验证
# 使用标准库 hmac 和 hashlib

# Redis
pip install redis

# JWT
pip install pyjwt

# HTTPS
pip install pyopenssl

# 安全扫描
pip install bandit safety
```

### 参考资源

- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [Flask Security Best Practices](https://flask.palletsprojects.com/en/latest/security/)
- [FastAPI Security](https://fastapi.tiangolo.com/tutorial/security/)
- [Redis Rate Limiting](https://redis.io/docs/manual/patterns/rate-limiter/)

---

## 下一步学习

完成本教程后，建议继续学习：

1. [Web 安全完整教程](Web安全-完整教程.md) - SQL 注入、XSS、CSRF 防护
2. [认证授权完整教程](认证授权-完整教程.md) - JWT、OAuth2、RBAC
3. [数据加密完整教程](数据加密-完整教程.md) - 密码哈希、对称加密、非对称加密

---

**记住：API 安全是系统安全的第一道防线，必须严格把关！** 🔒

@author erik.zhou
