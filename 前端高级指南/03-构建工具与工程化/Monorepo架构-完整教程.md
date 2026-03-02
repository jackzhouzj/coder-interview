# Monorepo架构 - 完整教程

## 课程信息
- **课程名称**: Monorepo架构完整教程
- **难度级别**: 高级
- **预计学时**: 10小时
- **核心内容**: Monorepo概念、工具对比、实战配置、最佳实践
- **@author**: erik.zhou

---

## 目录
1. [Monorepo概述](#1-monorepo概述)
2. [Monorepo vs Polyrepo](#2-monorepo-vs-polyrepo)
3. [Monorepo工具对比](#3-monorepo工具对比)
4. [Lerna实战](#4-lerna实战)
5. [Nx实战](#5-nx实战)
6. [pnpm Workspace实战](#6-pnpm-workspace实战)
7. [Turborepo实战](#7-turborepo实战)
8. [依赖管理](#8-依赖管理)
9. [构建优化](#9-构建优化)
10. [最佳实践](#10-最佳实践)

---

## 1. Monorepo概述

### 1.1 什么是Monorepo

Monorepo（单一代码库）是一种将多个项目存储在同一个代码仓库中的软件开发策略。

**核心特点**:
- 单一代码仓库
- 多个项目/包
- 共享依赖
- 统一工具链
- 原子化提交

### 1.2 Monorepo的优势

```javascript
// Monorepo结构示例
monorepo-project/
├── packages/
│   ├── web/              // Web应用
│   ├── mobile/           // 移动应用
│   ├── shared/           // 共享代码
│   ├── ui-components/    // UI组件库
│   └── utils/            // 工具函数库
├── package.json
└── pnpm-workspace.yaml
```

**优势**:
1. **代码共享**：轻松共享代码和组件
2. **统一管理**：统一的依赖、工具链、配置
3. **原子化提交**：跨项目的变更可以在一次提交中完成
4. **简化重构**：重构影响多个项目时更容易
5. **一致性**：确保所有项目使用相同的版本和配置

### 1.3 Monorepo的挑战

```javascript
const challenges = {
    // 1. 性能问题
    performance: {
        issue: '随着项目增多，构建和测试时间增加',
        solution: '使用增量构建、缓存、并行执行'
    },
    
    // 2. 权限管理
    permissions: {
        issue: '不同团队可能需要不同的访问权限',
        solution: '使用CODEOWNERS、分支保护'
    },
    
    // 3. 版本管理
    versioning: {
        issue: '多个包的版本管理复杂',
        solution: '使用统一版本或独立版本策略'
    },
    
    // 4. CI/CD复杂度
    cicd: {
        issue: 'CI/CD流程需要智能判断哪些包需要构建',
        solution: '使用affected命令、缓存策略'
    }
};
```



---

## 2. Monorepo vs Polyrepo

### 2.1 架构对比

```javascript
// Polyrepo结构
polyrepo/
├── web-app/              // 独立仓库
│   ├── .git/
│   ├── package.json
│   └── src/
├── mobile-app/           // 独立仓库
│   ├── .git/
│   ├── package.json
│   └── src/
└── shared-lib/           // 独立仓库
    ├── .git/
    ├── package.json
    └── src/

// Monorepo结构
monorepo/
├── .git/
├── packages/
│   ├── web-app/
│   │   ├── package.json
│   │   └── src/
│   ├── mobile-app/
│   │   ├── package.json
│   │   └── src/
│   └── shared-lib/
│       ├── package.json
│       └── src/
└── package.json
```

### 2.2 对比分析

| 维度 | Monorepo | Polyrepo |
|------|----------|----------|
| 代码共享 | ✅ 容易 | ❌ 需要发布npm包 |
| 依赖管理 | ✅ 统一管理 | ❌ 各自管理 |
| 版本控制 | ✅ 原子化提交 | ❌ 需要多次提交 |
| CI/CD | ⚠️ 复杂但可优化 | ✅ 简单独立 |
| 权限管理 | ❌ 粒度粗 | ✅ 粒度细 |
| 构建速度 | ⚠️ 需要优化 | ✅ 独立快速 |
| 学习成本 | ⚠️ 较高 | ✅ 较低 |
| 适用场景 | 紧密关联的项目 | 独立的项目 |

### 2.3 选择建议

```javascript
// 适合使用Monorepo的场景
const monorepoScenarios = [
    {
        scenario: '多个紧密关联的项目',
        example: 'Web应用 + 移动应用 + 管理后台',
        reason: '共享大量代码和组件'
    },
    {
        scenario: '组件库 + 文档站点',
        example: 'UI组件库 + Storybook + 文档网站',
        reason: '需要同步更新和测试'
    },
    {
        scenario: '微前端架构',
        example: '主应用 + 多个子应用',
        reason: '统一依赖和构建流程'
    },
    {
        scenario: '全栈项目',
        example: '前端 + 后端 + 共享类型定义',
        reason: '类型安全和代码复用'
    }
];

// 适合使用Polyrepo的场景
const polyrepoScenarios = [
    {
        scenario: '完全独立的项目',
        example: '不同客户的定制项目',
        reason: '无代码共享需求'
    },
    {
        scenario: '不同技术栈的项目',
        example: 'React项目 + Vue项目 + Angular项目',
        reason: '工具链差异大'
    },
    {
        scenario: '需要严格权限控制',
        example: '不同团队的独立项目',
        reason: '安全和权限要求'
    }
];
```

---

## 3. Monorepo工具对比

### 3.1 主流工具概览

```javascript
const monorepoTools = {
    lerna: {
        description: '老牌Monorepo工具',
        features: ['版本管理', '发布管理', '依赖管理'],
        pros: ['成熟稳定', '社区活跃', '文档完善'],
        cons: ['性能一般', '功能相对基础'],
        适用场景: 'npm包管理'
    },
    
    nx: {
        description: '智能构建系统',
        features: ['增量构建', '依赖图分析', '缓存', '代码生成'],
        pros: ['性能优秀', '功能强大', '插件丰富'],
        cons: ['学习曲线陡', '配置复杂'],
        适用场景: '大型企业级项目'
    },
    
    pnpm: {
        description: '快速的包管理器',
        features: ['Workspace', '硬链接', '节省磁盘空间'],
        pros: ['速度快', '节省空间', '严格依赖'],
        cons: ['生态相对较新', '部分工具兼容性问题'],
        适用场景: '注重性能和磁盘空间'
    },
    
    turborepo: {
        description: '高性能构建系统',
        features: ['远程缓存', '并行执行', '增量构建'],
        pros: ['极快的构建速度', '配置简单', '零配置缓存'],
        cons: ['功能相对单一', '生态较新'],
        适用场景: '注重构建性能'
    },
    
    yarn: {
        description: 'Facebook的包管理器',
        features: ['Workspace', 'Plug\'n\'Play', '零安装'],
        pros: ['成熟稳定', '性能好', 'PnP模式'],
        cons: ['PnP兼容性问题'],
        适用场景: '已使用Yarn的项目'
    }
};
```

### 3.2 性能对比

```javascript
// 构建性能对比（相对值）
const performanceComparison = {
    '首次构建': {
        lerna: 100,
        nx: 60,
        pnpm: 70,
        turborepo: 50,
        yarn: 75
    },
    
    '增量构建': {
        lerna: 100,
        nx: 20,
        pnpm: 80,
        turborepo: 15,
        yarn: 70
    },
    
    '缓存命中': {
        lerna: 100,
        nx: 5,
        pnpm: 90,
        turborepo: 3,
        yarn: 85
    },
    
    '依赖安装': {
        lerna: 100,
        nx: 90,
        pnpm: 40,
        turborepo: 90,
        yarn: 70
    }
};
```

### 3.3 功能对比矩阵

| 功能 | Lerna | Nx | pnpm | Turborepo | Yarn |
|------|-------|----|----|-----------|------|
| Workspace | ✅ | ✅ | ✅ | ✅ | ✅ |
| 依赖提升 | ✅ | ✅ | ⚠️ | ✅ | ✅ |
| 增量构建 | ❌ | ✅ | ❌ | ✅ | ❌ |
| 本地缓存 | ❌ | ✅ | ❌ | ✅ | ❌ |
| 远程缓存 | ❌ | ✅ | ❌ | ✅ | ❌ |
| 依赖图分析 | ⚠️ | ✅ | ❌ | ✅ | ❌ |
| 代码生成 | ❌ | ✅ | ❌ | ❌ | ❌ |
| 版本管理 | ✅ | ⚠️ | ❌ | ❌ | ⚠️ |
| 发布管理 | ✅ | ⚠️ | ❌ | ❌ | ⚠️ |
| 并行执行 | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## 4. Lerna实战

### 4.1 安装和初始化

```bash
# 全局安装Lerna
npm install -g lerna

# 初始化Lerna项目
mkdir my-monorepo
cd my-monorepo
lerna init

# 或者使用独立版本模式
lerna init --independent
```

### 4.2 项目结构

```javascript
// 初始化后的结构
my-monorepo/
├── packages/
│   └── (空)
├── lerna.json
└── package.json

// lerna.json配置
{
    "version": "0.0.0",
    "npmClient": "npm",
    "packages": [
        "packages/*"
    ],
    "command": {
        "publish": {
            "message": "chore(release): publish"
        }
    }
}
```

### 4.3 创建包

```bash
# 创建新包
cd packages
mkdir package-a
cd package-a
npm init -y

# 或使用Lerna命令
lerna create package-a
lerna create package-b
```

```javascript
// packages/package-a/package.json
{
    "name": "@my-monorepo/package-a",
    "version": "1.0.0",
    "main": "index.js",
    "scripts": {
        "test": "jest",
        "build": "babel src -d lib"
    },
    "dependencies": {
        "lodash": "^4.17.21"
    }
}

// packages/package-a/index.js
const _ = require('lodash');

function greet(name) {
    return _.capitalize(`hello, ${name}!`);
}

module.exports = { greet };
```

### 4.4 依赖管理

```bash
# 为所有包安装依赖
lerna bootstrap

# 为特定包添加依赖
lerna add lodash --scope=@my-monorepo/package-a

# 添加开发依赖
lerna add jest --dev

# 添加内部依赖
lerna add @my-monorepo/package-a --scope=@my-monorepo/package-b

# 清理node_modules
lerna clean
```

### 4.5 执行脚本

```bash
# 在所有包中执行脚本
lerna run test

# 在特定包中执行
lerna run test --scope=@my-monorepo/package-a

# 并行执行
lerna run build --parallel

# 按拓扑顺序执行
lerna run build --stream
```

### 4.6 版本管理

```bash
# 查看变更的包
lerna changed

# 查看差异
lerna diff

# 发布新版本
lerna publish

# 发布预发布版本
lerna publish --canary

# 从git标签发布
lerna publish from-git

# 从包发布
lerna publish from-package
```

### 4.7 完整配置示例

```javascript
// lerna.json
{
    "version": "independent",
    "npmClient": "pnpm",
    "useWorkspaces": true,
    "packages": [
        "packages/*"
    ],
    "command": {
        "bootstrap": {
            "hoist": true,
            "npmClientArgs": ["--no-package-lock"]
        },
        "publish": {
            "conventionalCommits": true,
            "message": "chore(release): publish",
            "registry": "https://registry.npmjs.org/"
        },
        "version": {
            "allowBranch": ["main", "develop"],
            "message": "chore(release): %s"
        }
    },
    "ignoreChanges": [
        "**/*.md",
        "**/*.test.js",
        "**/test/**"
    ]
}

// package.json
{
    "name": "my-monorepo",
    "private": true,
    "workspaces": [
        "packages/*"
    ],
    "scripts": {
        "bootstrap": "lerna bootstrap",
        "clean": "lerna clean",
        "test": "lerna run test",
        "build": "lerna run build --stream",
        "publish": "lerna publish"
    },
    "devDependencies": {
        "lerna": "^7.0.0"
    }
}
```



---

## 5. Nx实战

### 5.1 安装和初始化

```bash
# 使用npx创建Nx工作空间
npx create-nx-workspace@latest my-nx-workspace

# 选择预设
# - apps: 空工作空间
# - react: React应用
# - angular: Angular应用
# - next.js: Next.js应用
# - nest: Nest.js应用

# 或者手动安装
npm install -D @nrwl/workspace
npx nx init
```

### 5.2 项目结构

```javascript
// Nx工作空间结构
my-nx-workspace/
├── apps/
│   ├── web/                    // Web应用
│   │   ├── src/
│   │   ├── project.json
│   │   └── tsconfig.json
│   └── mobile/                 // 移动应用
│       ├── src/
│       ├── project.json
│       └── tsconfig.json
├── libs/
│   ├── shared/                 // 共享库
│   │   ├── ui/                 // UI组件
│   │   ├── data-access/        // 数据访问
│   │   └── utils/              // 工具函数
│   └── feature/                // 功能模块
├── tools/                      // 自定义工具
├── nx.json                     // Nx配置
├── workspace.json              // 工作空间配置
└── package.json
```

### 5.3 生成应用和库

```bash
# 生成React应用
nx generate @nrwl/react:app web

# 生成React库
nx generate @nrwl/react:lib shared-ui

# 生成组件
nx generate @nrwl/react:component button --project=shared-ui

# 生成Node应用
nx generate @nrwl/node:app api

# 生成库（通用）
nx generate @nrwl/workspace:lib utils
```

### 5.4 依赖图分析

```bash
# 查看依赖图
nx graph

# 查看特定项目的依赖
nx graph --focus=web

# 查看受影响的项目
nx affected:graph

# 导出依赖图
nx graph --file=output.html
```

```javascript
// nx.json - 依赖配置
{
    "implicitDependencies": {
        "package.json": {
            "dependencies": "*",
            "devDependencies": "*"
        },
        ".eslintrc.json": "*",
        "tsconfig.base.json": "*"
    },
    "affected": {
        "defaultBase": "main"
    }
}
```

### 5.5 任务执行

```bash
# 运行特定项目的任务
nx run web:build
nx run web:serve
nx run web:test

# 简写形式
nx build web
nx serve web
nx test web

# 运行所有项目的任务
nx run-many --target=build --all

# 运行特定项目的任务
nx run-many --target=build --projects=web,mobile

# 只运行受影响的项目
nx affected:build
nx affected:test
nx affected:lint

# 并行执行
nx run-many --target=build --all --parallel=3
```

### 5.6 缓存配置

```javascript
// nx.json - 缓存配置
{
    "tasksRunnerOptions": {
        "default": {
            "runner": "@nrwl/workspace/tasks-runners/default",
            "options": {
                "cacheableOperations": [
                    "build",
                    "test",
                    "lint"
                ],
                "parallel": 3,
                "cacheDirectory": "node_modules/.cache/nx"
            }
        }
    },
    "targetDefaults": {
        "build": {
            "dependsOn": ["^build"],
            "inputs": [
                "production",
                "^production"
            ],
            "outputs": ["{projectRoot}/dist"]
        },
        "test": {
            "inputs": [
                "default",
                "^production",
                "{workspaceRoot}/jest.preset.js"
            ],
            "cache": true
        }
    }
}
```

### 5.7 远程缓存

```bash
# 安装Nx Cloud
npm install -D @nrwl/nx-cloud

# 连接到Nx Cloud
npx nx connect-to-nx-cloud
```

```javascript
// nx.json - Nx Cloud配置
{
    "tasksRunnerOptions": {
        "default": {
            "runner": "@nrwl/nx-cloud",
            "options": {
                "accessToken": "YOUR_ACCESS_TOKEN",
                "cacheableOperations": [
                    "build",
                    "test",
                    "lint"
                ],
                "parallel": 3
            }
        }
    }
}
```

### 5.8 自定义执行器

```javascript
// tools/executors/custom-build/executor.ts
import { ExecutorContext } from '@nrwl/devkit';

export interface CustomBuildExecutorOptions {
    outputPath: string;
    tsConfig: string;
}

export default async function customBuildExecutor(
    options: CustomBuildExecutorOptions,
    context: ExecutorContext
) {
    console.log('Building project:', context.projectName);
    console.log('Options:', options);
    
    try {
        // 自定义构建逻辑
        await buildProject(options, context);
        
        return {
            success: true
        };
    } catch (error) {
        console.error('Build failed:', error);
        
        return {
            success: false
        };
    }
}

async function buildProject(
    options: CustomBuildExecutorOptions,
    context: ExecutorContext
) {
    // 实现构建逻辑
}

// tools/executors/custom-build/schema.json
{
    "$schema": "http://json-schema.org/schema",
    "type": "object",
    "properties": {
        "outputPath": {
            "type": "string",
            "description": "Output path for build artifacts"
        },
        "tsConfig": {
            "type": "string",
            "description": "TypeScript configuration file"
        }
    },
    "required": ["outputPath", "tsConfig"]
}
```

---

## 6. pnpm Workspace实战

### 6.1 安装和初始化

```bash
# 安装pnpm
npm install -g pnpm

# 初始化项目
mkdir my-pnpm-workspace
cd my-pnpm-workspace
pnpm init
```

### 6.2 配置Workspace

```yaml
# pnpm-workspace.yaml
packages:
    - 'packages/*'
    - 'apps/*'
    - 'tools/*'
```

```javascript
// package.json
{
    "name": "my-pnpm-workspace",
    "version": "1.0.0",
    "private": true,
    "scripts": {
        "dev": "pnpm -r --parallel run dev",
        "build": "pnpm -r run build",
        "test": "pnpm -r run test",
        "lint": "pnpm -r run lint"
    }
}
```

### 6.3 项目结构

```javascript
my-pnpm-workspace/
├── apps/
│   ├── web/
│   │   ├── package.json
│   │   └── src/
│   └── admin/
│       ├── package.json
│       └── src/
├── packages/
│   ├── ui-components/
│   │   ├── package.json
│   │   └── src/
│   ├── utils/
│   │   ├── package.json
│   │   └── src/
│   └── shared/
│       ├── package.json
│       └── src/
├── pnpm-workspace.yaml
├── pnpm-lock.yaml
└── package.json
```

### 6.4 依赖管理

```bash
# 安装所有依赖
pnpm install

# 为根项目添加依赖
pnpm add -D typescript -w

# 为特定包添加依赖
pnpm add lodash --filter web

# 添加workspace依赖
pnpm add @my-workspace/utils --filter web

# 为所有包添加依赖
pnpm add -r react

# 更新依赖
pnpm update

# 删除依赖
pnpm remove lodash --filter web
```

### 6.5 执行脚本

```bash
# 在所有包中执行脚本
pnpm -r run build

# 并行执行
pnpm -r --parallel run dev

# 在特定包中执行
pnpm --filter web run build

# 使用通配符
pnpm --filter "@my-workspace/*" run test

# 按拓扑顺序执行
pnpm -r --workspace-concurrency=1 run build
```

### 6.6 配置文件

```javascript
// .npmrc
# 使用严格的peer依赖
strict-peer-dependencies=true

# 提升依赖
shamefully-hoist=false

# 公共提升模式
public-hoist-pattern[]=*eslint*
public-hoist-pattern[]=*prettier*

# 忽略workspace根目录的scripts
ignore-workspace-root-check=true

# 使用硬链接
link-workspace-packages=true
```

### 6.7 过滤器使用

```bash
# 按包名过滤
pnpm --filter web run build

# 按路径过滤
pnpm --filter "./packages/**" run test

# 按依赖关系过滤
pnpm --filter "web..." run build  # web及其所有依赖
pnpm --filter "...web" run build  # web及其所有依赖者

# 排除特定包
pnpm --filter "!web" run test

# 组合过滤
pnpm --filter "@my-workspace/*" --filter "!web" run lint
```

### 6.8 Catalog功能

```yaml
# pnpm-workspace.yaml
packages:
    - 'packages/*'

catalog:
    react: ^18.2.0
    react-dom: ^18.2.0
    typescript: ^5.0.0
    '@types/react': ^18.2.0
```

```json
// packages/web/package.json
{
    "name": "@my-workspace/web",
    "dependencies": {
        "react": "catalog:",
        "react-dom": "catalog:"
    },
    "devDependencies": {
        "typescript": "catalog:",
        "@types/react": "catalog:"
    }
}
```

---

## 7. Turborepo实战

### 7.1 安装和初始化

```bash
# 使用npx创建
npx create-turbo@latest

# 或手动安装
npm install turbo --save-dev
npx turbo init
```

### 7.2 项目结构

```javascript
my-turborepo/
├── apps/
│   ├── web/
│   │   ├── package.json
│   │   └── src/
│   └── docs/
│       ├── package.json
│       └── src/
├── packages/
│   ├── ui/
│   │   ├── package.json
│   │   └── src/
│   ├── config/
│   │   ├── eslint-config/
│   │   └── tsconfig/
│   └── utils/
│       ├── package.json
│       └── src/
├── turbo.json
└── package.json
```

### 7.3 Turbo配置

```json
// turbo.json
{
    "$schema": "https://turbo.build/schema.json",
    "globalDependencies": [
        "**/.env.*local"
    ],
    "pipeline": {
        "build": {
            "dependsOn": ["^build"],
            "outputs": [
                "dist/**",
                ".next/**",
                "!.next/cache/**"
            ]
        },
        "test": {
            "dependsOn": ["build"],
            "outputs": ["coverage/**"],
            "inputs": [
                "src/**/*.tsx",
                "src/**/*.ts",
                "test/**/*.ts",
                "test/**/*.tsx"
            ]
        },
        "lint": {
            "outputs": []
        },
        "dev": {
            "cache": false,
            "persistent": true
        }
    }
}
```

### 7.4 任务执行

```bash
# 运行构建任务
turbo run build

# 运行多个任务
turbo run build test lint

# 只运行特定包
turbo run build --filter=web

# 运行依赖的包
turbo run build --filter=web...

# 并行执行
turbo run build --parallel

# 强制重新执行（忽略缓存）
turbo run build --force

# 查看执行计划
turbo run build --dry-run
```

### 7.5 缓存配置

```json
// turbo.json
{
    "pipeline": {
        "build": {
            "outputs": ["dist/**"],
            "cache": true
        },
        "test": {
            "outputs": ["coverage/**"],
            "cache": true
        },
        "dev": {
            "cache": false
        }
    },
    "remoteCache": {
        "signature": true
    }
}
```

### 7.6 远程缓存

```bash
# 登录Vercel
npx turbo login

# 链接项目
npx turbo link

# 使用远程缓存
turbo run build --remote-cache
```

```json
// turbo.json
{
    "remoteCache": {
        "enabled": true,
        "signature": true
    }
}
```

### 7.7 环境变量

```json
// turbo.json
{
    "globalEnv": [
        "NODE_ENV"
    ],
    "pipeline": {
        "build": {
            "env": [
                "API_URL",
                "API_KEY"
            ],
            "outputs": ["dist/**"]
        }
    }
}
```

```bash
# .env
NODE_ENV=production
API_URL=https://api.example.com
API_KEY=secret
```

### 7.8 过滤器

```bash
# 按包名过滤
turbo run build --filter=web

# 按作用域过滤
turbo run build --filter=@my-workspace/*

# 包含依赖
turbo run build --filter=web...

# 包含依赖者
turbo run build --filter=...web

# 排除包
turbo run build --filter=!web

# 组合过滤
turbo run build --filter=@my-workspace/* --filter=!web
```



---

## 8. 依赖管理

### 8.1 依赖提升

```javascript
// 依赖提升策略
const hoistingStrategies = {
    // 1. 完全提升（Yarn/npm默认）
    fullHoisting: {
        description: '所有依赖提升到根node_modules',
        pros: ['减少重复安装', '节省磁盘空间'],
        cons: ['幽灵依赖问题', '依赖版本冲突']
    },
    
    // 2. 选择性提升（pnpm）
    selectiveHoisting: {
        description: '只提升指定的依赖',
        pros: ['避免幽灵依赖', '严格的依赖管理'],
        cons: ['可能增加磁盘占用']
    },
    
    // 3. 不提升
    noHoisting: {
        description: '每个包独立管理依赖',
        pros: ['完全隔离', '无版本冲突'],
        cons: ['大量重复安装', '占用空间大']
    }
};

// pnpm配置示例
// .npmrc
shamefully-hoist=false
public-hoist-pattern[]=*eslint*
public-hoist-pattern[]=*prettier*
```

### 8.2 版本管理策略

```javascript
// 1. 固定版本（Fixed/Locked）
const fixedVersioning = {
    description: '所有包使用相同版本号',
    适用场景: '紧密关联的包',
    示例: {
        'lerna.json': {
            version: '1.0.0'
        }
    }
};

// 2. 独立版本（Independent）
const independentVersioning = {
    description: '每个包独立管理版本',
    适用场景: '松散关联的包',
    示例: {
        'lerna.json': {
            version: 'independent'
        }
    }
};

// 3. 语义化版本
const semanticVersioning = {
    major: '破坏性变更',
    minor: '新功能（向后兼容）',
    patch: 'Bug修复',
    示例: {
        '1.0.0': '初始版本',
        '1.1.0': '新增功能',
        '1.1.1': '修复bug',
        '2.0.0': '破坏性变更'
    }
};
```

### 8.3 内部依赖管理

```json
// packages/web/package.json
{
    "name": "@my-workspace/web",
    "version": "1.0.0",
    "dependencies": {
        // 使用workspace协议
        "@my-workspace/ui": "workspace:*",
        "@my-workspace/utils": "workspace:^1.0.0",
        
        // 外部依赖
        "react": "^18.2.0"
    }
}

// packages/ui/package.json
{
    "name": "@my-workspace/ui",
    "version": "1.0.0",
    "peerDependencies": {
        "react": ">=16.8.0"
    }
}
```

### 8.4 依赖冲突解决

```javascript
// 1. 使用resolutions（Yarn）
// package.json
{
    "resolutions": {
        "lodash": "4.17.21",
        "**/lodash": "4.17.21"
    }
}

// 2. 使用overrides（npm 8.3+）
// package.json
{
    "overrides": {
        "lodash": "4.17.21",
        "foo": {
            "lodash": "4.17.21"
        }
    }
}

// 3. 使用pnpm.overrides（pnpm）
// package.json
{
    "pnpm": {
        "overrides": {
            "lodash": "4.17.21",
            "foo>lodash": "4.17.21"
        }
    }
}
```

### 8.5 Peer依赖处理

```json
// packages/ui/package.json
{
    "name": "@my-workspace/ui",
    "peerDependencies": {
        "react": ">=16.8.0",
        "react-dom": ">=16.8.0"
    },
    "peerDependenciesMeta": {
        "react-dom": {
            "optional": true
        }
    }
}

// .npmrc（pnpm）
auto-install-peers=true
strict-peer-dependencies=false
```

---

## 9. 构建优化

### 9.1 增量构建

```javascript
// Nx增量构建配置
// nx.json
{
    "targetDefaults": {
        "build": {
            "dependsOn": ["^build"],
            "inputs": [
                "production",
                "^production"
            ],
            "outputs": ["{projectRoot}/dist"]
        }
    }
}

// Turborepo增量构建配置
// turbo.json
{
    "pipeline": {
        "build": {
            "dependsOn": ["^build"],
            "outputs": ["dist/**"],
            "inputs": [
                "src/**/*.ts",
                "src/**/*.tsx",
                "package.json"
            ]
        }
    }
}
```

### 9.2 并行构建

```bash
# Lerna并行构建
lerna run build --parallel

# Nx并行构建
nx run-many --target=build --all --parallel=3

# pnpm并行构建
pnpm -r --parallel run build

# Turborepo并行构建
turbo run build --parallel
```

```javascript
// 控制并行数量
// nx.json
{
    "tasksRunnerOptions": {
        "default": {
            "options": {
                "parallel": 3,
                "maxParallel": 5
            }
        }
    }
}
```

### 9.3 缓存策略

```javascript
// 本地缓存配置
const localCacheConfig = {
    nx: {
        cacheDirectory: 'node_modules/.cache/nx',
        cacheableOperations: ['build', 'test', 'lint']
    },
    
    turborepo: {
        outputs: ['dist/**', '.next/**'],
        cache: true
    }
};

// 远程缓存配置
const remoteCacheConfig = {
    nx: {
        runner: '@nrwl/nx-cloud',
        accessToken: 'YOUR_TOKEN'
    },
    
    turborepo: {
        remoteCache: {
            enabled: true,
            signature: true
        }
    }
};

// 缓存失效策略
const cacheInvalidation = {
    // 基于文件内容
    contentBased: {
        inputs: ['src/**/*.ts', 'package.json'],
        outputs: ['dist/**']
    },
    
    // 基于依赖关系
    dependencyBased: {
        dependsOn: ['^build']
    },
    
    // 基于环境变量
    envBased: {
        env: ['NODE_ENV', 'API_URL']
    }
};
```

### 9.4 按需构建

```bash
# Nx - 只构建受影响的项目
nx affected:build --base=main

# Turborepo - 使用过滤器
turbo run build --filter=[main]

# Lerna - 只发布变更的包
lerna publish --since main
```

```javascript
// CI/CD中的按需构建
// .github/workflows/ci.yml
name: CI

on:
    pull_request:
        branches: [main]

jobs:
    build:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v3
              with:
                  fetch-depth: 0
            
            - name: Setup Node.js
              uses: actions/setup-node@v3
              with:
                  node-version: '18'
            
            - name: Install dependencies
              run: npm ci
            
            - name: Build affected projects
              run: npx nx affected:build --base=origin/main
            
            - name: Test affected projects
              run: npx nx affected:test --base=origin/main
```

### 9.5 构建性能监控

```javascript
// 性能分析工具
const performanceTools = {
    // Nx性能分析
    nx: {
        command: 'nx run-many --target=build --all --verbose',
        report: 'nx.json中的tasksRunnerOptions'
    },
    
    // Turborepo性能分析
    turborepo: {
        command: 'turbo run build --profile',
        report: '.turbo/runs/*.json'
    }
};

// 构建时间统计
const buildMetrics = {
    totalTime: '总构建时间',
    cacheHitRate: '缓存命中率',
    parallelEfficiency: '并行效率',
    bottlenecks: '性能瓶颈'
};
```

---

## 10. 最佳实践

### 10.1 项目结构设计

```javascript
// 推荐的项目结构
monorepo/
├── apps/                       // 应用层
│   ├── web/                    // Web应用
│   ├── mobile/                 // 移动应用
│   └── admin/                  // 管理后台
├── packages/                   // 包层
│   ├── ui/                     // UI组件库
│   │   ├── components/         // 组件
│   │   ├── hooks/              // Hooks
│   │   └── styles/             // 样式
│   ├── shared/                 // 共享代码
│   │   ├── types/              // 类型定义
│   │   ├── constants/          // 常量
│   │   └── utils/              // 工具函数
│   ├── config/                 // 配置包
│   │   ├── eslint-config/      // ESLint配置
│   │   ├── tsconfig/           // TypeScript配置
│   │   └── jest-config/        // Jest配置
│   └── features/               // 功能模块
│       ├── auth/               // 认证模块
│       ├── user/               // 用户模块
│       └── payment/            // 支付模块
├── tools/                      // 工具脚本
│   ├── generators/             // 代码生成器
│   └── scripts/                // 自动化脚本
├── docs/                       // 文档
└── .github/                    // GitHub配置
    └── workflows/              // CI/CD工作流
```

### 10.2 命名规范

```javascript
// 包命名规范
const namingConventions = {
    // 使用作用域
    scoped: '@my-workspace/package-name',
    
    // 应用命名
    apps: {
        web: '@my-workspace/web',
        mobile: '@my-workspace/mobile',
        admin: '@my-workspace/admin'
    },
    
    // 库命名
    libs: {
        ui: '@my-workspace/ui',
        utils: '@my-workspace/utils',
        shared: '@my-workspace/shared'
    },
    
    // 配置包命名
    configs: {
        eslint: '@my-workspace/eslint-config',
        typescript: '@my-workspace/tsconfig',
        jest: '@my-workspace/jest-config'
    }
};
```

### 10.3 依赖管理原则

```javascript
// 依赖管理最佳实践
const dependencyPrinciples = {
    // 1. 版本统一
    unifiedVersions: {
        rule: '相同依赖使用相同版本',
        implementation: '使用catalog或resolutions'
    },
    
    // 2. 最小化依赖
    minimalDependencies: {
        rule: '只安装必要的依赖',
        implementation: '定期审查和清理未使用的依赖'
    },
    
    // 3. 明确依赖关系
    explicitDependencies: {
        rule: '避免隐式依赖',
        implementation: '使用strict模式，明确声明所有依赖'
    },
    
    // 4. 合理使用peer依赖
    peerDependencies: {
        rule: '共享库使用peer依赖',
        implementation: '避免版本冲突'
    }
};
```

### 10.4 代码共享策略

```javascript
// 代码共享层次
const sharingLayers = {
    // 1. 基础层
    foundation: {
        packages: ['utils', 'constants', 'types'],
        description: '最底层的工具和类型',
        dependencies: []
    },
    
    // 2. 组件层
    components: {
        packages: ['ui', 'icons', 'layouts'],
        description: 'UI组件和布局',
        dependencies: ['foundation']
    },
    
    // 3. 功能层
    features: {
        packages: ['auth', 'user', 'payment'],
        description: '业务功能模块',
        dependencies: ['foundation', 'components']
    },
    
    // 4. 应用层
    applications: {
        packages: ['web', 'mobile', 'admin'],
        description: '具体应用',
        dependencies: ['foundation', 'components', 'features']
    }
};

// 依赖方向规则
const dependencyRules = {
    allowed: [
        'app -> feature',
        'app -> component',
        'app -> foundation',
        'feature -> component',
        'feature -> foundation',
        'component -> foundation'
    ],
    forbidden: [
        'foundation -> component',
        'foundation -> feature',
        'component -> feature',
        'feature -> app'
    ]
};
```

### 10.5 CI/CD配置

```yaml
# .github/workflows/ci.yml
name: CI

on:
    push:
        branches: [main, develop]
    pull_request:
        branches: [main, develop]

jobs:
    # 安装依赖
    install:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v3
            
            - name: Setup Node.js
              uses: actions/setup-node@v3
              with:
                  node-version: '18'
                  cache: 'pnpm'
            
            - name: Install pnpm
              run: npm install -g pnpm
            
            - name: Install dependencies
              run: pnpm install --frozen-lockfile
            
            - name: Cache node_modules
              uses: actions/cache@v3
              with:
                  path: node_modules
                  key: ${{ runner.os }}-node-${{ hashFiles('**/pnpm-lock.yaml') }}
    
    # 代码检查
    lint:
        needs: install
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v3
            
            - name: Restore cache
              uses: actions/cache@v3
              with:
                  path: node_modules
                  key: ${{ runner.os }}-node-${{ hashFiles('**/pnpm-lock.yaml') }}
            
            - name: Run lint
              run: pnpm run lint
    
    # 类型检查
    typecheck:
        needs: install
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v3
            
            - name: Restore cache
              uses: actions/cache@v3
              with:
                  path: node_modules
                  key: ${{ runner.os }}-node-${{ hashFiles('**/pnpm-lock.yaml') }}
            
            - name: Run typecheck
              run: pnpm run typecheck
    
    # 测试
    test:
        needs: install
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v3
              with:
                  fetch-depth: 0
            
            - name: Restore cache
              uses: actions/cache@v3
              with:
                  path: node_modules
                  key: ${{ runner.os }}-node-${{ hashFiles('**/pnpm-lock.yaml') }}
            
            - name: Run tests
              run: pnpm run test --coverage
            
            - name: Upload coverage
              uses: codecov/codecov-action@v3
    
    # 构建
    build:
        needs: [lint, typecheck, test]
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v3
              with:
                  fetch-depth: 0
            
            - name: Restore cache
              uses: actions/cache@v3
              with:
                  path: node_modules
                  key: ${{ runner.os }}-node-${{ hashFiles('**/pnpm-lock.yaml') }}
            
            - name: Build affected projects
              run: pnpm run build
            
            - name: Upload artifacts
              uses: actions/upload-artifact@v3
              with:
                  name: build-artifacts
                  path: |
                      apps/*/dist
                      packages/*/dist
```



### 10.6 版本发布流程

```javascript
// 版本发布策略
const releaseStrategy = {
    // 1. 自动化版本管理
    automated: {
        tool: 'changesets',
        workflow: [
            '开发者添加changeset',
            'CI检查changeset',
            '合并后自动创建PR',
            '审核并合并发布PR',
            '自动发布到npm'
        ]
    },
    
    // 2. 手动版本管理
    manual: {
        tool: 'lerna',
        workflow: [
            '运行lerna version',
            '选择版本类型',
            '确认变更',
            '推送标签',
            '运行lerna publish'
        ]
    }
};

// 使用changesets
// .changeset/config.json
{
    "changelog": "@changesets/cli/changelog",
    "commit": false,
    "fixed": [],
    "linked": [],
    "access": "public",
    "baseBranch": "main",
    "updateInternalDependencies": "patch",
    "ignore": []
}
```

```bash
# Changesets工作流
# 1. 添加changeset
pnpm changeset

# 2. 查看待发布的变更
pnpm changeset status

# 3. 更新版本
pnpm changeset version

# 4. 发布
pnpm changeset publish
```

### 10.7 文档管理

```javascript
// 文档结构
docs/
├── README.md                   // 项目概述
├── CONTRIBUTING.md             // 贡献指南
├── ARCHITECTURE.md             // 架构设计
├── packages/                   // 包文档
│   ├── ui.md                   // UI组件文档
│   ├── utils.md                // 工具函数文档
│   └── shared.md               // 共享代码文档
├── guides/                     // 开发指南
│   ├── getting-started.md      // 快速开始
│   ├── development.md          // 开发指南
│   ├── testing.md              // 测试指南
│   └── deployment.md           // 部署指南
└── api/                        // API文档
    └── reference.md            // API参考

// 自动生成文档
// package.json
{
    "scripts": {
        "docs:generate": "typedoc --out docs/api packages/*/src",
        "docs:serve": "docsify serve docs",
        "docs:build": "vitepress build docs"
    }
}
```

### 10.8 监控和日志

```javascript
// 构建监控
const buildMonitoring = {
    metrics: {
        buildTime: '构建时间',
        cacheHitRate: '缓存命中率',
        parallelization: '并行化程度',
        dependencyGraph: '依赖图复杂度'
    },
    
    tools: {
        nx: 'Nx Cloud Dashboard',
        turborepo: 'Turbo Dashboard',
        custom: 'Grafana + Prometheus'
    }
};

// 日志配置
// nx.json
{
    "tasksRunnerOptions": {
        "default": {
            "options": {
                "verbose": true,
                "outputStyle": "stream"
            }
        }
    }
}
```

### 10.9 常见问题解决

```javascript
// 问题1：幽灵依赖
const ghostDependencies = {
    problem: '代码使用了未在package.json中声明的依赖',
    solution: [
        '使用pnpm的严格模式',
        '定期运行依赖检查',
        '使用ESLint插件检测'
    ],
    prevention: {
        pnpm: 'shamefully-hoist=false',
        eslint: 'plugin:import/recommended'
    }
};

// 问题2：循环依赖
const circularDependencies = {
    problem: '包之间存在循环依赖',
    solution: [
        '重构代码结构',
        '提取共享代码到新包',
        '使用依赖注入'
    ],
    detection: {
        nx: 'nx graph',
        madge: 'madge --circular packages/'
    }
};

// 问题3：构建性能
const buildPerformance = {
    problem: '构建速度慢',
    solution: [
        '启用增量构建',
        '使用缓存',
        '优化依赖图',
        '并行执行任务'
    ],
    optimization: {
        cache: '本地和远程缓存',
        parallel: '合理设置并行数',
        incremental: '只构建变更的包'
    }
};

// 问题4：版本冲突
const versionConflicts = {
    problem: '不同包依赖同一库的不同版本',
    solution: [
        '使用resolutions统一版本',
        '使用peerDependencies',
        '定期更新依赖'
    ],
    tools: {
        yarn: 'resolutions',
        npm: 'overrides',
        pnpm: 'pnpm.overrides'
    }
};
```

### 10.10 迁移指南

```javascript
// 从Polyrepo迁移到Monorepo
const migrationSteps = {
    // 阶段1：准备
    preparation: [
        '评估项目依赖关系',
        '选择合适的Monorepo工具',
        '制定迁移计划',
        '准备测试环境'
    ],
    
    // 阶段2：迁移
    migration: [
        '创建Monorepo结构',
        '迁移代码到packages',
        '配置workspace',
        '更新依赖关系',
        '配置构建工具',
        '迁移CI/CD配置'
    ],
    
    // 阶段3：验证
    validation: [
        '运行所有测试',
        '验证构建流程',
        '检查依赖关系',
        '性能对比测试'
    ],
    
    // 阶段4：优化
    optimization: [
        '启用缓存',
        '优化构建流程',
        '配置增量构建',
        '设置远程缓存'
    ]
};

// 迁移脚本示例
// scripts/migrate-to-monorepo.js
const fs = require('fs');
const path = require('path');

async function migrateToMonorepo() {
    // 1. 创建packages目录
    const packagesDir = path.join(process.cwd(), 'packages');
    
    if (!fs.existsSync(packagesDir)) {
        fs.mkdirSync(packagesDir, { recursive: true });
    }
    
    // 2. 移动项目到packages
    const projects = ['web', 'mobile', 'shared'];
    
    for (const project of projects) {
        const sourcePath = path.join(process.cwd(), '..', project);
        const targetPath = path.join(packagesDir, project);
        
        if (fs.existsSync(sourcePath)) {
            fs.cpSync(sourcePath, targetPath, { recursive: true });
            console.log(`Migrated ${project} to packages/${project}`);
        }
    }
    
    // 3. 创建workspace配置
    const workspaceConfig = {
        packages: ['packages/*']
    };
    
    fs.writeFileSync(
        'pnpm-workspace.yaml',
        `packages:\n  - 'packages/*'\n`
    );
    
    // 4. 更新package.json
    const rootPackageJson = {
        name: 'my-monorepo',
        private: true,
        scripts: {
            dev: 'pnpm -r --parallel run dev',
            build: 'pnpm -r run build',
            test: 'pnpm -r run test',
            lint: 'pnpm -r run lint'
        }
    };
    
    fs.writeFileSync(
        'package.json',
        JSON.stringify(rootPackageJson, null, 4)
    );
    
    console.log('Migration completed!');
}

migrateToMonorepo().catch(console.error);
```

---

## 总结

Monorepo是一种强大的代码组织方式，特别适合管理多个相关项目。

### 关键要点

1. **选择合适的工具**
   - Lerna：成熟稳定，适合npm包管理
   - Nx：功能强大，适合大型企业项目
   - pnpm：性能优秀，节省磁盘空间
   - Turborepo：构建速度快，配置简单

2. **优化构建性能**
   - 使用增量构建
   - 启用本地和远程缓存
   - 合理配置并行执行
   - 只构建受影响的项目

3. **管理依赖关系**
   - 统一依赖版本
   - 避免幽灵依赖
   - 合理使用peer依赖
   - 定期清理未使用的依赖

4. **建立规范流程**
   - 统一的代码规范
   - 自动化的CI/CD
   - 清晰的版本管理
   - 完善的文档体系

### 最佳实践总结

```javascript
const bestPractices = {
    structure: {
        principle: '清晰的分层结构',
        implementation: 'apps + packages + tools'
    },
    
    dependencies: {
        principle: '明确的依赖关系',
        implementation: '单向依赖，避免循环'
    },
    
    versioning: {
        principle: '合理的版本策略',
        implementation: '固定版本或独立版本'
    },
    
    building: {
        principle: '高效的构建流程',
        implementation: '增量构建 + 缓存 + 并行'
    },
    
    testing: {
        principle: '完善的测试覆盖',
        implementation: '单元测试 + 集成测试 + E2E测试'
    },
    
    documentation: {
        principle: '清晰的文档',
        implementation: 'README + API文档 + 开发指南'
    }
};
```

### 学习建议

1. 从小型项目开始实践
2. 逐步引入高级特性
3. 关注构建性能优化
4. 建立团队规范和流程
5. 持续学习和改进

### 参考资源

- [Lerna官方文档](https://lerna.js.org/)
- [Nx官方文档](https://nx.dev/)
- [pnpm官方文档](https://pnpm.io/)
- [Turborepo官方文档](https://turbo.build/)
- [Monorepo.tools](https://monorepo.tools/)

---

**最后更新时间：** 2026-02-27  
**@author** erik.zhou

