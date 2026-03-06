# HTTPS与SSL/TLS - 完整教程

## 课程简介
深入理解HTTPS协议和SSL/TLS加密机制，掌握证书管理、安全配置和性能优化。

## 学习目标
- 理解HTTPS工作原理
- 掌握SSL/TLS握手过程
- 学会证书管理和配置
- 了解加密算法和安全最佳实践
- 实践HTTPS性能优化

---

## 第一部分：HTTPS基础

### 1.1 HTTPS概述

#### HTTPS vs HTTP
```javascript
/**
 * HTTPS与HTTP对比
 * @author erik.zhou
 */

const protocolComparison = {
    http: {
        name: 'HTTP',
        fullName: 'HyperText Transfer Protocol',
        port: 80,
        security: {
            encryption: false,
            authentication: false,
            integrity: false
        },
        advantages: [
            '简单快速',
            '无需证书',
            '兼容性好'
        ],
        disadvantages: [
            '明文传输，易被窃听',
            '无法验证身份',
            '数据可被篡改'
        ],
        useCases: [
            '公开信息展示',
            '开发测试环境'
        ]
    },
    
    https: {
        name: 'HTTPS',
        fullName: 'HyperText Transfer Protocol Secure',
        port: 443,
        security: {
            encryption: true,
            authentication: true,
            integrity: true
        },
        advantages: [
            '数据加密传输',
            '身份认证',
            '数据完整性保护',
            'SEO友好',
            '浏览器信任标识'
        ],
        disadvantages: [
            '需要SSL证书',
            '性能开销（可优化）',
            '配置相对复杂'
        ],
        useCases: [
            '电商网站',
            '金融服务',
            '用户登录',
            '敏感数据传输',
            '所有生产环境'
        ]
    }
};

// 打印对比
function printComparison() {
    console.log('=== HTTP vs HTTPS ===\n');
    
    console.log('HTTP:');
    console.log(`  端口: ${protocolComparison.http.port}`);
    console.log(`  加密: ${protocolComparison.http.security.encryption ? '是' : '否'}`);
    console.log(`  优点: ${protocolComparison.http.advantages.join(', ')}`);
    console.log(`  缺点: ${protocolComparison.http.disadvantages.join(', ')}`);
    
    console.log('\nHTTPS:');
    console.log(`  端口: ${protocolComparison.https.port}`);
    console.log(`  加密: ${protocolComparison.https.security.encryption ? '是' : '否'}`);
    console.log(`  优点: ${protocolComparison.https.advantages.join(', ')}`);
    console.log(`  缺点: ${protocolComparison.https.disadvantages.join(', ')}`);
}

printComparison();
```

#### HTTPS工作原理
```javascript
/**
 * HTTPS工作原理示例
 * @author erik.zhou
 */

class HTTPSConnection {
    constructor() {
        this.steps = [];
    }
    
    // 建立HTTPS连接
    async establish(url) {
        console.log('=== HTTPS连接建立过程 ===');
        console.log(`连接到: ${url}\n`);
        
        // 1. TCP连接
        await this.tcpHandshake();
        
        // 2. TLS握手
        await this.tlsHandshake();
        
        // 3. 加密通信
        await this.encryptedCommunication();
        
        // 4. 关闭连接
        await this.closeConnection();
        
        console.log('\n完成步骤:', this.steps);
    }
    
    // TCP三次握手
    async tcpHandshake() {
        this.steps.push('TCP握手');
        console.log('第1步: TCP三次握手');
        console.log('  1. 客户端 -> SYN -> 服务器');
        await this.delay(50);
        
        console.log('  2. 服务器 -> SYN-ACK -> 客户端');
        await this.delay(50);
        
        console.log('  3. 客户端 -> ACK -> 服务器');
        await this.delay(50);
        
        console.log('  TCP连接已建立\n');
    }
    
    // TLS握手
    async tlsHandshake() {
        this.steps.push('TLS握手');
        console.log('第2步: TLS握手');
        
        // Client Hello
        console.log('  1. Client Hello');
        console.log('     - 支持的TLS版本');
        console.log('     - 支持的加密套件');
        console.log('     - 随机数');
        await this.delay(50);
        
        // Server Hello
        console.log('  2. Server Hello');
        console.log('     - 选择的TLS版本');
        console.log('     - 选择的加密套件');
        console.log('     - 随机数');
        console.log('     - 服务器证书');
        await this.delay(50);
        
        // 证书验证
        console.log('  3. 客户端验证证书');
        console.log('     - 检查证书有效期');
        console.log('     - 验证证书链');
        console.log('     - 检查域名匹配');
        await this.delay(50);
        
        // 密钥交换
        console.log('  4. 密钥交换');
        console.log('     - 生成预主密钥');
        console.log('     - 使用服务器公钥加密');
        console.log('     - 发送给服务器');
        await this.delay(50);
        
        // 生成会话密钥
        console.log('  5. 生成会话密钥');
        console.log('     - 客户端和服务器各自生成');
        console.log('     - 用于后续对称加密');
        await this.delay(50);
        
        // Finished消息
        console.log('  6. 交换Finished消息');
        console.log('     - 验证握手完整性');
        await this.delay(50);
        
        console.log('  TLS握手完成\n');
    }
    
    // 加密通信
    async encryptedCommunication() {
        this.steps.push('加密通信');
        console.log('第3步: 加密通信');
        console.log('  使用会话密钥加密所有数据');
        console.log('  - 请求数据加密');
        console.log('  - 响应数据加密');
        await this.delay(100);
        console.log('  数据传输完成\n');
    }
    
    // 关闭连接
    async closeConnection() {
        this.steps.push('关闭连接');
        console.log('第4步: 关闭连接');
        console.log('  发送close_notify警报');
        await this.delay(50);
        console.log('  连接已关闭');
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
const httpsConn = new HTTPSConnection();
httpsConn.establish('https://example.com');
```

