# Cypress - 完整教程

## 目录
1. [Cypress简介](#1-cypress简介)
2. [安装与配置](#2-安装与配置)
3. [基础用法](#3-基础用法)
4. [命令与断言](#4-命令与断言)
5. [网络请求Mock](#5-网络请求mock)
6. [自定义命令](#6-自定义命令)
7. [Fixtures与数据管理](#7-fixtures与数据管理)
8. [调试技巧](#8-调试技巧)
9. [CI/CD集成](#9-cicd集成)
10. [最佳实践](#10-最佳实践)

---

## 1. Cypress简介

### 1.1 什么是Cypress

```javascript
/**
 * Cypress核心特性
 * @author erik.zhou
 */

// Cypress是现代化的前端测试工具
// 核心优势：
// 1. 实时重载：代码改变自动重新运行测试
// 2. 时间旅行：查看每个命令执行时的应用状态
// 3. 自动等待：智能等待元素和请求
// 4. 网络控制：轻松stub和mock网络请求
// 5. 截图和视频：自动捕获失败时的状态
// 6. 调试友好：使用Chrome DevTools调试
// 7. 一致性结果：可靠的测试结果
```

### 1.2 Cypress架构

```javascript
/**
 * Cypress架构特点
 * @author erik.zhou
 */

// Cypress与其他测试工具的区别：
// 1. 运行在浏览器内部，与应用在同一运行循环
// 2. 可以访问所有对象（window、document、DOM等）
// 3. 可以控制网络层
// 4. 可以修改任何内容
// 5. 实时查看测试执行过程
```

---

## 2. 安装与配置

### 2.1 安装Cypress

```bash
# 安装Cypress
npm install --save-dev cypress

# 打开Cypress
npx cypress open

# 运行Cypress（headless模式）
npx cypress run

# 安装特定版本
npm install --save-dev cypress@12.0.0

# 验证安装
npx cypress verify
```

### 2.2 配置文件

```javascript
/**
 * Cypress配置文件
 * @author erik.zhou
 */

// cypress.config.js
const { defineConfig } = require('cypress');

module.exports = defineConfig({
    e2e: {
        // 基础URL
        baseUrl: 'http://localhost:3000',
        
        // 测试文件匹配模式
        specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}',
        
        // 支持文件
        supportFile: 'cypress/support/e2e.js',
        
        // 视口大小
        viewportWidth: 1280,
        viewportHeight: 720,
        
        // 超时设置
        defaultCommandTimeout: 10000,
        requestTimeout: 10000,
        responseTimeout: 30000,
        
        // 视频录制
        video: true,
        videoCompression: 32,
        
        // 截图
        screenshotOnRunFailure: true,
        
        // 重试次数
        retries: {
            runMode: 2,
            openMode: 0
        },
        
        // 环境变量
        env: {
            apiUrl: 'http://localhost:4000/api',
            username: 'testuser',
            password: 'password123'
        },
        
        // 设置钩子
        setupNodeEvents(on, config) {
            // 实现node事件监听器
            on('task', {
                log(message) {
                    console.log(message);
                    return null;
                }
            });
            
            return config;
        }
    },
    
    component: {
        devServer: {
            framework: 'react',
            bundler: 'vite'
        },
        specPattern: 'src/**/*.cy.{js,jsx,ts,tsx}'
    }
});
```

### 2.3 目录结构

```javascript
/**
 * Cypress目录结构
 * @author erik.zhou
 */

/*
cypress/
  e2e/
    auth/
      login.cy.js
      register.cy.js
    dashboard/
      overview.cy.js
  fixtures/
    users.json
    products.json
  support/
    commands.js
    e2e.js
  downloads/
  screenshots/
  videos/
cypress.config.js
*/
```

---

## 3. 基础用法

### 3.1 第一个测试

```javascript
/**
 * 第一个Cypress测试
 * @author erik.zhou
 */

describe('My First Test', () => {
    it('visits the app', () => {
        // 访问页面
        cy.visit('https://example.com');
        
        // 验证URL
        cy.url().should('include', 'example.com');
        
        // 查找元素
        cy.get('h1').should('contain', 'Example Domain');
        
        // 点击链接
        cy.contains('More information').click();
        
        // 验证导航
        cy.url().should('include', '/info');
    });
});
```

### 3.2 选择器

```javascript
/**
 * Cypress选择器示例
 * @author erik.zhou
 */

describe('Selectors', () => {
    beforeEach(() => {
        cy.visit('/');
    });
    
    it('uses different selectors', () => {
        // 1. CSS选择器
        cy.get('.my-class');
        cy.get('#my-id');
        cy.get('[data-testid="submit"]');
        
        // 2. 包含文本
        cy.contains('Submit');
        cy.contains('button', 'Submit');
        
        // 3. 查找子元素
        cy.get('form').find('input');
        cy.get('form').find('[type="email"]');
        
        // 4. 过滤元素
        cy.get('li').filter('.active');
        cy.get('li').not('.disabled');
        
        // 5. 第一个/最后一个
        cy.get('li').first();
        cy.get('li').last();
        cy.get('li').eq(2);
        
        // 6. 父元素/子元素
        cy.get('button').parent();
        cy.get('ul').children();
        
        // 7. 兄弟元素
        cy.get('.active').siblings();
        cy.get('.active').next();
        cy.get('.active').prev();
    });
});
```

### 3.3 用户交互

```javascript
/**
 * 用户交互示例
 * @author erik.zhou
 */

describe('User Interactions', () => {
    beforeEach(() => {
        cy.visit('/form');
    });
    
    it('interacts with form elements', () => {
        // 输入文本
        cy.get('[data-testid="username"]').type('testuser');
        
        // 清空输入
        cy.get('[data-testid="username"]').clear();
        
        // 输入特殊键
        cy.get('[data-testid="search"]').type('test{enter}');
        cy.get('[data-testid="input"]').type('{selectall}{backspace}');
        
        // 点击
        cy.get('button').click();
        cy.get('button').dblclick();
        cy.get('button').rightclick();
        
        // 勾选复选框
        cy.get('[type="checkbox"]').check();
        cy.get('[type="checkbox"]').uncheck();
        
        // 选择单选按钮
        cy.get('[type="radio"]').check('option1');
        
        // 选择下拉框
        cy.get('select').select('option1');
        cy.get('select').select(['option1', 'option2']);
        
        // 上传文件
        cy.get('input[type="file"]').selectFile('path/to/file.pdf');
        
        // 滚动
        cy.get('.element').scrollIntoView();
        cy.scrollTo('bottom');
        cy.scrollTo(0, 500);
        
        // 触发事件
        cy.get('button').trigger('mouseover');
        cy.get('input').trigger('focus');
    });
});
```

---

## 4. 命令与断言

### 4.1 常用命令

```javascript
/**
 * Cypress常用命令
 * @author erik.zhou
 */

describe('Common Commands', () => {
    it('uses various commands', () => {
        // 访问页面
        cy.visit('/');
        cy.visit('/about', { timeout: 10000 });
        
        // 重新加载
        cy.reload();
        cy.reload(true); // 强制刷新
        
        // 前进后退
        cy.go('back');
        cy.go('forward');
        cy.go(-1);
        
        // 等待
        cy.wait(1000);
        cy.wait('@apiRequest');
        
        // 获取元素
        cy.get('.element');
        cy.contains('text');
        
        // 查找
        cy.get('form').find('input');
        cy.get('form').within(() => {
            cy.get('input').type('test');
        });
        
        // 执行JavaScript
        cy.window().then((win) => {
            win.localStorage.setItem('key', 'value');
        });
        
        cy.document().then((doc) => {
            expect(doc.title).to.equal('My App');
        });
    });
});
```

### 4.2 断言

```javascript
/**
 * Cypress断言示例
 * @author erik.zhou
 */

describe('Assertions', () => {
    beforeEach(() => {
        cy.visit('/');
    });
    
    it('uses implicit assertions', () => {
        // should断言
        cy.get('h1').should('contain', 'Welcome');
        cy.get('h1').should('have.text', 'Welcome');
        cy.get('h1').should('be.visible');
        cy.get('h1').should('not.exist');
        
        // 链式断言
        cy.get('button')
            .should('be.visible')
            .and('be.enabled')
            .and('have.class', 'btn-primary');
        
        // 长度断言
        cy.get('li').should('have.length', 5);
        cy.get('li').should('have.length.greaterThan', 3);
        
        // 属性断言
        cy.get('a').should('have.attr', 'href', '/about');
        cy.get('input').should('have.value', 'test');
        
        // CSS断言
        cy.get('button').should('have.css', 'background-color', 'rgb(0, 123, 255)');
        
        // 类断言
        cy.get('.element').should('have.class', 'active');
        
        // 文本断言
        cy.get('p').should('contain', 'Hello');
        cy.get('p').should('match', /Hello \w+/);
    });
    
    it('uses explicit assertions', () => {
        // expect断言
        cy.get('h1').then(($h1) => {
            expect($h1).to.have.text('Welcome');
            expect($h1).to.be.visible;
        });
        
        // assert断言
        cy.get('input').then(($input) => {
            assert.equal($input.val(), 'test', 'input value is test');
            assert.isTrue($input.is(':visible'), 'input is visible');
        });
    });
});
```

### 4.3 别名

```javascript
/**
 * 别名使用示例
 * @author erik.zhou
 */

describe('Aliases', () => {
    beforeEach(() => {
        cy.visit('/');
    });
    
    it('uses aliases for elements', () => {
        // 为元素创建别名
        cy.get('button').as('submitButton');
        
        // 使用别名
        cy.get('@submitButton').click();
        cy.get('@submitButton').should('be.disabled');
    });
    
    it('uses aliases for routes', () => {
        // 为请求创建别名
        cy.intercept('GET', '/api/users').as('getUsers');
        
        cy.visit('/users');
        
        // 等待请求完成
        cy.wait('@getUsers').then((interception) => {
            expect(interception.response.statusCode).to.equal(200);
        });
    });
    
    it('uses aliases for fixtures', () => {
        // 为fixture创建别名
        cy.fixture('users.json').as('usersData');
        
        cy.get('@usersData').then((users) => {
            expect(users).to.have.length(10);
        });
    });
});
```

---

## 5. 网络请求Mock

### 5.1 拦截请求

```javascript
/**
 * 网络请求拦截
 * @author erik.zhou
 */

describe('Network Interception', () => {
    it('intercepts GET request', () => {
        // 拦截并mock响应
        cy.intercept('GET', '/api/users', {
            statusCode: 200,
            body: [
                { id: 1, name: 'User 1' },
                { id: 2, name: 'User 2' }
            ]
        }).as('getUsers');
        
        cy.visit('/users');
        
        cy.wait('@getUsers');
        cy.contains('User 1').should('be.visible');
    });
    
    it('intercepts POST request', () => {
        cy.intercept('POST', '/api/users', {
            statusCode: 201,
            body: { id: 3, name: 'New User' }
        }).as('createUser');
        
        cy.visit('/users/new');
        cy.get('[data-testid="name"]').type('New User');
        cy.get('[data-testid="submit"]').click();
        
        cy.wait('@createUser').its('request.body').should('deep.equal', {
            name: 'New User'
        });
    });
    
    it('modifies response', () => {
        cy.intercept('GET', '/api/users', (req) => {
            req.reply((res) => {
                // 修改响应
                res.body.push({ id: 999, name: 'Added User' });
                res.send();
            });
        });
        
        cy.visit('/users');
        cy.contains('Added User').should('be.visible');
    });
});
```

### 5.2 使用Fixtures

```javascript
/**
 * 使用Fixtures进行Mock
 * @author erik.zhou
 */

// cypress/fixtures/users.json
/*
[
    { "id": 1, "name": "John Doe", "email": "john@example.com" },
    { "id": 2, "name": "Jane Smith", "email": "jane@example.com" }
]
*/

describe('Using Fixtures', () => {
    it('uses fixture for mock data', () => {
        cy.intercept('GET', '/api/users', {
            fixture: 'users.json'
        }).as('getUsers');
        
        cy.visit('/users');
        
        cy.wait('@getUsers');
        cy.contains('John Doe').should('be.visible');
    });
    
    it('loads fixture programmatically', () => {
        cy.fixture('users.json').then((users) => {
            cy.intercept('GET', '/api/users', {
                statusCode: 200,
                body: users
            });
        });
        
        cy.visit('/users');
    });
});
```

### 5.3 网络错误模拟

```javascript
/**
 * 网络错误模拟
 * @author erik.zhou
 */

describe('Network Errors', () => {
    it('simulates 404 error', () => {
        cy.intercept('GET', '/api/users/999', {
            statusCode: 404,
            body: { error: 'User not found' }
        });
        
        cy.visit('/users/999');
        cy.contains('User not found').should('be.visible');
    });
    
    it('simulates 500 error', () => {
        cy.intercept('POST', '/api/users', {
            statusCode: 500,
            body: { error: 'Internal server error' }
        });
        
        cy.visit('/users/new');
        cy.get('[data-testid="submit"]').click();
        cy.contains('Internal server error').should('be.visible');
    });
    
    it('simulates network failure', () => {
        cy.intercept('GET', '/api/users', {
            forceNetworkError: true
        });
        
        cy.visit('/users');
        cy.contains('Network error').should('be.visible');
    });
    
    it('simulates slow response', () => {
        cy.intercept('GET', '/api/users', (req) => {
            req.reply({
                delay: 3000,
                body: []
            });
        });
        
        cy.visit('/users');
        cy.contains('Loading').should('be.visible');
    });
});
```


---

## 6. 自定义命令

### 6.1 创建自定义命令

```javascript
/**
 * 自定义命令
 * @author erik.zhou
 */

// cypress/support/commands.js

// 简单的自定义命令
Cypress.Commands.add('login', (username, password) => {
    cy.visit('/login');
    cy.get('[data-testid="username"]').type(username);
    cy.get('[data-testid="password"]').type(password);
    cy.get('[data-testid="submit"]').click();
});

// 使用自定义命令
describe('Custom Commands', () => {
    it('uses login command', () => {
        cy.login('testuser', 'password123');
        cy.url().should('include', '/dashboard');
    });
});

// 带选项的自定义命令
Cypress.Commands.add('loginByAPI', (username, password) => {
    cy.request({
        method: 'POST',
        url: '/api/login',
        body: { username, password }
    }).then((response) => {
        window.localStorage.setItem('token', response.body.token);
    });
});

// 覆盖现有命令
Cypress.Commands.overwrite('visit', (originalFn, url, options) => {
    const token = window.localStorage.getItem('token');
    
    return originalFn(url, {
        ...options,
        headers: {
            ...options?.headers,
            Authorization: token ? `Bearer ${token}` : ''
        }
    });
});
```

### 6.2 子命令

```javascript
/**
 * 子命令示例
 * @author erik.zhou
 */

// 创建子命令
Cypress.Commands.add('selectByText', { prevSubject: 'element' }, (subject, text) => {
    cy.wrap(subject).find('option').contains(text).then(($option) => {
        cy.wrap(subject).select($option.val());
    });
});

// 使用子命令
describe('Child Commands', () => {
    it('uses child command', () => {
        cy.get('select').selectByText('Option 1');
    });
});

// 双重命令（可以作为父命令或子命令）
Cypress.Commands.add('console', { prevSubject: 'optional' }, (subject, method = 'log') => {
    if (subject) {
        console[method]('Subject:', subject);
        return subject;
    }
    
    console[method]('No subject');
});
```

### 6.3 实用自定义命令

```javascript
/**
 * 实用自定义命令集合
 * @author erik.zhou
 */

// 等待加载完成
Cypress.Commands.add('waitForLoading', () => {
    cy.get('[data-testid="loading"]').should('not.exist');
});

// 清空数据库
Cypress.Commands.add('resetDatabase', () => {
    cy.task('db:reset');
});

// 创建测试用户
Cypress.Commands.add('createUser', (userData) => {
    cy.request('POST', '/api/users', userData);
});

// 设置本地存储
Cypress.Commands.add('setLocalStorage', (key, value) => {
    cy.window().then((win) => {
        win.localStorage.setItem(key, JSON.stringify(value));
    });
});

// 获取本地存储
Cypress.Commands.add('getLocalStorage', (key) => {
    cy.window().then((win) => {
        return JSON.parse(win.localStorage.getItem(key));
    });
});

// 拖拽元素
Cypress.Commands.add('drag', { prevSubject: 'element' }, (subject, targetSelector) => {
    cy.wrap(subject).trigger('mousedown', { which: 1 });
    cy.get(targetSelector).trigger('mousemove').trigger('mouseup', { force: true });
});
```

---

## 7. Fixtures与数据管理

### 7.1 使用Fixtures

```javascript
/**
 * Fixtures数据管理
 * @author erik.zhou
 */

// cypress/fixtures/users.json
/*
{
    "admin": {
        "username": "admin",
        "password": "admin123",
        "role": "admin"
    },
    "user": {
        "username": "testuser",
        "password": "password123",
        "role": "user"
    }
}
*/

describe('Using Fixtures', () => {
    it('loads fixture data', () => {
        cy.fixture('users.json').then((users) => {
            cy.login(users.admin.username, users.admin.password);
        });
    });
    
    it('uses fixture with alias', () => {
        cy.fixture('users.json').as('users');
        
        cy.get('@users').then((users) => {
            cy.login(users.user.username, users.user.password);
        });
    });
    
    it('loads fixture in beforeEach', () => {
        beforeEach(function() {
            cy.fixture('users.json').then((users) => {
                this.users = users;
            });
        });
        
        it('uses fixture from context', function() {
            cy.login(this.users.admin.username, this.users.admin.password);
        });
    });
});
```

### 7.2 动态数据生成

```javascript
/**
 * 动态数据生成
 * @author erik.zhou
 */

describe('Dynamic Data', () => {
    it('generates random data', () => {
        const randomEmail = `user${Date.now()}@example.com`;
        const randomUsername = `user${Math.random().toString(36).substring(7)}`;
        
        cy.visit('/register');
        cy.get('[data-testid="username"]').type(randomUsername);
        cy.get('[data-testid="email"]').type(randomEmail);
        cy.get('[data-testid="password"]').type('password123');
        cy.get('[data-testid="submit"]').click();
    });
    
    it('uses faker for data generation', () => {
        // 需要安装 @faker-js/faker
        const faker = require('@faker-js/faker').faker;
        
        const user = {
            name: faker.person.fullName(),
            email: faker.internet.email(),
            phone: faker.phone.number()
        };
        
        cy.visit('/register');
        cy.get('[data-testid="name"]').type(user.name);
        cy.get('[data-testid="email"]').type(user.email);
        cy.get('[data-testid="phone"]').type(user.phone);
    });
});
```

### 7.3 环境变量

```javascript
/**
 * 环境变量使用
 * @author erik.zhou
 */

// cypress.config.js
module.exports = defineConfig({
    e2e: {
        env: {
            apiUrl: 'http://localhost:4000/api',
            username: 'testuser',
            password: 'password123'
        }
    }
});

describe('Environment Variables', () => {
    it('uses environment variables', () => {
        const apiUrl = Cypress.env('apiUrl');
        const username = Cypress.env('username');
        const password = Cypress.env('password');
        
        cy.request(`${apiUrl}/login`, {
            username,
            password
        });
    });
    
    it('sets environment variable', () => {
        Cypress.env('token', 'abc123');
        
        const token = Cypress.env('token');
        expect(token).to.equal('abc123');
    });
});

// 从命令行传递环境变量
// npx cypress run --env apiUrl=http://api.example.com
```

---

## 8. 调试技巧

### 8.1 调试命令

```javascript
/**
 * Cypress调试技巧
 * @author erik.zhou
 */

describe('Debugging', () => {
    it('uses debug commands', () => {
        cy.visit('/');
        
        // 暂停执行
        cy.pause();
        
        // 打印到控制台
        cy.get('h1').debug();
        
        // 使用then查看值
        cy.get('h1').then(($h1) => {
            debugger; // 触发浏览器调试器
            console.log($h1.text());
        });
        
        // 打印日志
        cy.log('Custom log message');
        
        // 截图
        cy.screenshot('my-screenshot');
        
        // 获取当前状态
        cy.get('button').then(($btn) => {
            console.log('Button text:', $btn.text());
            console.log('Button classes:', $btn.attr('class'));
            console.log('Is visible:', $btn.is(':visible'));
        });
    });
});
```

### 8.2 时间旅行

```javascript
/**
 * 时间旅行调试
 * @author erik.zhou
 */

describe('Time Travel', () => {
    it('demonstrates time travel', () => {
        cy.visit('/');
        
        // 每个命令都会在命令日志中显示
        cy.get('input').type('test');
        cy.get('button').click();
        cy.url().should('include', '/results');
        
        // 点击命令日志中的任何命令，可以看到该时刻的DOM快照
        // 可以在浏览器DevTools中检查元素状态
    });
});
```

### 8.3 错误处理

```javascript
/**
 * 错误处理
 * @author erik.zhou
 */

describe('Error Handling', () => {
    it('handles expected errors', () => {
        // 捕获异常
        cy.on('uncaught:exception', (err, runnable) => {
            // 返回false阻止测试失败
            if (err.message.includes('Expected error')) {
                return false;
            }
        });
        
        cy.visit('/page-with-error');
    });
    
    it('uses fail handler', () => {
        cy.on('fail', (error, runnable) => {
            // 自定义失败处理
            console.log('Test failed:', error.message);
            throw error;
        });
        
        cy.get('.non-existent').should('exist');
    });
    
    it('retries on failure', () => {
        // 配置重试
        Cypress.currentTest.retries(2);
        
        cy.visit('/');
        cy.get('.element').should('be.visible');
    });
});
```


---

## 9. CI/CD集成

### 9.1 GitHub Actions配置

```yaml
# .github/workflows/cypress.yml
# @author erik.zhou

name: Cypress Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  cypress-run:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Cypress run
        uses: cypress-io/github-action@v5
        with:
          build: npm run build
          start: npm start
          wait-on: 'http://localhost:3000'
          wait-on-timeout: 120
          browser: chrome
          record: true
        env:
          CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
      
      - name: Upload screenshots
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: cypress-screenshots
          path: cypress/screenshots
      
      - name: Upload videos
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: cypress-videos
          path: cypress/videos
```

### 9.2 并行执行

```yaml
# 并行执行配置
# @author erik.zhou

name: Cypress Parallel Tests

on: [push]

jobs:
  cypress-run:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        containers: [1, 2, 3, 4]
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Cypress run
        uses: cypress-io/github-action@v5
        with:
          start: npm start
          wait-on: 'http://localhost:3000'
          record: true
          parallel: true
          group: 'UI Tests'
        env:
          CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
```

### 9.3 Docker配置

```dockerfile
# Dockerfile
# @author erik.zhou

FROM cypress/included:12.0.0

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

CMD ["npx", "cypress", "run"]
```

```yaml
# docker-compose.yml
# @author erik.zhou

version: '3.8'

services:
  cypress:
    build: .
    volumes:
      - ./cypress/videos:/app/cypress/videos
      - ./cypress/screenshots:/app/cypress/screenshots
    environment:
      - CYPRESS_baseUrl=http://web:3000
    depends_on:
      - web
  
  web:
    image: nginx:alpine
    ports:
      - "3000:80"
    volumes:
      - ./dist:/usr/share/nginx/html
```

---

## 10. 最佳实践

### 10.1 测试组织

```javascript
/**
 * 测试组织最佳实践
 * @author erik.zhou
 */

// ✅ 好的实践 - 按功能模块组织
/*
cypress/
  e2e/
    auth/
      login.cy.js
      register.cy.js
    dashboard/
      overview.cy.js
      settings.cy.js
    common/
      navigation.cy.js
*/

// ✅ 使用describe和context分组
describe('User Management', () => {
    context('When logged in as admin', () => {
        beforeEach(() => {
            cy.loginAsAdmin();
        });
        
        it('can create users', () => {
            // 测试逻辑
        });
        
        it('can delete users', () => {
            // 测试逻辑
        });
    });
    
    context('When logged in as regular user', () => {
        beforeEach(() => {
            cy.loginAsUser();
        });
        
        it('cannot create users', () => {
            // 测试逻辑
        });
    });
});
```

### 10.2 选择器最佳实践

```javascript
/**
 * 选择器最佳实践
 * @author erik.zhou
 */

describe('Selector Best Practices', () => {
    it('uses data attributes', () => {
        // ✅ 好的实践 - 使用data-testid
        cy.get('[data-testid="submit-button"]').click();
        
        // ❌ 不好的实践 - 使用CSS类
        cy.get('.btn-primary').click();
        
        // ❌ 不好的实践 - 使用文本内容
        cy.contains('Submit').click();
    });
    
    it('uses semantic selectors', () => {
        // ✅ 使用语义化选择器
        cy.get('form').within(() => {
            cy.get('input[name="username"]').type('testuser');
            cy.get('input[type="password"]').type('password123');
            cy.get('button[type="submit"]').click();
        });
    });
});
```

### 10.3 等待策略

```javascript
/**
 * 等待策略最佳实践
 * @author erik.zhou
 */

describe('Waiting Strategies', () => {
    it('uses automatic waiting', () => {
        // ✅ Cypress自动等待元素出现
        cy.get('[data-testid="button"]').click();
        
        // ✅ 等待请求完成
        cy.intercept('GET', '/api/users').as('getUsers');
        cy.visit('/users');
        cy.wait('@getUsers');
        
        // ✅ 等待元素状态
        cy.get('[data-testid="loading"]').should('not.exist');
        cy.get('[data-testid="content"]').should('be.visible');
    });
    
    it('avoids hard waits', () => {
        // ❌ 不好的实践 - 使用固定等待
        // cy.wait(5000);
        
        // ✅ 好的实践 - 等待特定条件
        cy.get('[data-testid="loading"]').should('not.exist');
    });
});
```

### 10.4 测试独立性

```javascript
/**
 * 测试独立性最佳实践
 * @author erik.zhou
 */

describe('Test Independence', () => {
    // ✅ 每个测试前重置状态
    beforeEach(() => {
        cy.resetDatabase();
        cy.visit('/');
    });
    
    // ✅ 测试之间互不依赖
    it('test 1', () => {
        cy.createUser({ name: 'User 1' });
        cy.contains('User 1').should('be.visible');
    });
    
    it('test 2', () => {
        cy.createUser({ name: 'User 2' });
        cy.contains('User 2').should('be.visible');
    });
    
    // ❌ 不好的实践 - 测试间有依赖
    // it('creates user', () => {
    //     cy.createUser({ name: 'User 1' });
    // });
    // 
    // it('edits user', () => {
    //     // 依赖上一个测试创建的用户
    //     cy.contains('User 1').click();
    // });
});
```

### 10.5 页面对象模式

```javascript
/**
 * 页面对象模式
 * @author erik.zhou
 */

// cypress/support/pages/LoginPage.js
export class LoginPage {
    visit() {
        cy.visit('/login');
    }
    
    fillUsername(username) {
        cy.get('[data-testid="username"]').type(username);
        return this;
    }
    
    fillPassword(password) {
        cy.get('[data-testid="password"]').type(password);
        return this;
    }
    
    submit() {
        cy.get('[data-testid="submit"]').click();
        return this;
    }
    
    login(username, password) {
        this.fillUsername(username)
            .fillPassword(password)
            .submit();
        return this;
    }
    
    getErrorMessage() {
        return cy.get('[data-testid="error"]');
    }
}

// 使用页面对象
import { LoginPage } from '../support/pages/LoginPage';

describe('Login Tests', () => {
    const loginPage = new LoginPage();
    
    it('logs in successfully', () => {
        loginPage.visit();
        loginPage.login('testuser', 'password123');
        
        cy.url().should('include', '/dashboard');
    });
    
    it('shows error for invalid credentials', () => {
        loginPage.visit();
        loginPage.login('invalid', 'wrong');
        
        loginPage.getErrorMessage()
            .should('be.visible')
            .and('contain', 'Invalid credentials');
    });
});
```

### 10.6 性能优化

```javascript
/**
 * 性能优化最佳实践
 * @author erik.zhou
 */

describe('Performance Optimization', () => {
    // ✅ 使用cy.session缓存登录状态
    beforeEach(() => {
        cy.session('user-session', () => {
            cy.visit('/login');
            cy.get('[data-testid="username"]').type('testuser');
            cy.get('[data-testid="password"]').type('password123');
            cy.get('[data-testid="submit"]').click();
            cy.url().should('include', '/dashboard');
        });
    });
    
    // ✅ 阻止不必要的资源加载
    beforeEach(() => {
        cy.intercept('**/*.{png,jpg,jpeg,gif}', { statusCode: 200, body: '' });
    });
    
    // ✅ 使用API而非UI进行数据准备
    it('prepares data via API', () => {
        cy.request('POST', '/api/users', {
            name: 'Test User',
            email: 'test@example.com'
        });
        
        cy.visit('/users');
        cy.contains('Test User').should('be.visible');
    });
});
```

### 10.7 错误处理

```javascript
/**
 * 错误处理最佳实践
 * @author erik.zhou
 */

describe('Error Handling', () => {
    // ✅ 全局错误处理
    Cypress.on('uncaught:exception', (err, runnable) => {
        // 忽略特定错误
        if (err.message.includes('ResizeObserver')) {
            return false;
        }
        return true;
    });
    
    it('handles API errors gracefully', () => {
        cy.intercept('GET', '/api/users', {
            statusCode: 500,
            body: { error: 'Server error' }
        });
        
        cy.visit('/users');
        cy.contains('Server error').should('be.visible');
    });
    
    it('retries flaky tests', () => {
        // 配置重试次数
        Cypress.currentTest.retries({
            runMode: 2,
            openMode: 0
        });
        
        cy.visit('/');
        cy.get('[data-testid="dynamic-content"]').should('be.visible');
    });
});
```

---

## 总结

### 核心要点

1. **实时重载**
   - 代码改变自动重新运行
   - 快速反馈循环
   - 提高开发效率

2. **时间旅行**
   - 查看每个命令执行时的状态
   - 强大的调试能力
   - 直观的测试过程

3. **自动等待**
   - 智能等待元素和请求
   - 减少flaky测试
   - 无需手动等待

4. **网络控制**
   - 轻松mock API响应
   - 模拟各种网络状况
   - 测试边界情况

5. **开发体验**
   - 友好的错误提示
   - 丰富的文档
   - 活跃的社区

### 学习资源

- [Cypress官方文档](https://docs.cypress.io/)
- [Cypress GitHub](https://github.com/cypress-io/cypress)
- [Cypress示例](https://github.com/cypress-io/cypress-example-recipes)
- [Cypress最佳实践](https://docs.cypress.io/guides/references/best-practices)

### 下一步

- 探索Cypress组件测试
- 学习视觉回归测试
- 掌握性能测试
- 了解可访问性测试

---

**最后更新时间：** 2026-03-05  
**@author erik.zhou**
