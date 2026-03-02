# Babel - 完整教程

## 目录
1. [Babel 简介](#babel-简介)
2. [快速开始](#快速开始)
3. [配置文件详解](#配置文件详解)
4. [Presets 预设](#presets-预设)
5. [Plugins 插件](#plugins-插件)
6. [Polyfill 和 Runtime](#polyfill-和-runtime)
7. [与构建工具集成](#与构建工具集成)
8. [实战案例](#实战案例)

---

## Babel 简介

### 1.1 什么是 Babel

Babel 是一个 JavaScript 编译器，主要用于将 ES6+ 代码转换为向后兼容的 JavaScript 版本。

```javascript
/**
 * Babel 转换示例
 * @author erik.zhou
 */

// 输入（ES6+）
const greet = (name) => {
    return `Hello, ${name}!`;
};

class Person {
    constructor(name) {
        this.name = name;
    }
    
    sayHello() {
        console.log(greet(this.name));
    }
}

// 输出（ES5）
"use strict";

var greet = function greet(name) {
    return "Hello, " + name + "!";
};

var Person = function() {
    function Person(name) {
        this.name = name;
    }
    
    Person.prototype.sayHello = function sayHello() {
        console.log(greet(this.name));
    };
    
    return Person;
}();
```

### 1.2 Babel 核心概念

```javascript
/**
 * Babel 工作流程
 * @author erik.zhou
 */

// 1. 解析（Parse）
// 将代码字符串解析成抽象语法树（AST）
const code = 'const a = 1;';
const ast = parse(code);

// 2. 转换（Transform）
// 遍历 AST，应用插件进行转换
const transformedAST = transform(ast, plugins);

// 3. 生成（Generate）
// 将转换后的 AST 生成新的代码
const output = generate(transformedAST);
```

### 1.3 Babel 生态系统

```javascript
/**
 * Babel 核心包
 * @author erik.zhou
 */

const babelEcosystem = {
    core: {
        '@babel/core': 'Babel 核心功能',
        '@babel/cli': '命令行工具',
        '@babel/parser': '解析器',
        '@babel/generator': '代码生成器',
        '@babel/traverse': 'AST 遍历器'
    },
    
    presets: {
        '@babel/preset-env': '智能预设，根据目标环境转换',
        '@babel/preset-react': 'React JSX 转换',
        '@babel/preset-typescript': 'TypeScript 转换',
        '@babel/preset-flow': 'Flow 类型转换'
    },
    
    plugins: {
        transform: '语法转换插件',
        proposal: '提案特性插件',
        syntax: '语法解析插件'
    },
    
    helpers: {
        '@babel/runtime': '运行时辅助函数',
        '@babel/plugin-transform-runtime': '运行时插件',
        '@babel/polyfill': 'Polyfill（已废弃）',
        'core-js': '现代 Polyfill 库'
    }
};
```

---

## 快速开始

### 2.1 安装 Babel

```bash
# 安装核心包
npm install --save-dev @babel/core @babel/cli

# 安装预设
npm install --save-dev @babel/preset-env

# 安装 React 预设（可选）
npm install --save-dev @babel/preset-react

# 安装 TypeScript 预设（可选）
npm install --save-dev @babel/preset-typescript
```

### 2.2 基础使用

```bash
# 编译单个文件
npx babel src/app.js --out-file dist/app.js

# 编译整个目录
npx babel src --out-dir dist

# 使用配置文件
npx babel src --out-dir dist --config-file ./babel.config.js

# 监听模式
npx babel src --out-dir dist --watch

# 生成源映射
npx babel src --out-dir dist --source-maps
```

### 2.3 配置文件

```javascript
/**
 * babel.config.js - 项目级配置
 * @author erik.zhou
 */

module.exports = {
    presets: [
        ['@babel/preset-env', {
            targets: {
                browsers: ['> 1%', 'last 2 versions', 'not dead']
            }
        }]
    ],
    plugins: []
};
```

```json
/**
 * .babelrc.json - 文件级配置
 * @author erik.zhou
 */
{
    "presets": [
        ["@babel/preset-env", {
            "targets": "> 0.25%, not dead"
        }]
    ],
    "plugins": []
}
```

---

## 配置文件详解

### 3.1 配置文件类型

```javascript
/**
 * babel.config.js - 推荐用于项目级配置
 * @author erik.zhou
 */

module.exports = function(api) {
    // 缓存配置
    api.cache(true);
    
    const presets = [
        '@babel/preset-env'
    ];
    
    const plugins = [];
    
    return {
        presets,
        plugins
    };
};
```

```json
/**
 * .babelrc.json - 用于文件级配置
 * @author erik.zhou
 */
{
    "presets": ["@babel/preset-env"],
    "plugins": [],
    "env": {
        "development": {
            "plugins": ["react-refresh/babel"]
        },
        "production": {
            "plugins": ["transform-remove-console"]
        }
    }
}
```

### 3.2 配置选项

```javascript
/**
 * 完整配置选项
 * @author erik.zhou
 */

module.exports = {
    // 预设
    presets: [],
    
    // 插件
    plugins: [],
    
    // 环境配置
    env: {
        development: {},
        production: {},
        test: {}
    },
    
    // 覆盖配置
    overrides: [
        {
            test: /\.tsx?$/,
            presets: ['@babel/preset-typescript']
        }
    ],
    
    // 忽略文件
    ignore: [
        'node_modules/**',
        '**/*.test.js'
    ],
    
    // 只处理的文件
    only: [
        'src/**'
    ],
    
    // 源映射
    sourceMaps: true,
    
    // 内联源映射
    sourceMaps: 'inline',
    
    // 源文件名
    sourceFileName: 'original.js',
    
    // 源根目录
    sourceRoot: '/src',
    
    // 注释
    comments: true,
    
    // 压缩
    compact: false,
    
    // 最小化
    minified: false,
    
    // 辅助函数
    auxiliaryCommentBefore: '/* before */',
    auxiliaryCommentAfter: '/* after */',
    
    // 模块 ID
    moduleId: 'myModule',
    
    // 模块根目录
    moduleRoot: '/src',
    
    // 文件名
    filename: 'app.js',
    
    // 文件名相对路径
    filenameRelative: 'src/app.js',
    
    // 代码
    code: true,
    
    // AST
    ast: false
};
```

---

## Presets 预设

### 4.1 @babel/preset-env

```javascript
/**
 * @babel/preset-env 详细配置
 * @author erik.zhou
 */

module.exports = {
    presets: [
        ['@babel/preset-env', {
            // 目标环境
            targets: {
                // 浏览器
                browsers: ['> 1%', 'last 2 versions', 'not dead'],
                
                // 或使用 browserslist 查询
                browsers: '> 0.25%, not dead',
                
                // Node.js 版本
                node: 'current',
                node: '14.0',
                
                // 特定浏览器
                chrome: '58',
                edge: '17',
                firefox: '60',
                safari: '11.1'
            },
            
            // 模块转换
            // false, 'auto', 'amd', 'umd', 'systemjs', 'commonjs'
            modules: false,
            
            // 调试模式
            debug: false,
            
            // 包含的插件
            include: ['transform-arrow-functions'],
            
            // 排除的插件
            exclude: ['transform-regenerator'],
            
            // Polyfill 使用方式
            // 'usage', 'entry', false
            useBuiltIns: 'usage',
            
            // core-js 版本
            corejs: {
                version: 3,
                proposals: true
            },
            
            // 强制所有转换
            forceAllTransforms: false,
            
            // 配置路径
            configPath: process.cwd(),
            
            // 忽略浏览器配置
            ignoreBrowserslistConfig: false,
            
            // 浏览器配置环境
            browserslistEnv: 'production',
            
            // 规范模式
            spec: false,
            
            // 宽松模式
            loose: false,
            
            // 模块 ID
            moduleId: undefined,
            
            // 模块根目录
            moduleRoot: undefined,
            
            // 模块名称
            moduleName: undefined,
            
            // 辅助函数
            useBuiltIns: 'usage'
        }]
    ]
};
```

```javascript
/**
 * 不同 useBuiltIns 选项的效果
 * @author erik.zhou
 */

// useBuiltIns: false
// 不自动导入 polyfill
const arr = [1, 2, 3];
arr.includes(1);

// useBuiltIns: 'entry'
// 需要手动导入 core-js
import 'core-js/stable';
import 'regenerator-runtime/runtime';

// useBuiltIns: 'usage'
// 自动按需导入 polyfill
import 'core-js/modules/es.array.includes.js';
const arr = [1, 2, 3];
arr.includes(1);
```

### 4.2 @babel/preset-react

```javascript
/**
 * @babel/preset-react 配置
 * @author erik.zhou
 */

module.exports = {
    presets: [
        ['@babel/preset-react', {
            // JSX 运行时
            // 'automatic', 'classic'
            runtime: 'automatic',
            
            // 开发模式
            development: process.env.NODE_ENV === 'development',
            
            // 抛出错误
            throwIfNamespace: true,
            
            // 导入来源
            importSource: 'react',
            
            // JSX Pragma
            pragma: 'React.createElement',
            
            // JSX Fragment Pragma
            pragmaFrag: 'React.Fragment',
            
            // 使用内置
            useBuiltIns: false,
            
            // 使用 Spread
            useSpread: false
        }]
    ]
};
```

```javascript
/**
 * React JSX 转换示例
 * @author erik.zhou
 */

// 输入（JSX）
function App() {
    return (
        <div className="app">
            <h1>Hello World</h1>
            <p>Welcome to React</p>
        </div>
    );
}

// 输出（runtime: 'automatic'）
import { jsx as _jsx } from 'react/jsx-runtime';
import { jsxs as _jsxs } from 'react/jsx-runtime';

function App() {
    return _jsxs('div', {
        className: 'app',
        children: [
            _jsx('h1', { children: 'Hello World' }),
            _jsx('p', { children: 'Welcome to React' })
        ]
    });
}

// 输出（runtime: 'classic'）
import React from 'react';

function App() {
    return React.createElement(
        'div',
        { className: 'app' },
        React.createElement('h1', null, 'Hello World'),
        React.createElement('p', null, 'Welcome to React')
    );
}
```

### 4.3 @babel/preset-typescript

```javascript
/**
 * @babel/preset-typescript 配置
 * @author erik.zhou
 */

module.exports = {
    presets: [
        ['@babel/preset-typescript', {
            // 允许声明文件
            allowDeclareFields: false,
            
            // 允许命名空间
            allowNamespaces: true,
            
            // 禁用类型检查
            onlyRemoveTypeImports: false,
            
            // 优化常量枚举
            optimizeConstEnums: false,
            
            // JSX
            isTSX: false,
            
            // 所有扩展名
            allExtensions: false
        }]
    ]
};
```

---

## Plugins 插件

### 5.1 常用转换插件

```javascript
/**
 * 常用转换插件配置
 * @author erik.zhou
 */

module.exports = {
    plugins: [
        // 类属性
        '@babel/plugin-proposal-class-properties',
        
        // 私有方法
        '@babel/plugin-proposal-private-methods',
        
        // 装饰器
        ['@babel/plugin-proposal-decorators', {
            legacy: true
        }],
        
        // 对象剩余展开
        '@babel/plugin-proposal-object-rest-spread',
        
        // 可选链
        '@babel/plugin-proposal-optional-chaining',
        
        // 空值合并
        '@babel/plugin-proposal-nullish-coalescing-operator',
        
        // 动态导入
        '@babel/plugin-syntax-dynamic-import',
        
        // 数字分隔符
        '@babel/plugin-proposal-numeric-separator',
        
        // 逻辑赋值
        '@babel/plugin-proposal-logical-assignment-operators'
    ]
};
```

### 5.2 插件示例

```javascript
/**
 * 类属性转换示例
 * @author erik.zhou
 */

// 输入
class Counter {
    count = 0;
    
    increment = () => {
        this.count++;
    }
}

// 输出
class Counter {
    constructor() {
        this.count = 0;
        this.increment = () => {
            this.count++;
        };
    }
}
```

```javascript
/**
 * 装饰器转换示例
 * @author erik.zhou
 */

// 输入
function readonly(target, key, descriptor) {
    descriptor.writable = false;
    return descriptor;
}

class Person {
    @readonly
    name = 'John';
}

// 输出
var _class;

function readonly(target, key, descriptor) {
    descriptor.writable = false;
    return descriptor;
}

let Person = (_class = class Person {
    constructor() {
        this.name = 'John';
    }
}, (_applyDecoratedDescriptor(_class.prototype, 'name', [readonly], 
    Object.getOwnPropertyDescriptor(_class.prototype, 'name'), 
    _class.prototype)), _class);
```

### 5.3 自定义插件

```javascript
/**
 * 自定义 Babel 插件
 * @author erik.zhou
 */

module.exports = function(babel) {
    const { types: t } = babel;
    
    return {
        name: 'my-custom-plugin',
        
        visitor: {
            // 访问标识符
            Identifier(path) {
                if (path.node.name === 'oldName') {
                    path.node.name = 'newName';
                }
            },
            
            // 访问函数声明
            FunctionDeclaration(path) {
                const name = path.node.id.name;
                console.log(`Found function: ${name}`);
            },
            
            // 访问箭头函数
            ArrowFunctionExpression(path) {
                // 转换为普通函数
                const func = t.functionExpression(
                    null,
                    path.node.params,
                    path.node.body,
                    path.node.generator,
                    path.node.async
                );
                path.replaceWith(func);
            },
            
            // 访问二元表达式
            BinaryExpression(path) {
                if (path.node.operator === '===') {
                    path.node.operator = '==';
                }
            }
        }
    };
};
```

```javascript
/**
 * 使用自定义插件
 * @author erik.zhou
 */

module.exports = {
    plugins: [
        './plugins/my-custom-plugin.js',
        
        // 带选项的插件
        ['./plugins/my-plugin.js', {
            option1: 'value1',
            option2: 'value2'
        }]
    ]
};
```

---

## Polyfill 和 Runtime

### 6.1 Polyfill 方案对比

```javascript
/**
 * Polyfill 方案对比
 * @author erik.zhou
 */

// 方案1：@babel/polyfill（已废弃）
// 全局污染，体积大
import '@babel/polyfill';

// 方案2：core-js + regenerator-runtime
// 手动导入，灵活控制
import 'core-js/stable';
import 'regenerator-runtime/runtime';

// 方案3：按需导入（推荐）
// 使用 @babel/preset-env 的 useBuiltIns: 'usage'
// 自动按需导入，无需手动引入

// 方案4：@babel/runtime（推荐用于库）
// 不污染全局，适合库开发
```

### 6.2 core-js 配置

```javascript
/**
 * core-js 按需导入配置
 * @author erik.zhou
 */

module.exports = {
    presets: [
        ['@babel/preset-env', {
            // 按需导入 polyfill
            useBuiltIns: 'usage',
            
            // core-js 版本
            corejs: {
                version: 3,
                // 包含提案特性
                proposals: true
            },
            
            // 目标环境
            targets: {
                browsers: '> 0.25%, not dead'
            }
        }]
    ]
};
```

```javascript
/**
 * core-js 使用示例
 * @author erik.zhou
 */

// 源代码
const arr = [1, 2, 3];
arr.includes(2);

const promise = Promise.resolve(42);

async function fetchData() {
    const response = await fetch('/api/data');
    return response.json();
}

// 编译后（自动导入需要的 polyfill）
import 'core-js/modules/es.array.includes.js';
import 'core-js/modules/es.promise.js';
import 'core-js/modules/web.dom-collections.iterator.js';

const arr = [1, 2, 3];
arr.includes(2);

const promise = Promise.resolve(42);

async function fetchData() {
    const response = await fetch('/api/data');
    return response.json();
}
```

### 6.3 @babel/runtime 配置

```javascript
/**
 * @babel/runtime 配置（适合库开发）
 * @author erik.zhou
 */

// 安装依赖
// npm install --save @babel/runtime
// npm install --save-dev @babel/plugin-transform-runtime

module.exports = {
    presets: [
        ['@babel/preset-env', {
            modules: false
        }]
    ],
    plugins: [
        ['@babel/plugin-transform-runtime', {
            // 使用 core-js
            corejs: 3,
            
            // 辅助函数
            helpers: true,
            
            // regenerator
            regenerator: true,
            
            // 使用 ES 模块
            useESModules: true,
            
            // 绝对路径
            absoluteRuntime: false,
            
            // 版本
            version: '^7.0.0'
        }]
    ]
};
```

```javascript
/**
 * @babel/runtime 使用示例
 * @author erik.zhou
 */

// 源代码
class Person {
    constructor(name) {
        this.name = name;
    }
}

const arr = [1, 2, 3];
arr.includes(2);

// 编译后（使用 runtime 辅助函数）
import _classCallCheck from '@babel/runtime/helpers/classCallCheck';
import _createClass from '@babel/runtime/helpers/createClass';
import _includesInstanceProperty from '@babel/runtime-corejs3/core-js-stable/instance/includes';

var Person = function() {
    function Person(name) {
        _classCallCheck(this, Person);
        this.name = name;
    }
    
    return _createClass(Person);
}();

var arr = [1, 2, 3];
_includesInstanceProperty(arr).call(arr, 2);
```

---

## 与构建工具集成

### 7.1 Webpack 集成

```javascript
/**
 * Webpack + Babel 配置
 * @author erik.zhou
 */

// webpack.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(js|jsx|ts|tsx)$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        // 缓存
                        cacheDirectory: true,
                        
                        // 压缩缓存
                        cacheCompression: false,
                        
                        // 预设
                        presets: [
                            ['@babel/preset-env', {
                                targets: '> 0.25%, not dead',
                                useBuiltIns: 'usage',
                                corejs: 3
                            }],
                            '@babel/preset-react',
                            '@babel/preset-typescript'
                        ],
                        
                        // 插件
                        plugins: [
                            '@babel/plugin-proposal-class-properties',
                            ['@babel/plugin-transform-runtime', {
                                corejs: 3
                            }]
                        ]
                    }
                }
            }
        ]
    }
};
```

### 7.2 Rollup 集成

```javascript
/**
 * Rollup + Babel 配置
 * @author erik.zhou
 */

import babel from '@rollup/plugin-babel';

export default {
    input: 'src/index.js',
    output: {
        file: 'dist/bundle.js',
        format: 'es'
    },
    plugins: [
        babel({
            // 辅助函数处理
            babelHelpers: 'bundled',
            
            // 排除文件
            exclude: 'node_modules/**',
            
            // 扩展名
            extensions: ['.js', '.jsx', '.ts', '.tsx'],
            
            // 预设
            presets: [
                ['@babel/preset-env', {
                    targets: '> 0.25%, not dead',
                    modules: false
                }],
                '@babel/preset-react',
                '@babel/preset-typescript'
            ],
            
            // 插件
            plugins: [
                '@babel/plugin-proposal-class-properties'
            ]
        })
    ]
};
```

### 7.3 Vite 集成

```javascript
/**
 * Vite + Babel 配置
 * @author erik.zhou
 */

// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        react({
            babel: {
                // Babel 配置
                presets: [
                    ['@babel/preset-env', {
                        targets: '> 0.25%, not dead'
                    }]
                ],
                plugins: [
                    '@babel/plugin-proposal-class-properties',
                    ['@babel/plugin-transform-runtime', {
                        corejs: 3
                    }]
                ]
            }
        })
    ]
});
```

### 7.4 Jest 集成

```javascript
/**
 * Jest + Babel 配置
 * @author erik.zhou
 */

// jest.config.js
module.exports = {
    transform: {
        '^.+\\.(js|jsx|ts|tsx)$': 'babel-jest'
    },
    transformIgnorePatterns: [
        'node_modules/(?!(module-to-transform)/)'
    ]
};

// babel.config.js
module.exports = {
    presets: [
        ['@babel/preset-env', {
            targets: {
                node: 'current'
            }
        }],
        '@babel/preset-react',
        '@babel/preset-typescript'
    ]
};
```

---

## 实战案例

### 8.1 React 项目配置

```javascript
/**
 * React 项目完整 Babel 配置
 * @author erik.zhou
 */

module.exports = function(api) {
    api.cache(true);
    
    const isDevelopment = process.env.NODE_ENV === 'development';
    const isProduction = process.env.NODE_ENV === 'production';
    
    return {
        presets: [
            ['@babel/preset-env', {
                targets: {
                    browsers: '> 0.25%, not dead'
                },
                useBuiltIns: 'usage',
                corejs: {
                    version: 3,
                    proposals: true
                },
                modules: false
            }],
            ['@babel/preset-react', {
                runtime: 'automatic',
                development: isDevelopment
            }],
            '@babel/preset-typescript'
        ],
        
        plugins: [
            // 类属性
            '@babel/plugin-proposal-class-properties',
            
            // 装饰器
            ['@babel/plugin-proposal-decorators', {
                legacy: true
            }],
            
            // 运行时
            ['@babel/plugin-transform-runtime', {
                corejs: 3,
                helpers: true,
                regenerator: true,
                useESModules: true
            }],
            
            // 开发环境：React Refresh
            isDevelopment && 'react-refresh/babel',
            
            // 生产环境：移除 console
            isProduction && 'transform-remove-console'
        ].filter(Boolean),
        
        env: {
            test: {
                presets: [
                    ['@babel/preset-env', {
                        targets: {
                            node: 'current'
                        }
                    }]
                ]
            }
        }
    };
};
```

### 8.2 Vue 项目配置

```javascript
/**
 * Vue 3 项目 Babel 配置
 * @author erik.zhou
 */

module.exports = {
    presets: [
        ['@babel/preset-env', {
            targets: {
                browsers: '> 0.25%, not dead'
            },
            useBuiltIns: 'usage',
            corejs: 3,
            modules: false
        }],
        '@babel/preset-typescript'
    ],
    
    plugins: [
        '@babel/plugin-proposal-class-properties',
        '@babel/plugin-proposal-optional-chaining',
        '@babel/plugin-proposal-nullish-coalescing-operator',
        ['@babel/plugin-transform-runtime', {
            corejs: 3
        }]
    ]
};
```

### 8.3 库项目配置

```javascript
/**
 * JavaScript 库 Babel 配置
 * @author erik.zhou
 */

module.exports = {
    presets: [
        ['@babel/preset-env', {
            targets: {
                browsers: '> 0.25%, not dead'
            },
            modules: false,
            loose: true
        }],
        '@babel/preset-typescript'
    ],
    
    plugins: [
        '@babel/plugin-proposal-class-properties',
        
        // 使用 runtime 避免全局污染
        ['@babel/plugin-transform-runtime', {
            corejs: 3,
            helpers: true,
            regenerator: true,
            useESModules: true
        }]
    ],
    
    // 不同输出格式的配置
    env: {
        esm: {
            presets: [
                ['@babel/preset-env', {
                    modules: false
                }]
            ]
        },
        cjs: {
            presets: [
                ['@babel/preset-env', {
                    modules: 'commonjs'
                }]
            ]
        }
    }
};
```

### 8.4 Monorepo 配置

```javascript
/**
 * Monorepo 根目录 babel.config.js
 * @author erik.zhou
 */

module.exports = {
    presets: [
        ['@babel/preset-env', {
            targets: {
                node: 'current'
            }
        }],
        '@babel/preset-typescript'
    ],
    
    // 覆盖配置
    overrides: [
        {
            // packages/web 使用浏览器配置
            test: './packages/web',
            presets: [
                ['@babel/preset-env', {
                    targets: {
                        browsers: '> 0.25%, not dead'
                    },
                    useBuiltIns: 'usage',
                    corejs: 3
                }],
                '@babel/preset-react'
            ]
        },
        {
            // packages/node 使用 Node.js 配置
            test: './packages/node',
            presets: [
                ['@babel/preset-env', {
                    targets: {
                        node: '14'
                    }
                }]
            ]
        }
    ]
};
```

### 8.5 性能优化配置

```javascript
/**
 * 性能优化的 Babel 配置
 * @author erik.zhou
 */

module.exports = {
    presets: [
        ['@babel/preset-env', {
            targets: '> 0.25%, not dead',
            useBuiltIns: 'usage',
            corejs: 3,
            
            // 宽松模式（更小的代码）
            loose: true,
            
            // 排除不需要的转换
            exclude: [
                'transform-typeof-symbol'
            ]
        }]
    ],
    
    plugins: [
        ['@babel/plugin-transform-runtime', {
            corejs: 3,
            helpers: true,
            regenerator: true,
            
            // 使用 ES 模块（更好的 tree shaking）
            useESModules: true
        }]
    ],
    
    // 缓存配置
    cacheDirectory: true,
    cacheCompression: false,
    
    // 紧凑输出
    compact: true,
    
    // 移除注释
    comments: false
};
```

---

## 总结

本教程详细介绍了 Babel 的核心功能：

1. Babel 简介：工作原理、核心概念、生态系统
2. 快速开始：安装、基础使用、配置文件
3. 配置文件详解：配置类型、配置选项
4. Presets 预设：preset-env、preset-react、preset-typescript
5. Plugins 插件：常用插件、插件示例、自定义插件
6. Polyfill 和 Runtime：方案对比、core-js、@babel/runtime
7. 与构建工具集成：Webpack、Rollup、Vite、Jest
8. 实战案例：React、Vue、库项目、Monorepo、性能优化

Babel 是现代 JavaScript 开发的基础工具，通过合理配置可以确保代码在各种环境中正常运行。

---

**最后更新时间：** 2026-02-27  
**@author erik.zhou**