### 1.2 SSL/TLS协议

#### TLS版本对比
```javascript
/**
 * TLS版本对比
 * @author erik.zhou
 */

const tlsVersions = {
    'SSL 2.0': {
        year: 1995,
        status: '已废弃',
        security: '不安全',
        vulnerabilities: ['多个严重漏洞'],
        recommendation: '禁止使用'
    },
    
    'SSL 3.0': {
        year: 1996,
        status: '已废弃',
        security: '不安全',
        vulnerabilities: ['POODLE攻击'],
        recommendation: '禁止使用'
    },
    
    'TLS 1.0': {
        year: 1999,
        status: '已废弃',
        security: '弱',
        vulnerabilities: ['BEAST攻击', '弱加密套件'],
        recommendation: '不推荐使用'
    },
    
    'TLS 1.1': {
        year: 2006,
        status: '已废弃',
        security: '弱',
        vulnerabilities: ['弱加密套件'],
        recommendation: '不推荐使用'
    },
    
    'TLS 1.2': {
        year: 2008,
        status: '广泛使用',
        security: '安全',
        features: [
            '支持AEAD加密',
            '支持SHA-256',
            '改进的密钥派生'
        ],
        recommendation: '推荐使用'
    },
    
    'TLS 1.3': {
        year: 2018,
        status: '最新标准',
        security: '最安全',
        features: [
            '简化握手（1-RTT）',
            '支持0-RTT',
            '移除弱加密算法',
            '前向保密',
            '更快的连接建立'
        ],
        improvements: [
            '握手时间减少',
            '更强的安全性',
            '更简洁的协议'
        ],
        recommendation: '强烈推荐'
    }
};

// 打印版本信息
function printTLSVersions() {
    console.log('=== TLS版本对比 ===\n');
    
    for (const [version, info] of Object.entries(tlsVersions)) {
        console.log(`${version} (${info.year})`);
        console.log(`  状态: ${info.status}`);
        console.log(`  安全性: ${info.security}`);
        
        if (info.features) {
            console.log(`  特性: ${info.features.join(', ')}`);
        }
        
        if (info.vulnerabilities) {
            console.log(`  漏洞: ${info.vulnerabilities.join(', ')}`);
        }
        
        console.log(`  建议: ${info.recommendation}\n`);
    }
}

printTLSVersions();
```

