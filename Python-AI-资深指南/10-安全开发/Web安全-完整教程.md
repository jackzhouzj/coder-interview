# Python Web 安全完整教程

> @author erik.zhou

## 📋 目录

1. [Web 安全概述](#web-安全概述)
2. [SQL 注入防护](#sql-注入防护)
3. [XSS 跨站脚本攻击防护](#xss-跨站脚本攻击防护)
4. [CSRF 跨站请求伪造防护](#csrf-跨站请求伪造防护)
5. [文件上传安全](#文件上传安全)
6. [命令注入防护](#命令注入防护)
7. [安全配置](#安全配置)
8. [实战案例](#实战案例)

---

## Web 安全概述

### OWASP Top 10 (2021)

| 排名 | 漏洞类型 | 严重程度 | 常见场景 |
|------|---------|---------|---------|
| A01 | 访问控制失效 | 严重 | 未授权访问 |
| A02 | 加密失败 | 严重 | 明文传输 |
| A03 | 注入攻击 | 严重 | SQL/命令注入 |
| A04 | 不安全设计 | 高 | 架构缺陷 |
| A05 | 安全配置错误 | 高 | 默认配置 |
| A06 | 易受攻击组件 | 高 | 过期依赖 |
| A07 | 身份认证失败 | 高 | 弱密码 |
| A08 | 软件数据完整性失败 | 中 | 未验证更新 |
| A09 | 安全日志不足 | 中 | 无审计 |
| A10 | 服务端请求伪造 | 中 | SSRF |

### 安全开发原则

```python
"""
安全开发的核心原则：

1. 永远不信任用户输入
2. 最小权限原则
3. 纵深防御
4. 安全默认配置
5. 失败安全
"""
```

---

## SQL 注入防护

### 什么是 SQL 注入？

```python
# ❌ 危险：SQL 注入漏洞
def get_user_unsafe(username):
    """直接拼接 SQL，存在注入风险"""
    query = f"SELECT * FROM users WHERE username = '{username}'"
    cursor.execute(query)
    return cursor.fetchone()

# 攻击示例
# username = "admin' OR '1'='1"
# 实际执行：SELECT * FROM users WHERE username = 'admin' OR '1'='1'
# 结果：返回所有用户数据
```

### 防护方法

```python
import sqlite3
from typing import Optional, Dict

# ✅ 安全：使用参数化查询
def get_user_safe(username: str) -> Optional[Dict]:
    """使用参数化查询防止 SQL 注入"""
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()
    
    # 使用占位符
    query = "SELECT * FROM users WHERE username = ?"
    cursor.execute(query, (username,))
    
    result = cursor.fetchone()
    conn.close()
    return result

# 使用 ORM（推荐）
from sqlalchemy import create_engine, Column, String, Integer
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    username = Column(String(50), unique=True)
    email = Column(String(100))

# 安全查询
def get_user_orm(username: str) -> Optional[User]:
    """使用 ORM 自动防止 SQL 注入"""
    engine = create_engine('sqlite:///database.db')
    Session = sessionmaker(bind=engine)
    session = Session()
    
    user = session.query(User).filter(User.username == username).first()
    session.close()
    return user
```

### 动态查询安全

```python
from sqlalchemy import text

# ❌ 危险：动态拼接
def search_users_unsafe(keyword: str):
    query = f"SELECT * FROM users WHERE username LIKE '%{keyword}%'"
    return session.execute(query)

# ✅ 安全：使用绑定参数
def search_users_safe(keyword: str):
    query = text("SELECT * FROM users WHERE username LIKE :keyword")
    return session.execute(query, {"keyword": f"%{keyword}%"})
```


---

## XSS 跨站脚本攻击防护

### 什么是 XSS？

```python
# ❌ 危险：XSS 漏洞
from flask import Flask, request, render_template_string

app = Flask(__name__)

@app.route('/search')
def search_unsafe():
    """未转义用户输入，存在 XSS 风险"""
    keyword = request.args.get('q', '')
    # 直接输出用户输入
    return f"<h1>搜索结果：{keyword}</h1>"

# 攻击示例
# URL: /search?q=<script>alert('XSS')</script>
# 结果：执行恶意脚本
```

### 防护方法

```python
from flask import Flask, request, render_template_string, escape
from markupsafe import Markup
import html

app = Flask(__name__)

# ✅ 安全：自动转义（推荐）
@app.route('/search')
def search_safe():
    """使用模板引擎自动转义"""
    keyword = request.args.get('q', '')
    # Jinja2 自动转义
    return render_template_string(
        "<h1>搜索结果：{{ keyword }}</h1>",
        keyword=keyword
    )

# ✅ 安全：手动转义
@app.route('/search2')
def search_manual():
    """手动转义用户输入"""
    keyword = request.args.get('q', '')
    # 使用 html.escape 转义
    safe_keyword = html.escape(keyword)
    return f"<h1>搜索结果：{safe_keyword}</h1>"

# ✅ 安全：使用白名单
import bleach

@app.route('/comment')
def post_comment():
    """允许部分 HTML 标签"""
    comment = request.form.get('comment', '')
    
    # 只允许特定标签
    allowed_tags = ['p', 'br', 'strong', 'em']
    allowed_attrs = {}
    
    clean_comment = bleach.clean(
        comment,
        tags=allowed_tags,
        attributes=allowed_attrs,
        strip=True
    )
    
    return clean_comment
```

### 内容安全策略（CSP）

```python
from flask import Flask, make_response

app = Flask(__name__)

@app.after_request
def set_csp(response):
    """设置 CSP 头"""
    response.headers['Content-Security-Policy'] = (
        "default-src 'self'; "
        "script-src 'self' 'unsafe-inline' https://cdn.example.com; "
        "style-src 'self' 'unsafe-inline'; "
        "img-src 'self' data: https:; "
        "font-src 'self' https://fonts.gstatic.com; "
        "connect-src 'self' https://api.example.com; "
        "frame-ancestors 'none';"
    )
    return response
```

---

## CSRF 跨站请求伪造防护

### 什么是 CSRF？

```python
# ❌ 危险：无 CSRF 保护
from flask import Flask, request

app = Flask(__name__)

@app.route('/transfer', methods=['POST'])
def transfer_unsafe():
    """未验证请求来源，存在 CSRF 风险"""
    to_account = request.form.get('to')
    amount = request.form.get('amount')
    
    # 直接执行转账
    transfer_money(to_account, amount)
    return "转账成功"

# 攻击示例：攻击者构造恶意页面
# <form action="https://bank.com/transfer" method="POST">
#   <input name="to" value="attacker_account">
#   <input name="amount" value="10000">
# </form>
# <script>document.forms[0].submit()</script>
```

### 防护方法

```python
from flask import Flask, request, session, abort
from flask_wtf.csrf import CSRFProtect
import secrets

app = Flask(__name__)
app.config['SECRET_KEY'] = secrets.token_hex(32)

# 方法 1：使用 Flask-WTF（推荐）
csrf = CSRFProtect(app)

@app.route('/transfer', methods=['POST'])
@csrf.exempt  # 如需豁免，显式标记
def transfer_safe():
    """自动验证 CSRF Token"""
    to_account = request.form.get('to')
    amount = request.form.get('amount')
    
    transfer_money(to_account, amount)
    return "转账成功"

# 前端模板
"""
<form method="POST" action="/transfer">
    <input type="hidden" name="csrf_token" value="{{ csrf_token() }}">
    <input name="to" placeholder="收款账户">
    <input name="amount" placeholder="金额">
    <button type="submit">转账</button>
</form>
"""

# 方法 2：手动实现 CSRF Token
def generate_csrf_token():
    """生成 CSRF Token"""
    if 'csrf_token' not in session:
        session['csrf_token'] = secrets.token_hex(32)
    return session['csrf_token']

def verify_csrf_token(token):
    """验证 CSRF Token"""
    return token == session.get('csrf_token')

@app.route('/transfer2', methods=['POST'])
def transfer_manual():
    """手动验证 CSRF Token"""
    token = request.form.get('csrf_token')
    
    if not verify_csrf_token(token):
        abort(403, "CSRF token 验证失败")
    
    to_account = request.form.get('to')
    amount = request.form.get('amount')
    
    transfer_money(to_account, amount)
    return "转账成功"

# 方法 3：双重 Cookie 验证
@app.route('/transfer3', methods=['POST'])
def transfer_double_cookie():
    """使用双重 Cookie 验证"""
    cookie_token = request.cookies.get('csrf_token')
    header_token = request.headers.get('X-CSRF-Token')
    
    if not cookie_token or cookie_token != header_token:
        abort(403, "CSRF 验证失败")
    
    # 执行业务逻辑
    return "转账成功"
```

### SameSite Cookie

```python
from flask import Flask, make_response

app = Flask(__name__)

@app.route('/login', methods=['POST'])
def login():
    """设置 SameSite Cookie"""
    response = make_response("登录成功")
    
    # 设置 SameSite 属性
    response.set_cookie(
        'session_id',
        value='xxx',
        httponly=True,      # 防止 JavaScript 访问
        secure=True,        # 仅 HTTPS 传输
        samesite='Strict'   # 严格模式，防止 CSRF
    )
    
    return response
```

---

## 文件上传安全

### 文件上传漏洞

```python
# ❌ 危险：未验证文件类型
from flask import Flask, request
import os

app = Flask(__name__)

@app.route('/upload', methods=['POST'])
def upload_unsafe():
    """未验证文件，存在安全风险"""
    file = request.files['file']
    
    # 直接使用用户提供的文件名
    filename = file.filename
    file.save(os.path.join('uploads', filename))
    
    return "上传成功"

# 攻击示例：
# 1. 上传 shell.php，获取服务器控制权
# 2. 上传 ../../../etc/passwd，路径遍历
# 3. 上传超大文件，DoS 攻击
```

### 安全的文件上传

```python
from flask import Flask, request, abort
from werkzeug.utils import secure_filename
import os
import magic
import hashlib
from pathlib import Path

app = Flask(__name__)

# 配置
UPLOAD_FOLDER = 'uploads'
MAX_FILE_SIZE = 5 * 1024 * 1024  # 5MB
ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif', 'pdf'}
ALLOWED_MIMETYPES = {
    'image/png', 'image/jpeg', 'image/gif', 'application/pdf'
}

def allowed_file(filename: str) -> bool:
    """检查文件扩展名"""
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

def verify_file_type(file_path: str) -> bool:
    """验证文件真实类型（基于文件内容）"""
    mime = magic.Magic(mime=True)
    file_type = mime.from_file(file_path)
    return file_type in ALLOWED_MIMETYPES

def generate_safe_filename(original_filename: str) -> str:
    """生成安全的文件名"""
    # 获取扩展名
    ext = original_filename.rsplit('.', 1)[1].lower()
    
    # 使用哈希生成唯一文件名
    unique_id = hashlib.md5(
        f"{original_filename}{os.urandom(16)}".encode()
    ).hexdigest()
    
    return f"{unique_id}.{ext}"

@app.route('/upload', methods=['POST'])
def upload_safe():
    """安全的文件上传"""
    # 1. 检查文件是否存在
    if 'file' not in request.files:
        abort(400, "未上传文件")
    
    file = request.files['file']
    
    if file.filename == '':
        abort(400, "文件名为空")
    
    # 2. 检查文件大小
    file.seek(0, os.SEEK_END)
    file_size = file.tell()
    file.seek(0)
    
    if file_size > MAX_FILE_SIZE:
        abort(400, f"文件过大，最大允许 {MAX_FILE_SIZE/1024/1024}MB")
    
    # 3. 检查文件扩展名
    if not allowed_file(file.filename):
        abort(400, "不允许的文件类型")
    
    # 4. 生成安全的文件名
    safe_filename = generate_safe_filename(file.filename)
    file_path = os.path.join(UPLOAD_FOLDER, safe_filename)
    
    # 5. 保存文件
    file.save(file_path)
    
    # 6. 验证文件真实类型
    if not verify_file_type(file_path):
        os.remove(file_path)
        abort(400, "文件类型验证失败")
    
    # 7. 设置文件权限（只读）
    os.chmod(file_path, 0o444)
    
    return {
        "message": "上传成功",
        "filename": safe_filename
    }

# 图片处理（防止图片马）
from PIL import Image

def sanitize_image(file_path: str):
    """清理图片，移除潜在恶意代码"""
    try:
        img = Image.open(file_path)
        
        # 重新保存图片，移除 EXIF 等元数据
        clean_path = f"{file_path}.clean"
        img.save(clean_path, quality=95)
        
        # 替换原文件
        os.replace(clean_path, file_path)
        
    except Exception as e:
        raise ValueError(f"图片处理失败: {e}")
```

---

## 命令注入防护

### 什么是命令注入？

```python
import os
import subprocess

# ❌ 危险：命令注入漏洞
def ping_unsafe(host: str):
    """直接拼接命令，存在注入风险"""
    command = f"ping -c 4 {host}"
    os.system(command)

# 攻击示例
# host = "8.8.8.8; rm -rf /"
# 实际执行：ping -c 4 8.8.8.8; rm -rf /
```

### 防护方法

```python
import subprocess
import shlex
import re
from typing import List

# ✅ 安全：使用列表参数
def ping_safe(host: str):
    """使用列表参数，防止命令注入"""
    # 验证输入
    if not re.match(r'^[\w\.-]+$', host):
        raise ValueError("无效的主机名")
    
    # 使用列表参数
    result = subprocess.run(
        ['ping', '-c', '4', host],
        capture_output=True,
        text=True,
        timeout=10
    )
    
    return result.stdout

# ✅ 安全：使用 shlex.quote
def execute_command_safe(command: str, args: List[str]):
    """使用 shlex.quote 转义参数"""
    safe_args = [shlex.quote(arg) for arg in args]
    full_command = f"{command} {' '.join(safe_args)}"
    
    result = subprocess.run(
        full_command,
        shell=True,
        capture_output=True,
        text=True,
        timeout=10
    )
    
    return result.stdout

# ✅ 最佳实践：避免使用 shell
def convert_image_safe(input_file: str, output_file: str):
    """图片转换（不使用 shell）"""
    # 验证文件路径
    if not os.path.exists(input_file):
        raise FileNotFoundError("输入文件不存在")
    
    # 使用列表参数，不使用 shell
    subprocess.run(
        ['convert', input_file, output_file],
        check=True,
        timeout=30
    )
```

---

## 安全配置

### Flask 安全配置

```python
from flask import Flask
import secrets

app = Flask(__name__)

# 1. 密钥配置
app.config['SECRET_KEY'] = secrets.token_hex(32)  # 随机生成

# 2. Session 配置
app.config['SESSION_COOKIE_SECURE'] = True      # 仅 HTTPS
app.config['SESSION_COOKIE_HTTPONLY'] = True    # 防止 XSS
app.config['SESSION_COOKIE_SAMESITE'] = 'Lax'   # 防止 CSRF
app.config['PERMANENT_SESSION_LIFETIME'] = 3600  # 1 小时过期

# 3. 安全头
@app.after_request
def set_security_headers(response):
    """设置安全响应头"""
    # 防止点击劫持
    response.headers['X-Frame-Options'] = 'DENY'
    
    # 防止 MIME 类型嗅探
    response.headers['X-Content-Type-Options'] = 'nosniff'
    
    # XSS 保护
    response.headers['X-XSS-Protection'] = '1; mode=block'
    
    # HTTPS 强制
    response.headers['Strict-Transport-Security'] = (
        'max-age=31536000; includeSubDomains'
    )
    
    # 引用策略
    response.headers['Referrer-Policy'] = 'strict-origin-when-cross-origin'
    
    # 权限策略
    response.headers['Permissions-Policy'] = (
        'geolocation=(), microphone=(), camera=()'
    )
    
    return response

# 4. 错误处理
@app.errorhandler(404)
def not_found(error):
    """不泄露敏感信息"""
    return {"error": "资源不存在"}, 404

@app.errorhandler(500)
def internal_error(error):
    """不泄露堆栈信息"""
    # 记录详细错误到日志
    app.logger.error(f"Internal error: {error}")
    # 返回通用错误信息
    return {"error": "服务器内部错误"}, 500
```

### Django 安全配置

```python
# settings.py

# 1. 密钥配置
import secrets
SECRET_KEY = secrets.token_hex(50)

# 2. 调试模式
DEBUG = False  # 生产环境必须关闭

# 3. 允许的主机
ALLOWED_HOSTS = ['example.com', 'www.example.com']

# 4. 安全中间件
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    # ...
]

# 5. HTTPS 配置
SECURE_SSL_REDIRECT = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True

# 6. HSTS
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# 7. Cookie 配置
SESSION_COOKIE_HTTPONLY = True
CSRF_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = 'Strict'
CSRF_COOKIE_SAMESITE = 'Strict'

# 8. 内容安全策略
CSP_DEFAULT_SRC = ("'self'",)
CSP_SCRIPT_SRC = ("'self'", "'unsafe-inline'")
CSP_STYLE_SRC = ("'self'", "'unsafe-inline'")

# 9. 密码验证
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {'min_length': 12}
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]
```

---

## 实战案例

### 案例 1：安全的用户注册

```python
from flask import Flask, request, jsonify
from werkzeug.security import generate_password_hash
import re
from typing import Dict

app = Flask(__name__)

def validate_email(email: str) -> bool:
    """验证邮箱格式"""
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(pattern, email) is not None

def validate_password(password: str) -> Dict[str, bool]:
    """验证密码强度"""
    checks = {
        'length': len(password) >= 12,
        'uppercase': bool(re.search(r'[A-Z]', password)),
        'lowercase': bool(re.search(r'[a-z]', password)),
        'digit': bool(re.search(r'\d', password)),
        'special': bool(re.search(r'[!@#$%^&*(),.?":{}|<>]', password))
    }
    return checks

@app.route('/register', methods=['POST'])
def register():
    """安全的用户注册"""
    data = request.get_json()
    
    # 1. 验证必填字段
    required_fields = ['username', 'email', 'password']
    for field in required_fields:
        if field not in data:
            return jsonify({"error": f"缺少字段: {field}"}), 400
    
    username = data['username']
    email = data['email']
    password = data['password']
    
    # 2. 验证用户名
    if not re.match(r'^[a-zA-Z0-9_]{3,20}$', username):
        return jsonify({
            "error": "用户名只能包含字母、数字、下划线，长度 3-20"
        }), 400
    
    # 3. 验证邮箱
    if not validate_email(email):
        return jsonify({"error": "邮箱格式不正确"}), 400
    
    # 4. 验证密码强度
    password_checks = validate_password(password)
    if not all(password_checks.values()):
        return jsonify({
            "error": "密码必须至少 12 位，包含大小写字母、数字和特殊字符",
            "checks": password_checks
        }), 400
    
    # 5. 检查用户名/邮箱是否已存在
    if user_exists(username, email):
        return jsonify({"error": "用户名或邮箱已存在"}), 400
    
    # 6. 哈希密码
    password_hash = generate_password_hash(
        password,
        method='pbkdf2:sha256',
        salt_length=16
    )
    
    # 7. 保存用户
    user_id = create_user(username, email, password_hash)
    
    # 8. 不返回敏感信息
    return jsonify({
        "message": "注册成功",
        "user_id": user_id
    }), 201

def user_exists(username: str, email: str) -> bool:
    """检查用户是否存在"""
    # 实现数据库查询
    pass

def create_user(username: str, email: str, password_hash: str) -> int:
    """创建用户"""
    # 实现数据库插入
    pass
```

### 案例 2：安全的 API 端点

```python
from flask import Flask, request, jsonify, g
from functools import wraps
import jwt
import time
from typing import Callable

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key'

def require_auth(f: Callable) -> Callable:
    """认证装饰器"""
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get('Authorization')
        
        if not token:
            return jsonify({"error": "缺少认证令牌"}), 401
        
        try:
            # 移除 "Bearer " 前缀
            if token.startswith('Bearer '):
                token = token[7:]
            
            # 验证 JWT
            payload = jwt.decode(
                token,
                app.config['SECRET_KEY'],
                algorithms=['HS256']
            )
            
            # 检查过期时间
            if payload['exp'] < time.time():
                return jsonify({"error": "令牌已过期"}), 401
            
            # 保存用户信息到 g 对象
            g.user_id = payload['user_id']
            
        except jwt.InvalidTokenError:
            return jsonify({"error": "无效的令牌"}), 401
        
        return f(*args, **kwargs)
    
    return decorated

def require_role(role: str) -> Callable:
    """角色权限装饰器"""
    def decorator(f: Callable) -> Callable:
        @wraps(f)
        def decorated(*args, **kwargs):
            user_role = get_user_role(g.user_id)
            
            if user_role != role:
                return jsonify({"error": "权限不足"}), 403
            
            return f(*args, **kwargs)
        
        return decorated
    return decorator

@app.route('/api/users', methods=['GET'])
@require_auth
@require_role('admin')
def get_users():
    """获取用户列表（需要管理员权限）"""
    users = fetch_users()
    return jsonify(users)

@app.route('/api/profile', methods=['GET'])
@require_auth
def get_profile():
    """获取当前用户信息"""
    user = fetch_user(g.user_id)
    return jsonify(user)

def get_user_role(user_id: int) -> str:
    """获取用户角色"""
    # 实现数据库查询
    pass

def fetch_users():
    """获取用户列表"""
    # 实现数据库查询
    pass

def fetch_user(user_id: int):
    """获取用户信息"""
    # 实现数据库查询
    pass
```

---

## 总结

### 安全检查清单

- [ ] 所有用户输入都经过验证和过滤
- [ ] 使用参数化查询防止 SQL 注入
- [ ] 对所有输出进行转义防止 XSS
- [ ] 实施 CSRF 保护
- [ ] 文件上传验证类型和大小
- [ ] 避免命令注入，不使用 shell=True
- [ ] 设置安全响应头
- [ ] 使用 HTTPS 传输
- [ ] 实施访问控制和权限验证
- [ ] 记录安全审计日志

### 关键要点

1. **永远不信任用户输入**：所有外部输入都要验证
2. **使用白名单而非黑名单**：明确允许什么，而不是禁止什么
3. **纵深防御**：多层安全措施
4. **最小权限原则**：只授予必要的权限
5. **安全默认配置**：默认配置应该是安全的

### 学习路径

1. 第 1 周：理解常见 Web 漏洞
2. 第 2 周：掌握 SQL 注入和 XSS 防护
3. 第 3 周：学习 CSRF 和文件上传安全
4. 第 4 周：实践安全配置和审计
5. 第 5 周：在实际项目中应用

**记住：安全是一个持续的过程，不是一次性的任务！** 🔒

@author erik.zhou
