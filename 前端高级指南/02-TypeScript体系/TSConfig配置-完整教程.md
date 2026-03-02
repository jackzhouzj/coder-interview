# TSConfig 配置 - 完整教程

## 目录
1. [TSConfig 基础](#tsconfig-基础)
2. [编译选项详解](#编译选项详解)
3. [模块解析配置](#模块解析配置)
4. [类型检查配置](#类型检查配置)
5. [输出配置](#输出配置)
6. [项目引用配置](#项目引用配置)
7. [常见场景配置](#常见场景配置)
8. [最佳实践](#最佳实践)

---

## TSConfig 基础

### 1.1 什么是 tsconfig.json

```json
/**
 * tsconfig.json 是 TypeScript 项目的配置文件
 * 用于指定编译选项和项目文件
 * @author erik.zhou
 */
{
    "compilerOptions": {
        "target": "ES2020",
        "module": "ESNext",
        "strict": true
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules", "dist"]
}
```

### 1.2 配置文件结构

```json
/**
 * 完整的配置文件结构
 * @author erik.zhou
 */
{
    // 编译选项
    "compilerOptions": {},
    
    // 包含的文件
    "include": [],
    
    // 排除的文件
    "exclude": [],
    
    // 指定要编译的文件
    "files": [],
    
    // 继承的配置
    "extends": "",
    
    // 项目引用
    "references": [],
    
    // 监听选项
    "watchOptions": {},
    
    // 类型获取选项
    "typeAcquisition": {}
}
```

### 1.3 配置文件继承

```json
/**
 * 基础配置文件 tsconfig.base.json
 * @author erik.zhou
 */
{
    "compilerOptions": {
        "target": "ES2020",
        "module": "ESNext",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true
    }
}
```

```json
/**
 * 开发环境配置 tsconfig.dev.json
 * @author erik.zhou
 */
{
    "extends": "./tsconfig.base.json",
    "compilerOptions": {
        "sourceMap": true,
        "incremental": true
    },
    "include": ["src/**/*"]
}
```

```json
/**
 * 生产环境配置 tsconfig.prod.json
 * @author erik.zhou
 */
{
    "extends": "./tsconfig.base.json",
    "compilerOptions": {
        "sourceMap": false,
        "removeComments": true,
        "declaration": true
    },
    "include": ["src/**/*"],
    "exclude": ["**/*.test.ts", "**/*.spec.ts"]
}
```

---

## 编译选项详解

### 2.1 目标和模块

```json
/**
 * 目标和模块配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        // 编译目标版本
        // ES3, ES5, ES6/ES2015, ES2016, ES2017, ES2018, ES2019, ES2020, ES2021, ES2022, ESNext
        "target": "ES2020",
        
        // 模块系统
        // none, commonjs, amd, system, umd, es6/es2015, es2020, es2022, esnext, node16, nodenext
        "module": "ESNext",
        
        // 指定库文件
        "lib": ["ES2020", "DOM", "DOM.Iterable"],
        
        // JSX 支持
        // preserve, react, react-native, react-jsx, react-jsxdev
        "jsx": "react-jsx",
        
        // 使用 define 定义模块
        "moduleDetection": "auto"
    }
}
```

### 2.2 严格模式选项

```json
/**
 * 严格模式配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        // 启用所有严格类型检查选项
        "strict": true,
        
        // 以下选项在 strict: true 时自动启用
        // 不允许隐式 any 类型
        "noImplicitAny": true,
        
        // 严格的 null 检查
        "strictNullChecks": true,
        
        // 严格的函数类型检查
        "strictFunctionTypes": true,
        
        // 严格的 bind/call/apply 检查
        "strictBindCallApply": true,
        
        // 严格的属性初始化检查
        "strictPropertyInitialization": true,
        
        // 不允许 this 为 any 类型
        "noImplicitThis": true,
        
        // 始终以严格模式检查
        "alwaysStrict": true
    }
}
```

### 2.3 额外检查选项

```json
/**
 * 额外检查配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        // 不允许未使用的局部变量
        "noUnusedLocals": true,
        
        // 不允许未使用的参数
        "noUnusedParameters": true,
        
        // 不允许隐式返回
        "noImplicitReturns": true,
        
        // 不允许 switch 语句贯穿
        "noFallthroughCasesInSwitch": true,
        
        // 不允许无法访问的代码
        "allowUnreachableCode": false,
        
        // 不允许未使用的标签
        "allowUnusedLabels": false,
        
        // 精确的可选属性类型
        "exactOptionalPropertyTypes": true,
        
        // 不允许索引签名的未检查访问
        "noUncheckedIndexedAccess": true,
        
        // 不允许属性覆盖
        "noPropertyAccessFromIndexSignature": false
    }
}
```

### 2.4 模块解析选项

```json
/**
 * 模块解析配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        // 模块解析策略
        // classic, node, node16, nodenext, bundler
        "moduleResolution": "bundler",
        
        // 基础路径
        "baseUrl": "./",
        
        // 路径映射
        "paths": {
            "@/*": ["src/*"],
            "@components/*": ["src/components/*"],
            "@utils/*": ["src/utils/*"],
            "@types/*": ["src/types/*"]
        },
        
        // 根目录
        "rootDirs": ["src", "generated"],
        
        // 类型根目录
        "typeRoots": ["./node_modules/@types", "./types"],
        
        // 要包含的类型包
        "types": ["node", "jest", "react"],
        
        // 允许导入 JSON 模块
        "resolveJsonModule": true,
        
        // 允许默认导入没有默认导出的模块
        "allowSyntheticDefaultImports": true,
        
        // ES 模块互操作
        "esModuleInterop": true
    }
}
```

---

## 模块解析配置

### 3.1 路径映射详解

```json
/**
 * 路径映射配置示例
 * @author erik.zhou
 */
{
    "compilerOptions": {
        "baseUrl": "./",
        "paths": {
            // 简单映射
            "@/*": ["src/*"],
            
            // 多个备选路径
            "jquery": [
                "node_modules/jquery/dist/jquery",
                "lib/jquery"
            ],
            
            // 通配符映射
            "*": [
                "node_modules/*",
                "src/types/*"
            ],
            
            // 具体模块映射
            "app/*": ["src/app/*"],
            "config/*": ["src/config/*"],
            "shared/*": ["src/shared/*"],
            "helpers/*": ["src/helpers/*"],
            "tests/*": ["tests/*"]
        }
    }
}
```

```typescript
/**
 * 使用路径映射的示例代码
 * @author erik.zhou
 */

// 使用 @ 别名
import { Button } from '@/components/Button';
import { formatDate } from '@/utils/date';
import { User } from '@/types/user';

// 使用具体模块别名
import { AppConfig } from 'config/app';
import { apiClient } from 'shared/api';
import { validateEmail } from 'helpers/validation';
```

### 3.2 Monorepo 配置

```json
/**
 * Monorepo 项目配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        "baseUrl": ".",
        "paths": {
            "@workspace/core": ["packages/core/src"],
            "@workspace/utils": ["packages/utils/src"],
            "@workspace/ui": ["packages/ui/src"],
            "@workspace/*": ["packages/*/src"]
        },
        "composite": true,
        "declaration": true,
        "declarationMap": true
    }
}
```

---

## 类型检查配置

### 4.1 类型声明文件

```json
/**
 * 类型声明配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        // 生成声明文件
        "declaration": true,
        
        // 生成声明文件的源映射
        "declarationMap": true,
        
        // 只生成声明文件
        "emitDeclarationOnly": false,
        
        // 声明文件输出目录
        "declarationDir": "./types",
        
        // 跳过库文件的类型检查
        "skipLibCheck": true,
        
        // 跳过默认库文件的类型检查
        "skipDefaultLibCheck": false
    }
}
```

### 4.2 JavaScript 支持

```json
/**
 * JavaScript 支持配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        // 允许编译 JavaScript 文件
        "allowJs": true,
        
        // 检查 JavaScript 文件
        "checkJs": false,
        
        // JavaScript 文件的最大节点数
        "maxNodeModuleJsDepth": 0
    },
    "include": ["src/**/*.ts", "src/**/*.js"]
}
```

### 4.3 实验性特性

```json
/**
 * 实验性特性配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        // 启用装饰器
        "experimentalDecorators": true,
        
        // 装饰器元数据
        "emitDecoratorMetadata": true,
        
        // 使用 define 语义定义类字段
        "useDefineForClassFields": true
    }
}
```

---

## 输出配置

### 5.1 输出目录和文件

```json
/**
 * 输出配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        // 输出目录
        "outDir": "./dist",
        
        // 根目录
        "rootDir": "./src",
        
        // 输出单个文件
        "outFile": "./dist/bundle.js",
        
        // 移除注释
        "removeComments": true,
        
        // 不生成输出文件
        "noEmit": false,
        
        // 导入辅助函数
        "importHelpers": true,
        
        // 降级迭代器
        "downlevelIteration": true,
        
        // 生成源映射
        "sourceMap": true,
        
        // 内联源映射
        "inlineSourceMap": false,
        
        // 内联源代码
        "inlineSources": false,
        
        // 源映射根路径
        "sourceRoot": "",
        
        // 映射根路径
        "mapRoot": ""
    }
}
```

### 5.2 增量编译

```json
/**
 * 增量编译配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        // 启用增量编译
        "incremental": true,
        
        // 增量编译信息文件
        "tsBuildInfoFile": "./.tsbuildinfo",
        
        // 复合项目
        "composite": false
    }
}
```

### 5.3 输出格式化

```json
/**
 * 输出格式化配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        // 新行字符
        "newLine": "lf",
        
        // 保留常量枚举
        "preserveConstEnums": false,
        
        // 保留值导入
        "preserveValueImports": false,
        
        // 保留监视输出
        "preserveWatchOutput": false,
        
        // 漂亮打印
        "pretty": true
    }
}
```

---

## 项目引用配置

### 6.1 基础项目引用

```json
/**
 * 主项目配置 tsconfig.json
 * @author erik.zhou
 */
{
    "files": [],
    "references": [
        { "path": "./packages/core" },
        { "path": "./packages/utils" },
        { "path": "./packages/ui" }
    ]
}
```

```json
/**
 * 子项目配置 packages/core/tsconfig.json
 * @author erik.zhou
 */
{
    "compilerOptions": {
        "composite": true,
        "declaration": true,
        "declarationMap": true,
        "rootDir": "./src",
        "outDir": "./dist"
    },
    "include": ["src/**/*"]
}
```

### 6.2 项目引用依赖

```json
/**
 * UI 包配置 packages/ui/tsconfig.json
 * 依赖 core 和 utils 包
 * @author erik.zhou
 */
{
    "compilerOptions": {
        "composite": true,
        "declaration": true,
        "rootDir": "./src",
        "outDir": "./dist"
    },
    "references": [
        { "path": "../core" },
        { "path": "../utils" }
    ],
    "include": ["src/**/*"]
}
```

---

## 常见场景配置

### 7.1 React 项目配置

```json
/**
 * React 项目 TypeScript 配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        "target": "ES2020",
        "lib": ["ES2020", "DOM", "DOM.Iterable"],
        "module": "ESNext",
        "moduleResolution": "bundler",
        "jsx": "react-jsx",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true,
        "resolveJsonModule": true,
        "isolatedModules": true,
        "noEmit": true,
        "baseUrl": "./",
        "paths": {
            "@/*": ["src/*"],
            "@components/*": ["src/components/*"],
            "@hooks/*": ["src/hooks/*"],
            "@utils/*": ["src/utils/*"],
            "@types/*": ["src/types/*"]
        }
    },
    "include": ["src"],
    "exclude": ["node_modules", "dist", "build"]
}
```

### 7.2 Vue 项目配置

```json
/**
 * Vue 3 项目 TypeScript 配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        "target": "ES2020",
        "module": "ESNext",
        "lib": ["ES2020", "DOM", "DOM.Iterable"],
        "moduleResolution": "bundler",
        "jsx": "preserve",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "resolveJsonModule": true,
        "isolatedModules": true,
        "baseUrl": "./",
        "paths": {
            "@/*": ["src/*"]
        },
        "types": ["vite/client"]
    },
    "include": [
        "src/**/*.ts",
        "src/**/*.d.ts",
        "src/**/*.tsx",
        "src/**/*.vue"
    ],
    "exclude": ["node_modules", "dist"]
}
```

### 7.3 Node.js 项目配置

```json
/**
 * Node.js 项目 TypeScript 配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        "target": "ES2020",
        "module": "commonjs",
        "lib": ["ES2020"],
        "outDir": "./dist",
        "rootDir": "./src",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true,
        "resolveJsonModule": true,
        "moduleResolution": "node",
        "declaration": true,
        "declarationMap": true,
        "sourceMap": true,
        "types": ["node"],
        "baseUrl": "./",
        "paths": {
            "@/*": ["src/*"]
        }
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### 7.4 库项目配置

```json
/**
 * 库项目 TypeScript 配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        "target": "ES2015",
        "module": "ESNext",
        "lib": ["ES2015"],
        "declaration": true,
        "declarationMap": true,
        "outDir": "./dist",
        "rootDir": "./src",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true,
        "moduleResolution": "node",
        "removeComments": true,
        "sourceMap": true
    },
    "include": ["src/**/*"],
    "exclude": [
        "node_modules",
        "dist",
        "**/*.test.ts",
        "**/*.spec.ts",
        "examples"
    ]
}
```

### 7.5 测试配置

```json
/**
 * Jest 测试配置 tsconfig.test.json
 * @author erik.zhou
 */
{
    "extends": "./tsconfig.json",
    "compilerOptions": {
        "types": ["jest", "node"],
        "esModuleInterop": true
    },
    "include": [
        "src/**/*.test.ts",
        "src/**/*.spec.ts",
        "tests/**/*"
    ]
}
```

### 7.6 Next.js 项目配置

```json
/**
 * Next.js 项目 TypeScript 配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        "target": "ES2017",
        "lib": ["dom", "dom.iterable", "esnext"],
        "allowJs": true,
        "skipLibCheck": true,
        "strict": true,
        "forceConsistentCasingInFileNames": true,
        "noEmit": true,
        "esModuleInterop": true,
        "module": "esnext",
        "moduleResolution": "bundler",
        "resolveJsonModule": true,
        "isolatedModules": true,
        "jsx": "preserve",
        "incremental": true,
        "plugins": [
            {
                "name": "next"
            }
        ],
        "paths": {
            "@/*": ["./*"]
        }
    },
    "include": [
        "next-env.d.ts",
        "**/*.ts",
        "**/*.tsx",
        ".next/types/**/*.ts"
    ],
    "exclude": ["node_modules"]
}
```

---

## 最佳实践

### 8.1 推荐的基础配置

```json
/**
 * 推荐的基础配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        // 目标和模块
        "target": "ES2020",
        "module": "ESNext",
        "lib": ["ES2020", "DOM"],
        
        // 严格模式
        "strict": true,
        "noUnusedLocals": true,
        "noUnusedParameters": true,
        "noImplicitReturns": true,
        "noFallthroughCasesInSwitch": true,
        
        // 模块解析
        "moduleResolution": "bundler",
        "esModuleInterop": true,
        "allowSyntheticDefaultImports": true,
        "resolveJsonModule": true,
        
        // 其他
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true,
        "isolatedModules": true
    }
}
```

### 8.2 性能优化配置

```json
/**
 * 性能优化配置
 * @author erik.zhou
 */
{
    "compilerOptions": {
        // 启用增量编译
        "incremental": true,
        
        // 跳过库文件检查
        "skipLibCheck": true,
        
        // 不检查默认库
        "skipDefaultLibCheck": true,
        
        // 假设导入总是有副作用
        "assumeChangesOnlyAffectDirectDependencies": true
    },
    "exclude": [
        "node_modules",
        "**/*.spec.ts",
        "**/*.test.ts"
    ]
}
```

### 8.3 多环境配置

```json
/**
 * 基础配置 tsconfig.base.json
 * @author erik.zhou
 */
{
    "compilerOptions": {
        "target": "ES2020",
        "module": "ESNext",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true,
        "baseUrl": "./",
        "paths": {
            "@/*": ["src/*"]
        }
    }
}
```

```json
/**
 * 开发环境 tsconfig.json
 * @author erik.zhou
 */
{
    "extends": "./tsconfig.base.json",
    "compilerOptions": {
        "sourceMap": true,
        "incremental": true,
        "noEmit": true
    },
    "include": ["src/**/*", "tests/**/*"]
}
```

```json
/**
 * 生产构建 tsconfig.build.json
 * @author erik.zhou
 */
{
    "extends": "./tsconfig.base.json",
    "compilerOptions": {
        "outDir": "./dist",
        "declaration": true,
        "declarationMap": true,
        "sourceMap": false,
        "removeComments": true
    },
    "include": ["src/**/*"],
    "exclude": ["**/*.test.ts", "**/*.spec.ts"]
}
```

### 8.4 常见问题解决

```typescript
/**
 * 常见配置问题及解决方案
 * @author erik.zhou
 */

// 问题1：无法解析模块
// 解决方案：配置 paths 和 baseUrl
{
    "compilerOptions": {
        "baseUrl": "./",
        "paths": {
            "@/*": ["src/*"]
        }
    }
}

// 问题2：装饰器报错
// 解决方案：启用装饰器支持
{
    "compilerOptions": {
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true
    }
}

// 问题3：JSX 语法错误
// 解决方案：配置正确的 JSX 选项
{
    "compilerOptions": {
        "jsx": "react-jsx" // React 17+
        // 或 "jsx": "react" // React 16
    }
}

// 问题4：类型声明找不到
// 解决方案：配置 types 和 typeRoots
{
    "compilerOptions": {
        "typeRoots": ["./node_modules/@types", "./types"],
        "types": ["node", "jest"]
    }
}

// 问题5：编译速度慢
// 解决方案：启用增量编译和跳过库检查
{
    "compilerOptions": {
        "incremental": true,
        "skipLibCheck": true
    }
}
```

### 8.5 配置检查清单

```typescript
/**
 * TSConfig 配置检查清单
 * @author erik.zhou
 */

// ✅ 必须配置项
const mustHave = {
    target: 'ES2020',           // 编译目标
    module: 'ESNext',           // 模块系统
    strict: true,               // 严格模式
    esModuleInterop: true,      // ES 模块互操作
    skipLibCheck: true,         // 跳过库检查
    forceConsistentCasingInFileNames: true  // 强制文件名大小写一致
};

// ✅ 推荐配置项
const recommended = {
    noUnusedLocals: true,       // 检查未使用的局部变量
    noUnusedParameters: true,   // 检查未使用的参数
    noImplicitReturns: true,    // 检查隐式返回
    noFallthroughCasesInSwitch: true,  // 检查 switch 贯穿
    resolveJsonModule: true,    // 解析 JSON 模块
    isolatedModules: true       // 隔离模块
};

// ✅ 项目特定配置
const projectSpecific = {
    baseUrl: './',              // 基础路径
    paths: {},                  // 路径映射
    jsx: 'react-jsx',          // JSX 配置（React 项目）
    types: [],                  // 类型包
    lib: []                     // 库文件
};

// ✅ 性能优化配置
const performance = {
    incremental: true,          // 增量编译
    tsBuildInfoFile: './.tsbuildinfo'  // 构建信息文件
};
```

---

## 总结

本教程详细介绍了 TSConfig 的各项配置：

1. TSConfig 基础：配置文件结构、继承机制
2. 编译选项：目标版本、模块系统、严格模式
3. 模块解析：路径映射、Monorepo 配置
4. 类型检查：声明文件、JavaScript 支持
5. 输出配置：输出目录、增量编译、格式化
6. 项目引用：多项目管理、依赖配置
7. 常见场景：React、Vue、Node.js、库项目
8. 最佳实践：推荐配置、性能优化、问题解决

通过合理配置 TSConfig，可以提升开发体验、编译性能和代码质量。

---

**最后更新时间：** 2026-02-26  
**@author erik.zhou**