#### TLS 1.3握手优化
```javascript
/**
 * TLS 1.3握手优化
 * @author erik.zhou
 */

class TLS13Handshake {
    constructor() {
        this.sessionCache = new Map();
    }
    
    // 1-RTT握手（首次连接）
    async oneRTTHandshake(domain) {
        console.log('=== TLS 1.3 1-RTT握手 ===');
        console.log(`连接到: ${domain}\n`);
        
        const startTime = Date.now();
        
        // 第一次往返
        console.log('第1次往返:');
        console.log('  客户端 -> ClientHello + KeyShare');
        await this.delay(50);
        
        console.log('  服务器 -> ServerHello + KeyShare + Certificate + Finished');
        await this.delay(50);
        
        // 立即可以发送应用数据
        console.log('\n可以发送应用数据');
        console.log('  客户端 -> Finished + 应用数据');
        await this.delay(50);
        
        const duration = Date.now() - startTime;
        console.log(`\n握手完成，耗时: ${duration}ms`);
        
        // 保存会话信息
        this.saveSession(domain);
        
        return duration;
    }
    
    // 0-RTT握手（会话恢复）
    async zeroRTTHandshake(domain) {
        console.log('\n=== TLS 1.3 0-RTT握手（会话恢复）===');
        console.log(`连接到: ${domain}\n`);
        
        const startTime = Date.now();
        
        // 检查会话票据
        if (!this.hasSession(domain)) {
            console.log('无会话票据，使用1-RTT握手');
            return await this.oneRTTHandshake(domain);
        }
        
        // 0-RTT：立即发送应用数据
        console.log('使用会话票据:');
        console.log('  客户端 -> ClientHello + 0-RTT数据');
        await this.delay(25);
        
        console.log('  服务器 -> ServerHello + 应用数据');
        await this.delay(25);
        
        const duration = Date.now() - startTime;
        console.log(`\n握手完成，耗时: ${duration}ms`);
        console.log(`性能提升: 约50%`);
        
        return duration;
    }
    
    // TLS 1.2握手对比
    async tls12Handshake(domain) {
        console.log('\n=== TLS 1.2握手（对比）===');
        console.log(`连接到: ${domain}\n`);
        
        const startTime = Date.now();
        
        // 第一次往返
        console.log('第1次往返:');
        console.log('  客户端 -> ClientHello');
        await this.delay(50);
        
        console.log('  服务器 -> ServerHello + Certificate');
        await this.delay(50);
        
        // 第二次往返
        console.log('\n第2次往返:');
        console.log('  客户端 -> KeyExchange + ChangeCipherSpec + Finished');
        await this.delay(50);
        
        console.log('  服务器 -> ChangeCipherSpec + Finished');
        await this.delay(50);
        
        // 现在才能发送应用数据
        console.log('\n可以发送应用数据');
        await this.delay(50);
        
        const duration = Date.now() - startTime;
        console.log(`\n握手完成，耗时: ${duration}ms`);
        
        return duration;
    }
    
    // 保存会话
    saveSession(domain) {
        this.sessionCache.set(domain, {
            ticket: 'session-ticket-' + Math.random().toString(36),
            timestamp: Date.now()
        });
    }
    
    // 检查会话
    hasSession(domain) {
        return this.sessionCache.has(domain);
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
async function compareTLSHandshakes() {
    const tls = new TLS13Handshake();
    const domain = 'example.com';
    
    // TLS 1.2握手
    const tls12Time = await tls.tls12Handshake(domain);
    
    // TLS 1.3 1-RTT握手
    const tls13Time = await tls.oneRTTHandshake(domain);
    
    // TLS 1.3 0-RTT握手
    const tls13ZeroTime = await tls.zeroRTTHandshake(domain);
    
    // 性能对比
    console.log('\n=== 性能对比 ===');
    console.log(`TLS 1.2: ${tls12Time}ms`);
    console.log(`TLS 1.3 (1-RTT): ${tls13Time}ms (提升 ${((1 - tls13Time / tls12Time) * 100).toFixed(0)}%)`);
    console.log(`TLS 1.3 (0-RTT): ${tls13ZeroTime}ms (提升 ${((1 - tls13ZeroTime / tls12Time) * 100).toFixed(0)}%)`);
}

compareTLSHandshakes();
```

## 第二部分：证书管理

### 2.1 SSL证书

#### 证书类型
```javascript
/**
 * SSL证书类型
 * @author erik.zhou
 */

const certificateTypes = {
    dv: {
        name: 'DV证书',
        fullName: 'Domain Validation',
        validation: '域名验证',
        validationTime: '几分钟到几小时',
        security: '基础',
        price: '免费或低价',
        features: [
            '仅验证域名所有权',
            '快速签发',
            '适合个人网站'
        ],
        useCases: [
            '个人博客',
            '小型网站',
            '测试环境'
        ],
        providers: ['Let\'s Encrypt', 'Cloudflare', 'ZeroSSL']
    },
    
    ov: {
        name: 'OV证书',
        fullName: 'Organization Validation',
        validation: '组织验证',
        validationTime: '1-3天',
        security: '中等',
        price: '中等',
        features: [
            '验证域名和组织身份',
            '证书中显示组织信息',
            '适合企业网站'
        ],
        useCases: [
            '企业官网',
            '电商平台',
            '在线服务'
        ],
        providers: ['DigiCert', 'GlobalSign', 'Sectigo']
    },
    
    ev: {
        name: 'EV证书',
        fullName: 'Extended Validation',
        validation: '扩展验证',
        validationTime: '1-2周',
        security: '最高',
        price: '高',
        features: [
            '最严格的身份验证',
            '浏览器地址栏显示组织名称',
            '最高信任度'
        ],
        useCases: [
            '金融机构',
            '大型电商',
            '政府网站'
        ],
        providers: ['DigiCert', 'GlobalSign', 'Entrust']
    },
    
    wildcard: {
        name: '通配符证书',
        fullName: 'Wildcard Certificate',
        validation: 'DV或OV',
        validationTime: '取决于验证类型',
        security: '取决于验证类型',
        price: '中等到高',
        features: [
            '保护主域名和所有子域名',
            '*.example.com',
            '节省成本'
        ],
        useCases: [
            '多子域名网站',
            'SaaS平台',
            'API服务'
        ],
        example: '*.example.com 可保护 api.example.com, www.example.com 等'
    },
    
    san: {
        name: 'SAN证书',
        fullName: 'Subject Alternative Name',
        validation: 'DV、OV或EV',
        validationTime: '取决于验证类型',
        security: '取决于验证类型',
        price: '中等到高',
        features: [
            '保护多个不同域名',
            '一个证书多个域名',
            '灵活管理'
        ],
        useCases: [
            '多品牌网站',
            '多域名服务',
            '统一管理'
        ],
        example: '可保护 example.com, example.net, example.org'
    }
};

// 打印证书类型
function printCertificateTypes() {
    console.log('=== SSL证书类型 ===\n');
    
    for (const [key, cert] of Object.entries(certificateTypes)) {
        console.log(`${cert.name} (${cert.fullName})`);
        console.log(`  验证方式: ${cert.validation}`);
        console.log(`  签发时间: ${cert.validationTime}`);
        console.log(`  安全级别: ${cert.security}`);
        console.log(`  价格: ${cert.price}`);
        console.log(`  特点: ${cert.features.join(', ')}`);
        console.log(`  适用场景: ${cert.useCases.join(', ')}`);
        
        if (cert.providers) {
            console.log(`  提供商: ${cert.providers.join(', ')}`);
        }
        
        if (cert.example) {
            console.log(`  示例: ${cert.example}`);
        }
        
        console.log('');
    }
}

printCertificateTypes();
```

