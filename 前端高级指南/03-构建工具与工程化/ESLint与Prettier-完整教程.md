# ESLint与Prettier - 完整教程

## 课程信息
- **课程名称**: ESLint与Prettier完整教程
- **难度级别**: 中级
- **预计学时**: 8小时
- **核心内容**: 代码规范工具配置、规则详解、集成方案、自动化流程
- **@author**: erik.zhou

---

## 目录
1. [ESLint基础](#1-eslint基础)
2. [ESLint配置详解](#2-eslint配置详解)
3. [ESLint规则体系](#3-eslint规则体系)
4. [Prettier基础](#4-prettier基础)
5. [ESLint与Prettier集成](#5-eslint与prettier集成)
6. [编辑器集成](#6-编辑器集成)
7. [Git Hooks集成](#7-git-hooks集成)
8. [自定义规则开发](#8-自定义规则开发)
9. [团队规范实践](#9-团队规范实践)
10. [常见问题与最佳实践](#10-常见问题与最佳实践)

---

## 1. ESLint基础

### 1.1 什么是ESLint

ESLint是一个开源的JavaScript代码检查工具，用于识别和报告代码中的模式问题。

**核心特性**:
- 可配置的规则系统
- 支持插件扩展
- 自动修复功能
- 支持多种解析器
- 与编辑器深度集成

### 1.2 安装ESLint

```bash
# 项目级安装
npm install eslint --save-dev

# 初始化配置
npx eslint --init

# 全局安装（不推荐）
npm install -g eslint
```

### 1.3 基础使用

```bash
# 检查单个文件
npx eslint file.js

# 检查目录
npx eslint src/

# 自动修复
npx eslint src/ --fix

# 指定配置文件
npx eslint src/ --config .eslintrc.json
```



### 1.4 第一个ESLint配置

```javascript
// .eslintrc.js
module.exports = {
    env: {
        browser: true,
        es2021: true,
        node: true
    },
    extends: 'eslint:recommended',
    parserOptions: {
        ecmaVersion: 'latest',
        sourceType: 'module'
    },
    rules: {
        'no-console': 'warn',
        'no-unused-vars': 'error',
        'semi': ['error', 'always'],
        'quotes': ['error', 'single']
    }
};
```

### 1.5 配置文件格式

ESLint支持多种配置文件格式：

```javascript
// 1. .eslintrc.js (推荐)
module.exports = {
    rules: {
        semi: ['error', 'always']
    }
};

// 2. .eslintrc.json
{
    "rules": {
        "semi": ["error", "always"]
    }
}

// 3. .eslintrc.yml
rules:
    semi:
        - error
        - always

// 4. package.json
{
    "eslintConfig": {
        "rules": {
            "semi": ["error", "always"]
        }
    }
}
```

---

## 2. ESLint配置详解

### 2.1 环境配置（env）

```javascript
module.exports = {
    env: {
        // 浏览器全局变量
        browser: true,
        
        // Node.js全局变量和作用域
        node: true,
        
        // ES2021全局变量
        es2021: true,
        
        // CommonJS全局变量和作用域
        commonjs: true,
        
        // jQuery全局变量
        jquery: true,
        
        // Jest全局变量
        jest: true,
        
        // Mocha全局变量
        mocha: true,
        
        // Web Workers全局变量
        worker: true,
        
        // Service Worker全局变量
        serviceworker: true
    }
};
```

### 2.2 全局变量配置（globals）

```javascript
module.exports = {
    globals: {
        // 只读全局变量
        myGlobalVar: 'readonly',
        
        // 可写全局变量
        myWritableGlobal: 'writable',
        
        // 禁用全局变量
        myDisabledGlobal: 'off',
        
        // 旧语法（兼容）
        $: false,  // 只读
        jQuery: true  // 可写
    }
};
```

### 2.3 解析器配置（parser & parserOptions）

```javascript
module.exports = {
    // 指定解析器
    parser: '@babel/eslint-parser',
    
    parserOptions: {
        // ECMAScript版本
        ecmaVersion: 2021,
        
        // 源码类型
        sourceType: 'module',  // 'script' | 'module'
        
        // 额外的语言特性
        ecmaFeatures: {
            jsx: true,
            globalReturn: false,
            impliedStrict: true
        },
        
        // Babel解析器特定配置
        requireConfigFile: false,
        babelOptions: {
            presets: ['@babel/preset-react']
        }
    }
};
```



### 2.4 扩展配置（extends）

```javascript
module.exports = {
    extends: [
        // ESLint推荐规则
        'eslint:recommended',
        
        // Airbnb规范
        'airbnb',
        'airbnb/hooks',
        
        // Standard规范
        'standard',
        
        // Google规范
        'google',
        
        // React规则
        'plugin:react/recommended',
        
        // TypeScript规则
        'plugin:@typescript-eslint/recommended',
        
        // Vue规则
        'plugin:vue/vue3-recommended',
        
        // Prettier集成（必须放在最后）
        'prettier'
    ]
};
```

### 2.5 插件配置（plugins）

```javascript
module.exports = {
    plugins: [
        // React插件
        'react',
        'react-hooks',
        
        // TypeScript插件
        '@typescript-eslint',
        
        // Vue插件
        'vue',
        
        // Import插件
        'import',
        
        // JSX a11y插件
        'jsx-a11y',
        
        // Promise插件
        'promise',
        
        // Node插件
        'node'
    ],
    
    rules: {
        // 使用插件规则
        'react/prop-types': 'error',
        'react-hooks/rules-of-hooks': 'error',
        '@typescript-eslint/no-unused-vars': 'error'
    }
};
```

### 2.6 规则配置（rules）

```javascript
module.exports = {
    rules: {
        // 关闭规则
        'no-console': 'off',
        
        // 警告级别
        'no-unused-vars': 'warn',
        
        // 错误级别
        'semi': 'error',
        
        // 带选项的规则
        'quotes': ['error', 'single'],
        'indent': ['error', 4],
        'comma-dangle': ['error', 'never'],
        
        // 对象形式配置
        'max-len': ['error', {
            code: 120,
            tabWidth: 4,
            ignoreUrls: true,
            ignoreStrings: true
        }],
        
        // 覆盖扩展的规则
        'import/no-unresolved': 'off'
    }
};
```

### 2.7 覆盖配置（overrides）

```javascript
module.exports = {
    rules: {
        'no-console': 'error'
    },
    
    overrides: [
        // 测试文件特殊配置
        {
            files: ['**/*.test.js', '**/*.spec.js'],
            env: {
                jest: true
            },
            rules: {
                'no-console': 'off'
            }
        },
        
        // TypeScript文件配置
        {
            files: ['**/*.ts', '**/*.tsx'],
            parser: '@typescript-eslint/parser',
            plugins: ['@typescript-eslint'],
            rules: {
                '@typescript-eslint/explicit-function-return-type': 'error'
            }
        },
        
        // 配置文件特殊配置
        {
            files: ['*.config.js', '.eslintrc.js'],
            env: {
                node: true
            },
            rules: {
                'no-console': 'off'
            }
        }
    ]
};
```

### 2.8 忽略文件配置

```javascript
// .eslintignore
# 依赖目录
node_modules/
bower_components/

# 构建产物
dist/
build/
coverage/

# 配置文件
*.config.js
.eslintrc.js

# 第三方库
public/libs/
src/vendor/

# 临时文件
*.tmp
*.log
```

---

## 3. ESLint规则体系

### 3.1 可能的错误（Possible Errors）

```javascript
module.exports = {
    rules: {
        // 禁止条件表达式中的常量
        'no-constant-condition': 'error',
        
        // 禁止在正则表达式中使用控制字符
        'no-control-regex': 'error',
        
        // 禁止使用debugger
        'no-debugger': 'error',
        
        // 禁止对象字面量中出现重复的key
        'no-dupe-keys': 'error',
        
        // 禁止重复的case标签
        'no-duplicate-case': 'error',
        
        // 禁止空语句块
        'no-empty': 'error',
        
        // 禁止在正则表达式中使用空字符集
        'no-empty-character-class': 'error',
        
        // 禁止对catch子句的参数重新赋值
        'no-ex-assign': 'error',
        
        // 禁止不必要的布尔转换
        'no-extra-boolean-cast': 'error',
        
        // 禁止不必要的括号
        'no-extra-parens': ['error', 'functions'],
        
        // 禁止不必要的分号
        'no-extra-semi': 'error'
    }
};
```



### 3.2 最佳实践（Best Practices）

```javascript
module.exports = {
    rules: {
        // 强制getter和setter在对象中成对出现
        'accessor-pairs': 'error',
        
        // 强制数组方法的回调函数中有return语句
        'array-callback-return': 'error',
        
        // 强制把变量的使用限制在其定义的作用域范围内
        'block-scoped-var': 'error',
        
        // 限制圈复杂度
        'complexity': ['error', 10],
        
        // 要求return语句要么总是指定返回的值，要么不指定
        'consistent-return': 'error',
        
        // 强制所有控制语句使用一致的括号风格
        'curly': ['error', 'all'],
        
        // 要求switch语句中有default分支
        'default-case': 'error',
        
        // 强制在点号之前和之后一致的换行
        'dot-location': ['error', 'property'],
        
        // 强制使用点号
        'dot-notation': 'error',
        
        // 要求使用===和!==
        'eqeqeq': ['error', 'always'],
        
        // 要求for-in循环中有一个if语句
        'guard-for-in': 'error',
        
        // 禁用alert、confirm和prompt
        'no-alert': 'warn',
        
        // 禁用arguments.caller或arguments.callee
        'no-caller': 'error',
        
        // 禁止使用看起来像除法的正则表达式
        'no-div-regex': 'error',
        
        // 禁止if语句中return语句之后有else块
        'no-else-return': 'error',
        
        // 禁止使用空解构模式
        'no-empty-pattern': 'error',
        
        // 禁止使用eval()
        'no-eval': 'error',
        
        // 禁止扩展原生类型
        'no-extend-native': 'error',
        
        // 禁止不必要的.bind()调用
        'no-extra-bind': 'error',
        
        // 禁用不必要的标签
        'no-extra-label': 'error',
        
        // 禁止case语句落空
        'no-fallthrough': 'error',
        
        // 禁止数字字面量中使用前导和末尾小数点
        'no-floating-decimal': 'error',
        
        // 禁止使用短符号进行类型转换
        'no-implicit-coercion': 'error',
        
        // 禁止在全局范围内使用变量声明和function声明
        'no-implicit-globals': 'error',
        
        // 禁止使用类似eval()的方法
        'no-implied-eval': 'error',
        
        // 禁止this关键字出现在类和类对象之外
        'no-invalid-this': 'error',
        
        // 禁用__iterator__属性
        'no-iterator': 'error',
        
        // 禁用标签语句
        'no-labels': 'error',
        
        // 禁用不必要的嵌套块
        'no-lone-blocks': 'error',
        
        // 禁止在循环中出现function声明和表达式
        'no-loop-func': 'error',
        
        // 禁用魔术数字
        'no-magic-numbers': ['error', {
            ignore: [0, 1, -1],
            ignoreArrayIndexes: true
        }],
        
        // 禁止使用多个空格
        'no-multi-spaces': 'error',
        
        // 禁止使用多行字符串
        'no-multi-str': 'error',
        
        // 禁止使用new以避免产生副作用
        'no-new': 'error',
        
        // 禁止对Function对象使用new操作符
        'no-new-func': 'error',
        
        // 禁止对String、Number和Boolean使用new操作符
        'no-new-wrappers': 'error',
        
        // 禁用八进制字面量
        'no-octal': 'error',
        
        // 禁止在字符串中使用八进制转义序列
        'no-octal-escape': 'error',
        
        // 禁止对function的参数进行重新赋值
        'no-param-reassign': 'error',
        
        // 禁用__proto__属性
        'no-proto': 'error',
        
        // 禁止多次声明同一变量
        'no-redeclare': 'error',
        
        // 禁止使用对象的某些属性
        'no-restricted-properties': ['error', {
            object: 'Math',
            property: 'pow',
            message: 'Use ** operator instead.'
        }],
        
        // 禁止在return语句中使用赋值语句
        'no-return-assign': 'error',
        
        // 禁止不必要的return await
        'no-return-await': 'error',
        
        // 禁止使用javascript:url
        'no-script-url': 'error',
        
        // 禁止自我赋值
        'no-self-assign': 'error',
        
        // 禁止自身比较
        'no-self-compare': 'error',
        
        // 禁用逗号操作符
        'no-sequences': 'error',
        
        // 禁止抛出异常字面量
        'no-throw-literal': 'error',
        
        // 禁用一成不变的循环条件
        'no-unmodified-loop-condition': 'error',
        
        // 禁止出现未使用过的表达式
        'no-unused-expressions': 'error',
        
        // 禁用未使用过的标签
        'no-unused-labels': 'error',
        
        // 禁止不必要的.call()和.apply()
        'no-useless-call': 'error',
        
        // 禁止不必要的catch子句
        'no-useless-catch': 'error',
        
        // 禁止不必要的字符串字面量或模板字面量的连接
        'no-useless-concat': 'error',
        
        // 禁用不必要的转义字符
        'no-useless-escape': 'error',
        
        // 禁止多余的return语句
        'no-useless-return': 'error',
        
        // 禁用void操作符
        'no-void': 'error',
        
        // 禁止在注释中使用特定的警告术语
        'no-warning-comments': 'warn',
        
        // 禁用with语句
        'no-with': 'error',
        
        // 要求使用Error对象作为Promise拒绝的原因
        'prefer-promise-reject-errors': 'error',
        
        // 要求IIFE使用括号括起来
        'wrap-iife': ['error', 'inside'],
        
        // 要求或禁止Yoda条件
        'yoda': 'error'
    }
};
```



### 3.3 变量声明（Variables）

```javascript
module.exports = {
    rules: {
        // 要求或禁止var声明中的初始化
        'init-declarations': ['error', 'always'],
        
        // 禁止删除变量
        'no-delete-var': 'error',
        
        // 禁止标签与变量同名
        'no-label-var': 'error',
        
        // 禁用特定的全局变量
        'no-restricted-globals': ['error', 'event', 'fdescribe'],
        
        // 禁止变量声明与外层作用域的变量同名
        'no-shadow': 'error',
        
        // 禁止将标识符定义为受限的名字
        'no-shadow-restricted-names': 'error',
        
        // 禁用未声明的变量
        'no-undef': 'error',
        
        // 禁止将变量初始化为undefined
        'no-undef-init': 'error',
        
        // 禁止将undefined作为标识符
        'no-undefined': 'error',
        
        // 禁止出现未使用过的变量
        'no-unused-vars': ['error', {
            vars: 'all',
            args: 'after-used',
            ignoreRestSiblings: true
        }],
        
        // 禁止在变量定义之前使用它们
        'no-use-before-define': ['error', {
            functions: false,
            classes: true,
            variables: true
        }]
    }
};
```

### 3.4 代码风格（Stylistic Issues）

```javascript
module.exports = {
    rules: {
        // 强制数组方括号中使用一致的空格
        'array-bracket-spacing': ['error', 'never'],
        
        // 强制在代码块中使用一致的大括号风格
        'brace-style': ['error', '1tbs'],
        
        // 强制使用骆驼拼写法命名约定
        'camelcase': ['error', {
            properties: 'never',
            ignoreDestructuring: false
        }],
        
        // 要求或禁止末尾逗号
        'comma-dangle': ['error', 'never'],
        
        // 强制在逗号前后使用一致的空格
        'comma-spacing': ['error', {
            before: false,
            after: true
        }],
        
        // 强制使用一致的逗号风格
        'comma-style': ['error', 'last'],
        
        // 强制在计算的属性的方括号中使用一致的空格
        'computed-property-spacing': ['error', 'never'],
        
        // 当获取当前执行环境的上下文时，强制使用一致的命名
        'consistent-this': ['error', 'that'],
        
        // 要求或禁止文件末尾存在空行
        'eol-last': ['error', 'always'],
        
        // 强制在函数标识符和其调用之间有空格
        'func-call-spacing': ['error', 'never'],
        
        // 要求函数名与赋值给它们的变量名或属性名相匹配
        'func-name-matching': 'error',
        
        // 要求或禁止使用命名的function表达式
        'func-names': ['error', 'as-needed'],
        
        // 强制一致地使用function声明或表达式
        'func-style': ['error', 'declaration', {
            allowArrowFunctions: true
        }],
        
        // 强制在函数括号内使用一致的换行
        'function-paren-newline': ['error', 'multiline'],
        
        // 禁用指定的标识符
        'id-blacklist': ['error', 'data', 'err', 'e', 'cb', 'callback'],
        
        // 强制标识符的最小和最大长度
        'id-length': ['error', {
            min: 2,
            max: 30,
            exceptions: ['i', 'j', 'x', 'y', '_']
        }],
        
        // 要求标识符匹配一个指定的正则表达式
        'id-match': ['error', '^[a-z]+([A-Z][a-z]+)*$', {
            properties: false,
            onlyDeclarations: true
        }],
        
        // 强制执行箭头函数体的位置
        'implicit-arrow-linebreak': ['error', 'beside'],
        
        // 强制使用一致的缩进
        'indent': ['error', 4, {
            SwitchCase: 1,
            VariableDeclarator: 1,
            outerIIFEBody: 1,
            MemberExpression: 1,
            FunctionDeclaration: {
                parameters: 1,
                body: 1
            },
            FunctionExpression: {
                parameters: 1,
                body: 1
            },
            CallExpression: {
                arguments: 1
            },
            ArrayExpression: 1,
            ObjectExpression: 1,
            ImportDeclaration: 1,
            flatTernaryExpressions: false,
            ignoreComments: false
        }],
        
        // 强制在JSX属性中一致地使用双引号或单引号
        'jsx-quotes': ['error', 'prefer-double'],
        
        // 强制在对象字面量的属性中键和值之间使用一致的间距
        'key-spacing': ['error', {
            beforeColon: false,
            afterColon: true
        }],
        
        // 强制在关键字前后使用一致的空格
        'keyword-spacing': ['error', {
            before: true,
            after: true
        }],
        
        // 强制行注释的位置
        'line-comment-position': ['error', {
            position: 'above'
        }],
        
        // 强制使用一致的换行风格
        'linebreak-style': ['error', 'unix'],
        
        // 要求在注释周围有空行
        'lines-around-comment': ['error', {
            beforeBlockComment: true,
            afterBlockComment: false,
            beforeLineComment: true,
            afterLineComment: false,
            allowBlockStart: true,
            allowBlockEnd: true,
            allowObjectStart: true,
            allowObjectEnd: true,
            allowArrayStart: true,
            allowArrayEnd: true
        }],
        
        // 要求或禁止类成员之间出现空行
        'lines-between-class-members': ['error', 'always'],
        
        // 强制可嵌套的块的最大深度
        'max-depth': ['error', 4],
        
        // 强制一行的最大长度
        'max-len': ['error', {
            code: 120,
            tabWidth: 4,
            ignoreUrls: true,
            ignoreStrings: true,
            ignoreTemplateLiterals: true,
            ignoreRegExpLiterals: true
        }],
        
        // 强制最大行数
        'max-lines': ['error', {
            max: 500,
            skipBlankLines: true,
            skipComments: true
        }],
        
        // 强制回调函数最大嵌套深度
        'max-nested-callbacks': ['error', 3],
        
        // 强制函数定义中最多允许的参数数量
        'max-params': ['error', 5],
        
        // 强制函数块最多允许的的语句数量
        'max-statements': ['error', 30],
        
        // 强制每一行中所允许的最大语句数量
        'max-statements-per-line': ['error', {
            max: 1
        }],
        
        // 要求或禁止在三元操作数中间换行
        'multiline-ternary': ['error', 'always-multiline'],
        
        // 要求构造函数首字母大写
        'new-cap': ['error', {
            newIsCap: true,
            capIsNew: false
        }],
        
        // 要求调用无参构造函数时有圆括号
        'new-parens': 'error',
        
        // 要求方法链中每个调用都有一个换行符
        'newline-per-chained-call': ['error', {
            ignoreChainWithDepth: 2
        }],
        
        // 禁用Array构造函数
        'no-array-constructor': 'error',
        
        // 禁用按位运算符
        'no-bitwise': 'error',
        
        // 禁用continue语句
        'no-continue': 'error',
        
        // 禁止在代码后使用内联注释
        'no-inline-comments': 'error',
        
        // 禁止if作为唯一的语句出现在else语句中
        'no-lonely-if': 'error',
        
        // 禁止混合使用不同的操作符
        'no-mixed-operators': 'error',
        
        // 禁止空格和tab的混合缩进
        'no-mixed-spaces-and-tabs': 'error',
        
        // 禁止出现多行空行
        'no-multiple-empty-lines': ['error', {
            max: 2,
            maxEOF: 1,
            maxBOF: 0
        }],
        
        // 禁用否定的表达式
        'no-negated-condition': 'error',
        
        // 禁用嵌套的三元表达式
        'no-nested-ternary': 'error',
        
        // 禁用Object的构造函数
        'no-new-object': 'error',
        
        // 禁用一元操作符++和--
        'no-plusplus': ['error', {
            allowForLoopAfterthoughts: true
        }],
        
        // 禁用特定的语法
        'no-restricted-syntax': ['error', 'WithStatement'],
        
        // 禁用tab
        'no-tabs': 'error',
        
        // 禁用三元操作符
        'no-ternary': 'off',
        
        // 禁止行尾空格
        'no-trailing-spaces': 'error',
        
        // 禁止标识符中有悬空下划线
        'no-underscore-dangle': ['error', {
            allowAfterThis: true,
            allowAfterSuper: true
        }],
        
        // 禁止可以在有更简单的可替代的表达式时使用三元操作符
        'no-unneeded-ternary': 'error',
        
        // 禁止属性前有空白
        'no-whitespace-before-property': 'error',
        
        // 强制单个语句的位置
        'nonblock-statement-body-position': ['error', 'beside'],
        
        // 强制大括号内换行符的一致性
        'object-curly-newline': ['error', {
            multiline: true,
            consistent: true
        }],
        
        // 强制在大括号中使用一致的空格
        'object-curly-spacing': ['error', 'always'],
        
        // 强制将对象的属性放在不同的行上
        'object-property-newline': ['error', {
            allowAllPropertiesOnSameLine: true
        }],
        
        // 强制函数中的变量要么一起声明要么分开声明
        'one-var': ['error', 'never'],
        
        // 要求或禁止在变量声明周围换行
        'one-var-declaration-per-line': ['error', 'always'],
        
        // 要求或禁止在可能的情况下使用简化的赋值操作符
        'operator-assignment': ['error', 'always'],
        
        // 强制操作符使用一致的换行符
        'operator-linebreak': ['error', 'before'],
        
        // 要求或禁止块内填充
        'padded-blocks': ['error', 'never'],
        
        // 要求或禁止在语句间填充空行
        'padding-line-between-statements': [
            'error',
            { blankLine: 'always', prev: '*', next: 'return' },
            { blankLine: 'always', prev: ['const', 'let', 'var'], next: '*' },
            { blankLine: 'any', prev: ['const', 'let', 'var'], next: ['const', 'let', 'var'] }
        ],
        
        // 要求对象字面量属性名称用引号括起来
        'quote-props': ['error', 'as-needed'],
        
        // 强制使用一致的反勾号、双引号或单引号
        'quotes': ['error', 'single', {
            avoidEscape: true,
            allowTemplateLiterals: true
        }],
        
        // 要求或禁止使用分号
        'semi': ['error', 'always'],
        
        // 强制分号之前和之后使用一致的空格
        'semi-spacing': ['error', {
            before: false,
            after: true
        }],
        
        // 强制分号的位置
        'semi-style': ['error', 'last'],
        
        // 要求对象属性按序排列
        'sort-keys': ['error', 'asc', {
            caseSensitive: false,
            natural: true
        }],
        
        // 要求同一个声明块中的变量按顺序排列
        'sort-vars': 'error',
        
        // 强制在块之前使用一致的空格
        'space-before-blocks': 'error',
        
        // 强制在function的左括号之前使用一致的空格
        'space-before-function-paren': ['error', {
            anonymous: 'always',
            named: 'never',
            asyncArrow: 'always'
        }],
        
        // 强制在圆括号内使用一致的空格
        'space-in-parens': ['error', 'never'],
        
        // 要求操作符周围有空格
        'space-infix-ops': 'error',
        
        // 强制在一元操作符前后使用一致的空格
        'space-unary-ops': ['error', {
            words: true,
            nonwords: false
        }],
        
        // 强制在注释中//或/*使用一致的空格
        'spaced-comment': ['error', 'always'],
        
        // 强制在switch的冒号左右有空格
        'switch-colon-spacing': ['error', {
            after: true,
            before: false
        }],
        
        // 要求或禁止在模板标记和它们的字面量之间有空格
        'template-tag-spacing': ['error', 'never'],
        
        // 要求或禁止Unicode字节顺序标记(BOM)
        'unicode-bom': ['error', 'never'],
        
        // 要求正则表达式被括号括起来
        'wrap-regex': 'error'
    }
};
```



### 3.5 ES6+规则（ECMAScript 6）

```javascript
module.exports = {
    rules: {
        // 要求箭头函数体使用大括号
        'arrow-body-style': ['error', 'as-needed'],
        
        // 要求箭头函数的参数使用圆括号
        'arrow-parens': ['error', 'always'],
        
        // 强制箭头函数的箭头前后使用一致的空格
        'arrow-spacing': ['error', {
            before: true,
            after: true
        }],
        
        // 要求在构造函数中有super()的调用
        'constructor-super': 'error',
        
        // 强制generator函数中*号周围使用一致的空格
        'generator-star-spacing': ['error', {
            before: false,
            after: true
        }],
        
        // 禁止修改类声明的变量
        'no-class-assign': 'error',
        
        // 禁止在可能与比较操作符相混淆的地方使用箭头函数
        'no-confusing-arrow': ['error', {
            allowParens: true
        }],
        
        // 禁止修改const声明的变量
        'no-const-assign': 'error',
        
        // 禁止类成员中出现重复的名称
        'no-dupe-class-members': 'error',
        
        // 禁止重复模块导入
        'no-duplicate-imports': 'error',
        
        // 禁止Symbolnew操作符和new一起使用
        'no-new-symbol': 'error',
        
        // 禁止使用指定的import加载的模块
        'no-restricted-imports': ['error', 'lodash'],
        
        // 禁止在构造函数中，在调用super()之前使用this或super
        'no-this-before-super': 'error',
        
        // 禁止在对象中使用不必要的计算属性
        'no-useless-computed-key': 'error',
        
        // 禁用不必要的构造函数
        'no-useless-constructor': 'error',
        
        // 禁止在import和export和解构赋值时将引用重命名为相同的名字
        'no-useless-rename': 'error',
        
        // 要求使用let或const而不是var
        'no-var': 'error',
        
        // 要求或禁止对象字面量中方法和属性使用简写语法
        'object-shorthand': ['error', 'always'],
        
        // 要求使用箭头函数作为回调
        'prefer-arrow-callback': 'error',
        
        // 要求使用const声明那些声明后不再被修改的变量
        'prefer-const': 'error',
        
        // 要求在合适的地方使用解构
        'prefer-destructuring': ['error', {
            array: true,
            object: true
        }, {
            enforceForRenamedProperties: false
        }],
        
        // 禁用parseInt()和Number.parseInt()，使用二进制，八进制和十六进制字面量
        'prefer-numeric-literals': 'error',
        
        // 要求使用剩余参数而不是arguments
        'prefer-rest-params': 'error',
        
        // 要求使用扩展运算符而非.apply()
        'prefer-spread': 'error',
        
        // 要求使用模板字面量而非字符串连接
        'prefer-template': 'error',
        
        // 要求generator函数内有yield
        'require-yield': 'error',
        
        // 强制剩余和扩展运算符及其表达式之间有空格
        'rest-spread-spacing': ['error', 'never'],
        
        // 强制模块内的import排序
        'sort-imports': ['error', {
            ignoreCase: false,
            ignoreDeclarationSort: false,
            ignoreMemberSort: false,
            memberSyntaxSortOrder: ['none', 'all', 'multiple', 'single']
        }],
        
        // 要求symbol描述
        'symbol-description': 'error',
        
        // 要求或禁止模板字符串中的嵌入表达式周围空格的使用
        'template-curly-spacing': ['error', 'never'],
        
        // 强制在yield*表达式中*周围使用空格
        'yield-star-spacing': ['error', 'after']
    }
};
```

---

## 4. Prettier基础

### 4.1 什么是Prettier

Prettier是一个代码格式化工具，支持多种语言，能够自动格式化代码。

**核心特性**:
- 支持多种语言（JavaScript、TypeScript、CSS、HTML等）
- 配置简单
- 与编辑器深度集成
- 支持保存时自动格式化
- 与ESLint配合使用

### 4.2 安装Prettier

```bash
# 项目级安装
npm install --save-dev prettier

# 全局安装（不推荐）
npm install -g prettier
```

### 4.3 基础使用

```bash
# 格式化单个文件
npx prettier --write file.js

# 格式化目录
npx prettier --write src/

# 检查格式（不修改）
npx prettier --check src/

# 指定配置文件
npx prettier --write src/ --config .prettierrc.json
```

### 4.4 Prettier配置

```javascript
// .prettierrc.js
module.exports = {
    // 每行最大长度
    printWidth: 120,
    
    // 缩进空格数
    tabWidth: 4,
    
    // 使用tab缩进
    useTabs: false,
    
    // 语句末尾添加分号
    semi: true,
    
    // 使用单引号
    singleQuote: true,
    
    // 对象属性引号
    quoteProps: 'as-needed',
    
    // JSX使用单引号
    jsxSingleQuote: false,
    
    // 尾随逗号
    trailingComma: 'none',
    
    // 对象字面量的括号间空格
    bracketSpacing: true,
    
    // JSX标签的>单独一行
    bracketSameLine: false,
    
    // 箭头函数参数括号
    arrowParens: 'always',
    
    // 格式化范围
    rangeStart: 0,
    rangeEnd: Infinity,
    
    // 指定解析器
    parser: 'babel',
    
    // 文件路径
    filepath: undefined,
    
    // 是否需要pragma
    requirePragma: false,
    
    // 是否插入pragma
    insertPragma: false,
    
    // Markdown文本换行
    proseWrap: 'preserve',
    
    // HTML空格敏感性
    htmlWhitespaceSensitivity: 'css',
    
    // Vue文件script和style标签缩进
    vueIndentScriptAndStyle: false,
    
    // 换行符
    endOfLine: 'lf',
    
    // 格式化嵌入的代码
    embeddedLanguageFormatting: 'auto'
};
```

### 4.5 忽略文件配置

```javascript
// .prettierignore
# 依赖目录
node_modules/
bower_components/

# 构建产物
dist/
build/
coverage/

# 配置文件
*.config.js
.prettierrc.js

# 第三方库
public/libs/
src/vendor/

# 临时文件
*.tmp
*.log

# Markdown文件
*.md
```

---

## 5. ESLint与Prettier集成

### 5.1 为什么需要集成

ESLint和Prettier各有侧重：
- ESLint：代码质量检查（逻辑错误、最佳实践）
- Prettier：代码格式化（缩进、引号、分号）

集成可以避免规则冲突，统一代码风格。

### 5.2 安装集成插件

```bash
# 安装eslint-config-prettier（禁用ESLint中与Prettier冲突的规则）
npm install --save-dev eslint-config-prettier

# 安装eslint-plugin-prettier（将Prettier作为ESLint规则运行）
npm install --save-dev eslint-plugin-prettier
```

### 5.3 配置集成

```javascript
// .eslintrc.js
module.exports = {
    extends: [
        'eslint:recommended',
        'plugin:react/recommended',
        // 必须放在最后，用于关闭ESLint中与Prettier冲突的规则
        'prettier'
    ],
    plugins: [
        'prettier'
    ],
    rules: {
        // 将Prettier错误作为ESLint错误显示
        'prettier/prettier': 'error'
    }
};
```

### 5.4 完整配置示例

```javascript
// .eslintrc.js
module.exports = {
    env: {
        browser: true,
        es2021: true,
        node: true
    },
    extends: [
        'eslint:recommended',
        'plugin:react/recommended',
        'plugin:@typescript-eslint/recommended',
        'prettier'
    ],
    parser: '@typescript-eslint/parser',
    parserOptions: {
        ecmaFeatures: {
            jsx: true
        },
        ecmaVersion: 'latest',
        sourceType: 'module'
    },
    plugins: [
        'react',
        '@typescript-eslint',
        'prettier'
    ],
    rules: {
        'prettier/prettier': 'error',
        'no-console': 'warn',
        'no-unused-vars': 'off',
        '@typescript-eslint/no-unused-vars': 'error'
    }
};

// .prettierrc.js
module.exports = {
    printWidth: 120,
    tabWidth: 4,
    useTabs: false,
    semi: true,
    singleQuote: true,
    trailingComma: 'none',
    bracketSpacing: true,
    arrowParens: 'always',
    endOfLine: 'lf'
};
```



---

## 6. 编辑器集成

### 6.1 VS Code集成

#### 6.1.1 安装插件

在VS Code中安装以下插件：
- ESLint
- Prettier - Code formatter

#### 6.1.2 配置settings.json

```json
{
    // 启用ESLint
    "eslint.enable": true,
    
    // ESLint验证的语言
    "eslint.validate": [
        "javascript",
        "javascriptreact",
        "typescript",
        "typescriptreact",
        "vue"
    ],
    
    // 保存时自动修复
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": true
    },
    
    // 设置Prettier为默认格式化工具
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    
    // 保存时自动格式化
    "editor.formatOnSave": true,
    
    // 针对特定语言的配置
    "[javascript]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[typescript]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[json]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    
    // Prettier配置
    "prettier.requireConfig": true,
    "prettier.useEditorConfig": false
}
```

#### 6.1.3 工作区配置

```json
// .vscode/settings.json
{
    "eslint.workingDirectories": [
        "./packages/*"
    ],
    "eslint.options": {
        "overrideConfigFile": ".eslintrc.js"
    },
    "prettier.configPath": ".prettierrc.js"
}
```

### 6.2 WebStorm集成

#### 6.2.1 启用ESLint

1. 打开 Settings/Preferences
2. 导航到 Languages & Frameworks > JavaScript > Code Quality Tools > ESLint
3. 选择 Automatic ESLint configuration
4. 勾选 Run eslint --fix on save

#### 6.2.2 启用Prettier

1. 打开 Settings/Preferences
2. 导航到 Languages & Frameworks > JavaScript > Prettier
3. 设置 Prettier package 路径
4. 勾选 On save
5. 勾选 On code reformat

### 6.3 Sublime Text集成

#### 6.3.1 安装Package Control

1. 按 Ctrl+Shift+P (Windows/Linux) 或 Cmd+Shift+P (Mac)
2. 输入 Install Package Control

#### 6.3.2 安装插件

1. 按 Ctrl+Shift+P
2. 输入 Package Control: Install Package
3. 安装 ESLint 和 JsPrettier

#### 6.3.3 配置

```json
// Preferences > Package Settings > JsPrettier > Settings - User
{
    "auto_format_on_save": true,
    "prettier_cli_path": "./node_modules/.bin/prettier"
}
```

---

## 7. Git Hooks集成

### 7.1 使用Husky

#### 7.1.1 安装Husky

```bash
# 安装husky
npm install --save-dev husky

# 初始化husky
npx husky install

# 添加prepare脚本
npm set-script prepare "husky install"
```

#### 7.1.2 添加pre-commit钩子

```bash
# 添加pre-commit钩子
npx husky add .husky/pre-commit "npm run lint"
```

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npm run lint
```

### 7.2 使用lint-staged

#### 7.2.1 安装lint-staged

```bash
npm install --save-dev lint-staged
```

#### 7.2.2 配置lint-staged

```javascript
// package.json
{
    "lint-staged": {
        "*.{js,jsx,ts,tsx}": [
            "eslint --fix",
            "prettier --write"
        ],
        "*.{css,scss,less}": [
            "prettier --write"
        ],
        "*.{json,md}": [
            "prettier --write"
        ]
    }
}

// 或者 .lintstagedrc.js
module.exports = {
    '*.{js,jsx,ts,tsx}': [
        'eslint --fix',
        'prettier --write'
    ],
    '*.{css,scss,less}': [
        'prettier --write'
    ],
    '*.{json,md}': [
        'prettier --write'
    ]
};
```

#### 7.2.3 更新pre-commit钩子

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
```

### 7.3 使用commitlint

#### 7.3.1 安装commitlint

```bash
npm install --save-dev @commitlint/cli @commitlint/config-conventional
```

#### 7.3.2 配置commitlint

```javascript
// commitlint.config.js
module.exports = {
    extends: ['@commitlint/config-conventional'],
    rules: {
        'type-enum': [
            2,
            'always',
            [
                'feat',     // 新功能
                'fix',      // 修复bug
                'docs',     // 文档变更
                'style',    // 代码格式
                'refactor', // 重构
                'perf',     // 性能优化
                'test',     // 测试
                'chore',    // 构建过程或辅助工具的变动
                'revert',   // 回退
                'build'     // 打包
            ]
        ],
        'subject-case': [0]
    }
};
```

#### 7.3.3 添加commit-msg钩子

```bash
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'
```

### 7.4 完整的package.json配置

```json
{
    "name": "my-project",
    "version": "1.0.0",
    "scripts": {
        "prepare": "husky install",
        "lint": "eslint src/ --ext .js,.jsx,.ts,.tsx",
        "lint:fix": "eslint src/ --ext .js,.jsx,.ts,.tsx --fix",
        "format": "prettier --write \"src/**/*.{js,jsx,ts,tsx,css,scss,json,md}\"",
        "format:check": "prettier --check \"src/**/*.{js,jsx,ts,tsx,css,scss,json,md}\""
    },
    "devDependencies": {
        "@commitlint/cli": "^17.0.0",
        "@commitlint/config-conventional": "^17.0.0",
        "eslint": "^8.0.0",
        "eslint-config-prettier": "^8.0.0",
        "eslint-plugin-prettier": "^4.0.0",
        "husky": "^8.0.0",
        "lint-staged": "^13.0.0",
        "prettier": "^2.0.0"
    },
    "lint-staged": {
        "*.{js,jsx,ts,tsx}": [
            "eslint --fix",
            "prettier --write"
        ],
        "*.{css,scss,less}": [
            "prettier --write"
        ],
        "*.{json,md}": [
            "prettier --write"
        ]
    }
}
```

---

## 8. 自定义规则开发

### 8.1 ESLint规则结构

```javascript
// rules/no-console-log.js
module.exports = {
    meta: {
        type: 'suggestion',
        docs: {
            description: '禁止使用console.log',
            category: 'Best Practices',
            recommended: false
        },
        fixable: 'code',
        schema: []
    },
    
    create(context) {
        return {
            CallExpression(node) {
                const callee = node.callee;
                
                if (
                    callee.type === 'MemberExpression' &&
                    callee.object.name === 'console' &&
                    callee.property.name === 'log'
                ) {
                    context.report({
                        node,
                        message: '不允许使用console.log，请使用logger',
                        fix(fixer) {
                            return fixer.replaceText(
                                callee,
                                'logger.info'
                            );
                        }
                    });
                }
            }
        };
    }
};
```

### 8.2 创建ESLint插件

```javascript
// eslint-plugin-custom/index.js
module.exports = {
    rules: {
        'no-console-log': require('./rules/no-console-log'),
        'no-magic-numbers': require('./rules/no-magic-numbers')
    },
    configs: {
        recommended: {
            rules: {
                'custom/no-console-log': 'error',
                'custom/no-magic-numbers': 'warn'
            }
        }
    }
};
```

### 8.3 使用自定义插件

```javascript
// .eslintrc.js
module.exports = {
    plugins: [
        'custom'
    ],
    extends: [
        'plugin:custom/recommended'
    ],
    rules: {
        'custom/no-console-log': 'error'
    }
};
```

### 8.4 测试自定义规则

```javascript
// tests/rules/no-console-log.test.js
const { RuleTester } = require('eslint');
const rule = require('../../rules/no-console-log');

const ruleTester = new RuleTester({
    parserOptions: {
        ecmaVersion: 2021
    }
});

ruleTester.run('no-console-log', rule, {
    valid: [
        'logger.info("test")',
        'console.error("error")',
        'console.warn("warning")'
    ],
    
    invalid: [
        {
            code: 'console.log("test")',
            errors: [{
                message: '不允许使用console.log，请使用logger'
            }],
            output: 'logger.info("test")'
        }
    ]
});
```



---

## 9. 团队规范实践

### 9.1 制定团队规范

#### 9.1.1 规范制定原则

```javascript
// .eslintrc.js - 团队规范示例
module.exports = {
    // 1. 基础配置
    env: {
        browser: true,
        es2021: true,
        node: true
    },
    
    // 2. 继承业界标准
    extends: [
        'eslint:recommended',
        'airbnb-base',
        'prettier'
    ],
    
    // 3. 团队自定义规则
    rules: {
        // 代码质量规则（严格执行）
        'no-console': ['error', {
            allow: ['warn', 'error']
        }],
        'no-debugger': 'error',
        'no-alert': 'error',
        
        // 代码风格规则（适度宽松）
        'max-len': ['warn', {
            code: 120,
            ignoreUrls: true,
            ignoreStrings: true
        }],
        
        // 团队约定规则
        'prefer-const': 'error',
        'no-var': 'error',
        'arrow-body-style': ['error', 'as-needed'],
        
        // 注释规范
        'spaced-comment': ['error', 'always', {
            markers: ['/']
        }]
    }
};
```

#### 9.1.2 分层规范配置

```javascript
// 基础配置 - .eslintrc.base.js
module.exports = {
    env: {
        browser: true,
        es2021: true
    },
    extends: [
        'eslint:recommended',
        'prettier'
    ],
    rules: {
        'no-console': 'warn',
        'no-debugger': 'error'
    }
};

// React项目配置 - .eslintrc.react.js
module.exports = {
    extends: [
        './.eslintrc.base.js',
        'plugin:react/recommended',
        'plugin:react-hooks/recommended'
    ],
    settings: {
        react: {
            version: 'detect'
        }
    },
    rules: {
        'react/prop-types': 'off',
        'react/react-in-jsx-scope': 'off'
    }
};

// TypeScript项目配置 - .eslintrc.ts.js
module.exports = {
    extends: [
        './.eslintrc.base.js',
        'plugin:@typescript-eslint/recommended'
    ],
    parser: '@typescript-eslint/parser',
    rules: {
        '@typescript-eslint/no-explicit-any': 'warn',
        '@typescript-eslint/explicit-function-return-type': 'off'
    }
};
```

### 9.2 规范文档化

#### 9.2.1 编写规范文档

```markdown
# 前端代码规范

## 1. 命名规范

### 1.1 变量命名
- 使用camelCase命名法
- 布尔值以is/has/can开头
- 常量使用UPPER_SNAKE_CASE

```javascript
// ✅ 正确
const userName = 'John';
const isActive = true;
const MAX_COUNT = 100;

// ❌ 错误
const user_name = 'John';
const active = true;
const maxCount = 100;
```

### 1.2 函数命名
- 使用动词开头
- 表达清晰的意图

```javascript
// ✅ 正确
function getUserInfo() {}
function handleClick() {}
function validateForm() {}

// ❌ 错误
function user() {}
function click() {}
function form() {}
```

## 2. 代码格式

### 2.1 缩进
- 使用4个空格缩进
- 禁止使用Tab

### 2.2 引号
- 统一使用单引号
- JSX属性使用双引号

### 2.3 分号
- 语句末尾必须加分号

## 3. 注释规范

### 3.1 单行注释
```javascript
// 获取用户信息
const userInfo = getUserInfo();
```

### 3.2 多行注释
```javascript
/**
 * 用户登录函数
 * @param {string} username - 用户名
 * @param {string} password - 密码
 * @returns {Promise<User>} 用户信息
 */
function login(username, password) {
    // 实现代码
}
```

## 4. 最佳实践

### 4.1 避免使用var
```javascript
// ✅ 正确
const count = 0;
let index = 0;

// ❌ 错误
var count = 0;
```

### 4.2 使用箭头函数
```javascript
// ✅ 正确
const add = (a, b) => a + b;

// ❌ 错误
const add = function(a, b) {
    return a + b;
};
```

### 4.3 对象解构
```javascript
// ✅ 正确
const { name, age } = user;

// ❌ 错误
const name = user.name;
const age = user.age;
```
```

#### 9.2.2 规范检查清单

```markdown
# 代码审查清单

## 代码质量
- [ ] 没有console.log调试代码
- [ ] 没有debugger语句
- [ ] 没有未使用的变量
- [ ] 没有未使用的导入
- [ ] 异常处理完整

## 代码风格
- [ ] 缩进统一（4空格）
- [ ] 引号统一（单引号）
- [ ] 分号使用正确
- [ ] 命名规范符合约定
- [ ] 代码行长度不超过120字符

## 注释文档
- [ ] 复杂逻辑有注释说明
- [ ] 公共函数有JSDoc注释
- [ ] 没有无用的注释

## 最佳实践
- [ ] 使用const/let替代var
- [ ] 使用箭头函数
- [ ] 使用模板字符串
- [ ] 使用对象解构
- [ ] 使用扩展运算符

## 性能优化
- [ ] 避免不必要的重渲染
- [ ] 合理使用useMemo/useCallback
- [ ] 避免在循环中创建函数
- [ ] 合理使用懒加载

## 安全性
- [ ] 输入验证完整
- [ ] 没有XSS漏洞
- [ ] 没有敏感信息泄露
- [ ] API调用有错误处理
```

### 9.3 规范培训

#### 9.3.1 新人培训流程

```javascript
// 培训材料结构
const trainingMaterials = {
    // 第一周：基础规范
    week1: {
        topics: [
            '代码规范概述',
            'ESLint基础使用',
            'Prettier配置',
            '编辑器集成'
        ],
        practice: [
            '配置开发环境',
            '运行第一个检查',
            '修复常见问题'
        ]
    },
    
    // 第二周：进阶规范
    week2: {
        topics: [
            '命名规范详解',
            '注释规范',
            '最佳实践',
            'Git Hooks集成'
        ],
        practice: [
            '编写符合规范的代码',
            '代码审查实践',
            '规范问题修复'
        ]
    },
    
    // 第三周：团队协作
    week3: {
        topics: [
            '团队规范定制',
            '规范文档编写',
            '代码审查流程',
            '持续改进'
        ],
        practice: [
            '参与规范讨论',
            '提交规范建议',
            '审查他人代码'
        ]
    }
};
```

#### 9.3.2 培训考核

```javascript
// 考核题目示例
const examQuestions = [
    {
        question: '以下哪种命名方式符合规范？',
        options: [
            'A. const user_name = "John"',
            'B. const UserName = "John"',
            'C. const userName = "John"',
            'D. const USERNAME = "John"'
        ],
        answer: 'C',
        explanation: '变量名使用camelCase命名法'
    },
    {
        question: '如何正确使用ESLint自动修复？',
        options: [
            'A. eslint --fix src/',
            'B. eslint src/ --fix',
            'C. eslint fix src/',
            'D. A和B都正确'
        ],
        answer: 'D',
        explanation: '--fix参数可以放在命令的任意位置'
    },
    {
        question: '以下哪个不是Prettier的配置项？',
        options: [
            'A. printWidth',
            'B. tabWidth',
            'C. noConsole',
            'D. semi'
        ],
        answer: 'C',
        explanation: 'noConsole是ESLint规则，不是Prettier配置'
    }
];
```

### 9.4 规范落地

#### 9.4.1 CI/CD集成

```yaml
# .github/workflows/lint.yml
name: Lint Check

on:
    pull_request:
        branches: [main, develop]
    push:
        branches: [main, develop]

jobs:
    lint:
        runs-on: ubuntu-latest
        
        steps:
            - name: Checkout代码
              uses: actions/checkout@v3
            
            - name: 安装Node.js
              uses: actions/setup-node@v3
              with:
                  node-version: '18'
            
            - name: 安装依赖
              run: npm ci
            
            - name: 运行ESLint
              run: npm run lint
            
            - name: 运行Prettier检查
              run: npm run format:check
            
            - name: 上传检查报告
              if: failure()
              uses: actions/upload-artifact@v3
              with:
                  name: lint-report
                  path: lint-report.json
```

#### 9.4.2 代码审查流程

```javascript
// 代码审查配置 - .github/CODEOWNERS
# 代码所有者配置
* @team-lead

# 特定目录的审查者
/src/components/ @frontend-team
/src/utils/ @core-team
/src/api/ @backend-team

# 配置文件需要技术负责人审查
*.config.js @tech-lead
.eslintrc.js @tech-lead
.prettierrc.js @tech-lead
```

```markdown
# Pull Request模板 - .github/pull_request_template.md

## 变更说明
<!-- 描述本次变更的内容 -->

## 变更类型
- [ ] 新功能
- [ ] Bug修复
- [ ] 代码重构
- [ ] 文档更新
- [ ] 样式调整
- [ ] 性能优化

## 检查清单
- [ ] 代码通过ESLint检查
- [ ] 代码通过Prettier格式化
- [ ] 添加了必要的注释
- [ ] 更新了相关文档
- [ ] 添加了单元测试
- [ ] 所有测试通过

## 相关Issue
<!-- 关联的Issue编号 -->
Closes #

## 截图
<!-- 如果有UI变更，请提供截图 -->
```

### 9.5 规范维护

#### 9.5.1 定期审查

```javascript
// 规范审查计划
const reviewSchedule = {
    monthly: {
        // 每月审查
        tasks: [
            '收集规范问题反馈',
            '统计规范违规情况',
            '讨论规范优化建议'
        ],
        participants: ['团队成员', '技术负责人']
    },
    
    quarterly: {
        // 每季度审查
        tasks: [
            '评估规范执行效果',
            '更新规范文档',
            '调整规则配置',
            '组织规范培训'
        ],
        participants: ['全体成员', '技术委员会']
    },
    
    yearly: {
        // 每年审查
        tasks: [
            '全面评估规范体系',
            '对比业界最佳实践',
            '制定下一年规范计划',
            '更新工具链'
        ],
        participants: ['技术委员会', '外部专家']
    }
};
```

#### 9.5.2 规范演进

```javascript
// 规范版本管理
const ruleVersions = {
    'v1.0.0': {
        date: '2024-01-01',
        changes: [
            '初始版本',
            '基础规则配置',
            'ESLint + Prettier集成'
        ]
    },
    
    'v1.1.0': {
        date: '2024-04-01',
        changes: [
            '新增TypeScript规则',
            '优化React Hooks规则',
            '添加自定义规则'
        ],
        migration: {
            breaking: [
                '移除no-console规则的allow配置'
            ],
            steps: [
                '更新.eslintrc.js配置',
                '运行npm run lint:fix',
                '手动修复剩余问题'
            ]
        }
    },
    
    'v2.0.0': {
        date: '2024-07-01',
        changes: [
            '升级到ESLint 9.0',
            '采用Flat Config',
            '重构规则体系'
        ],
        migration: {
            breaking: [
                '配置文件格式变更',
                '部分规则名称变更',
                '插件配置方式变更'
            ],
            steps: [
                '阅读迁移指南',
                '使用迁移工具转换配置',
                '测试并修复问题',
                '更新CI/CD配置'
            ]
        }
    }
};
```



---

## 10. 常见问题与最佳实践

### 10.1 常见问题

#### 10.1.1 ESLint与Prettier冲突

```javascript
// 问题：ESLint和Prettier规则冲突
// 解决方案：使用eslint-config-prettier

// 安装
npm install --save-dev eslint-config-prettier

// 配置（prettier必须放在extends数组最后）
module.exports = {
    extends: [
        'eslint:recommended',
        'plugin:react/recommended',
        'prettier'  // 必须放在最后
    ]
};
```

#### 10.1.2 规则过于严格

```javascript
// 问题：某些规则过于严格，影响开发效率
// 解决方案：调整规则级别或禁用特定规则

module.exports = {
    rules: {
        // 将error改为warn
        'no-console': 'warn',
        
        // 针对特定场景禁用
        'max-len': ['error', {
            code: 120,
            ignoreUrls: true,
            ignoreStrings: true,
            ignoreTemplateLiterals: true
        }],
        
        // 完全禁用
        'no-plusplus': 'off'
    }
};
```

#### 10.1.3 性能问题

```javascript
// 问题：ESLint检查速度慢
// 解决方案：优化配置和使用缓存

module.exports = {
    // 1. 限制检查范围
    ignorePatterns: [
        'node_modules/',
        'dist/',
        'build/',
        '*.min.js'
    ],
    
    // 2. 使用缓存
    cache: true,
    cacheLocation: '.eslintcache',
    
    // 3. 只检查变更的文件（配合lint-staged）
    // package.json
    "lint-staged": {
        "*.{js,jsx,ts,tsx}": [
            "eslint --cache --fix"
        ]
    }
};
```

#### 10.1.4 Monorepo配置

```javascript
// 问题：Monorepo项目中不同包需要不同配置
// 解决方案：使用overrides或多个配置文件

// 根目录 .eslintrc.js
module.exports = {
    root: true,
    extends: ['eslint:recommended'],
    
    overrides: [
        // React应用配置
        {
            files: ['packages/web/**/*.{js,jsx}'],
            extends: ['plugin:react/recommended'],
            env: {
                browser: true
            }
        },
        
        // Node.js服务配置
        {
            files: ['packages/server/**/*.js'],
            env: {
                node: true
            },
            rules: {
                'no-console': 'off'
            }
        },
        
        // 工具包配置
        {
            files: ['packages/utils/**/*.js'],
            rules: {
                'no-console': 'error'
            }
        }
    ]
};
```

### 10.2 最佳实践

#### 10.2.1 渐进式引入

```javascript
// 阶段1：只启用错误级别规则
module.exports = {
    extends: ['eslint:recommended'],
    rules: {
        // 只修复明显的错误
        'no-undef': 'error',
        'no-unused-vars': 'error'
    }
};

// 阶段2：引入警告级别规则
module.exports = {
    extends: ['eslint:recommended'],
    rules: {
        'no-undef': 'error',
        'no-unused-vars': 'error',
        // 添加警告规则
        'no-console': 'warn',
        'prefer-const': 'warn'
    }
};

// 阶段3：完整规范
module.exports = {
    extends: [
        'eslint:recommended',
        'airbnb-base',
        'prettier'
    ],
    rules: {
        // 团队自定义规则
    }
};
```

#### 10.2.2 规则分类管理

```javascript
// eslint-rules/errors.js - 错误规则
module.exports = {
    rules: {
        'no-console': 'error',
        'no-debugger': 'error',
        'no-alert': 'error'
    }
};

// eslint-rules/best-practices.js - 最佳实践
module.exports = {
    rules: {
        'eqeqeq': 'error',
        'no-eval': 'error',
        'prefer-const': 'error'
    }
};

// eslint-rules/style.js - 代码风格
module.exports = {
    rules: {
        'indent': ['error', 4],
        'quotes': ['error', 'single'],
        'semi': ['error', 'always']
    }
};

// .eslintrc.js - 主配置
module.exports = {
    extends: [
        './eslint-rules/errors',
        './eslint-rules/best-practices',
        './eslint-rules/style'
    ]
};
```

#### 10.2.3 自动化修复策略

```javascript
// package.json
{
    "scripts": {
        // 检查但不修复
        "lint": "eslint src/",
        
        // 自动修复
        "lint:fix": "eslint src/ --fix",
        
        // 只检查变更的文件
        "lint:staged": "lint-staged",
        
        // 格式化
        "format": "prettier --write \"src/**/*.{js,jsx,ts,tsx,css,scss,json,md}\"",
        
        // 检查格式
        "format:check": "prettier --check \"src/**/*.{js,jsx,ts,tsx,css,scss,json,md}\"",
        
        // 完整检查
        "check": "npm run lint && npm run format:check"
    }
}
```

#### 10.2.4 团队协作规范

```javascript
// 1. 统一工具版本
// package.json
{
    "engines": {
        "node": ">=18.0.0",
        "npm": ">=9.0.0"
    },
    "devDependencies": {
        "eslint": "8.50.0",
        "prettier": "3.0.0"
    }
}

// 2. 锁定依赖版本
// 使用package-lock.json或yarn.lock

// 3. 编辑器配置统一
// .editorconfig
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 4
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false

// 4. Git配置
// .gitattributes
* text=auto eol=lf
*.js text eol=lf
*.jsx text eol=lf
*.ts text eol=lf
*.tsx text eol=lf
```

#### 10.2.5 性能优化技巧

```javascript
// 1. 使用ESLint缓存
// .eslintrc.js
module.exports = {
    cache: true,
    cacheLocation: '.eslintcache',
    cacheStrategy: 'content'
};

// 2. 并行处理
// package.json
{
    "scripts": {
        "lint": "eslint --cache --max-warnings=0 src/"
    }
}

// 3. 只检查必要的文件
// .eslintignore
node_modules/
dist/
build/
coverage/
*.min.js
*.bundle.js

// 4. 使用lint-staged只检查变更文件
// package.json
{
    "lint-staged": {
        "*.{js,jsx,ts,tsx}": [
            "eslint --cache --fix",
            "prettier --write"
        ]
    }
}
```

### 10.3 故障排查

#### 10.3.1 配置不生效

```bash
# 检查配置文件是否被正确加载
npx eslint --print-config src/index.js

# 检查规则是否启用
npx eslint --debug src/index.js

# 清除缓存
rm -rf .eslintcache
rm -rf node_modules/.cache
```

#### 10.3.2 插件冲突

```javascript
// 问题：多个插件规则冲突
// 解决方案：明确规则优先级

module.exports = {
    extends: [
        'plugin:react/recommended',
        'plugin:@typescript-eslint/recommended',
        'prettier'  // prettier必须最后，用于禁用冲突规则
    ],
    rules: {
        // 明确覆盖规则
        'react/prop-types': 'off',
        '@typescript-eslint/explicit-function-return-type': 'off'
    }
};
```

#### 10.3.3 编辑器集成问题

```json
// VS Code settings.json
{
    // 启用ESLint
    "eslint.enable": true,
    
    // 指定工作目录
    "eslint.workingDirectories": [
        { "mode": "auto" }
    ],
    
    // 验证的语言
    "eslint.validate": [
        "javascript",
        "javascriptreact",
        "typescript",
        "typescriptreact"
    ],
    
    // 保存时自动修复
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": true
    },
    
    // 禁用其他格式化工具
    "javascript.format.enable": false,
    "typescript.format.enable": false
}
```

### 10.4 实战案例

#### 10.4.1 React项目完整配置

```javascript
// .eslintrc.js
module.exports = {
    env: {
        browser: true,
        es2021: true,
        node: true,
        jest: true
    },
    
    extends: [
        'eslint:recommended',
        'plugin:react/recommended',
        'plugin:react-hooks/recommended',
        'plugin:jsx-a11y/recommended',
        'plugin:import/recommended',
        'prettier'
    ],
    
    parser: '@babel/eslint-parser',
    
    parserOptions: {
        ecmaFeatures: {
            jsx: true
        },
        ecmaVersion: 'latest',
        sourceType: 'module',
        requireConfigFile: false,
        babelOptions: {
            presets: ['@babel/preset-react']
        }
    },
    
    plugins: [
        'react',
        'react-hooks',
        'jsx-a11y',
        'import'
    ],
    
    settings: {
        react: {
            version: 'detect'
        },
        'import/resolver': {
            node: {
                extensions: ['.js', '.jsx']
            }
        }
    },
    
    rules: {
        // React规则
        'react/react-in-jsx-scope': 'off',
        'react/prop-types': 'off',
        'react/jsx-uses-react': 'off',
        
        // Hooks规则
        'react-hooks/rules-of-hooks': 'error',
        'react-hooks/exhaustive-deps': 'warn',
        
        // Import规则
        'import/order': ['error', {
            groups: [
                'builtin',
                'external',
                'internal',
                'parent',
                'sibling',
                'index'
            ],
            'newlines-between': 'always',
            alphabetize: {
                order: 'asc',
                caseInsensitive: true
            }
        }],
        
        // 通用规则
        'no-console': ['warn', {
            allow: ['warn', 'error']
        }],
        'no-unused-vars': ['error', {
            argsIgnorePattern: '^_'
        }],
        'prefer-const': 'error',
        'no-var': 'error'
    }
};

// .prettierrc.js
module.exports = {
    printWidth: 120,
    tabWidth: 4,
    useTabs: false,
    semi: true,
    singleQuote: true,
    quoteProps: 'as-needed',
    jsxSingleQuote: false,
    trailingComma: 'none',
    bracketSpacing: true,
    bracketSameLine: false,
    arrowParens: 'always',
    endOfLine: 'lf'
};
```

#### 10.4.2 TypeScript项目完整配置

```javascript
// .eslintrc.js
module.exports = {
    env: {
        browser: true,
        es2021: true,
        node: true
    },
    
    extends: [
        'eslint:recommended',
        'plugin:@typescript-eslint/recommended',
        'plugin:react/recommended',
        'plugin:react-hooks/recommended',
        'prettier'
    ],
    
    parser: '@typescript-eslint/parser',
    
    parserOptions: {
        ecmaFeatures: {
            jsx: true
        },
        ecmaVersion: 'latest',
        sourceType: 'module',
        project: './tsconfig.json'
    },
    
    plugins: [
        '@typescript-eslint',
        'react',
        'react-hooks'
    ],
    
    settings: {
        react: {
            version: 'detect'
        }
    },
    
    rules: {
        // TypeScript规则
        '@typescript-eslint/no-explicit-any': 'warn',
        '@typescript-eslint/explicit-function-return-type': 'off',
        '@typescript-eslint/explicit-module-boundary-types': 'off',
        '@typescript-eslint/no-unused-vars': ['error', {
            argsIgnorePattern: '^_'
        }],
        
        // React规则
        'react/react-in-jsx-scope': 'off',
        'react/prop-types': 'off',
        
        // 通用规则
        'no-console': 'warn',
        'prefer-const': 'error'
    }
};
```

---

## 总结

ESLint和Prettier是现代前端开发中不可或缺的工具，它们帮助团队：

1. **统一代码风格**：消除个人编码习惯差异
2. **提高代码质量**：及早发现潜在问题
3. **提升开发效率**：自动化格式化和修复
4. **降低维护成本**：代码更易读、更易维护
5. **促进团队协作**：统一的规范减少沟通成本

### 关键要点

- ESLint负责代码质量检查，Prettier负责代码格式化
- 使用eslint-config-prettier避免规则冲突
- 通过Git Hooks确保代码提交前通过检查
- 渐进式引入规范，避免一次性改动过大
- 定期审查和更新规范，保持与业界最佳实践同步

### 学习建议

1. 从基础配置开始，逐步深入
2. 理解每个规则的意义，而不是盲目遵守
3. 根据团队实际情况调整规则
4. 持续学习业界最佳实践
5. 积极参与团队规范讨论

---

**最后更新时间：** 2026-02-27  
**@author** erik.zhou

