# Rollup - 完整教程

## 目录
1. [Rollup 简介](#rollup-简介)
2. [快速开始](#快速开始)
3. [配置文件详解](#配置文件详解)
4. [插件系统](#插件系统)
5. [代码分割](#代码分割)
6. [Tree Shaking](#tree-shaking)
7. [插件开发](#插件开发)
8. [实战案例](#实战案例)

---

## Rollup 简介

### 1.1 什么是 Rollup

Rollup 是一个 JavaScript 模块打包器，专注于 ES 模块，特别适合构建库和工具。

```javascript
/**
 * Rollup 的核心特点
 * @author erik.zhou
 */

// 1. ES 模块优先
// Rollup 原生支持 ES6 模块语法
import { sum } from './math.js';

// 2. Tree Shaking
// 自动移除未使用的代码
export function add(a, b) {
    return a + b;
}

export function subtract(a, b) {
    return a - b;
}

// 如果只导入 add，subtract 会被移除

// 3. 输出格式多样
// 支持 ES、CJS、UMD、IIFE 等多种格式
```

### 1.2 Rollup vs Webpack

```javascript
/**
 * Rollup 与 Webpack 的对比
 * @author erik.zhou
 */

// Rollup 适用场景：
const rollupUseCases = {
    libraries: '构建 JavaScript 库',
    frameworks: '构建框架和工具',
    simpleApps: '简单的应用程序',
    nodeModules: 'Node.js 模块'
};

// Webpack 适用场景：
const webpackUseCases = {
    complexApps: '复杂的 Web 应用',
    spa: '单页应用（SPA）',
    assets: '需要处理多种资源类型',
    hmr: '需要热模块替换（HMR）'
};

// Rollup 优势：
const rollupAdvantages = [
    '更小的打包体积',
    '更快的构建速度',
    '更好的 Tree Shaking',
    '输出代码更清晰'
];
```

---

## 快速开始

### 2.1 安装 Rollup

```bash
# 安装 Rollup
npm install --save-dev rollup

# 或使用 yarn
yarn add --dev rollup

# 或使用 pnpm
pnpm add -D rollup
```

### 2.2 基础使用

```javascript
/**
 * 基础打包示例
 * @author erik.zhou
 */

// src/main.js
import { version } from './version.js';

export function greet(name) {
    console.log(`Hello ${name}! Version: ${version}`);
}

// src/version.js
export const version = '1.0.0';
```

```bash
# 命令行打包
rollup src/main.js --file dist/bundle.js --format es

# 指定输出格式
rollup src/main.js --file dist/bundle.cjs.js --format cjs
rollup src/main.js --file dist/bundle.umd.js --format umd --name MyLibrary
```

### 2.3 使用配置文件

```javascript
/**
 * rollup.config.js - 基础配置
 * @author erik.zhou
 */

export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        format: 'es'
    }
};
```

```bash
# 使用配置文件打包
rollup -c

# 指定配置文件
rollup -c rollup.config.prod.js

# 监听模式
rollup -c -w
```

---

## 配置文件详解

### 3.1 输入配置

```javascript
/**
 * 输入配置选项
 * @author erik.zhou
 */

export default {
    // 单入口
    input: 'src/main.js',
    
    // 多入口
    input: {
        main: 'src/main.js',
        utils: 'src/utils.js'
    },
    
    // 数组形式（生成多个独立的包）
    input: ['src/main.js', 'src/utils.js'],
    
    // 外部依赖（不打包进输出文件）
    external: ['lodash', 'react'],
    
    // 外部依赖（使用函数）
    external: (id) => {
        return /node_modules/.test(id);
    },
    
    // 插件
    plugins: []
};
```

### 3.2 输出配置

```javascript
/**
 * 输出配置选项
 * @author erik.zhou
 */

export default {
    input: 'src/main.js',
    output: {
        // 输出文件路径
        file: 'dist/bundle.js',
        
        // 输出格式
        // es, cjs, amd, umd, iife, system
        format: 'es',
        
        // 全局变量名（umd/iife 格式需要）
        name: 'MyLibrary',
        
        // 源映射
        sourcemap: true,
        
        // 内联源映射
        sourcemap: 'inline',
        
        // 全局变量映射（umd 格式）
        globals: {
            'lodash': '_',
            'react': 'React'
        },
        
        // 输出目录（多入口时使用）
        dir: 'dist',
        
        // 入口文件名模式
        entryFileNames: '[name].js',
        
        // 代码分割文件名模式
        chunkFileNames: '[name]-[hash].js',
        
        // 资源文件名模式
        assetFileNames: 'assets/[name]-[hash][extname]',
        
        // 导出模式
        exports: 'auto', // auto, default, named, none
        
        // 严格模式
        strict: true,
        
        // 压缩输出
        compact: false,
        
        // 保留模块结构
        preserveModules: false,
        
        // 保留模块根目录
        preserveModulesRoot: 'src'
    }
};
```

### 3.3 多输出配置

```javascript
/**
 * 多输出格式配置
 * @author erik.zhou
 */

export default {
    input: 'src/main.js',
    output: [
        // ES 模块
        {
            file: 'dist/bundle.esm.js',
            format: 'es',
            sourcemap: true
        },
        // CommonJS
        {
            file: 'dist/bundle.cjs.js',
            format: 'cjs',
            sourcemap: true
        },
        // UMD
        {
            file: 'dist/bundle.umd.js',
            format: 'umd',
            name: 'MyLibrary',
            sourcemap: true,
            globals: {
                'react': 'React'
            }
        },
        // IIFE（浏览器直接使用）
        {
            file: 'dist/bundle.iife.js',
            format: 'iife',
            name: 'MyLibrary',
            sourcemap: true
        }
    ],
    external: ['react']
};
```

### 3.4 高级配置

```javascript
/**
 * 高级配置选项
 * @author erik.zhou
 */

export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        format: 'es'
    },
    
    // 监听选项
    watch: {
        include: 'src/**',
        exclude: 'node_modules/**',
        clearScreen: false,
        buildDelay: 100
    },
    
    // 性能警告阈值
    perf: true,
    
    // 缓存
    cache: true,
    
    // 上下文（this 的值）
    context: 'window',
    
    // 模块上下文
    moduleContext: {
        'src/some-module.js': 'window'
    },
    
    // 保留符号链接
    preserveSymlinks: false,
    
    // 最大并行文件读取数
    maxParallelFileReads: 20,
    
    // 实验性功能
    experimentalCacheExpiry: 10,
    
    // 树摇优化
    treeshake: {
        moduleSideEffects: true,
        propertyReadSideEffects: true,
        tryCatchDeoptimization: true,
        unknownGlobalSideEffects: true
    }
};
```

---

## 插件系统

### 4.1 常用插件

```javascript
/**
 * Rollup 常用插件配置
 * @author erik.zhou
 */

import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';
import typescript from '@rollup/plugin-typescript';
import json from '@rollup/plugin-json';
import terser from '@rollup/plugin-terser';
import replace from '@rollup/plugin-replace';
import alias from '@rollup/plugin-alias';

export default {
    input: 'src/main.ts',
    output: {
        file: 'dist/bundle.js',
        format: 'es',
        sourcemap: true
    },
    plugins: [
        // 解析 node_modules 中的模块
        resolve({
            extensions: ['.js', '.ts', '.json'],
            browser: true,
            preferBuiltins: false
        }),
        
        // 转换 CommonJS 模块为 ES6
        commonjs({
            include: 'node_modules/**',
            extensions: ['.js', '.cjs']
        }),
        
        // TypeScript 支持
        typescript({
            tsconfig: './tsconfig.json',
            declaration: true,
            declarationDir: 'dist/types'
        }),
        
        // Babel 转译
        babel({
            babelHelpers: 'bundled',
            exclude: 'node_modules/**',
            extensions: ['.js', '.ts', '.tsx']
        }),
        
        // JSON 文件支持
        json(),
        
        // 环境变量替换
        replace({
            'process.env.NODE_ENV': JSON.stringify('production'),
            preventAssignment: true
        }),
        
        // 路径别名
        alias({
            entries: [
                { find: '@', replacement: './src' },
                { find: '@utils', replacement: './src/utils' }
            ]
        }),
        
        // 代码压缩
        terser({
            compress: {
                drop_console: true,
                drop_debugger: true
            },
            format: {
                comments: false
            }
        })
    ]
};
```

### 4.2 插件详解

```javascript
/**
 * @rollup/plugin-node-resolve 详解
 * @author erik.zhou
 */

import resolve from '@rollup/plugin-node-resolve';

export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        format: 'es'
    },
    plugins: [
        resolve({
            // 浏览器环境
            browser: true,
            
            // 优先使用内置模块
            preferBuiltins: false,
            
            // 主字段
            mainFields: ['module', 'main', 'browser'],
            
            // 扩展名
            extensions: ['.mjs', '.js', '.json', '.node'],
            
            // 导出条件
            exportConditions: ['import', 'module', 'default'],
            
            // 模块目录
            moduleDirectories: ['node_modules'],
            
            // 仅解析指定的依赖
            resolveOnly: ['lodash-es']
        })
    ]
};
```

```javascript
/**
 * @rollup/plugin-commonjs 详解
 * @author erik.zhou
 */

import commonjs from '@rollup/plugin-commonjs';

export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        format: 'es'
    },
    plugins: [
        commonjs({
            // 包含的文件
            include: 'node_modules/**',
            
            // 排除的文件
            exclude: [],
            
            // 扩展名
            extensions: ['.js', '.cjs'],
            
            // 忽略动态 require
            ignoreDynamicRequires: false,
            
            // 忽略全局变量
            ignoreGlobal: false,
            
            // 源映射
            sourceMap: true,
            
            // 转换混合模块
            transformMixedEsModules: false,
            
            // 默认导出模式
            defaultIsModuleExports: 'auto',
            
            // 命名导出
            namedExports: {
                'react': ['createElement', 'Component']
            }
        })
    ]
};
```

```javascript
/**
 * @rollup/plugin-babel 详解
 * @author erik.zhou
 */

import babel from '@rollup/plugin-babel';

export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        format: 'es'
    },
    plugins: [
        babel({
            // Babel 辅助函数处理方式
            // bundled, runtime, inline, external
            babelHelpers: 'bundled',
            
            // 排除的文件
            exclude: 'node_modules/**',
            
            // 包含的文件
            include: ['src/**/*.js'],
            
            // 扩展名
            extensions: ['.js', '.jsx', '.ts', '.tsx'],
            
            // Babel 配置
            presets: [
                ['@babel/preset-env', {
                    targets: '> 0.25%, not dead',
                    modules: false
                }],
                '@babel/preset-react',
                '@babel/preset-typescript'
            ],
            
            plugins: [
                '@babel/plugin-proposal-class-properties',
                '@babel/plugin-proposal-object-rest-spread'
            ],
            
            // 跳过 Babel 配置文件
            babelrc: false,
            configFile: false
        })
    ]
};
```

### 4.3 CSS 处理插件

```javascript
/**
 * CSS 处理插件配置
 * @author erik.zhou
 */

import postcss from 'rollup-plugin-postcss';
import autoprefixer from 'autoprefixer';
import cssnano from 'cssnano';

export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        format: 'es'
    },
    plugins: [
        postcss({
            // 提取 CSS 到单独文件
            extract: true,
            
            // 输出文件名
            extract: 'styles.css',
            
            // 压缩 CSS
            minimize: true,
            
            // 源映射
            sourceMap: true,
            
            // PostCSS 插件
            plugins: [
                autoprefixer(),
                cssnano({
                    preset: 'default'
                })
            ],
            
            // CSS 模块
            modules: {
                generateScopedName: '[name]__[local]___[hash:base64:5]'
            },
            
            // 预处理器
            use: [
                ['sass', {
                    includePaths: ['./src/styles']
                }],
                ['less', {
                    javascriptEnabled: true
                }]
            ],
            
            // 注入到 head
            inject: false,
            
            // 扩展名
            extensions: ['.css', '.scss', '.sass', '.less']
        })
    ]
};
```

### 4.4 资源处理插件

```javascript
/**
 * 资源处理插件配置
 * @author erik.zhou
 */

import image from '@rollup/plugin-image';
import url from '@rollup/plugin-url';
import svg from 'rollup-plugin-svg';

export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        format: 'es'
    },
    plugins: [
        // 图片处理
        image({
            // 输出目录
            output: 'dist/images',
            
            // 文件名模式
            fileName: '[name][extname]',
            
            // 包含的扩展名
            include: ['**/*.png', '**/*.jpg', '**/*.gif']
        }),
        
        // URL 处理
        url({
            // 文件大小限制（字节）
            limit: 10 * 1024, // 10KB
            
            // 包含的文件
            include: ['**/*.svg', '**/*.png', '**/*.jpg'],
            
            // 输出目录
            destDir: 'dist/assets',
            
            // 文件名
            fileName: '[name][extname]',
            
            // 公共路径
            publicPath: '/assets/'
        }),
        
        // SVG 处理
        svg({
            base64: false
        })
    ]
};
```

---

## 代码分割

### 5.1 动态导入

```javascript
/**
 * 动态导入实现代码分割
 * @author erik.zhou
 */

// src/main.js
export async function loadModule() {
    // 动态导入会自动进行代码分割
    const module = await import('./heavy-module.js');
    return module.default();
}

// src/heavy-module.js
export default function heavyFunction() {
    console.log('This is a heavy module');
    return 'Heavy result';
}
```

```javascript
/**
 * 代码分割配置
 * @author erik.zhou
 */

export default {
    input: 'src/main.js',
    output: {
        dir: 'dist',
        format: 'es',
        
        // 代码分割文件名
        chunkFileNames: 'chunks/[name]-[hash].js',
        
        // 手动分割
        manualChunks: {
            'vendor': ['react', 'react-dom'],
            'utils': ['lodash-es']
        },
        
        // 或使用函数
        manualChunks(id) {
            if (id.includes('node_modules')) {
                return 'vendor';
            }
            if (id.includes('src/utils')) {
                return 'utils';
            }
        }
    }
};
```

### 5.2 多入口代码分割

```javascript
/**
 * 多入口代码分割
 * @author erik.zhou
 */

export default {
    input: {
        main: 'src/main.js',
        admin: 'src/admin.js',
        user: 'src/user.js'
    },
    output: {
        dir: 'dist',
        format: 'es',
        entryFileNames: '[name].js',
        chunkFileNames: 'chunks/[name]-[hash].js'
    },
    plugins: []
};
```

---

## Tree Shaking

### 6.1 Tree Shaking 原理

```javascript
/**
 * Tree Shaking 示例
 * @author erik.zhou
 */

// utils.js - 导出多个函数
export function add(a, b) {
    return a + b;
}

export function subtract(a, b) {
    return a - b;
}

export function multiply(a, b) {
    return a * b;
}

export function divide(a, b) {
    return a / b;
}

// main.js - 只使用部分函数
import { add, multiply } from './utils.js';

console.log(add(1, 2));
console.log(multiply(3, 4));

// 打包后，subtract 和 divide 会被移除
```

### 6.2 Tree Shaking 配置

```javascript
/**
 * Tree Shaking 配置优化
 * @author erik.zhou
 */

export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        format: 'es'
    },
    treeshake: {
        // 模块副作用
        moduleSideEffects: true,
        
        // 属性读取副作用
        propertyReadSideEffects: true,
        
        // try-catch 去优化
        tryCatchDeoptimization: true,
        
        // 未知全局副作用
        unknownGlobalSideEffects: true,
        
        // 注解
        annotations: true,
        
        // 预设
        preset: 'smallest' // smallest, safest, recommended
    }
};
```

### 6.3 副作用标记

```json
/**
 * package.json 中标记副作用
 * @author erik.zhou
 */
{
    "name": "my-library",
    "version": "1.0.0",
    "sideEffects": false,
    
    // 或指定有副作用的文件
    "sideEffects": [
        "*.css",
        "*.scss",
        "src/polyfills.js"
    ]
}
```

---

## 插件开发

### 7.1 基础插件结构

```javascript
/**
 * Rollup 插件基础结构
 * @author erik.zhou
 */

function myPlugin(options = {}) {
    return {
        name: 'my-plugin',
        
        // 构建开始
        buildStart(options) {
            console.log('Build started');
        },
        
        // 解析模块 ID
        resolveId(source, importer) {
            if (source === 'virtual-module') {
                return source;
            }
            return null;
        },
        
        // 加载模块
        load(id) {
            if (id === 'virtual-module') {
                return 'export default "This is virtual!"';
            }
            return null;
        },
        
        // 转换代码
        transform(code, id) {
            if (id.endsWith('.txt')) {
                return {
                    code: `export default ${JSON.stringify(code)}`,
                    map: null
                };
            }
            return null;
        },
        
        // 生成 bundle
        generateBundle(options, bundle) {
            console.log('Bundle generated');
        },
        
        // 构建结束
        buildEnd(error) {
            if (error) {
                console.error('Build failed:', error);
            } else {
                console.log('Build completed');
            }
        }
    };
}

export default myPlugin;
```

### 7.2 实用插件示例

```javascript
/**
 * 自定义插件：添加版权信息
 * @author erik.zhou
 */

function copyrightPlugin(options = {}) {
    const { author = 'Unknown', year = new Date().getFullYear() } = options;
    
    return {
        name: 'copyright-plugin',
        
        renderChunk(code, chunk, options) {
            const banner = `/**
 * @author ${author}
 * @copyright ${year}
 * @license MIT
 */\n\n`;
            
            return {
                code: banner + code,
                map: null
            };
        }
    };
}

// 使用插件
export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        format: 'es'
    },
    plugins: [
        copyrightPlugin({
            author: 'erik.zhou',
            year: 2026
        })
    ]
};
```

```javascript
/**
 * 自定义插件：环境变量注入
 * @author erik.zhou
 */

function envPlugin(env = {}) {
    return {
        name: 'env-plugin',
        
        transform(code, id) {
            // 替换环境变量
            let transformedCode = code;
            
            Object.keys(env).forEach((key) => {
                const regex = new RegExp(`process\\.env\\.${key}`, 'g');
                transformedCode = transformedCode.replace(
                    regex,
                    JSON.stringify(env[key])
                );
            });
            
            return {
                code: transformedCode,
                map: null
            };
        }
    };
}

// 使用插件
export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        format: 'es'
    },
    plugins: [
        envPlugin({
            API_URL: 'https://api.example.com',
            VERSION: '1.0.0'
        })
    ]
};
```

---

## 实战案例

### 8.1 构建 JavaScript 库

```javascript
/**
 * JavaScript 库完整配置
 * @author erik.zhou
 */

import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';
import terser from '@rollup/plugin-terser';
import { dts } from 'rollup-plugin-dts';

const packageJson = require('./package.json');

export default [
    // 主构建配置
    {
        input: 'src/index.js',
        output: [
            // ES 模块
            {
                file: packageJson.module,
                format: 'es',
                sourcemap: true
            },
            // CommonJS
            {
                file: packageJson.main,
                format: 'cjs',
                sourcemap: true,
                exports: 'named'
            },
            // UMD（浏览器使用）
            {
                file: packageJson.browser,
                format: 'umd',
                name: 'MyLibrary',
                sourcemap: true,
                globals: {
                    'react': 'React'
                }
            }
        ],
        external: ['react', 'react-dom'],
        plugins: [
            resolve(),
            commonjs(),
            babel({
                babelHelpers: 'bundled',
                exclude: 'node_modules/**',
                presets: [
                    ['@babel/preset-env', {
                        targets: '> 0.25%, not dead'
                    }]
                ]
            }),
            terser()
        ]
    },
    // 类型声明文件
    {
        input: 'src/index.d.ts',
        output: {
            file: 'dist/index.d.ts',
            format: 'es'
        },
        plugins: [dts()]
    }
];
```

### 8.2 构建 React 组件库

```javascript
/**
 * React 组件库配置
 * @author erik.zhou
 */

import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';
import typescript from '@rollup/plugin-typescript';
import postcss from 'rollup-plugin-postcss';
import peerDepsExternal from 'rollup-plugin-peer-deps-external';
import { terser } from 'rollup-plugin-terser';

export default {
    input: 'src/index.ts',
    output: [
        {
            file: 'dist/index.js',
            format: 'cjs',
            sourcemap: true,
            exports: 'named'
        },
        {
            file: 'dist/index.esm.js',
            format: 'es',
            sourcemap: true
        }
    ],
    plugins: [
        // 自动处理 peerDependencies
        peerDepsExternal(),
        
        resolve({
            extensions: ['.js', '.jsx', '.ts', '.tsx']
        }),
        
        commonjs(),
        
        typescript({
            tsconfig: './tsconfig.json',
            declaration: true,
            declarationDir: 'dist/types'
        }),
        
        babel({
            babelHelpers: 'runtime',
            exclude: 'node_modules/**',
            extensions: ['.js', '.jsx', '.ts', '.tsx'],
            presets: [
                '@babel/preset-env',
                '@babel/preset-react',
                '@babel/preset-typescript'
            ],
            plugins: [
                '@babel/plugin-transform-runtime'
            ]
        }),
        
        postcss({
            extract: true,
            modules: true,
            minimize: true
        }),
        
        terser()
    ],
    external: ['react', 'react-dom']
};
```

### 8.3 Monorepo 配置

```javascript
/**
 * Monorepo 项目配置
 * @author erik.zhou
 */

// packages/core/rollup.config.js
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import typescript from '@rollup/plugin-typescript';

export default {
    input: 'src/index.ts',
    output: [
        {
            file: 'dist/index.js',
            format: 'cjs',
            sourcemap: true
        },
        {
            file: 'dist/index.esm.js',
            format: 'es',
            sourcemap: true
        }
    ],
    plugins: [
        resolve(),
        commonjs(),
        typescript({
            tsconfig: './tsconfig.json'
        })
    ]
};

// packages/utils/rollup.config.js
export default {
    input: 'src/index.ts',
    output: [
        {
            file: 'dist/index.js',
            format: 'cjs',
            sourcemap: true
        },
        {
            file: 'dist/index.esm.js',
            format: 'es',
            sourcemap: true
        }
    ],
    // 引用 core 包
    external: ['@mylib/core'],
    plugins: [
        resolve(),
        commonjs(),
        typescript()
    ]
};
```

### 8.4 完整的生产配置

```javascript
/**
 * 生产环境完整配置
 * @author erik.zhou
 */

import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import babel from '@rollup/plugin-babel';
import typescript from '@rollup/plugin-typescript';
import terser from '@rollup/plugin-terser';
import replace from '@rollup/plugin-replace';
import { visualizer } from 'rollup-plugin-visualizer';
import filesize from 'rollup-plugin-filesize';
import progress from 'rollup-plugin-progress';
import cleanup from 'rollup-plugin-cleanup';

const production = process.env.NODE_ENV === 'production';

export default {
    input: 'src/index.ts',
    output: [
        {
            file: 'dist/index.js',
            format: 'cjs',
            sourcemap: !production,
            exports: 'named'
        },
        {
            file: 'dist/index.esm.js',
            format: 'es',
            sourcemap: !production
        },
        production && {
            file: 'dist/index.min.js',
            format: 'umd',
            name: 'MyLibrary',
            sourcemap: false,
            plugins: [terser()]
        }
    ].filter(Boolean),
    
    plugins: [
        // 进度显示
        progress(),
        
        // 环境变量替换
        replace({
            'process.env.NODE_ENV': JSON.stringify(
                production ? 'production' : 'development'
            ),
            preventAssignment: true
        }),
        
        resolve({
            extensions: ['.js', '.ts', '.tsx']
        }),
        
        commonjs(),
        
        typescript({
            tsconfig: './tsconfig.json',
            declaration: true,
            declarationDir: 'dist/types'
        }),
        
        babel({
            babelHelpers: 'bundled',
            exclude: 'node_modules/**',
            extensions: ['.js', '.ts', '.tsx']
        }),
        
        // 清理注释
        cleanup(),
        
        // 生产环境压缩
        production && terser({
            compress: {
                drop_console: true,
                drop_debugger: true,
                pure_funcs: ['console.log']
            },
            format: {
                comments: false
            }
        }),
        
        // 文件大小报告
        filesize(),
        
        // 打包分析
        production && visualizer({
            filename: 'dist/stats.html',
            open: true,
            gzipSize: true,
            brotliSize: true
        })
    ].filter(Boolean),
    
    external: ['react', 'react-dom']
};
```

### 8.5 性能优化配置

```javascript
/**
 * 性能优化配置
 * @author erik.zhou
 */

export default {
    input: 'src/index.js',
    output: {
        dir: 'dist',
        format: 'es',
        sourcemap: false
    },
    
    // 缓存
    cache: true,
    
    // 树摇优化
    treeshake: {
        preset: 'smallest',
        moduleSideEffects: false,
        propertyReadSideEffects: false
    },
    
    // 性能监控
    perf: true,
    
    // 最大并行读取
    maxParallelFileReads: 20,
    
    // 实验性缓存过期
    experimentalCacheExpiry: 10,
    
    plugins: [
        // 插件配置...
    ]
};
```

---

## 总结

本教程详细介绍了 Rollup 的核心功能：

1. Rollup 简介：特点、适用场景、与 Webpack 对比
2. 快速开始：安装、基础使用、配置文件
3. 配置文件详解：输入输出配置、多输出、高级选项
4. 插件系统：常用插件、插件配置、CSS 和资源处理
5. 代码分割：动态导入、多入口分割
6. Tree Shaking：原理、配置、副作用标记
7. 插件开发：基础结构、实用插件示例
8. 实战案例：库构建、React 组件库、Monorepo、生产配置

Rollup 是构建 JavaScript 库和工具的最佳选择，通过合理配置可以获得最小的打包体积和最佳的性能。

---

**最后更新时间：** 2026-02-26  
**@author erik.zhou**