#### 证书验证
```javascript
/**
 * SSL证书验证
 * @author erik.zhou
 */

class CertificateValidator {
    constructor() {
        this.trustedCAs = new Set([
            'DigiCert',
            'Let\'s Encrypt',
            'GlobalSign',
            'Sectigo'
        ]);
    }
    
    // 验证证书
    async validate(certificate) {
        console.log('=== 证书验证流程 ===\n');
        
        const validations = [
            this.checkExpiry(certificate),
            this.checkDomain(certificate),
            this.checkChain(certificate),
            this.checkRevocation(certificate),
            this.checkSignature(certificate)
        ];
        
        const results = await Promise.all(validations);
        
        const isValid = results.every(result => result.valid);
        
        console.log('\n=== 验证结果 ===');
        console.log(`证书有效: ${isValid ? '是' : '否'}`);
        
        return {
            valid: isValid,
            results: results
        };
    }
    
    // 检查有效期
    async checkExpiry(certificate) {
        console.log('1. 检查证书有效期');
        
        const now = new Date();
        const notBefore = new Date(certificate.notBefore);
        const notAfter = new Date(certificate.notAfter);
        
        const valid = now >= notBefore && now <= notAfter;
        
        console.log(`   有效期: ${certificate.notBefore} 至 ${certificate.notAfter}`);
        console.log(`   状态: ${valid ? '有效' : '已过期'}`);
        
        return {
            check: 'expiry',
            valid: valid,
            message: valid ? '证书在有效期内' : '证书已过期'
        };
    }
    
    // 检查域名匹配
    async checkDomain(certificate) {
        console.log('\n2. 检查域名匹配');
        
        const requestDomain = certificate.requestDomain;
        const certDomain = certificate.commonName;
        const altNames = certificate.subjectAltNames || [];
        
        let valid = false;
        
        // 检查CN
        if (this.matchDomain(requestDomain, certDomain)) {
            valid = true;
            console.log(`   CN匹配: ${certDomain}`);
        }
        
        // 检查SAN
        for (const altName of altNames) {
            if (this.matchDomain(requestDomain, altName)) {
                valid = true;
                console.log(`   SAN匹配: ${altName}`);
                break;
            }
        }
        
        console.log(`   状态: ${valid ? '匹配' : '不匹配'}`);
        
        return {
            check: 'domain',
            valid: valid,
            message: valid ? '域名匹配' : '域名不匹配'
        };
    }
    
    // 检查证书链
    async checkChain(certificate) {
        console.log('\n3. 检查证书链');
        
        const chain = certificate.chain || [];
        let valid = true;
        
        console.log(`   证书链长度: ${chain.length}`);
        
        for (let i = 0; i < chain.length; i++) {
            const cert = chain[i];
            console.log(`   [${i}] ${cert.subject}`);
            
            // 检查是否是受信任的CA
            if (i === chain.length - 1) {
                if (!this.trustedCAs.has(cert.issuer)) {
                    valid = false;
                    console.log(`       警告: 根证书不受信任`);
                }
            }
        }
        
        console.log(`   状态: ${valid ? '有效' : '无效'}`);
        
        return {
            check: 'chain',
            valid: valid,
            message: valid ? '证书链有效' : '证书链无效'
        };
    }
    
    // 检查吊销状态
    async checkRevocation(certificate) {
        console.log('\n4. 检查证书吊销状态');
        
        // 模拟OCSP查询
        console.log('   查询OCSP服务器...');
        await this.delay(100);
        
        const revoked = false; // 模拟结果
        
        console.log(`   状态: ${revoked ? '已吊销' : '未吊销'}`);
        
        return {
            check: 'revocation',
            valid: !revoked,
            message: revoked ? '证书已被吊销' : '证书未被吊销'
        };
    }
    
    // 检查签名
    async checkSignature(certificate) {
        console.log('\n5. 检查证书签名');
        
        console.log(`   签名算法: ${certificate.signatureAlgorithm}`);
        console.log('   验证签名...');
        await this.delay(50);
        
        const valid = true; // 模拟验证结果
        
        console.log(`   状态: ${valid ? '有效' : '无效'}`);
        
        return {
            check: 'signature',
            valid: valid,
            message: valid ? '签名有效' : '签名无效'
        };
    }
    
    // 域名匹配（支持通配符）
    matchDomain(requestDomain, certDomain) {
        if (certDomain === requestDomain) {
            return true;
        }
        
        // 通配符匹配
        if (certDomain.startsWith('*.')) {
            const pattern = certDomain.substring(2);
            const parts = requestDomain.split('.');
            if (parts.length > 1) {
                const domain = parts.slice(1).join('.');
                return domain === pattern;
            }
        }
        
        return false;
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
const validator = new CertificateValidator();

const certificate = {
    requestDomain: 'www.example.com',
    commonName: '*.example.com',
    subjectAltNames: ['example.com', '*.example.com'],
    notBefore: '2024-01-01T00:00:00Z',
    notAfter: '2025-01-01T00:00:00Z',
    signatureAlgorithm: 'SHA256-RSA',
    chain: [
        { subject: '*.example.com', issuer: 'Let\'s Encrypt' },
        { subject: 'Let\'s Encrypt', issuer: 'Let\'s Encrypt' }
    ]
};

validator.validate(certificate);
```

