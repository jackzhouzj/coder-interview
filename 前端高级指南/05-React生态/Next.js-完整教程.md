# Next.js - 完整教程

## 课程信息
- **课程名称**: Next.js完整教程
- **难度级别**: 中高级
- **预计学时**: 8小时
- **核心内容**: SSR/SSG、路由系统、API Routes、性能优化、部署
- **@author**: erik.zhou

---

## 目录
1. [Next.js概述](#1-nextjs概述)
2. [项目初始化](#2-项目初始化)
3. [路由系统](#3-路由系统)
4. [数据获取](#4-数据获取)
5. [API Routes](#5-api-routes)
6. [样式方案](#6-样式方案)
7. [图片优化](#7-图片优化)
8. [性能优化](#8-性能优化)
9. [部署上线](#9-部署上线)
10. [实战案例](#10-实战案例)

---

## 1. Next.js概述

### 1.1 什么是Next.js

Next.js是一个基于React的全栈框架，提供了服务端渲染（SSR）、静态站点生成（SSG）等功能。

**核心特点**:
- 零配置开箱即用
- 混合渲染模式（SSR/SSG/ISR）
- 文件系统路由
- API Routes支持
- 自动代码分割
- 内置CSS/Sass支持
- 图片优化
- TypeScript支持

### 1.2 为什么使用Next.js

```javascript
// 传统React应用的问题
const traditionalReactProblems = {
    SEO差: '客户端渲染不利于搜索引擎',
    首屏慢: '需要下载完整JS才能渲染',
    配置复杂: '需要手动配置Webpack、Babel等',
    路由繁琐: '需要额外安装配置路由库',
    API开发: '需要单独搭建后端服务'
};

// Next.js的优势
const nextjsBenefits = {
    SEO友好: '服务端渲染，搜索引擎可直接抓取',
    性能优秀: '首屏快速渲染，自动优化',
    开发体验: '零配置，约定优于配置',
    全栈能力: '内置API Routes，无需单独后端',
    灵活渲染: 'SSR/SSG/ISR多种模式可选'
};
```

### 1.3 渲染模式对比

```javascript
/**
 * Next.js渲染模式
 * @author erik.zhou
 */

// 1. SSR - 服务端渲染（每次请求都渲染）
export async function getServerSideProps() {
    const data = await fetchData();
    return { props: { data } };
}

// 2. SSG - 静态站点生成（构建时渲染）
export async function getStaticProps() {
    const data = await fetchData();
    return { props: { data } };
}

// 3. ISR - 增量静态再生（定时重新生成）
export async function getStaticProps() {
    const data = await fetchData();
    return {
        props: { data },
        revalidate: 60 // 60秒后重新生成
    };
}

// 4. CSR - 客户端渲染（传统React方式）
function Page() {
    const [data, setData] = useState(null);
    
    useEffect(() => {
        fetchData().then(setData);
    }, []);
    
    return <div>{data}</div>;
}
```

---

## 2. 项目初始化

### 2.1 创建项目

```bash
# 使用create-next-app创建项目
npx create-next-app@latest my-next-app

# 选择配置
# ✔ Would you like to use TypeScript? Yes
# ✔ Would you like to use ESLint? Yes
# ✔ Would you like to use Tailwind CSS? Yes
# ✔ Would you like to use `src/` directory? Yes
# ✔ Would you like to use App Router? Yes
# ✔ Would you like to customize the default import alias? No

# 进入项目目录
cd my-next-app

# 启动开发服务器
npm run dev
```

### 2.2 项目结构

```
my-next-app/
├── src/
│   ├── app/              # App Router目录
│   │   ├── layout.tsx    # 根布局
│   │   ├── page.tsx      # 首页
│   │   └── globals.css   # 全局样式
│   ├── components/       # 组件目录
│   └── lib/             # 工具函数
├── public/              # 静态资源
├── next.config.js       # Next.js配置
├── package.json
└── tsconfig.json
```

### 2.3 基础配置

```javascript
// next.config.js
/**
 * Next.js配置文件
 * @author erik.zhou
 */
/** @type {import('next').NextConfig} */
const nextConfig = {
    // 严格模式
    reactStrictMode: true,
    
    // 图片域名白名单
    images: {
        domains: ['example.com', 'cdn.example.com']
    },
    
    // 环境变量
    env: {
        API_URL: process.env.API_URL
    },
    
    // 重定向
    async redirects() {
        return [
            {
                source: '/old-path',
                destination: '/new-path',
                permanent: true
            }
        ];
    },
    
    // 重写
    async rewrites() {
        return [
            {
                source: '/api/:path*',
                destination: 'https://api.example.com/:path*'
            }
        ];
    }
};

module.exports = nextConfig;
```

---

## 3. 路由系统

### 3.1 文件系统路由

```typescript
/**
 * App Router文件系统路由
 * @author erik.zhou
 */

// app/page.tsx - 首页 (/)
export default function HomePage() {
    return <h1>首页</h1>;
}

// app/about/page.tsx - 关于页 (/about)
export default function AboutPage() {
    return <h1>关于我们</h1>;
}

// app/blog/page.tsx - 博客列表 (/blog)
export default function BlogPage() {
    return <h1>博客列表</h1>;
}

// app/blog/[slug]/page.tsx - 博客详情 (/blog/xxx)
export default function BlogPostPage({ params }: { params: { slug: string } }) {
    return <h1>博客: {params.slug}</h1>;
}
```

### 3.2 动态路由

```typescript
// app/products/[id]/page.tsx
/**
 * 动态路由 - 产品详情页
 * @author erik.zhou
 */
interface ProductPageProps {
    params: {
        id: string;
    };
}

export default function ProductPage({ params }: ProductPageProps) {
    return (
        <div>
            <h1>产品ID: {params.id}</h1>
        </div>
    );
}

// 生成静态路径
export async function generateStaticParams() {
    const products = await fetchProducts();
    
    return products.map((product) => ({
        id: product.id.toString()
    }));
}
```

```typescript
// app/blog/[...slug]/page.tsx
/**
 * 捕获所有路由
 * @author erik.zhou
 */
interface CatchAllPageProps {
    params: {
        slug: string[];
    };
}

export default function CatchAllPage({ params }: CatchAllPageProps) {
    // /blog/2024/01/post -> ['2024', '01', 'post']
    return (
        <div>
            <h1>路径: {params.slug.join('/')}</h1>
        </div>
    );
}
```

### 3.3 路由组和布局

```typescript
// app/layout.tsx
/**
 * 根布局
 * @author erik.zhou
 */
import './globals.css';
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export const metadata = {
    title: 'My Next.js App',
    description: 'Created with Next.js'
};

export default function RootLayout({
    children
}: {
    children: React.ReactNode;
}) {
    return (
        <html lang="zh-CN">
            <body className={inter.className}>
                <header>
                    <nav>导航栏</nav>
                </header>
                <main>{children}</main>
                <footer>页脚</footer>
            </body>
        </html>
    );
}
```

```typescript
// app/dashboard/layout.tsx
/**
 * 仪表盘布局
 * @author erik.zhou
 */
export default function DashboardLayout({
    children
}: {
    children: React.ReactNode;
}) {
    return (
        <div className="dashboard">
            <aside>侧边栏</aside>
            <div className="content">{children}</div>
        </div>
    );
}
```

### 3.4 路由导航

```typescript
// components/Navigation.tsx
/**
 * 导航组件
 * @author erik.zhou
 */
'use client';

import Link from 'next/link';
import { usePathname, useRouter } from 'next/navigation';

export default function Navigation() {
    const pathname = usePathname();
    const router = useRouter();
    
    const handleNavigate = () => {
        router.push('/about');
        // router.replace('/about'); // 替换历史记录
        // router.back(); // 返回上一页
        // router.forward(); // 前进
    };
    
    return (
        <nav>
            <Link 
                href="/" 
                className={pathname === '/' ? 'active' : ''}
            >
                首页
            </Link>
            
            <Link 
                href="/about"
                className={pathname === '/about' ? 'active' : ''}
            >
                关于
            </Link>
            
            <Link 
                href="/blog"
                prefetch={true} // 预加载
            >
                博客
            </Link>
            
            <button onClick={handleNavigate}>
                编程式导航
            </button>
        </nav>
    );
}
```

---

## 4. 数据获取

### 4.1 服务端组件数据获取

```typescript
// app/posts/page.tsx
/**
 * 服务端组件 - 获取文章列表
 * @author erik.zhou
 */
interface Post {
    id: number;
    title: string;
    content: string;
}

async function getPosts(): Promise<Post[]> {
    const res = await fetch('https://api.example.com/posts', {
        cache: 'no-store' // 不缓存，每次都重新获取
    });
    
    if (!res.ok) {
        throw new Error('获取文章失败');
    }
    
    return res.json();
}

export default async function PostsPage() {
    const posts = await getPosts();
    
    return (
        <div>
            <h1>文章列表</h1>
            {posts.map(post => (
                <article key={post.id}>
                    <h2>{post.title}</h2>
                    <p>{post.content}</p>
                </article>
            ))}
        </div>
    );
}
```

### 4.2 缓存策略

```typescript
// app/products/page.tsx
/**
 * 不同的缓存策略
 * @author erik.zhou
 */

// 1. 强制缓存（默认）
async function getProducts1() {
    const res = await fetch('https://api.example.com/products');
    return res.json();
}

// 2. 不缓存
async function getProducts2() {
    const res = await fetch('https://api.example.com/products', {
        cache: 'no-store'
    });
    return res.json();
}

// 3. 定时重新验证
async function getProducts3() {
    const res = await fetch('https://api.example.com/products', {
        next: { revalidate: 60 } // 60秒后重新验证
    });
    return res.json();
}

// 4. 按需重新验证
async function getProducts4() {
    const res = await fetch('https://api.example.com/products', {
        next: { tags: ['products'] }
    });
    return res.json();
}

export default async function ProductsPage() {
    const products = await getProducts3();
    
    return (
        <div>
            {products.map((product: any) => (
                <div key={product.id}>{product.name}</div>
            ))}
        </div>
    );
}
```

### 4.3 客户端数据获取

```typescript
// app/users/page.tsx
/**
 * 客户端组件 - 数据获取
 * @author erik.zhou
 */
'use client';

import { useState, useEffect } from 'react';

interface User {
    id: number;
    name: string;
    email: string;
}

export default function UsersPage() {
    const [users, setUsers] = useState<User[]>([]);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState<string | null>(null);
    
    useEffect(() => {
        fetch('/api/users')
            .then(res => {
                if (!res.ok) {
                    throw new Error('获取用户失败');
                }
                return res.json();
            })
            .then(data => {
                setUsers(data);
                setLoading(false);
            })
            .catch(err => {
                setError(err.message);
                setLoading(false);
            });
    }, []);
    
    if (loading) {
        return <div>加载中...</div>;
    }
    
    if (error) {
        return <div>错误: {error}</div>;
    }
    
    return (
        <div>
            <h1>用户列表</h1>
            {users.map(user => (
                <div key={user.id}>
                    <h3>{user.name}</h3>
                    <p>{user.email}</p>
                </div>
            ))}
        </div>
    );
}
```

### 4.4 并行数据获取

```typescript
// app/dashboard/page.tsx
/**
 * 并行获取多个数据源
 * @author erik.zhou
 */
async function getUser() {
    const res = await fetch('https://api.example.com/user');
    return res.json();
}

async function getPosts() {
    const res = await fetch('https://api.example.com/posts');
    return res.json();
}

async function getComments() {
    const res = await fetch('https://api.example.com/comments');
    return res.json();
}

export default async function DashboardPage() {
    // 并行获取数据
    const [user, posts, comments] = await Promise.all([
        getUser(),
        getPosts(),
        getComments()
    ]);
    
    return (
        <div>
            <h1>欢迎, {user.name}</h1>
            <section>
                <h2>文章数: {posts.length}</h2>
            </section>
            <section>
                <h2>评论数: {comments.length}</h2>
            </section>
        </div>
    );
}
```

---

## 5. API Routes

### 5.1 基础API路由

```typescript
// app/api/hello/route.ts
/**
 * GET请求处理
 * @author erik.zhou
 */
import { NextResponse } from 'next/server';

export async function GET() {
    return NextResponse.json({
        message: 'Hello from API'
    });
}
```

```typescript
// app/api/users/route.ts
/**
 * 用户API - CRUD操作
 * @author erik.zhou
 */
import { NextRequest, NextResponse } from 'next/server';

// GET - 获取用户列表
export async function GET(request: NextRequest) {
    const searchParams = request.nextUrl.searchParams;
    const page = searchParams.get('page') || '1';
    
    // 模拟数据库查询
    const users = [
        { id: 1, name: '张三', email: 'zhangsan@example.com' },
        { id: 2, name: '李四', email: 'lisi@example.com' }
    ];
    
    return NextResponse.json({
        data: users,
        page: parseInt(page)
    });
}

// POST - 创建用户
export async function POST(request: NextRequest) {
    const body = await request.json();
    
    // 验证数据
    if (!body.name || !body.email) {
        return NextResponse.json(
            { error: '姓名和邮箱不能为空' },
            { status: 400 }
        );
    }
    
    // 模拟创建用户
    const newUser = {
        id: Date.now(),
        name: body.name,
        email: body.email
    };
    
    return NextResponse.json(newUser, { status: 201 });
}
```

### 5.2 动态API路由

```typescript
// app/api/users/[id]/route.ts
/**
 * 动态API路由 - 单个用户操作
 * @author erik.zhou
 */
import { NextRequest, NextResponse } from 'next/server';

interface RouteParams {
    params: {
        id: string;
    };
}

// GET - 获取单个用户
export async function GET(
    request: NextRequest,
    { params }: RouteParams
) {
    const userId = params.id;
    
    // 模拟数据库查询
    const user = {
        id: userId,
        name: '张三',
        email: 'zhangsan@example.com'
    };
    
    return NextResponse.json(user);
}

// PUT - 更新用户
export async function PUT(
    request: NextRequest,
    { params }: RouteParams
) {
    const userId = params.id;
    const body = await request.json();
    
    // 模拟更新用户
    const updatedUser = {
        id: userId,
        ...body
    };
    
    return NextResponse.json(updatedUser);
}

// DELETE - 删除用户
export async function DELETE(
    request: NextRequest,
    { params }: RouteParams
) {
    const userId = params.id;
    
    // 模拟删除用户
    return NextResponse.json({
        message: `用户 ${userId} 已删除`
    });
}
```

### 5.3 中间件

```typescript
// middleware.ts
/**
 * 全局中间件
 * @author erik.zhou
 */
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
    // 获取token
    const token = request.cookies.get('token')?.value;
    
    // 检查是否访问受保护的路由
    if (request.nextUrl.pathname.startsWith('/dashboard')) {
        if (!token) {
            // 重定向到登录页
            return NextResponse.redirect(new URL('/login', request.url));
        }
    }
    
    // 添加自定义响应头
    const response = NextResponse.next();
    response.headers.set('x-custom-header', 'my-value');
    
    return response;
}

// 配置中间件匹配路径
export const config = {
    matcher: ['/dashboard/:path*', '/api/:path*']
};
```

### 5.4 错误处理

```typescript
// app/api/posts/route.ts
/**
 * API错误处理
 * @author erik.zhou
 */
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
    try {
        const res = await fetch('https://api.example.com/posts');
        
        if (!res.ok) {
            throw new Error('获取文章失败');
        }
        
        const data = await res.json();
        return NextResponse.json(data);
        
    } catch (error) {
        console.error('API错误:', error);
        
        return NextResponse.json(
            {
                error: '服务器错误',
                message: error instanceof Error ? error.message : '未知错误'
            },
            { status: 500 }
        );
    }
}
```

---

## 6. 样式方案

### 6.1 CSS Modules

```typescript
// app/components/Button.tsx
/**
 * CSS Modules示例
 * @author erik.zhou
 */
import styles from './Button.module.css';

interface ButtonProps {
    children: React.ReactNode;
    variant?: 'primary' | 'secondary';
}

export default function Button({ children, variant = 'primary' }: ButtonProps) {
    return (
        <button className={`${styles.button} ${styles[variant]}`}>
            {children}
        </button>
    );
}
```

```css
/* Button.module.css */
.button {
    padding: 12px 24px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
}

.primary {
    background-color: #0070f3;
    color: white;
}

.secondary {
    background-color: #eaeaea;
    color: #333;
}
```

### 6.2 Tailwind CSS

```typescript
// app/components/Card.tsx
/**
 * Tailwind CSS示例
 * @author erik.zhou
 */
interface CardProps {
    title: string;
    description: string;
    image?: string;
}

export default function Card({ title, description, image }: CardProps) {
    return (
        <div className="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-xl transition-shadow">
            {image && (
                <img 
                    src={image} 
                    alt={title}
                    className="w-full h-48 object-cover"
                />
            )}
            <div className="p-6">
                <h3 className="text-xl font-bold mb-2 text-gray-800">
                    {title}
                </h3>
                <p className="text-gray-600">
                    {description}
                </p>
            </div>
        </div>
    );
}
```

### 6.3 CSS-in-JS (Styled Components)

```typescript
// app/components/StyledButton.tsx
/**
 * Styled Components示例
 * @author erik.zhou
 */
'use client';

import styled from 'styled-components';

const StyledButton = styled.button<{ variant?: 'primary' | 'secondary' }>`
    padding: 12px 24px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
    background-color: ${props => 
        props.variant === 'primary' ? '#0070f3' : '#eaeaea'
    };
    color: ${props => 
        props.variant === 'primary' ? 'white' : '#333'
    };
    
    &:hover {
        opacity: 0.8;
    }
`;

export default function Button({ 
    children, 
    variant = 'primary' 
}: {
    children: React.ReactNode;
    variant?: 'primary' | 'secondary';
}) {
    return (
        <StyledButton variant={variant}>
            {children}
        </StyledButton>
    );
}
```

### 6.4 全局样式

```css
/* app/globals.css */
/**
 * 全局样式
 * @author erik.zhou
 */
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
    --primary-color: #0070f3;
    --secondary-color: #eaeaea;
    --text-color: #333;
}

* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 
        'Helvetica Neue', Arial, sans-serif;
    color: var(--text-color);
    line-height: 1.6;
}

a {
    color: var(--primary-color);
    text-decoration: none;
}

a:hover {
    text-decoration: underline;
}
```

---

## 7. 图片优化

### 7.1 Image组件基础

```typescript
// app/components/OptimizedImage.tsx
/**
 * Next.js Image组件
 * @author erik.zhou
 */
import Image from 'next/image';

export default function OptimizedImage() {
    return (
        <div>
            {/* 本地图片 */}
            <Image
                src="/images/hero.jpg"
                alt="Hero Image"
                width={800}
                height={600}
                priority // 优先加载
            />
            
            {/* 远程图片 */}
            <Image
                src="https://example.com/image.jpg"
                alt="Remote Image"
                width={800}
                height={600}
                loading="lazy" // 懒加载
            />
            
            {/* 填充容器 */}
            <div style={{ position: 'relative', width: '100%', height: '400px' }}>
                <Image
                    src="/images/banner.jpg"
                    alt="Banner"
                    fill
                    style={{ objectFit: 'cover' }}
                />
            </div>
        </div>
    );
}
```

### 7.2 响应式图片

```typescript
// app/components/ResponsiveImage.tsx
/**
 * 响应式图片
 * @author erik.zhou
 */
import Image from 'next/image';

export default function ResponsiveImage() {
    return (
        <div className="relative w-full">
            <Image
                src="/images/product.jpg"
                alt="Product"
                width={1200}
                height={800}
                sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
                style={{
                    width: '100%',
                    height: 'auto'
                }}
            />
        </div>
    );
}
```

### 7.3 图片加载占位

```typescript
// app/components/ImageWithPlaceholder.tsx
/**
 * 带占位符的图片
 * @author erik.zhou
 */
import Image from 'next/image';

export default function ImageWithPlaceholder() {
    return (
        <div>
            {/* 模糊占位符 */}
            <Image
                src="/images/photo.jpg"
                alt="Photo"
                width={600}
                height={400}
                placeholder="blur"
                blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRg..." // Base64编码的模糊图片
            />
            
            {/* 自定义占位符 */}
            <div className="relative w-full h-64 bg-gray-200">
                <Image
                    src="/images/avatar.jpg"
                    alt="Avatar"
                    fill
                    className="rounded-full"
                />
            </div>
        </div>
    );
}
```

---

## 8. 性能优化

### 8.1 代码分割

```typescript
// app/components/DynamicComponent.tsx
/**
 * 动态导入组件
 * @author erik.zhou
 */
'use client';

import dynamic from 'next/dynamic';
import { Suspense } from 'react';

// 动态导入，不包含在初始bundle中
const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
    loading: () => <div>加载中...</div>,
    ssr: false // 禁用服务端渲染
});

const Chart = dynamic(() => import('./Chart'), {
    loading: () => <div>图表加载中...</div>
});

export default function DynamicComponent() {
    return (
        <div>
            <h1>动态组件示例</h1>
            
            <Suspense fallback={<div>加载中...</div>}>
                <HeavyComponent />
            </Suspense>
            
            <Chart />
        </div>
    );
}
```

### 8.2 字体优化

```typescript
// app/layout.tsx
/**
 * 字体优化
 * @author erik.zhou
 */
import { Inter, Roboto_Mono } from 'next/font/google';

// Google字体
const inter = Inter({
    subsets: ['latin'],
    display: 'swap',
    variable: '--font-inter'
});

const robotoMono = Roboto_Mono({
    subsets: ['latin'],
    display: 'swap',
    variable: '--font-roboto-mono'
});

export default function RootLayout({
    children
}: {
    children: React.ReactNode;
}) {
    return (
        <html lang="zh-CN" className={`${inter.variable} ${robotoMono.variable}`}>
            <body className={inter.className}>
                {children}
            </body>
        </html>
    );
}
```

```typescript
// app/fonts/localFont.ts
/**
 * 本地字体
 * @author erik.zhou
 */
import localFont from 'next/font/local';

export const myFont = localFont({
    src: [
        {
            path: './fonts/MyFont-Regular.woff2',
            weight: '400',
            style: 'normal'
        },
        {
            path: './fonts/MyFont-Bold.woff2',
            weight: '700',
            style: 'normal'
        }
    ],
    variable: '--font-my-font'
});
```

### 8.3 元数据优化

```typescript
// app/layout.tsx
/**
 * 静态元数据
 * @author erik.zhou
 */
import type { Metadata } from 'next';

export const metadata: Metadata = {
    title: {
        default: '我的网站',
        template: '%s | 我的网站'
    },
    description: '这是一个使用Next.js构建的网站',
    keywords: ['Next.js', 'React', 'TypeScript'],
    authors: [{ name: 'erik.zhou' }],
    openGraph: {
        title: '我的网站',
        description: '这是一个使用Next.js构建的网站',
        url: 'https://example.com',
        siteName: '我的网站',
        images: [
            {
                url: 'https://example.com/og-image.jpg',
                width: 1200,
                height: 630
            }
        ],
        locale: 'zh_CN',
        type: 'website'
    },
    twitter: {
        card: 'summary_large_image',
        title: '我的网站',
        description: '这是一个使用Next.js构建的网站',
        images: ['https://example.com/twitter-image.jpg']
    },
    robots: {
        index: true,
        follow: true
    }
};
```

```typescript
// app/blog/[slug]/page.tsx
/**
 * 动态元数据
 * @author erik.zhou
 */
import type { Metadata } from 'next';

interface PageProps {
    params: { slug: string };
}

export async function generateMetadata({ params }: PageProps): Promise<Metadata> {
    const post = await getPost(params.slug);
    
    return {
        title: post.title,
        description: post.excerpt,
        openGraph: {
            title: post.title,
            description: post.excerpt,
            images: [post.coverImage]
        }
    };
}

export default async function BlogPost({ params }: PageProps) {
    const post = await getPost(params.slug);
    
    return (
        <article>
            <h1>{post.title}</h1>
            <div dangerouslySetInnerHTML={{ __html: post.content }} />
        </article>
    );
}
```

### 8.4 流式渲染

```typescript
// app/dashboard/page.tsx
/**
 * 流式渲染和Suspense
 * @author erik.zhou
 */
import { Suspense } from 'react';

async function SlowComponent() {
    // 模拟慢速数据获取
    await new Promise(resolve => setTimeout(resolve, 3000));
    return <div>慢速组件内容</div>;
}

async function FastComponent() {
    await new Promise(resolve => setTimeout(resolve, 500));
    return <div>快速组件内容</div>;
}

export default function DashboardPage() {
    return (
        <div>
            <h1>仪表盘</h1>
            
            {/* 快速组件立即显示 */}
            <Suspense fallback={<div>加载快速组件...</div>}>
                <FastComponent />
            </Suspense>
            
            {/* 慢速组件延迟显示 */}
            <Suspense fallback={<div>加载慢速组件...</div>}>
                <SlowComponent />
            </Suspense>
        </div>
    );
}
```

---

## 9. 部署上线

### 9.1 Vercel部署

```bash
# 安装Vercel CLI
npm install -g vercel

# 登录Vercel
vercel login

# 部署到生产环境
vercel --prod

# 或者通过Git自动部署
# 1. 推送代码到GitHub
# 2. 在Vercel导入项目
# 3. 自动部署
```

```javascript
// vercel.json
/**
 * Vercel配置文件
 * @author erik.zhou
 */
{
    "buildCommand": "npm run build",
    "outputDirectory": ".next",
    "devCommand": "npm run dev",
    "installCommand": "npm install",
    "framework": "nextjs",
    "regions": ["sin1"],
    "env": {
        "API_URL": "@api-url"
    },
    "headers": [
        {
            "source": "/(.*)",
            "headers": [
                {
                    "key": "X-Content-Type-Options",
                    "value": "nosniff"
                },
                {
                    "key": "X-Frame-Options",
                    "value": "DENY"
                }
            ]
        }
    ]
}
```

### 9.2 Docker部署

```dockerfile
# Dockerfile
# @author erik.zhou

# 构建阶段
FROM node:18-alpine AS builder

WORKDIR /app

# 复制依赖文件
COPY package*.json ./

# 安装依赖
RUN npm ci

# 复制源代码
COPY . .

# 构建应用
RUN npm run build

# 生产阶段
FROM node:18-alpine AS runner

WORKDIR /app

ENV NODE_ENV production

# 创建非root用户
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# 复制构建产物
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

```yaml
# docker-compose.yml
# @author erik.zhou
version: '3.8'

services:
    nextjs:
        build: .
        ports:
            - "3000:3000"
        environment:
            - NODE_ENV=production
            - API_URL=https://api.example.com
        restart: unless-stopped
```

### 9.3 环境变量配置

```bash
# .env.local (本地开发)
# @author erik.zhou
NEXT_PUBLIC_API_URL=http://localhost:8000
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
SECRET_KEY=your-secret-key

# .env.production (生产环境)
NEXT_PUBLIC_API_URL=https://api.example.com
DATABASE_URL=postgresql://user:password@prod-db:5432/mydb
SECRET_KEY=production-secret-key
```

```typescript
// lib/env.ts
/**
 * 环境变量类型定义
 * @author erik.zhou
 */
export const env = {
    // 公开环境变量（客户端可访问）
    apiUrl: process.env.NEXT_PUBLIC_API_URL!,
    
    // 私有环境变量（仅服务端可访问）
    databaseUrl: process.env.DATABASE_URL!,
    secretKey: process.env.SECRET_KEY!
};

// 验证必需的环境变量
function validateEnv() {
    const required = ['NEXT_PUBLIC_API_URL', 'DATABASE_URL', 'SECRET_KEY'];
    
    for (const key of required) {
        if (!process.env[key]) {
            throw new Error(`缺少必需的环境变量: ${key}`);
        }
    }
}

validateEnv();
```

### 9.4 性能监控

```typescript
// app/layout.tsx
/**
 * 性能监控
 * @author erik.zhou
 */
import { Analytics } from '@vercel/analytics/react';
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({
    children
}: {
    children: React.ReactNode;
}) {
    return (
        <html lang="zh-CN">
            <body>
                {children}
                <Analytics />
                <SpeedInsights />
            </body>
        </html>
    );
}
```

```typescript
// lib/monitoring.ts
/**
 * 自定义性能监控
 * @author erik.zhou
 */
export function reportWebVitals(metric: any) {
    // 发送到分析服务
    if (metric.label === 'web-vital') {
        console.log(metric);
        
        // 发送到Google Analytics
        window.gtag?.('event', metric.name, {
            value: Math.round(
                metric.name === 'CLS' ? metric.value * 1000 : metric.value
            ),
            event_label: metric.id,
            non_interaction: true
        });
    }
}
```

---

## 10. 实战案例

### 10.1 博客系统

```typescript
// app/blog/page.tsx
/**
 * 博客列表页
 * @author erik.zhou
 */
import Link from 'next/link';
import { getPosts } from '@/lib/posts';

export const metadata = {
    title: '博客',
    description: '技术博客文章列表'
};

export default async function BlogPage() {
    const posts = await getPosts();
    
    return (
        <div className="container mx-auto px-4 py-8">
            <h1 className="text-4xl font-bold mb-8">博客文章</h1>
            
            <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
                {posts.map(post => (
                    <article 
                        key={post.slug}
                        className="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-xl transition-shadow"
                    >
                        <Link href={`/blog/${post.slug}`}>
                            <img
                                src={post.coverImage}
                                alt={post.title}
                                className="w-full h-48 object-cover"
                            />
                            <div className="p-6">
                                <h2 className="text-xl font-bold mb-2">
                                    {post.title}
                                </h2>
                                <p className="text-gray-600 mb-4">
                                    {post.excerpt}
                                </p>
                                <div className="flex items-center text-sm text-gray-500">
                                    <time>{post.date}</time>
                                    <span className="mx-2">·</span>
                                    <span>{post.readingTime}分钟阅读</span>
                                </div>
                            </div>
                        </Link>
                    </article>
                ))}
            </div>
        </div>
    );
}
```

```typescript
// app/blog/[slug]/page.tsx
/**
 * 博客详情页
 * @author erik.zhou
 */
import { notFound } from 'next/navigation';
import { getPost, getAllPostSlugs } from '@/lib/posts';
import { MDXRemote } from 'next-mdx-remote/rsc';

interface PageProps {
    params: { slug: string };
}

export async function generateStaticParams() {
    const slugs = await getAllPostSlugs();
    return slugs.map(slug => ({ slug }));
}

export async function generateMetadata({ params }: PageProps) {
    const post = await getPost(params.slug);
    
    if (!post) {
        return {};
    }
    
    return {
        title: post.title,
        description: post.excerpt,
        openGraph: {
            title: post.title,
            description: post.excerpt,
            images: [post.coverImage]
        }
    };
}

export default async function BlogPostPage({ params }: PageProps) {
    const post = await getPost(params.slug);
    
    if (!post) {
        notFound();
    }
    
    return (
        <article className="container mx-auto px-4 py-8 max-w-3xl">
            <header className="mb-8">
                <h1 className="text-4xl font-bold mb-4">{post.title}</h1>
                <div className="flex items-center text-gray-600">
                    <time>{post.date}</time>
                    <span className="mx-2">·</span>
                    <span>{post.readingTime}分钟阅读</span>
                </div>
            </header>
            
            <div className="prose prose-lg max-w-none">
                <MDXRemote source={post.content} />
            </div>
        </article>
    );
}
```

```typescript
// lib/posts.ts
/**
 * 博客文章数据处理
 * @author erik.zhou
 */
import fs from 'fs';
import path from 'path';
import matter from 'gray-matter';

const postsDirectory = path.join(process.cwd(), 'content/posts');

export interface Post {
    slug: string;
    title: string;
    date: string;
    excerpt: string;
    coverImage: string;
    content: string;
    readingTime: number;
}

export async function getPosts(): Promise<Post[]> {
    const fileNames = fs.readdirSync(postsDirectory);
    
    const posts = fileNames.map(fileName => {
        const slug = fileName.replace(/\.mdx$/, '');
        const fullPath = path.join(postsDirectory, fileName);
        const fileContents = fs.readFileSync(fullPath, 'utf8');
        const { data, content } = matter(fileContents);
        
        return {
            slug,
            title: data.title,
            date: data.date,
            excerpt: data.excerpt,
            coverImage: data.coverImage,
            content,
            readingTime: Math.ceil(content.split(' ').length / 200)
        };
    });
    
    return posts.sort((a, b) => (a.date > b.date ? -1 : 1));
}

export async function getPost(slug: string): Promise<Post | null> {
    try {
        const fullPath = path.join(postsDirectory, `${slug}.mdx`);
        const fileContents = fs.readFileSync(fullPath, 'utf8');
        const { data, content } = matter(fileContents);
        
        return {
            slug,
            title: data.title,
            date: data.date,
            excerpt: data.excerpt,
            coverImage: data.coverImage,
            content,
            readingTime: Math.ceil(content.split(' ').length / 200)
        };
    } catch {
        return null;
    }
}

export async function getAllPostSlugs(): Promise<string[]> {
    const fileNames = fs.readdirSync(postsDirectory);
    return fileNames.map(fileName => fileName.replace(/\.mdx$/, ''));
}
```

### 10.2 电商产品页

```typescript
// app/products/page.tsx
/**
 * 产品列表页
 * @author erik.zhou
 */
'use client';

import { useState } from 'react';
import { useRouter, useSearchParams } from 'next/navigation';
import ProductCard from '@/components/ProductCard';
import Pagination from '@/components/Pagination';

interface Product {
    id: number;
    name: string;
    price: number;
    image: string;
    category: string;
}

export default function ProductsPage() {
    const router = useRouter();
    const searchParams = useSearchParams();
    const [products, setProducts] = useState<Product[]>([]);
    const [loading, setLoading] = useState(false);
    
    const page = parseInt(searchParams.get('page') || '1');
    const category = searchParams.get('category') || 'all';
    
    const handleCategoryChange = (newCategory: string) => {
        const params = new URLSearchParams(searchParams);
        params.set('category', newCategory);
        params.set('page', '1');
        router.push(`/products?${params.toString()}`);
    };
    
    const handlePageChange = (newPage: number) => {
        const params = new URLSearchParams(searchParams);
        params.set('page', newPage.toString());
        router.push(`/products?${params.toString()}`);
    };
    
    return (
        <div className="container mx-auto px-4 py-8">
            <h1 className="text-4xl font-bold mb-8">产品列表</h1>
            
            {/* 分类筛选 */}
            <div className="mb-8">
                <button
                    onClick={() => handleCategoryChange('all')}
                    className={`mr-4 ${category === 'all' ? 'font-bold' : ''}`}
                >
                    全部
                </button>
                <button
                    onClick={() => handleCategoryChange('electronics')}
                    className={`mr-4 ${category === 'electronics' ? 'font-bold' : ''}`}
                >
                    电子产品
                </button>
                <button
                    onClick={() => handleCategoryChange('clothing')}
                    className={`mr-4 ${category === 'clothing' ? 'font-bold' : ''}`}
                >
                    服装
                </button>
            </div>
            
            {/* 产品网格 */}
            {loading ? (
                <div>加载中...</div>
            ) : (
                <>
                    <div className="grid gap-6 md:grid-cols-3 lg:grid-cols-4">
                        {products.map(product => (
                            <ProductCard key={product.id} product={product} />
                        ))}
                    </div>
                    
                    <Pagination
                        currentPage={page}
                        totalPages={10}
                        onPageChange={handlePageChange}
                    />
                </>
            )}
        </div>
    );
}
```

```typescript
// app/products/[id]/page.tsx
/**
 * 产品详情页
 * @author erik.zhou
 */
import { notFound } from 'next/navigation';
import Image from 'next/image';
import AddToCartButton from '@/components/AddToCartButton';

interface Product {
    id: number;
    name: string;
    price: number;
    description: string;
    images: string[];
    specifications: Record<string, string>;
}

async function getProduct(id: string): Promise<Product | null> {
    const res = await fetch(`https://api.example.com/products/${id}`, {
        next: { revalidate: 3600 }
    });
    
    if (!res.ok) {
        return null;
    }
    
    return res.json();
}

export async function generateMetadata({ params }: { params: { id: string } }) {
    const product = await getProduct(params.id);
    
    if (!product) {
        return {};
    }
    
    return {
        title: product.name,
        description: product.description,
        openGraph: {
            images: [product.images[0]]
        }
    };
}

export default async function ProductPage({ params }: { params: { id: string } }) {
    const product = await getProduct(params.id);
    
    if (!product) {
        notFound();
    }
    
    return (
        <div className="container mx-auto px-4 py-8">
            <div className="grid md:grid-cols-2 gap-8">
                {/* 产品图片 */}
                <div>
                    <Image
                        src={product.images[0]}
                        alt={product.name}
                        width={600}
                        height={600}
                        className="rounded-lg"
                    />
                </div>
                
                {/* 产品信息 */}
                <div>
                    <h1 className="text-3xl font-bold mb-4">{product.name}</h1>
                    <p className="text-2xl text-red-600 font-bold mb-6">
                        ¥{product.price}
                    </p>
                    <p className="text-gray-600 mb-6">{product.description}</p>
                    
                    {/* 规格参数 */}
                    <div className="mb-6">
                        <h3 className="font-bold mb-2">规格参数</h3>
                        <dl className="grid grid-cols-2 gap-2">
                            {Object.entries(product.specifications).map(([key, value]) => (
                                <div key={key}>
                                    <dt className="text-gray-600">{key}</dt>
                                    <dd className="font-medium">{value}</dd>
                                </div>
                            ))}
                        </dl>
                    </div>
                    
                    <AddToCartButton productId={product.id} />
                </div>
            </div>
        </div>
    );
}
```

### 10.3 用户认证系统

```typescript
// app/api/auth/login/route.ts
/**
 * 登录API
 * @author erik.zhou
 */
import { NextRequest, NextResponse } from 'next/server';
import { SignJWT } from 'jose';
import { cookies } from 'next/headers';

export async function POST(request: NextRequest) {
    try {
        const { email, password } = await request.json();
        
        // 验证用户凭据（示例）
        if (email === 'user@example.com' && password === 'password') {
            // 生成JWT token
            const secret = new TextEncoder().encode(process.env.JWT_SECRET);
            const token = await new SignJWT({ email })
                .setProtectedHeader({ alg: 'HS256' })
                .setExpirationTime('24h')
                .sign(secret);
            
            // 设置cookie
            cookies().set('token', token, {
                httpOnly: true,
                secure: process.env.NODE_ENV === 'production',
                sameSite: 'lax',
                maxAge: 60 * 60 * 24 // 24小时
            });
            
            return NextResponse.json({
                success: true,
                message: '登录成功'
            });
        }
        
        return NextResponse.json(
            { error: '邮箱或密码错误' },
            { status: 401 }
        );
        
    } catch (error) {
        return NextResponse.json(
            { error: '登录失败' },
            { status: 500 }
        );
    }
}
```

```typescript
// middleware.ts
/**
 * 认证中间件
 * @author erik.zhou
 */
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { jwtVerify } from 'jose';

export async function middleware(request: NextRequest) {
    const token = request.cookies.get('token')?.value;
    
    // 检查是否访问受保护的路由
    if (request.nextUrl.pathname.startsWith('/dashboard')) {
        if (!token) {
            return NextResponse.redirect(new URL('/login', request.url));
        }
        
        try {
            const secret = new TextEncoder().encode(process.env.JWT_SECRET);
            await jwtVerify(token, secret);
        } catch {
            return NextResponse.redirect(new URL('/login', request.url));
        }
    }
    
    return NextResponse.next();
}

export const config = {
    matcher: ['/dashboard/:path*']
};
```

```typescript
// app/login/page.tsx
/**
 * 登录页面
 * @author erik.zhou
 */
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';

export default function LoginPage() {
    const router = useRouter();
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    const [error, setError] = useState('');
    const [loading, setLoading] = useState(false);
    
    const handleSubmit = async (e: React.FormEvent) => {
        e.preventDefault();
        setError('');
        setLoading(true);
        
        try {
            const res = await fetch('/api/auth/login', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ email, password })
            });
            
            const data = await res.json();
            
            if (res.ok) {
                router.push('/dashboard');
            } else {
                setError(data.error);
            }
        } catch {
            setError('登录失败，请重试');
        } finally {
            setLoading(false);
        }
    };
    
    return (
        <div className="min-h-screen flex items-center justify-center bg-gray-50">
            <div className="max-w-md w-full bg-white rounded-lg shadow-md p-8">
                <h2 className="text-2xl font-bold mb-6 text-center">登录</h2>
                
                <form onSubmit={handleSubmit}>
                    <div className="mb-4">
                        <label className="block text-gray-700 mb-2">邮箱</label>
                        <input
                            type="email"
                            value={email}
                            onChange={(e) => setEmail(e.target.value)}
                            className="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
                            required
                        />
                    </div>
                    
                    <div className="mb-6">
                        <label className="block text-gray-700 mb-2">密码</label>
                        <input
                            type="password"
                            value={password}
                            onChange={(e) => setPassword(e.target.value)}
                            className="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
                            required
                        />
                    </div>
                    
                    {error && (
                        <div className="mb-4 text-red-600 text-sm">{error}</div>
                    )}
                    
                    <button
                        type="submit"
                        disabled={loading}
                        className="w-full bg-blue-600 text-white py-2 rounded-lg hover:bg-blue-700 disabled:opacity-50"
                    >
                        {loading ? '登录中...' : '登录'}
                    </button>
                </form>
            </div>
        </div>
    );
}
```

---

## 总结

### 核心要点

1. **渲染模式**
   - SSR：服务端渲染，适合动态内容
   - SSG：静态生成，适合静态内容
   - ISR：增量静态再生，兼顾性能和实时性
   - CSR：客户端渲染，适合交互密集场景

2. **路由系统**
   - 文件系统路由，约定优于配置
   - 动态路由和捕获所有路由
   - 布局和嵌套路由
   - 路由组和并行路由

3. **数据获取**
   - 服务端组件直接获取数据
   - 灵活的缓存策略
   - 并行和串行数据获取
   - 流式渲染和Suspense

4. **性能优化**
   - 自动代码分割
   - 图片和字体优化
   - 动态导入
   - 元数据优化

5. **部署方案**
   - Vercel一键部署
   - Docker容器化
   - 环境变量管理
   - 性能监控

### 学习路径

1. **基础阶段**（2-3天）
   - 理解Next.js核心概念
   - 掌握路由系统
   - 学习数据获取方法
   - 了解渲染模式

2. **进阶阶段**（1周）
   - 掌握API Routes
   - 学习样式方案
   - 理解性能优化
   - 实践中间件

3. **实战阶段**（2-3周）
   - 构建完整项目
   - 实现用户认证
   - 优化SEO
   - 部署上线

4. **高级阶段**（持续学习）
   - 深入理解渲染原理
   - 掌握高级优化技巧
   - 学习最佳实践
   - 探索新特性

### 推荐资源

**官方文档**
- [Next.js官方文档](https://nextjs.org/docs)
- [Next.js学习课程](https://nextjs.org/learn)
- [Next.js示例](https://github.com/vercel/next.js/tree/canary/examples)

**学习资源**
- [Next.js中文文档](https://www.nextjs.cn/)
- [Next.js最佳实践](https://nextjs.org/docs/pages/building-your-application/deploying/production-checklist)
- [Vercel博客](https://vercel.com/blog)

**社区资源**
- [Next.js Discord](https://discord.com/invite/bUG2bvbtHy)
- [GitHub Discussions](https://github.com/vercel/next.js/discussions)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/next.js)

**视频教程**
- [Next.js 13完整教程](https://www.youtube.com/watch?v=ZVnjOPwW4ZA)
- [Next.js实战项目](https://www.youtube.com/watch?v=843nec-IvW0)
- [Next.js性能优化](https://www.youtube.com/watch?v=SUD5MdHl-Ik)

---

## 附录

### A. 常用命令

```bash
# 创建项目
npx create-next-app@latest

# 开发模式
npm run dev

# 构建生产版本
npm run build

# 启动生产服务器
npm run start

# 代码检查
npm run lint

# 类型检查
npm run type-check
```

### B. 配置模板

```javascript
// next.config.js完整配置
/**
 * @author erik.zhou
 */
/** @type {import('next').NextConfig} */
const nextConfig = {
    reactStrictMode: true,
    swcMinify: true,
    
    images: {
        domains: ['example.com'],
        formats: ['image/avif', 'image/webp']
    },
    
    experimental: {
        serverActions: true
    },
    
    async headers() {
        return [
            {
                source: '/:path*',
                headers: [
                    {
                        key: 'X-DNS-Prefetch-Control',
                        value: 'on'
                    }
                ]
            }
        ];
    },
    
    async redirects() {
        return [
            {
                source: '/old',
                destination: '/new',
                permanent: true
            }
        ];
    },
    
    async rewrites() {
        return [
            {
                source: '/api/:path*',
                destination: 'https://api.example.com/:path*'
            }
        ];
    }
};

module.exports = nextConfig;
```

### C. 常见问题

**Q1: 如何选择渲染模式？**

- 静态内容（博客、文档）→ SSG
- 动态内容（用户数据）→ SSR
- 需要实时性但可接受延迟 → ISR
- 交互密集型应用 → CSR

**Q2: 如何优化首屏加载？**

```typescript
// 1. 使用动态导入
const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
    ssr: false
});

// 2. 图片优化
<Image src="/hero.jpg" priority />

// 3. 字体优化
const inter = Inter({ subsets: ['latin'], display: 'swap' });

// 4. 代码分割
export const dynamic = 'force-dynamic';
```

**Q3: 如何处理环境变量？**

```bash
# .env.local
NEXT_PUBLIC_API_URL=http://localhost:3000
DATABASE_URL=postgresql://...
```

**Q4: 如何实现国际化？**

```typescript
// next.config.js
module.exports = {
    i18n: {
        locales: ['zh-CN', 'en-US'],
        defaultLocale: 'zh-CN'
    }
};
```

**Q5: 如何调试Next.js应用？**

```json
// .vscode/launch.json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Next.js: debug server-side",
            "type": "node-terminal",
            "request": "launch",
            "command": "npm run dev"
        }
    ]
}
```

### D. 性能优化清单

- [ ] 使用Image组件优化图片
- [ ] 启用字体优化
- [ ] 实现代码分割
- [ ] 配置合适的缓存策略
- [ ] 使用Suspense和流式渲染
- [ ] 优化元数据和SEO
- [ ] 启用压缩和minify
- [ ] 配置CDN
- [ ] 监控Core Web Vitals
- [ ] 使用性能分析工具

---

## 结语

Next.js是现代Web开发的强大框架，它将React的灵活性与服务端渲染的性能优势完美结合。通过本教程的学习，你应该已经掌握了：

- Next.js的核心概念和渲染模式
- 路由系统和数据获取
- API Routes和中间件
- 样式方案和图片优化
- 性能优化技巧
- 部署和生产环境配置

记住，Next.js的强大之处在于它的灵活性。根据项目需求选择合适的渲染模式，合理使用缓存策略，持续优化性能，你就能构建出高性能的Web应用。

继续实践，不断探索，祝你在Next.js的学习之路上越走越远！

---

**@author**: erik.zhou  
**最后更新**: 2026-02-27  
**版本**: v1.0.0
