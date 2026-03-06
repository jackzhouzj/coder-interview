# Playwright - 完整教程

## 目录
1. [Playwright简介](#1-playwright简介)
2. [安装与配置](#2-安装与配置)
3. [基础用法](#3-基础用法)
4. [页面对象模型](#4-页面对象模型)
5. [API测试](#5-api测试)
6. [视觉回归测试](#6-视觉回归测试)
7. [移动端测试](#7-移动端测试)
8. [CI/CD集成](#8-cicd集成)
9. [高级特性](#9-高级特性)
10. [最佳实践](#10-最佳实践)

---

## 1. Playwright简介

### 1.1 什么是Playwright

```typescript
/**
 * Playwright核心特性
 * @author erik.zhou
 */

// Playwright是微软开发的现代化端到端测试框架
// 核心优势：
// 1. 跨浏览器支持：Chromium、Firefox、WebKit
// 2. 自动等待：智能等待元素可交互
// 3. 网络拦截：Mock API响应
// 4. 多标签页/多窗口支持
// 5. 移动端模拟
// 6. 视频录制和截图
// 7. 并行执行
// 8. TypeScript原生支持
```

### 1.2 与其他测试框架对比

```typescript
/**
 * Playwright vs Cypress vs Selenium
 * @author erik.zhou
 */

// Playwright优势：
// - 真正的跨浏览器支持（包括WebKit）
// - 更快的执行速度
// - 更好的网络控制
// - 原生支持多标签页
// - 更强大的自动等待机制

// Cypress优势：
// - 更简单的API
// - 更好的开发体验
// - 实时重载
// - 时间旅行调试

// Selenium优势：
// - 最成熟的生态系统
// - 支持更多编程语言
// - 更多的社区资源
```

---

## 2. 安装与配置

### 2.1 安装Playwright

```bash
# 安装Playwright
npm init playwright@latest

# 或手动安装
npm install -D @playwright/test

# 安装浏览器
npx playwright install

# 安装特定浏览器
npx playwright install chromium
npx playwright install firefox
npx playwright install webkit
```

### 2.2 配置文件

```typescript
/**
 * Playwright配置文件
 * @author erik.zhou
 */

// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
    // 测试目录
    testDir: './tests',
    
    // 测试匹配模式
    testMatch: '**/*.spec.ts',
    
    // 全局超时时间
    timeout: 30000,
    
    // 期望超时时间
    expect: {
        timeout: 5000
    },
    
    // 失败重试次数
    retries: process.env.CI ? 2 : 0,
    
    // 并行执行的worker数量
    workers: process.env.CI ? 1 : undefined,
    
    // 报告配置
    reporter: [
        ['html'],
        ['json', { outputFile: 'test-results.json' }],
        ['junit', { outputFile: 'test-results.xml' }]
    ],
    
    // 全局配置
    use: {
        // 基础URL
        baseURL: 'http://localhost:3000',
        
        // 浏览器选项
        headless: true,
        
        // 视口大小
        viewport: { width: 1280, height: 720 },
        
        // 忽略HTTPS错误
        ignoreHTTPSErrors: true,
        
        // 截图配置
        screenshot: 'only-on-failure',
        
        // 视频录制
        video: 'retain-on-failure',
        
        // 追踪
        trace: 'on-first-retry'
    },
    
    // 项目配置（多浏览器测试）
    projects: [
        {
            name: 'chromium',
            use: { ...devices['Desktop Chrome'] }
        },
        {
            name: 'firefox',
            use: { ...devices['Desktop Firefox'] }
        },
        {
            name: 'webkit',
            use: { ...devices['Desktop Safari'] }
        },
        {
            name: 'mobile-chrome',
            use: { ...devices['Pixel 5'] }
        },
        {
            name: 'mobile-safari',
            use: { ...devices['iPhone 12'] }
        }
    ],
    
    // Web服务器配置
    webServer: {
        command: 'npm run dev',
        port: 3000,
        timeout: 120000,
        reuseExistingServer: !process.env.CI
    }
});
```

### 2.3 TypeScript配置

```json
/**
 * TypeScript配置
 * @author erik.zhou
 */

// tsconfig.json
{
    "compilerOptions": {
        "target": "ES2020",
        "module": "commonjs",
        "lib": ["ES2020"],
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true,
        "types": ["@playwright/test"]
    },
    "include": ["tests/**/*"]
}
```

---

## 3. 基础用法

### 3.1 第一个测试

```typescript
/**
 * 第一个Playwright测试
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

test('basic test', async ({ page }) => {
    // 导航到页面
    await page.goto('https://playwright.dev/');
    
    // 验证标题
    await expect(page).toHaveTitle(/Playwright/);
    
    // 点击链接
    await page.getByRole('link', { name: 'Get started' }).click();
    
    // 验证URL
    await expect(page).toHaveURL(/.*intro/);
});
```

### 3.2 定位器（Locators）

```typescript
/**
 * Playwright定位器示例
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

test('locators', async ({ page }) => {
    await page.goto('http://localhost:3000');
    
    // 1. 通过角色定位（推荐）
    const button = page.getByRole('button', { name: 'Submit' });
    await button.click();
    
    // 2. 通过文本定位
    const heading = page.getByText('Welcome');
    await expect(heading).toBeVisible();
    
    // 3. 通过标签定位
    const input = page.getByLabel('Username');
    await input.fill('testuser');
    
    // 4. 通过占位符定位
    const emailInput = page.getByPlaceholder('Enter email');
    await emailInput.fill('test@example.com');
    
    // 5. 通过测试ID定位
    const element = page.getByTestId('custom-element');
    await element.click();
    
    // 6. 通过CSS选择器定位
    const cssElement = page.locator('.my-class');
    await cssElement.click();
    
    // 7. 通过XPath定位
    const xpathElement = page.locator('xpath=//button[@type="submit"]');
    await xpathElement.click();
    
    // 8. 链式定位
    const form = page.locator('form');
    const submitButton = form.getByRole('button', { name: 'Submit' });
    await submitButton.click();
    
    // 9. 过滤定位器
    const items = page.getByRole('listitem');
    const activeItem = items.filter({ hasText: 'Active' });
    await activeItem.click();
});
```

### 3.3 用户交互

```typescript
/**
 * 用户交互示例
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

test('user interactions', async ({ page }) => {
    await page.goto('http://localhost:3000');
    
    // 点击
    await page.getByRole('button', { name: 'Click me' }).click();
    
    // 双击
    await page.getByRole('button', { name: 'Double click' }).dblclick();
    
    // 右键点击
    await page.getByRole('button', { name: 'Context menu' }).click({ button: 'right' });
    
    // 悬停
    await page.getByRole('button', { name: 'Hover' }).hover();
    
    // 输入文本
    await page.getByLabel('Username').fill('testuser');
    
    // 逐字输入（模拟真实打字）
    await page.getByLabel('Password').type('password123', { delay: 100 });
    
    // 清空输入
    await page.getByLabel('Username').clear();
    
    // 选择下拉框
    await page.getByLabel('Country').selectOption('US');
    
    // 勾选复选框
    await page.getByLabel('Accept terms').check();
    
    // 取消勾选
    await page.getByLabel('Accept terms').uncheck();
    
    // 选择单选按钮
    await page.getByLabel('Male').check();
    
    // 上传文件
    await page.getByLabel('Upload').setInputFiles('path/to/file.pdf');
    
    // 上传多个文件
    await page.getByLabel('Upload').setInputFiles([
        'path/to/file1.pdf',
        'path/to/file2.pdf'
    ]);
    
    // 键盘操作
    await page.keyboard.press('Enter');
    await page.keyboard.press('Control+A');
    await page.keyboard.type('Hello World');
    
    // 鼠标操作
    await page.mouse.move(100, 200);
    await page.mouse.click(100, 200);
    await page.mouse.dblclick(100, 200);
});
```

### 3.4 断言

```typescript
/**
 * Playwright断言示例
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

test('assertions', async ({ page }) => {
    await page.goto('http://localhost:3000');
    
    // 页面断言
    await expect(page).toHaveTitle('My App');
    await expect(page).toHaveURL('http://localhost:3000');
    
    // 元素可见性
    await expect(page.getByText('Welcome')).toBeVisible();
    await expect(page.getByText('Hidden')).toBeHidden();
    
    // 元素状态
    await expect(page.getByRole('button')).toBeEnabled();
    await expect(page.getByRole('button')).toBeDisabled();
    await expect(page.getByRole('checkbox')).toBeChecked();
    
    // 文本内容
    await expect(page.getByRole('heading')).toHaveText('Welcome');
    await expect(page.getByRole('heading')).toContainText('Wel');
    
    // 属性值
    await expect(page.getByRole('link')).toHaveAttribute('href', '/about');
    await expect(page.getByRole('button')).toHaveClass('btn-primary');
    
    // 输入值
    await expect(page.getByLabel('Username')).toHaveValue('testuser');
    
    // 元素数量
    await expect(page.getByRole('listitem')).toHaveCount(5);
    
    // CSS属性
    await expect(page.getByRole('button')).toHaveCSS('background-color', 'rgb(0, 123, 255)');
    
    // 截图对比
    await expect(page).toHaveScreenshot('homepage.png');
});
```


---

## 4. 页面对象模型

### 4.1 基础页面对象

```typescript
/**
 * 页面对象模型基础
 * @author erik.zhou
 */

import { Page, Locator } from '@playwright/test';

export class LoginPage {
    readonly page: Page;
    readonly usernameInput: Locator;
    readonly passwordInput: Locator;
    readonly submitButton: Locator;
    readonly errorMessage: Locator;
    
    constructor(page: Page) {
        this.page = page;
        this.usernameInput = page.getByLabel('Username');
        this.passwordInput = page.getByLabel('Password');
        this.submitButton = page.getByRole('button', { name: 'Login' });
        this.errorMessage = page.getByRole('alert');
    }
    
    async goto() {
        await this.page.goto('/login');
    }
    
    async login(username: string, password: string) {
        await this.usernameInput.fill(username);
        await this.passwordInput.fill(password);
        await this.submitButton.click();
    }
    
    async getErrorMessage() {
        return await this.errorMessage.textContent();
    }
}

// 使用页面对象
import { test, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

test('login with valid credentials', async ({ page }) => {
    const loginPage = new LoginPage(page);
    
    await loginPage.goto();
    await loginPage.login('testuser', 'password123');
    
    await expect(page).toHaveURL('/dashboard');
});
```

### 4.2 组件对象模型

```typescript
/**
 * 组件对象模型
 * @author erik.zhou
 */

import { Page, Locator } from '@playwright/test';

export class NavigationComponent {
    readonly page: Page;
    readonly container: Locator;
    
    constructor(page: Page) {
        this.page = page;
        this.container = page.locator('nav');
    }
    
    async clickLink(name: string) {
        await this.container.getByRole('link', { name }).click();
    }
    
    async isLinkVisible(name: string) {
        return await this.container.getByRole('link', { name }).isVisible();
    }
}

export class DashboardPage {
    readonly page: Page;
    readonly navigation: NavigationComponent;
    readonly welcomeMessage: Locator;
    
    constructor(page: Page) {
        this.page = page;
        this.navigation = new NavigationComponent(page);
        this.welcomeMessage = page.getByRole('heading', { name: /welcome/i });
    }
    
    async goto() {
        await this.page.goto('/dashboard');
    }
    
    async navigateToProfile() {
        await this.navigation.clickLink('Profile');
    }
}
```

### 4.3 页面工厂模式

```typescript
/**
 * 页面工厂模式
 * @author erik.zhou
 */

import { Page } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';
import { DashboardPage } from './pages/DashboardPage';
import { ProfilePage } from './pages/ProfilePage';

export class PageFactory {
    constructor(private page: Page) {}
    
    loginPage() {
        return new LoginPage(this.page);
    }
    
    dashboardPage() {
        return new DashboardPage(this.page);
    }
    
    profilePage() {
        return new ProfilePage(this.page);
    }
}

// 使用工厂模式
test('user flow with page factory', async ({ page }) => {
    const pages = new PageFactory(page);
    
    const loginPage = pages.loginPage();
    await loginPage.goto();
    await loginPage.login('testuser', 'password123');
    
    const dashboardPage = pages.dashboardPage();
    await expect(dashboardPage.welcomeMessage).toBeVisible();
    
    await dashboardPage.navigateToProfile();
    
    const profilePage = pages.profilePage();
    await expect(profilePage.heading).toBeVisible();
});
```

---

## 5. API测试

### 5.1 基础API测试

```typescript
/**
 * API测试基础
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

test('GET request', async ({ request }) => {
    const response = await request.get('https://api.example.com/users');
    
    expect(response.ok()).toBeTruthy();
    expect(response.status()).toBe(200);
    
    const data = await response.json();
    expect(data).toHaveLength(10);
});

test('POST request', async ({ request }) => {
    const response = await request.post('https://api.example.com/users', {
        data: {
            name: 'John Doe',
            email: 'john@example.com'
        }
    });
    
    expect(response.ok()).toBeTruthy();
    expect(response.status()).toBe(201);
    
    const data = await response.json();
    expect(data.name).toBe('John Doe');
    expect(data.email).toBe('john@example.com');
});

test('PUT request', async ({ request }) => {
    const response = await request.put('https://api.example.com/users/1', {
        data: {
            name: 'Jane Doe',
            email: 'jane@example.com'
        }
    });
    
    expect(response.ok()).toBeTruthy();
    expect(response.status()).toBe(200);
});

test('DELETE request', async ({ request }) => {
    const response = await request.delete('https://api.example.com/users/1');
    
    expect(response.ok()).toBeTruthy();
    expect(response.status()).toBe(204);
});
```

### 5.2 API认证

```typescript
/**
 * API认证测试
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

test('API with Bearer token', async ({ request }) => {
    const response = await request.get('https://api.example.com/protected', {
        headers: {
            'Authorization': 'Bearer your-token-here'
        }
    });
    
    expect(response.ok()).toBeTruthy();
});

test('API with Basic Auth', async ({ request }) => {
    const response = await request.get('https://api.example.com/protected', {
        headers: {
            'Authorization': `Basic ${Buffer.from('username:password').toString('base64')}`
        }
    });
    
    expect(response.ok()).toBeTruthy();
});

// 使用fixture设置全局认证
test.use({
    extraHTTPHeaders: {
        'Authorization': 'Bearer your-token-here'
    }
});

test('authenticated request', async ({ request }) => {
    const response = await request.get('https://api.example.com/protected');
    expect(response.ok()).toBeTruthy();
});
```

### 5.3 网络拦截与Mock

```typescript
/**
 * 网络拦截与Mock
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

test('mock API response', async ({ page }) => {
    // 拦截API请求并返回mock数据
    await page.route('**/api/users', route => {
        route.fulfill({
            status: 200,
            contentType: 'application/json',
            body: JSON.stringify([
                { id: 1, name: 'Mock User 1' },
                { id: 2, name: 'Mock User 2' }
            ])
        });
    });
    
    await page.goto('/users');
    
    await expect(page.getByText('Mock User 1')).toBeVisible();
    await expect(page.getByText('Mock User 2')).toBeVisible();
});

test('modify API response', async ({ page }) => {
    await page.route('**/api/users', async route => {
        const response = await route.fetch();
        const json = await response.json();
        
        // 修改响应数据
        json.push({ id: 999, name: 'Added User' });
        
        await route.fulfill({
            response,
            json
        });
    });
    
    await page.goto('/users');
    
    await expect(page.getByText('Added User')).toBeVisible();
});

test('abort specific requests', async ({ page }) => {
    // 阻止图片加载
    await page.route('**/*.{png,jpg,jpeg}', route => route.abort());
    
    await page.goto('/');
});

test('wait for API response', async ({ page }) => {
    const responsePromise = page.waitForResponse('**/api/users');
    
    await page.goto('/users');
    
    const response = await responsePromise;
    expect(response.status()).toBe(200);
    
    const data = await response.json();
    expect(data).toHaveLength(10);
});
```

---

## 6. 视觉回归测试

### 6.1 截图对比

```typescript
/**
 * 视觉回归测试
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

test('full page screenshot', async ({ page }) => {
    await page.goto('/');
    
    // 全页面截图对比
    await expect(page).toHaveScreenshot('homepage.png');
});

test('element screenshot', async ({ page }) => {
    await page.goto('/');
    
    const header = page.locator('header');
    
    // 元素截图对比
    await expect(header).toHaveScreenshot('header.png');
});

test('screenshot with options', async ({ page }) => {
    await page.goto('/');
    
    await expect(page).toHaveScreenshot('homepage-full.png', {
        fullPage: true,
        animations: 'disabled',
        mask: [page.locator('.dynamic-content')],
        maxDiffPixels: 100
    });
});
```

### 6.2 视觉测试配置

```typescript
/**
 * 视觉测试配置
 * @author erik.zhou
 */

// playwright.config.ts
export default defineConfig({
    expect: {
        toHaveScreenshot: {
            // 最大像素差异
            maxDiffPixels: 100,
            
            // 最大像素差异比例
            maxDiffPixelRatio: 0.1,
            
            // 阈值
            threshold: 0.2,
            
            // 禁用动画
            animations: 'disabled',
            
            // CSS缩放
            scale: 'css'
        }
    }
});

test('visual regression with custom config', async ({ page }) => {
    await page.goto('/');
    
    await expect(page).toHaveScreenshot({
        maxDiffPixels: 50,
        threshold: 0.1
    });
});
```


---

## 7. 移动端测试

### 7.1 移动设备模拟

```typescript
/**
 * 移动设备模拟
 * @author erik.zhou
 */

import { test, expect, devices } from '@playwright/test';

test('mobile viewport', async ({ page }) => {
    // 设置移动端视口
    await page.setViewportSize({ width: 375, height: 667 });
    
    await page.goto('/');
    
    // 验证移动端布局
    await expect(page.locator('.mobile-menu')).toBeVisible();
});

test('iPhone 12 simulation', async ({ browser }) => {
    const iPhone12 = devices['iPhone 12'];
    const context = await browser.newContext({
        ...iPhone12
    });
    
    const page = await context.newPage();
    await page.goto('/');
    
    await expect(page).toHaveScreenshot('iphone12.png');
    
    await context.close();
});

test('iPad simulation', async ({ browser }) => {
    const iPad = devices['iPad Pro'];
    const context = await browser.newContext({
        ...iPad
    });
    
    const page = await context.newPage();
    await page.goto('/');
    
    await context.close();
});
```

### 7.2 触摸手势

```typescript
/**
 * 触摸手势测试
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

test('tap gesture', async ({ page }) => {
    await page.goto('/');
    
    // 点击
    await page.locator('.button').tap();
});

test('swipe gesture', async ({ page }) => {
    await page.goto('/carousel');
    
    const carousel = page.locator('.carousel');
    
    // 获取元素位置
    const box = await carousel.boundingBox();
    
    if (box) {
        // 向左滑动
        await page.touchscreen.tap(box.x + box.width - 10, box.y + box.height / 2);
        await page.mouse.move(box.x + 10, box.y + box.height / 2);
    }
});

test('pinch zoom', async ({ page }) => {
    await page.goto('/map');
    
    // 模拟双指缩放
    await page.evaluate(() => {
        const event = new WheelEvent('wheel', {
            deltaY: -100,
            ctrlKey: true
        });
        document.dispatchEvent(event);
    });
});
```

### 7.3 地理位置

```typescript
/**
 * 地理位置测试
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

test('geolocation', async ({ context, page }) => {
    // 设置地理位置
    await context.setGeolocation({
        latitude: 37.7749,
        longitude: -122.4194
    });
    
    // 授予地理位置权限
    await context.grantPermissions(['geolocation']);
    
    await page.goto('/map');
    
    // 验证地图显示正确位置
    await expect(page.locator('.location-marker')).toBeVisible();
});
```

---

## 8. CI/CD集成

### 8.1 GitHub Actions配置

```yaml
# .github/workflows/playwright.yml
# @author erik.zhou

name: Playwright Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright Browsers
        run: npx playwright install --with-deps
      
      - name: Run Playwright tests
        run: npx playwright test
      
      - uses: actions/upload-artifact@v3
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

### 8.2 Docker配置

```dockerfile
# Dockerfile
# @author erik.zhou

FROM mcr.microsoft.com/playwright:v1.40.0-focal

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

CMD ["npx", "playwright", "test"]
```

```yaml
# docker-compose.yml
# @author erik.zhou

version: '3.8'

services:
  playwright:
    build: .
    volumes:
      - ./test-results:/app/test-results
      - ./playwright-report:/app/playwright-report
    environment:
      - CI=true
```

### 8.3 并行执行

```typescript
/**
 * 并行执行配置
 * @author erik.zhou
 */

// playwright.config.ts
export default defineConfig({
    // 设置worker数量
    workers: process.env.CI ? 2 : undefined,
    
    // 完全并行
    fullyParallel: true,
    
    // 失败重试
    retries: process.env.CI ? 2 : 0,
    
    // 项目并行执行
    projects: [
        {
            name: 'chromium',
            use: { ...devices['Desktop Chrome'] }
        },
        {
            name: 'firefox',
            use: { ...devices['Desktop Firefox'] }
        }
    ]
});

// 使用test.describe.parallel
test.describe.parallel('parallel suite', () => {
    test('test 1', async ({ page }) => {
        // 这些测试将并行执行
    });
    
    test('test 2', async ({ page }) => {
        // 这些测试将并行执行
    });
});
```

---

## 9. 高级特性

### 9.1 Fixtures

```typescript
/**
 * 自定义Fixtures
 * @author erik.zhou
 */

import { test as base, expect } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

type MyFixtures = {
    loginPage: LoginPage;
    authenticatedPage: Page;
};

const test = base.extend<MyFixtures>({
    loginPage: async ({ page }, use) => {
        const loginPage = new LoginPage(page);
        await use(loginPage);
    },
    
    authenticatedPage: async ({ page }, use) => {
        // 自动登录
        await page.goto('/login');
        await page.getByLabel('Username').fill('testuser');
        await page.getByLabel('Password').fill('password123');
        await page.getByRole('button', { name: 'Login' }).click();
        
        await use(page);
    }
});

// 使用自定义fixture
test('use login page fixture', async ({ loginPage }) => {
    await loginPage.goto();
    await loginPage.login('testuser', 'password123');
});

test('use authenticated page', async ({ authenticatedPage }) => {
    // 页面已经登录
    await expect(authenticatedPage).toHaveURL('/dashboard');
});
```

### 9.2 全局设置和清理

```typescript
/**
 * 全局设置和清理
 * @author erik.zhou
 */

// global-setup.ts
import { chromium, FullConfig } from '@playwright/test';

async function globalSetup(config: FullConfig) {
    const browser = await chromium.launch();
    const page = await browser.newPage();
    
    // 执行全局设置
    await page.goto('http://localhost:3000/setup');
    await page.getByRole('button', { name: 'Initialize' }).click();
    
    await browser.close();
}

export default globalSetup;

// global-teardown.ts
async function globalTeardown(config: FullConfig) {
    // 执行全局清理
    console.log('Cleaning up...');
}

export default globalTeardown;

// playwright.config.ts
export default defineConfig({
    globalSetup: require.resolve('./global-setup'),
    globalTeardown: require.resolve('./global-teardown')
});
```

### 9.3 追踪和调试

```typescript
/**
 * 追踪和调试
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

test('with tracing', async ({ page, context }) => {
    // 开始追踪
    await context.tracing.start({
        screenshots: true,
        snapshots: true,
        sources: true
    });
    
    await page.goto('/');
    await page.getByRole('button', { name: 'Click me' }).click();
    
    // 停止追踪并保存
    await context.tracing.stop({
        path: 'trace.zip'
    });
});

// 使用调试模式
test('debug mode', async ({ page }) => {
    // 设置断点
    await page.pause();
    
    await page.goto('/');
    
    // 慢速执行
    await page.setDefaultTimeout(5000);
});

// 查看Playwright Inspector
// npx playwright test --debug

// 查看追踪文件
// npx playwright show-trace trace.zip
```

### 9.4 视频录制

```typescript
/**
 * 视频录制
 * @author erik.zhou
 */

// playwright.config.ts
export default defineConfig({
    use: {
        // 录制所有测试
        video: 'on',
        
        // 仅失败时录制
        video: 'retain-on-failure',
        
        // 视频大小
        videoSize: { width: 1280, height: 720 }
    }
});

test('with video', async ({ page }) => {
    await page.goto('/');
    await page.getByRole('button').click();
    
    // 视频将自动保存到test-results目录
});
```


---

## 10. 最佳实践

### 10.1 测试组织

```typescript
/**
 * 测试组织最佳实践
 * @author erik.zhou
 */

// ✅ 好的实践 - 按功能模块组织
/*
tests/
  auth/
    login.spec.ts
    register.spec.ts
    logout.spec.ts
  dashboard/
    overview.spec.ts
    settings.spec.ts
  pages/
    LoginPage.ts
    DashboardPage.ts
  fixtures/
    auth.ts
  utils/
    helpers.ts
*/

// ✅ 使用describe分组
test.describe('User Authentication', () => {
    test.describe('Login', () => {
        test('with valid credentials', async ({ page }) => {
            // 测试逻辑
        });
        
        test('with invalid credentials', async ({ page }) => {
            // 测试逻辑
        });
    });
    
    test.describe('Logout', () => {
        test('should logout successfully', async ({ page }) => {
            // 测试逻辑
        });
    });
});
```

### 10.2 等待策略

```typescript
/**
 * 等待策略最佳实践
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

test('waiting strategies', async ({ page }) => {
    await page.goto('/');
    
    // ✅ 好的实践 - 使用自动等待
    await page.getByRole('button', { name: 'Submit' }).click();
    
    // ✅ 等待导航
    await page.waitForURL('/success');
    
    // ✅ 等待特定状态
    await page.waitForLoadState('networkidle');
    
    // ✅ 等待选择器
    await page.waitForSelector('.result', { state: 'visible' });
    
    // ✅ 等待函数
    await page.waitForFunction(() => {
        return document.querySelectorAll('.item').length > 5;
    });
    
    // ❌ 不好的实践 - 使用固定延迟
    // await page.waitForTimeout(5000);
});
```

### 10.3 错误处理

```typescript
/**
 * 错误处理最佳实践
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

test('error handling', async ({ page }) => {
    // ✅ 使用try-catch处理预期错误
    try {
        await page.goto('/non-existent-page');
    } catch (error) {
        expect(error.message).toContain('404');
    }
    
    // ✅ 验证错误消息
    await page.goto('/form');
    await page.getByRole('button', { name: 'Submit' }).click();
    
    await expect(page.getByRole('alert')).toHaveText('Please fill all fields');
});

test('handle network errors', async ({ page }) => {
    // 模拟网络错误
    await page.route('**/api/data', route => {
        route.abort('failed');
    });
    
    await page.goto('/');
    
    // 验证错误处理
    await expect(page.getByText(/network error/i)).toBeVisible();
});
```

### 10.4 性能优化

```typescript
/**
 * 性能优化最佳实践
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

// ✅ 复用浏览器上下文
test.describe('Performance Tests', () => {
    let context;
    let page;
    
    test.beforeAll(async ({ browser }) => {
        context = await browser.newContext();
        page = await context.newPage();
    });
    
    test.afterAll(async () => {
        await context.close();
    });
    
    test('test 1', async () => {
        await page.goto('/page1');
    });
    
    test('test 2', async () => {
        await page.goto('/page2');
    });
});

// ✅ 阻止不必要的资源加载
test('block unnecessary resources', async ({ context }) => {
    await context.route('**/*.{png,jpg,jpeg,gif,svg}', route => route.abort());
    await context.route('**/*.{woff,woff2,ttf}', route => route.abort());
    
    const page = await context.newPage();
    await page.goto('/');
});

// ✅ 并行执行独立测试
test.describe.parallel('Independent Tests', () => {
    test('test 1', async ({ page }) => {
        // 独立测试
    });
    
    test('test 2', async ({ page }) => {
        // 独立测试
    });
});
```

### 10.5 数据管理

```typescript
/**
 * 测试数据管理
 * @author erik.zhou
 */

import { test, expect } from '@playwright/test';

// ✅ 使用测试数据工厂
class UserFactory {
    static create(overrides = {}) {
        return {
            username: 'testuser',
            email: 'test@example.com',
            password: 'password123',
            ...overrides
        };
    }
    
    static createAdmin() {
        return this.create({
            username: 'admin',
            role: 'admin'
        });
    }
}

test('use test data factory', async ({ page }) => {
    const user = UserFactory.create({ username: 'john' });
    
    await page.goto('/register');
    await page.getByLabel('Username').fill(user.username);
    await page.getByLabel('Email').fill(user.email);
    await page.getByLabel('Password').fill(user.password);
});

// ✅ 从文件加载测试数据
import testData from './fixtures/users.json';

test('use data from file', async ({ page }) => {
    const user = testData.users[0];
    
    await page.goto('/login');
    await page.getByLabel('Username').fill(user.username);
    await page.getByLabel('Password').fill(user.password);
});
```

### 10.6 可维护性

```typescript
/**
 * 可维护性最佳实践
 * @author erik.zhou
 */

// ✅ 使用常量管理选择器
export const SELECTORS = {
    LOGIN: {
        USERNAME: '[data-testid="username"]',
        PASSWORD: '[data-testid="password"]',
        SUBMIT: '[data-testid="submit"]'
    },
    DASHBOARD: {
        WELCOME: '[data-testid="welcome"]',
        MENU: '[data-testid="menu"]'
    }
};

test('use selector constants', async ({ page }) => {
    await page.goto('/login');
    await page.locator(SELECTORS.LOGIN.USERNAME).fill('testuser');
    await page.locator(SELECTORS.LOGIN.PASSWORD).fill('password123');
    await page.locator(SELECTORS.LOGIN.SUBMIT).click();
});

// ✅ 提取通用操作
class TestHelpers {
    static async login(page: Page, username: string, password: string) {
        await page.goto('/login');
        await page.getByLabel('Username').fill(username);
        await page.getByLabel('Password').fill(password);
        await page.getByRole('button', { name: 'Login' }).click();
    }
    
    static async logout(page: Page) {
        await page.getByRole('button', { name: 'Logout' }).click();
    }
}

test('use helper functions', async ({ page }) => {
    await TestHelpers.login(page, 'testuser', 'password123');
    await expect(page).toHaveURL('/dashboard');
    
    await TestHelpers.logout(page);
    await expect(page).toHaveURL('/login');
});
```

### 10.7 报告和监控

```typescript
/**
 * 报告和监控配置
 * @author erik.zhou
 */

// playwright.config.ts
export default defineConfig({
    reporter: [
        // HTML报告
        ['html', { open: 'never' }],
        
        // JSON报告
        ['json', { outputFile: 'test-results.json' }],
        
        // JUnit报告（用于CI）
        ['junit', { outputFile: 'junit.xml' }],
        
        // 自定义报告
        ['./custom-reporter.ts']
    ]
});

// custom-reporter.ts
import { Reporter, TestCase, TestResult } from '@playwright/test/reporter';

class CustomReporter implements Reporter {
    onTestEnd(test: TestCase, result: TestResult) {
        console.log(`Test ${test.title}: ${result.status}`);
        
        if (result.status === 'failed') {
            // 发送通知或记录日志
            console.error(`Test failed: ${test.title}`);
        }
    }
}

export default CustomReporter;
```

---

## 总结

### 核心要点

1. **跨浏览器支持**
   - Chromium、Firefox、WebKit全覆盖
   - 真实浏览器环境测试
   - 移动端设备模拟

2. **强大的自动等待**
   - 智能等待元素可交互
   - 减少flaky测试
   - 无需手动添加等待

3. **网络控制**
   - API拦截和Mock
   - 网络条件模拟
   - 请求/响应修改

4. **页面对象模型**
   - 提高代码复用性
   - 降低维护成本
   - 清晰的测试结构

5. **CI/CD集成**
   - 并行执行
   - Docker支持
   - 丰富的报告格式

### 学习资源

- [Playwright官方文档](https://playwright.dev/)
- [Playwright GitHub](https://github.com/microsoft/playwright)
- [Playwright示例](https://github.com/microsoft/playwright/tree/main/examples)
- [Playwright社区](https://playwright.dev/community/welcome)

### 下一步

- 学习Cypress进行E2E测试
- 探索性能测试工具
- 了解可访问性测试
- 掌握测试策略和规划

---

**最后更新时间：** 2026-03-05  
**@author erik.zhou**