### 2.2 证书申请与配置

#### Let's Encrypt免费证书
```javascript
/**
 * Let's Encrypt证书申请流程
 * @author erik.zhou
 */

class LetsEncryptClient {
    constructor(domain, email) {
        this.domain = domain;
        this.email = email;
        this.challenges = [];
    }
    
    // 申请证书
    async requestCertificate() {
        console.log('=== Let\'s Encrypt证书申请 ===\n');
        console.log(`域名: ${this.domain}`);
        console.log(`邮箱: ${this.email}\n`);
        
        try {
            // 1. 创建账户
            await this.createAccount();
            
            // 2. 创建订单
            await this.createOrder();
            
            // 3. 完成验证
            await this.completeChallenge();
            
            // 4. 获取证书
            const certificate = await this.downloadCertificate();
            
            console.log('\n证书申请成功！');
            return certificate;
            
        } catch (error) {
            console.error('证书申请失败:', error.message);
            throw error;
        }
    }
    
    // 创建账户
    async createAccount() {
        console.log('步骤1: 创建Let\'s Encrypt账户');
        console.log(`  邮箱: ${this.email}`);
        
        await this.delay(100);
        
        console.log('  账户创建成功\n');
    }
    
    // 创建订单
    async createOrder() {
        console.log('步骤2: 创建证书订单');
        console.log(`  域名: ${this.domain}`);
        
        await this.delay(100);
        
        // 生成验证挑战
        this.challenges = [
            {
                type: 'http-01',
                token: 'random-token-' + Math.random().toString(36),
                url: `http://${this.domain}/.well-known/acme-challenge/token`
            },
            {
                type: 'dns-01',
                token: 'dns-token-' + Math.random().toString(36),
                record: `_acme-challenge.${this.domain}`
            }
        ];
        
        console.log('  订单创建成功');
        console.log('  验证方式:');
        console.log('    - HTTP-01: 文件验证');
        console.log('    - DNS-01: DNS记录验证\n');
    }
    
    // 完成验证
    async completeChallenge() {
        console.log('步骤3: 完成域名验证');
        
        // 选择HTTP-01验证
        const challenge = this.challenges[0];
        
        console.log(`  验证类型: ${challenge.type}`);
        console.log(`  验证URL: ${challenge.url}`);
        console.log(`  Token: ${challenge.token}`);
        
        console.log('\n  请在服务器上创建验证文件:');
        console.log(`  路径: .well-known/acme-challenge/${challenge.token}`);
        console.log(`  内容: ${challenge.token}.account-thumbprint`);
        
        console.log('\n  等待验证...');
        await this.delay(2000);
        
        console.log('  验证成功\n');
    }
    
    // 下载证书
    async downloadCertificate() {
        console.log('步骤4: 下载证书');
        
        await this.delay(500);
        
        const certificate = {
            domain: this.domain,
            cert: '-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----',
            key: '-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----',
            chain: '-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----',
            fullchain: 'cert + chain',
            expiresAt: new Date(Date.now() + 90 * 24 * 60 * 60 * 1000) // 90天后
        };
        
        console.log('  证书下载成功');
        console.log(`  有效期: 90天`);
        console.log(`  过期时间: ${certificate.expiresAt.toISOString()}`);
        
        return certificate;
    }
    
    // 自动续期
    async autoRenew(certificate) {
        console.log('\n=== 证书自动续期 ===');
        
        const now = new Date();
        const expiresAt = new Date(certificate.expiresAt);
        const daysLeft = Math.floor((expiresAt - now) / (24 * 60 * 60 * 1000));
        
        console.log(`剩余天数: ${daysLeft}天`);
        
        if (daysLeft <= 30) {
            console.log('证书即将过期，开始续期...');
            return await this.requestCertificate();
        } else {
            console.log('证书仍在有效期内，无需续期');
            return certificate;
        }
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
async function applyLetsEncrypt() {
    const client = new LetsEncryptClient('example.com', 'admin@example.com');
    
    // 申请证书
    const certificate = await client.requestCertificate();
    
    // 模拟30天后自动续期
    console.log('\n--- 30天后 ---');
    certificate.expiresAt = new Date(Date.now() + 25 * 24 * 60 * 60 * 1000);
    await client.autoRenew(certificate);
}

applyLetsEncrypt();
```

#### Nginx HTTPS配置
```javascript
/**
 * Nginx HTTPS配置生成器
 * @author erik.zhou
 */

class NginxHTTPSConfig {
    constructor(domain, certPath, keyPath) {
        this.domain = domain;
        this.certPath = certPath;
        this.keyPath = keyPath;
    }
    
    // 生成基础配置
    generateBasicConfig() {
        return `
# HTTP重定向到HTTPS
server {
    listen 80;
    server_name ${this.domain};
    return 301 https://$server_name$request_uri;
}

# HTTPS配置
server {
    listen 443 ssl http2;
    server_name ${this.domain};
    
    # SSL证书
    ssl_certificate ${this.certPath};
    ssl_certificate_key ${this.keyPath};
    
    # SSL协议
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # 加密套件
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    # 网站根目录
    root /var/www/html;
    index index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
`.trim();
    }
    
    // 生成优化配置
    generateOptimizedConfig() {
        return `
# HTTP重定向到HTTPS
server {
    listen 80;
    server_name ${this.domain};
    return 301 https://$server_name$request_uri;
}

# HTTPS配置（优化版）
server {
    listen 443 ssl http2;
    server_name ${this.domain};
    
    # SSL证书
    ssl_certificate ${this.certPath};
    ssl_certificate_key ${this.keyPath};
    
    # SSL协议（仅TLS 1.2和1.3）
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # 加密套件（推荐配置）
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    
    # SSL会话缓存
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_session_tickets off;
    
    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate ${this.certPath};
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;
    
    # 安全头部
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    
    # 网站根目录
    root /var/www/html;
    index index.html;
    
    # Gzip压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/json application/javascript application/xml+rss;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
`.trim();
    }
    
    // 生成反向代理配置
    generateProxyConfig(backendUrl) {
        return `
# HTTPS反向代理配置
server {
    listen 443 ssl http2;
    server_name ${this.domain};
    
    # SSL证书
    ssl_certificate ${this.certPath};
    ssl_certificate_key ${this.keyPath};
    
    # SSL优化配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # 安全头部
    add_header Strict-Transport-Security "max-age=31536000" always;
    
    # 反向代理
    location / {
        proxy_pass ${backendUrl};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket支持
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
`.trim();
    }
    
    // 打印所有配置
    printAllConfigs() {
        console.log('=== Nginx HTTPS配置 ===\n');
        
        console.log('1. 基础配置:');
        console.log(this.generateBasicConfig());
        
        console.log('\n\n2. 优化配置:');
        console.log(this.generateOptimizedConfig());
        
        console.log('\n\n3. 反向代理配置:');
        console.log(this.generateProxyConfig('http://localhost:3000'));
    }
}

// 使用示例
const nginxConfig = new NginxHTTPSConfig(
    'example.com',
    '/etc/letsencrypt/live/example.com/fullchain.pem',
    '/etc/letsencrypt/live/example.com/privkey.pem'
);

nginxConfig.printAllConfigs();
```

## 第三部分：加密算法

### 3.1 对称加密

#### AES加密
```javascript
/**
 * AES对称加密示例
 * @author erik.zhou
 */

class AESEncryption {
    constructor() {
        this.algorithm = 'AES-256-GCM';
    }
    
    // 加密数据
    async encrypt(plaintext, key) {
        console.log('=== AES加密 ===');
        console.log(`明文: ${plaintext}`);
        console.log(`密钥长度: ${key.length * 8} bits`);
        
        // 生成随机IV
        const iv = this.generateIV();
        
        // 模拟加密过程
        const ciphertext = this.mockEncrypt(plaintext, key, iv);
        
        console.log(`IV: ${iv}`);
        console.log(`密文: ${ciphertext}`);
        
        return {
            ciphertext: ciphertext,
            iv: iv,
            algorithm: this.algorithm
        };
    }
    
    // 解密数据
    async decrypt(ciphertext, key, iv) {
        console.log('\n=== AES解密 ===');
        console.log(`密文: ${ciphertext}`);
        console.log(`IV: ${iv}`);
        
        // 模拟解密过程
        const plaintext = this.mockDecrypt(ciphertext, key, iv);
        
        console.log(`明文: ${plaintext}`);
        
        return plaintext;
    }
    
    // 生成IV
    generateIV() {
        return Math.random().toString(36).substring(2, 18);
    }
    
    // 模拟加密
    mockEncrypt(plaintext, key, iv) {
        return Buffer.from(plaintext).toString('base64');
    }
    
    // 模拟解密
    mockDecrypt(ciphertext, key, iv) {
        return Buffer.from(ciphertext, 'base64').toString('utf8');
    }
}

// 使用示例
async function demonstrateAES() {
    const aes = new AESEncryption();
    const key = 'my-secret-key-32-bytes-long!!';
    const plaintext = 'Hello, HTTPS!';
    
    // 加密
    const encrypted = await aes.encrypt(plaintext, key);
    
    // 解密
    const decrypted = await aes.decrypt(encrypted.ciphertext, key, encrypted.iv);
    
    console.log(`\n验证: ${plaintext === decrypted ? '成功' : '失败'}`);
}

demonstrateAES();
```

### 3.2 非对称加密

#### RSA加密
```javascript
/**
 * RSA非对称加密示例
 * @author erik.zhou
 */

class RSAEncryption {
    constructor() {
        this.keySize = 2048;
    }
    
    // 生成密钥对
    async generateKeyPair() {
        console.log('=== 生成RSA密钥对 ===');
        console.log(`密钥长度: ${this.keySize} bits`);
        
        // 模拟密钥生成
        await this.delay(100);
        
        const keyPair = {
            publicKey: '-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----',
            privateKey: '-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----'
        };
        
        console.log('密钥对生成成功\n');
        
        return keyPair;
    }
    
    // 公钥加密
    async encryptWithPublicKey(plaintext, publicKey) {
        console.log('=== 公钥加密 ===');
        console.log(`明文: ${plaintext}`);
        
        // 模拟加密
        const ciphertext = Buffer.from(plaintext).toString('base64');
        
        console.log(`密文: ${ciphertext}\n`);
        
        return ciphertext;
    }
    
    // 私钥解密
    async decryptWithPrivateKey(ciphertext, privateKey) {
        console.log('=== 私钥解密 ===');
        console.log(`密文: ${ciphertext}`);
        
        // 模拟解密
        const plaintext = Buffer.from(ciphertext, 'base64').toString('utf8');
        
        console.log(`明文: ${plaintext}\n`);
        
        return plaintext;
    }
    
    // 私钥签名
    async sign(data, privateKey) {
        console.log('=== 私钥签名 ===');
        console.log(`数据: ${data}`);
        
        // 模拟签名
        const signature = 'signature-' + Buffer.from(data).toString('base64');
        
        console.log(`签名: ${signature}\n`);
        
        return signature;
    }
    
    // 公钥验证
    async verify(data, signature, publicKey) {
        console.log('=== 公钥验证签名 ===');
        console.log(`数据: ${data}`);
        console.log(`签名: ${signature}`);
        
        // 模拟验证
        const valid = signature.startsWith('signature-');
        
        console.log(`验证结果: ${valid ? '有效' : '无效'}\n`);
        
        return valid;
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
async function demonstrateRSA() {
    const rsa = new RSAEncryption();
    
    // 生成密钥对
    const keyPair = await rsa.generateKeyPair();
    
    // 加密解密
    const plaintext = 'Secret Message';
    const ciphertext = await rsa.encryptWithPublicKey(plaintext, keyPair.publicKey);
    const decrypted = await rsa.decryptWithPrivateKey(ciphertext, keyPair.privateKey);
    
    // 签名验证
    const data = 'Important Data';
    const signature = await rsa.sign(data, keyPair.privateKey);
    const valid = await rsa.verify(data, signature, keyPair.publicKey);
}

demonstrateRSA();
```

## 第四部分：HTTPS性能优化

### 4.1 会话复用

#### Session Resumption
```javascript
/**
 * TLS会话复用
 * @author erik.zhou
 */

class SessionResumption {
    constructor() {
        this.sessionCache = new Map();
        this.sessionTickets = new Map();
    }
    
    // Session ID复用
    async sessionIDResumption(domain) {
        console.log('=== Session ID复用 ===');
        console.log(`连接到: ${domain}\n`);
        
        // 检查缓存
        if (this.sessionCache.has(domain)) {
            console.log('发现缓存的Session ID');
            const session = this.sessionCache.get(domain);
            
            console.log('1. 客户端 -> ClientHello + Session ID');
            await this.delay(50);
            
            console.log('2. 服务器验证Session ID');
            await this.delay(50);
            
            console.log('3. 服务器 -> ServerHello + 恢复会话');
            await this.delay(50);
            
            console.log('\n会话复用成功，节省1个RTT');
            
            return session;
        }
        
        // 完整握手
        console.log('无缓存，执行完整握手');
        const session = await this.fullHandshake(domain);
        
        // 缓存Session ID
        this.sessionCache.set(domain, session);
        
        return session;
    }
    
    // Session Ticket复用
    async sessionTicketResumption(domain) {
        console.log('\n=== Session Ticket复用 ===');
        console.log(`连接到: ${domain}\n`);
        
        // 检查票据
        if (this.sessionTickets.has(domain)) {
            console.log('发现Session Ticket');
            const ticket = this.sessionTickets.get(domain);
            
            console.log('1. 客户端 -> ClientHello + Session Ticket');
            await this.delay(50);
            
            console.log('2. 服务器解密Ticket');
            await this.delay(50);
            
            console.log('3. 服务器 -> ServerHello + 恢复会话');
            await this.delay(50);
            
            console.log('\n会话复用成功，无需服务器存储');
            
            return ticket;
        }
        
        // 完整握手
        console.log('无票据，执行完整握手');
        const session = await this.fullHandshake(domain);
        
        // 生成Session Ticket
        const ticket = {
            ...session,
            ticket: 'encrypted-ticket-' + Math.random().toString(36)
        };
        
        this.sessionTickets.set(domain, ticket);
        
        return ticket;
    }
    
    // 完整握手
    async fullHandshake(domain) {
        console.log('执行完整TLS握手...');
        await this.delay(200);
        
        return {
            domain: domain,
            sessionId: 'session-' + Math.random().toString(36),
            masterSecret: 'master-secret',
            createdAt: Date.now()
        };
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
async function demonstrateSessionResumption() {
    const session = new SessionResumption();
    const domain = 'example.com';
    
    // 首次连接
    console.log('--- 首次连接 ---');
    await session.sessionIDResumption(domain);
    
    // 第二次连接（Session ID复用）
    console.log('\n--- 第二次连接 ---');
    await session.sessionIDResumption(domain);
    
    // Session Ticket
    console.log('\n--- Session Ticket ---');
    await session.sessionTicketResumption(domain);
    await session.sessionTicketResumption(domain);
}

demonstrateSessionResumption();
```

### 4.2 OCSP Stapling

#### OCSP装订
```javascript
/**
 * OCSP Stapling实现
 * @author erik.zhou
 */

class OCSPStapling {
    constructor() {
        this.ocspCache = new Map();
    }
    
    // 传统OCSP查询
    async traditionalOCSP(certificate) {
        console.log('=== 传统OCSP查询 ===');
        console.log('客户端查询OCSP服务器\n');
        
        const startTime = Date.now();
        
        // 1. 客户端连接OCSP服务器
        console.log('1. 连接OCSP服务器');
        await this.delay(100);
        
        // 2. 查询证书状态
        console.log('2. 查询证书状态');
        await this.delay(100);
        
        // 3. 接收响应
        console.log('3. 接收OCSP响应');
        await this.delay(100);
        
        const duration = Date.now() - startTime;
        console.log(`\n总耗时: ${duration}ms`);
        console.log('缺点: 增加延迟、泄露隐私\n');
        
        return {
            status: 'good',
            duration: duration
        };
    }
    
    // OCSP Stapling
    async ocspStapling(certificate) {
        console.log('=== OCSP Stapling ===');
        console.log('服务器预先查询并装订OCSP响应\n');
        
        const startTime = Date.now();
        
        // 服务器预先查询
        if (!this.ocspCache.has(certificate.serial)) {
            console.log('服务器查询OCSP（后台）');
            await this.delay(100);
            
            const ocspResponse = {
                status: 'good',
                timestamp: Date.now(),
                nextUpdate: Date.now() + 24 * 60 * 60 * 1000
            };
            
            this.ocspCache.set(certificate.serial, ocspResponse);
        }
        
        // TLS握手时直接发送
        console.log('1. TLS握手');
        await this.delay(50);
        
        console.log('2. 服务器发送证书 + OCSP响应');
        await this.delay(50);
        
        console.log('3. 客户端验证OCSP响应');
        await this.delay(10);
        
        const duration = Date.now() - startTime;
        console.log(`\n总耗时: ${duration}ms`);
        console.log('优点: 无额外延迟、保护隐私\n');
        
        return {
            status: 'good',
            duration: duration
        };
    }
    
    // 性能对比
    async compare(certificate) {
        console.log('=== 性能对比 ===\n');
        
        const traditional = await this.traditionalOCSP(certificate);
        const stapling = await this.ocspStapling(certificate);
        
        const improvement = ((traditional.duration - stapling.duration) / traditional.duration * 100).toFixed(0);
        
        console.log('结果对比:');
        console.log(`传统OCSP: ${traditional.duration}ms`);
        console.log(`OCSP Stapling: ${stapling.duration}ms`);
        console.log(`性能提升: ${improvement}%`);
    }
    
    delay(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
}

// 使用示例
const ocsp = new OCSPStapling();
const certificate = { serial: 'ABC123' };

ocsp.compare(certificate);
```

## 总结

### HTTPS核心要点

1. HTTPS基础
   - 加密传输保护数据安全
   - 身份认证防止中间人攻击
   - 数据完整性防止篡改
   - SEO友好，浏览器信任

2. SSL/TLS协议
   - TLS 1.2: 当前主流版本
   - TLS 1.3: 最新标准，性能更优
   - 握手优化: 1-RTT和0-RTT
   - 前向保密保护历史数据

3. 证书管理
   - DV/OV/EV证书类型
   - Let's Encrypt免费证书
   - 证书验证流程
   - 自动续期机制

4. 加密算法
   - 对称加密: AES-256-GCM
   - 非对称加密: RSA-2048/4096
   - 哈希算法: SHA-256
   - 密钥交换: ECDHE

5. 性能优化
   - 会话复用
   - OCSP Stapling
   - HTTP/2支持
   - 证书链优化

### 最佳实践

1. 使用TLS 1.2及以上版本
2. 配置强加密套件
3. 启用HSTS
4. 实施OCSP Stapling
5. 使用HTTP/2或HTTP/3
6. 定期更新证书
7. 监控证书过期时间
8. 配置安全响应头

### 学习资源

- [Mozilla SSL配置生成器](https://ssl-config.mozilla.org/)
- [SSL Labs测试工具](https://www.ssllabs.com/ssltest/)
- [Let's Encrypt文档](https://letsencrypt.org/docs/)
- [TLS 1.3规范](https://tools.ietf.org/html/rfc8446)

---

**@author erik.zhou**
**最后更新时间：** 2026-03-06
