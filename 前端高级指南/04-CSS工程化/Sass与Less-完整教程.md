# Sass与Less - 完整教程

## 课程信息
- **课程名称**: Sass与Less完整教程
- **难度级别**: 中级
- **预计学时**: 8小时
- **核心内容**: CSS预处理器、变量、嵌套、混合、函数、模块化
- **@author**: erik.zhou

---

## 目录
1. [CSS预处理器概述](#1-css预处理器概述)
2. [Sass基础](#2-sass基础)
3. [Sass进阶](#3-sass进阶)
4. [Less基础](#4-less基础)
5. [Less进阶](#5-less进阶)
6. [Sass vs Less](#6-sass-vs-less)
7. [构建工具集成](#7-构建工具集成)
8. [最佳实践](#8-最佳实践)
9. [性能优化](#9-性能优化)
10. [实战案例](#10-实战案例)

---

## 1. CSS预处理器概述

### 1.1 什么是CSS预处理器

CSS预处理器是一种脚本语言，扩展了CSS的功能，最终编译成标准的CSS。

**核心特性**:
- 变量
- 嵌套
- 混合（Mixins）
- 函数
- 运算
- 模块化

### 1.2 为什么使用预处理器

```scss
// 传统CSS的问题
.header {
    background-color: #3498db;
}

.header .nav {
    color: #3498db;
}

.header .nav .item {
    border-color: #3498db;
}

// 使用Sass解决
$primary-color: #3498db;

.header {
    background-color: $primary-color;
    
    .nav {
        color: $primary-color;
        
        .item {
            border-color: $primary-color;
        }
    }
}
```

**优势**:
1. **提高可维护性**：变量和混合减少重复
2. **增强可读性**：嵌套反映HTML结构
3. **提升开发效率**：函数和运算简化计算
4. **模块化**：@import和@use组织代码

---

## 2. Sass基础

### 2.1 安装和使用

```bash
# 安装Sass
npm install -g sass

# 编译单个文件
sass input.scss output.css

# 监听文件变化
sass --watch input.scss:output.css

# 监听目录
sass --watch scss:css

# 压缩输出
sass input.scss output.css --style=compressed
```

### 2.2 变量

```scss
// 定义变量
$primary-color: #3498db;
$secondary-color: #2ecc71;
$font-size-base: 16px;
$font-family: 'Helvetica Neue', Arial, sans-serif;

// 使用变量
body {
    font-family: $font-family;
    font-size: $font-size-base;
    color: $primary-color;
}

// 变量作用域
$global-var: red;

.container {
    $local-var: blue;  // 局部变量
    color: $local-var;
}

// 默认变量
$primary-color: #3498db !default;  // 如果已定义则不覆盖

// 变量插值
$side: left;

.box {
    margin-#{$side}: 10px;  // margin-left: 10px
}
```

### 2.3 嵌套

```scss
// 基础嵌套
.nav {
    background: #333;
    
    ul {
        list-style: none;
        margin: 0;
        padding: 0;
    }
    
    li {
        display: inline-block;
        
        a {
            color: white;
            text-decoration: none;
            padding: 10px 15px;
            
            &:hover {  // & 表示父选择器
                background: #555;
            }
        }
    }
}

// 属性嵌套
.box {
    font: {
        family: Arial;
        size: 16px;
        weight: bold;
    }
    
    border: {
        top: 1px solid #ccc;
        bottom: 1px solid #ccc;
    }
}

// 编译结果
.box {
    font-family: Arial;
    font-size: 16px;
    font-weight: bold;
    border-top: 1px solid #ccc;
    border-bottom: 1px solid #ccc;
}
```

### 2.4 混合（Mixins）

```scss
// 定义混合
@mixin border-radius($radius) {
    -webkit-border-radius: $radius;
    -moz-border-radius: $radius;
    border-radius: $radius;
}

// 使用混合
.button {
    @include border-radius(5px);
}

// 带默认参数的混合
@mixin box-shadow($x: 0, $y: 2px, $blur: 4px, $color: rgba(0,0,0,0.1)) {
    box-shadow: $x $y $blur $color;
}

.card {
    @include box-shadow();  // 使用默认值
}

.modal {
    @include box-shadow(0, 4px, 8px, rgba(0,0,0,0.2));  // 自定义值
}

// 可变参数
@mixin transition($properties...) {
    transition: $properties;
}

.button {
    @include transition(background 0.3s, color 0.3s, transform 0.2s);
}

// 内容块混合
@mixin media-query($breakpoint) {
    @if $breakpoint == mobile {
        @media (max-width: 767px) {
            @content;
        }
    } @else if $breakpoint == tablet {
        @media (min-width: 768px) and (max-width: 1023px) {
            @content;
        }
    } @else if $breakpoint == desktop {
        @media (min-width: 1024px) {
            @content;
        }
    }
}

.container {
    width: 100%;
    
    @include media-query(mobile) {
        padding: 10px;
    }
    
    @include media-query(desktop) {
        max-width: 1200px;
        margin: 0 auto;
    }
}
```


### 2.5 继承

```scss
// 基础继承
.message {
    padding: 15px;
    border: 1px solid #ccc;
    border-radius: 4px;
}

.success {
    @extend .message;
    border-color: #2ecc71;
    background: #d4edda;
}

.error {
    @extend .message;
    border-color: #e74c3c;
    background: #f8d7da;
}

// 占位符选择器（不会被编译）
%button-base {
    display: inline-block;
    padding: 10px 20px;
    border: none;
    cursor: pointer;
    text-align: center;
}

.button-primary {
    @extend %button-base;
    background: #3498db;
    color: white;
}

.button-secondary {
    @extend %button-base;
    background: #95a5a6;
    color: white;
}

// 多重继承
.alert {
    padding: 10px;
}

.important {
    font-weight: bold;
}

.alert-important {
    @extend .alert;
    @extend .important;
    color: red;
}
```

### 2.6 导入

```scss
// 传统@import（已废弃）
@import 'variables';
@import 'mixins';
@import 'base';

// 部分文件（Partials）
// _variables.scss
$primary-color: #3498db;
$secondary-color: #2ecc71;

// main.scss
@import 'variables';  // 不需要下划线和扩展名

// 导入多个文件
@import 'variables', 'mixins', 'base';

// 嵌套导入
.theme-dark {
    @import 'dark-theme';
}

// 导入CSS文件
@import url('https://fonts.googleapis.com/css2?family=Roboto');
```

### 2.7 运算符

```scss
// 数学运算
$width: 100px;

.container {
    width: $width * 2;        // 200px
    height: $width / 2;       // 50px
    margin: $width + 20px;    // 120px
    padding: $width - 10px;   // 90px
}

// 除法运算（需要注意）
.box {
    // 使用括号
    width: (100px / 2);       // 50px
    
    // 使用变量
    $total: 100px;
    width: $total / 2;        // 50px
    
    // 使用其他运算
    width: 100px / 2 + 0px;   // 50px
}

// 颜色运算
$base-color: #3498db;

.box {
    background: $base-color + #111;     // 颜色变亮
    border-color: $base-color - #111;   // 颜色变暗
}

// 字符串运算
$family: 'Arial';

.text {
    font-family: $family + ', sans-serif';  // 'Arial, sans-serif'
}

// 比较运算
$width: 100px;

.box {
    @if $width > 50px {
        padding: 20px;
    }
}

// 逻辑运算
$mobile: true;
$tablet: false;

.container {
    @if $mobile and not $tablet {
        width: 100%;
    }
}
```

---

## 3. Sass进阶

### 3.1 函数

```scss
// 内置颜色函数
$primary: #3498db;

.button {
    background: $primary;
    
    &:hover {
        background: darken($primary, 10%);    // 变暗10%
    }
    
    &:active {
        background: lighten($primary, 10%);   // 变亮10%
    }
}

// 更多颜色函数
.color-demo {
    // 调整色相
    color: adjust-hue($primary, 45deg);
    
    // 调整饱和度
    background: saturate($primary, 20%);
    border-color: desaturate($primary, 20%);
    
    // 调整透明度
    box-shadow: 0 2px 4px rgba($primary, 0.3);
    
    // 混合颜色
    background: mix(#3498db, #2ecc71, 50%);
    
    // 获取颜色通道
    $red: red($primary);
    $green: green($primary);
    $blue: blue($primary);
}

// 自定义函数
@function calculate-rem($size) {
    $rem-size: $size / 16px;
    @return #{$rem-size}rem;
}

.text {
    font-size: calculate-rem(24px);  // 1.5rem
}

// 带默认参数的函数
@function strip-unit($number) {
    @if type-of($number) == 'number' and not unitless($number) {
        @return $number / ($number * 0 + 1);
    }
    @return $number;
}

// 复杂函数示例
@function power($base, $exponent) {
    $result: 1;
    @for $i from 1 through $exponent {
        $result: $result * $base;
    }
    @return $result;
}

.box {
    width: power(2, 3) * 10px;  // 80px (2^3 * 10)
}
```

### 3.2 控制指令

```scss
// @if 条件判断
$theme: dark;

.container {
    @if $theme == dark {
        background: #333;
        color: white;
    } @else if $theme == light {
        background: white;
        color: #333;
    } @else {
        background: #f5f5f5;
        color: #666;
    }
}

// @for 循环
@for $i from 1 through 12 {
    .col-#{$i} {
        width: percentage($i / 12);
    }
}

// 编译结果
// .col-1 { width: 8.33333%; }
// .col-2 { width: 16.66667%; }
// ...
// .col-12 { width: 100%; }

// @each 遍历列表
$colors: (
    primary: #3498db,
    success: #2ecc71,
    warning: #f39c12,
    danger: #e74c3c
);

@each $name, $color in $colors {
    .btn-#{$name} {
        background: $color;
        
        &:hover {
            background: darken($color, 10%);
        }
    }
}

// @while 循环
$i: 1;
@while $i <= 6 {
    .heading-#{$i} {
        font-size: 32px - ($i * 2px);
    }
    $i: $i + 1;
}

// 遍历简单列表
$sizes: 10px, 20px, 30px, 40px;

@each $size in $sizes {
    .margin-#{$size} {
        margin: $size;
    }
}

// 遍历多维列表
$icons: (
    home '\f015',
    user '\f007',
    search '\f002'
);

@each $name, $code in $icons {
    .icon-#{$name}::before {
        content: $code;
    }
}
```

### 3.3 模块系统（@use和@forward）

```scss
// _variables.scss
$primary-color: #3498db;
$secondary-color: #2ecc71;

// _mixins.scss
@mixin button-style {
    padding: 10px 20px;
    border: none;
    cursor: pointer;
}

// main.scss（使用@use）
@use 'variables';
@use 'mixins';

.button {
    background: variables.$primary-color;
    @include mixins.button-style;
}

// 使用别名
@use 'variables' as vars;
@use 'mixins' as *;  // 导入所有成员到当前命名空间

.button {
    background: vars.$primary-color;
    @include button-style;  // 不需要前缀
}

// 配置模块
// _theme.scss
$primary: #3498db !default;
$secondary: #2ecc71 !default;

// main.scss
@use 'theme' with (
    $primary: #e74c3c,
    $secondary: #f39c12
);

// @forward 转发模块
// _index.scss
@forward 'variables';
@forward 'mixins';
@forward 'functions';

// main.scss
@use 'index';  // 一次性导入所有模块

.button {
    background: index.$primary-color;
}
```

### 3.4 Maps和Lists

```scss
// Maps（映射）
$colors: (
    primary: #3498db,
    secondary: #2ecc71,
    danger: #e74c3c
);

// 访问Map值
.button {
    background: map-get($colors, primary);
}

// Map函数
$theme: (
    primary: #3498db,
    secondary: #2ecc71,
    text: #333
);

.demo {
    // 检查键是否存在
    @if map-has-key($theme, primary) {
        color: map-get($theme, primary);
    }
    
    // 获取所有键
    // map-keys($theme) => (primary, secondary, text)
    
    // 获取所有值
    // map-values($theme) => (#3498db, #2ecc71, #333)
    
    // 合并Map
    $extended: map-merge($theme, (accent: #f39c12));
}

// 嵌套Map
$breakpoints: (
    mobile: (
        min: 0,
        max: 767px
    ),
    tablet: (
        min: 768px,
        max: 1023px
    ),
    desktop: (
        min: 1024px,
        max: null
    )
);

// 访问嵌套值
$mobile-max: map-get(map-get($breakpoints, mobile), max);

// Lists（列表）
$fonts: 'Helvetica', 'Arial', sans-serif;
$sizes: 12px 14px 16px 18px 20px;

.text {
    font-family: $fonts;
    
    // 列表函数
    // length($fonts) => 3
    // nth($fonts, 2) => 'Arial'
    // index($fonts, 'Arial') => 2
}

// 列表操作
$list: 10px 20px 30px;

.box {
    // 追加元素
    $new-list: append($list, 40px);  // 10px 20px 30px 40px
    
    // 合并列表
    $merged: join($list, (40px 50px));  // 10px 20px 30px 40px 50px
    
    // 列表切片
    // 需要自定义函数实现
}
```

---

## 4. Less基础

### 4.1 安装和使用

```bash
# 安装Less
npm install -g less

# 编译文件
lessc input.less output.css

# 压缩输出
lessc input.less output.css --compress

# 生成Source Map
lessc input.less output.css --source-map
```

### 4.2 变量

```less
// 定义变量
@primary-color: #3498db;
@secondary-color: #2ecc71;
@font-size: 16px;

// 使用变量
body {
    color: @primary-color;
    font-size: @font-size;
}

// 变量插值
@side: left;

.box {
    margin-@{side}: 10px;  // margin-left: 10px
}

// 选择器插值
@selector: button;

.@{selector} {
    padding: 10px;
}

// URL插值
@images: '../images';

.logo {
    background: url('@{images}/logo.png');
}

// 属性名插值
@property: color;

.text {
    @{property}: red;
}

// 变量的变量
@primary: blue;
@color-name: primary;

.box {
    color: @@color-name;  // blue
}

// 延迟加载（Lazy Loading）
@var: 0;
.class {
    @var: 1;
    .brass {
        @var: 2;
        three: @var;  // 3
        @var: 3;
    }
    one: @var;  // 1
}
```

### 4.3 嵌套

```less
// 基础嵌套
.nav {
    background: #333;
    
    ul {
        list-style: none;
    }
    
    li {
        display: inline-block;
        
        a {
            color: white;
            
            &:hover {
                color: #3498db;
            }
        }
    }
}

// @规则嵌套
.component {
    width: 300px;
    
    @media (min-width: 768px) {
        width: 600px;
    }
    
    @supports (display: grid) {
        display: grid;
    }
}

// 编译结果
.component {
    width: 300px;
}

@media (min-width: 768px) {
    .component {
        width: 600px;
    }
}

@supports (display: grid) {
    .component {
        display: grid;
    }
}
```

### 4.4 混合（Mixins）

```less
// 基础混合
.border-radius(@radius) {
    -webkit-border-radius: @radius;
    -moz-border-radius: @radius;
    border-radius: @radius;
}

.button {
    .border-radius(5px);
}

// 带默认值的混合
.box-shadow(@x: 0, @y: 2px, @blur: 4px, @color: rgba(0,0,0,0.1)) {
    box-shadow: @x @y @blur @color;
}

.card {
    .box-shadow();  // 使用默认值
}

// 不输出的混合
.hidden-mixin() {
    color: red;
}

.visible-mixin {
    color: blue;
}

// 只有.visible-mixin会被编译到CSS

// 模式匹配
.mixin(dark, @color) {
    background: darken(@color, 10%);
}

.mixin(light, @color) {
    background: lighten(@color, 10%);
}

.mixin(@_, @color) {
    color: @color;
}

.button-dark {
    .mixin(dark, #3498db);
}

.button-light {
    .mixin(light, #3498db);
}

// Guards（守卫）
.mixin(@a) when (@a > 10) {
    width: 100px;
}

.mixin(@a) when (@a <= 10) {
    width: 50px;
}

.box1 {
    .mixin(15);  // width: 100px
}

.box2 {
    .mixin(5);   // width: 50px
}

// 多条件守卫
.mixin(@a) when (@a > 10) and (@a < 20) {
    width: 100px;
}

.mixin(@a) when (@a = 10), (@a = 20) {
    width: 50px;
}
```

### 4.5 运算

```less
// 数学运算
@width: 100px;

.container {
    width: @width * 2;        // 200px
    height: @width / 2;       // 50px
    margin: @width + 20px;    // 120px
    padding: @width - 10px;   // 90px
}

// 颜色运算
@base-color: #3498db;

.box {
    background: @base-color + #111;
    border-color: @base-color - #111;
}

// calc()运算
.box {
    width: calc(100% - 50px);
    height: calc(~"100vh - 80px");  // 使用~转义
}

// 字符串运算
@base-url: 'https://example.com';
@path: '/images';

.image {
    background: url('@{base-url}@{path}/logo.png');
}
```

---

## 5. Less进阶

### 5.1 函数

```less
// 颜色函数
@primary: #3498db;

.button {
    background: @primary;
    
    &:hover {
        background: darken(@primary, 10%);
    }
    
    &:active {
        background: lighten(@primary, 10%);
    }
}

// 更多颜色函数
.color-demo {
    // 饱和度
    color: saturate(@primary, 20%);
    background: desaturate(@primary, 20%);
    
    // 透明度
    border-color: fade(@primary, 50%);
    box-shadow: 0 2px 4px fadein(@primary, 20%);
    
    // 混合颜色
    background: mix(@primary, #2ecc71, 50%);
    
    // 色调旋转
    color: spin(@primary, 45);
}

// 数学函数
.math-demo {
    width: ceil(10.3px);      // 11px
    height: floor(10.8px);    // 10px
    margin: round(10.5px);    // 11px
    padding: percentage(0.5); // 50%
    
    // 最小值和最大值
    width: min(100px, 50%);
    height: max(100px, 50%);
}

// 字符串函数
@string: 'Hello World';

.string-demo {
    // 转义
    content: e(@string);
    
    // 替换
    content: replace(@string, 'World', 'Less');
}

// 列表函数
@list: 10px, 20px, 30px;

.list-demo {
    // 获取长度
    // length(@list) => 3
    
    // 提取元素
    margin: extract(@list, 2);  // 20px
}
```

### 5.2 命名空间

```less
// 定义命名空间
#bundle() {
    .button {
        display: inline-block;
        padding: 10px 20px;
        border: none;
    }
    
    .tab {
        background: #f5f5f5;
        border: 1px solid #ddd;
    }
    
    .citation {
        font-style: italic;
    }
}

// 使用命名空间
.my-button {
    #bundle.button();
}

.my-tab {
    #bundle.tab();
}

// 嵌套命名空间
#theme() {
    #dark() {
        @background: #333;
        @color: white;
    }
    
    #light() {
        @background: white;
        @color: #333;
    }
}

.dark-theme {
    background: #theme.#dark[@background];
    color: #theme.#dark[@color];
}
```

### 5.3 作用域

```less
// 变量作用域
@color: red;

.container {
    @color: blue;
    
    .box {
        color: @color;  // blue（就近原则）
    }
}

.other {
    color: @color;  // red
}

// 混合作用域
.mixin() {
    @width: 100px;
}

.box {
    .mixin();
    width: @width;  // 100px（混合中的变量可以被访问）
}

// 父选择器作用域
.parent {
    @var: parent;
    
    .child {
        @var: child;
        value: @var;  // child
        
        .grandchild {
            value: @var;  // child（继承父级）
        }
    }
}
```

### 5.4 导入

```less
// 导入Less文件
@import 'variables';
@import 'mixins.less';

// 导入CSS文件
@import (css) 'reset.css';

// 导入选项
@import (reference) 'library.less';  // 只导入，不输出
@import (inline) 'external.css';     // 内联导入
@import (less) 'styles.css';         // 作为Less文件导入
@import (once) 'common.less';        // 只导入一次（默认）
@import (multiple) 'theme.less';     // 允许多次导入

// 条件导入
@import 'mobile.less' (max-width: 767px);
```

---

## 6. Sass vs Less

### 6.1 语法对比

```scss
// Sass变量
$primary-color: #3498db;

// Less变量
@primary-color: #3498db;
```

```scss
// Sass混合
@mixin border-radius($radius) {
    border-radius: $radius;
}

// Less混合
.border-radius(@radius) {
    border-radius: @radius;
}
```

```scss
// Sass条件判断
@if $theme == dark {
    background: #333;
}

// Less条件判断（使用Guards）
.mixin() when (@theme = dark) {
    background: #333;
}
```

### 6.2 功能对比

| 特性 | Sass | Less | 说明 |
|------|------|------|------|
| 变量符号 | $ | @ | Sass使用$，Less使用@ |
| 混合定义 | @mixin | .mixin() | Sass用@mixin，Less用类选择器 |
| 混合调用 | @include | .mixin() | Sass需要@include，Less直接调用 |
| 条件判断 | @if/@else | Guards | Sass更直观，Less使用守卫 |
| 循环 | @for/@each/@while | 递归混合 | Sass内置循环，Less需要递归 |
| 模块系统 | @use/@forward | @import | Sass有现代模块系统 |
| 继承 | @extend | :extend() | 语法不同 |
| 函数 | @function | 混合+Guards | Sass有独立函数 |
| 运行环境 | Dart/Ruby | JavaScript | Sass需要编译器，Less可浏览器运行 |

### 6.3 选择建议

**选择Sass的场景**:
- 大型项目，需要强大的模块系统
- 需要复杂的逻辑控制（循环、条件）
- 团队熟悉Ruby/Dart生态
- 需要丰富的内置函数
- 追求最佳实践和现代化

**选择Less的场景**:
- 中小型项目
- 需要浏览器端编译
- 团队熟悉JavaScript生态
- 快速上手，学习成本低
- 与Bootstrap等框架集成

---

## 7. 构建工具集成

### 7.1 Webpack集成

```javascript
// webpack.config.js

// Sass配置
module.exports = {
    module: {
        rules: [
            {
                test: /\.scss$/,
                use: [
                    'style-loader',
                    'css-loader',
                    {
                        loader: 'sass-loader',
                        options: {
                            implementation: require('sass'),
                            sassOptions: {
                                outputStyle: 'compressed'
                            }
                        }
                    }
                ]
            }
        ]
    }
};

// Less配置
module.exports = {
    module: {
        rules: [
            {
                test: /\.less$/,
                use: [
                    'style-loader',
                    'css-loader',
                    {
                        loader: 'less-loader',
                        options: {
                            lessOptions: {
                                javascriptEnabled: true,
                                modifyVars: {
                                    'primary-color': '#1890ff'
                                }
                            }
                        }
                    }
                ]
            }
        ]
    }
};
```

### 7.2 Vite集成

```javascript
// vite.config.js

export default {
    css: {
        preprocessorOptions: {
            // Sass配置
            scss: {
                additionalData: `@import "@/styles/variables.scss";`,
                charset: false
            },
            
            // Less配置
            less: {
                modifyVars: {
                    'primary-color': '#1890ff',
                    'link-color': '#1890ff'
                },
                javascriptEnabled: true
            }
        }
    }
};
```

### 7.3 Gulp集成

```javascript
// gulpfile.js
const gulp = require('gulp');
const sass = require('gulp-sass')(require('sass'));
const less = require('gulp-less');
const autoprefixer = require('gulp-autoprefixer');
const cleanCSS = require('gulp-clean-css');
const sourcemaps = require('gulp-sourcemaps');

// Sass任务
gulp.task('sass', function() {
    return gulp.src('src/scss/**/*.scss')
        .pipe(sourcemaps.init())
        .pipe(sass({
            outputStyle: 'compressed'
        }).on('error', sass.logError))
        .pipe(autoprefixer())
        .pipe(cleanCSS())
        .pipe(sourcemaps.write('.'))
        .pipe(gulp.dest('dist/css'));
});

// Less任务
gulp.task('less', function() {
    return gulp.src('src/less/**/*.less')
        .pipe(sourcemaps.init())
        .pipe(less())
        .pipe(autoprefixer())
        .pipe(cleanCSS())
        .pipe(sourcemaps.write('.'))
        .pipe(gulp.dest('dist/css'));
});

// 监听任务
gulp.task('watch', function() {
    gulp.watch('src/scss/**/*.scss', gulp.series('sass'));
    gulp.watch('src/less/**/*.less', gulp.series('less'));
});
```

---

## 8. 最佳实践

### 8.1 文件组织

```
styles/
├── abstracts/          # 抽象层
│   ├── _variables.scss # 变量
│   ├── _mixins.scss    # 混合
│   └── _functions.scss # 函数
├── base/               # 基础层
│   ├── _reset.scss     # 重置样式
│   ├── _typography.scss# 排版
│   └── _utilities.scss # 工具类
├── components/         # 组件层
│   ├── _buttons.scss
│   ├── _cards.scss
│   └── _forms.scss
├── layout/             # 布局层
│   ├── _header.scss
│   ├── _footer.scss
│   └── _grid.scss
├── pages/              # 页面层
│   ├── _home.scss
│   └── _about.scss
├── themes/             # 主题层
│   ├── _dark.scss
│   └── _light.scss
└── main.scss           # 主文件
```

### 8.2 命名规范

```scss
// BEM命名规范
.block {
    &__element {
        // 元素样式
    }
    
    &--modifier {
        // 修饰符样式
    }
    
    &__element--modifier {
        // 元素修饰符
    }
}

// 示例
.card {
    padding: 20px;
    
    &__header {
        font-size: 18px;
        font-weight: bold;
    }
    
    &__body {
        margin-top: 10px;
    }
    
    &--featured {
        border: 2px solid gold;
    }
}

// 编译结果
.card { padding: 20px; }
.card__header { font-size: 18px; font-weight: bold; }
.card__body { margin-top: 10px; }
.card--featured { border: 2px solid gold; }
```

### 8.3 变量管理

```scss
// _variables.scss

// 颜色系统
$colors: (
    primary: #3498db,
    secondary: #2ecc71,
    success: #27ae60,
    warning: #f39c12,
    danger: #e74c3c,
    info: #3498db,
    light: #ecf0f1,
    dark: #2c3e50
);

// 间距系统
$spacers: (
    0: 0,
    1: 0.25rem,
    2: 0.5rem,
    3: 1rem,
    4: 1.5rem,
    5: 3rem
);

// 断点系统
$breakpoints: (
    xs: 0,
    sm: 576px,
    md: 768px,
    lg: 992px,
    xl: 1200px,
    xxl: 1400px
);

// 字体系统
$font-family-base: 'Helvetica Neue', Arial, sans-serif;
$font-family-monospace: 'Courier New', monospace;

$font-sizes: (
    xs: 0.75rem,
    sm: 0.875rem,
    base: 1rem,
    lg: 1.125rem,
    xl: 1.25rem,
    2xl: 1.5rem,
    3xl: 1.875rem,
    4xl: 2.25rem
);
```

### 8.4 混合最佳实践

```scss
// 响应式混合
@mixin respond-to($breakpoint) {
    @if map-has-key($breakpoints, $breakpoint) {
        @media (min-width: map-get($breakpoints, $breakpoint)) {
            @content;
        }
    } @else {
        @warn "Breakpoint #{$breakpoint} not found in $breakpoints";
    }
}

// 使用示例
.container {
    width: 100%;
    padding: 0 15px;
    
    @include respond-to(md) {
        max-width: 720px;
    }
    
    @include respond-to(lg) {
        max-width: 960px;
    }
    
    @include respond-to(xl) {
        max-width: 1140px;
    }
}

// Flexbox混合
@mixin flex-center {
    display: flex;
    justify-content: center;
    align-items: center;
}

@mixin flex-between {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

// 文本截断混合
@mixin text-truncate {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

@mixin line-clamp($lines) {
    display: -webkit-box;
    -webkit-line-clamp: $lines;
    -webkit-box-orient: vertical;
    overflow: hidden;
}

// 使用示例
.title {
    @include text-truncate;
}

.description {
    @include line-clamp(3);
}
```

### 8.5 避免过度嵌套

```scss
// ❌ 不好的做法（嵌套过深）
.header {
    .nav {
        .menu {
            .item {
                .link {
                    color: blue;
                    
                    &:hover {
                        color: red;
                    }
                }
            }
        }
    }
}

// ✅ 好的做法（控制嵌套层级）
.header {
    // 头部样式
}

.nav {
    // 导航样式
}

.nav-menu {
    // 菜单样式
}

.nav-item {
    // 菜单项样式
}

.nav-link {
    color: blue;
    
    &:hover {
        color: red;
    }
}
```

---

## 9. 性能优化

### 9.1 减少编译时间

```scss
// ❌ 避免在循环中使用复杂计算
@for $i from 1 through 100 {
    .col-#{$i} {
        width: percentage($i / 100);
        // 复杂的颜色计算
        background: darken(lighten(saturate(#3498db, 20%), 10%), 5%);
    }
}

// ✅ 预先计算，减少重复
$base-color: #3498db;
$processed-color: darken(lighten(saturate($base-color, 20%), 10%), 5%);

@for $i from 1 through 100 {
    .col-#{$i} {
        width: percentage($i / 100);
        background: $processed-color;
    }
}
```

### 9.2 优化输出CSS

```scss
// 使用占位符选择器避免重复
%button-base {
    display: inline-block;
    padding: 10px 20px;
    border: none;
    cursor: pointer;
}

.btn-primary {
    @extend %button-base;
    background: #3498db;
}

.btn-secondary {
    @extend %button-base;
    background: #95a5a6;
}

// 编译结果（合并选择器）
.btn-primary, .btn-secondary {
    display: inline-block;
    padding: 10px 20px;
    border: none;
    cursor: pointer;
}

.btn-primary {
    background: #3498db;
}

.btn-secondary {
    background: #95a5a6;
}
```

### 9.3 按需导入

```scss
// ❌ 导入整个库
@import 'bootstrap/scss/bootstrap';

// ✅ 只导入需要的模块
@import 'bootstrap/scss/functions';
@import 'bootstrap/scss/variables';
@import 'bootstrap/scss/mixins';
@import 'bootstrap/scss/grid';
@import 'bootstrap/scss/utilities';
```

### 9.4 Source Map配置

```javascript
// webpack.config.js
module.exports = {
    devtool: 'source-map',  // 生产环境
    // devtool: 'eval-source-map',  // 开发环境
    
    module: {
        rules: [
            {
                test: /\.scss$/,
                use: [
                    'style-loader',
                    {
                        loader: 'css-loader',
                        options: {
                            sourceMap: true
                        }
                    },
                    {
                        loader: 'sass-loader',
                        options: {
                            sourceMap: true
                        }
                    }
                ]
            }
        ]
    }
};
```

---

## 10. 实战案例

### 10.1 响应式网格系统

```scss
// _grid.scss

// 变量定义
$grid-columns: 12;
$grid-gutter: 30px;
$container-max-widths: (
    sm: 540px,
    md: 720px,
    lg: 960px,
    xl: 1140px,
    xxl: 1320px
);

// 容器
.container {
    width: 100%;
    padding-right: $grid-gutter / 2;
    padding-left: $grid-gutter / 2;
    margin-right: auto;
    margin-left: auto;
    
    @each $breakpoint, $width in $container-max-widths {
        @include respond-to($breakpoint) {
            max-width: $width;
        }
    }
}

// 行
.row {
    display: flex;
    flex-wrap: wrap;
    margin-right: -$grid-gutter / 2;
    margin-left: -$grid-gutter / 2;
}

// 列
@mixin make-col($size, $columns: $grid-columns) {
    flex: 0 0 percentage($size / $columns);
    max-width: percentage($size / $columns);
}

// 生成列类
@for $i from 1 through $grid-columns {
    .col-#{$i} {
        @include make-col($i);
        padding-right: $grid-gutter / 2;
        padding-left: $grid-gutter / 2;
    }
}

// 响应式列
@each $breakpoint in map-keys($breakpoints) {
    @include respond-to($breakpoint) {
        @for $i from 1 through $grid-columns {
            .col-#{$breakpoint}-#{$i} {
                @include make-col($i);
                padding-right: $grid-gutter / 2;
                padding-left: $grid-gutter / 2;
            }
        }
    }
}
```

### 10.2 主题切换系统

```scss
// _themes.scss

// 主题配置
$themes: (
    light: (
        background: #ffffff,
        text: #333333,
        primary: #3498db,
        secondary: #2ecc71,
        border: #e0e0e0
    ),
    dark: (
        background: #1a1a1a,
        text: #ffffff,
        primary: #5dade2,
        secondary: #58d68d,
        border: #404040
    )
);

// 主题混合
@mixin themed() {
    @each $theme, $map in $themes {
        .theme-#{$theme} & {
            $theme-map: () !global;
            @each $key, $value in $map {
                $theme-map: map-merge($theme-map, ($key: $value)) !global;
            }
            @content;
            $theme-map: null !global;
        }
    }
}

// 获取主题值函数
@function theme-get($key) {
    @return map-get($theme-map, $key);
}

// 使用示例
.card {
    @include themed() {
        background: theme-get(background);
        color: theme-get(text);
        border: 1px solid theme-get(border);
    }
}

.button {
    @include themed() {
        background: theme-get(primary);
        color: theme-get(background);
        
        &:hover {
            background: darken(theme-get(primary), 10%);
        }
    }
}
```

### 10.3 组件库样式系统

```scss
// _button.scss

// 按钮基础样式
.btn {
    display: inline-block;
    padding: 0.5rem 1rem;
    font-size: 1rem;
    font-weight: 400;
    line-height: 1.5;
    text-align: center;
    text-decoration: none;
    border: 1px solid transparent;
    border-radius: 0.25rem;
    cursor: pointer;
    transition: all 0.3s ease;
    
    &:disabled {
        opacity: 0.6;
        cursor: not-allowed;
    }
}

// 按钮尺寸
$button-sizes: (
    sm: (
        padding: 0.25rem 0.5rem,
        font-size: 0.875rem
    ),
    md: (
        padding: 0.5rem 1rem,
        font-size: 1rem
    ),
    lg: (
        padding: 0.75rem 1.5rem,
        font-size: 1.125rem
    )
);

@each $size, $props in $button-sizes {
    .btn-#{$size} {
        padding: map-get($props, padding);
        font-size: map-get($props, font-size);
    }
}

// 按钮颜色变体
@each $name, $color in $colors {
    .btn-#{$name} {
        background: $color;
        color: white;
        border-color: $color;
        
        &:hover {
            background: darken($color, 10%);
            border-color: darken($color, 10%);
        }
        
        &:active {
            background: darken($color, 15%);
            border-color: darken($color, 15%);
        }
    }
    
    // 轮廓按钮
    .btn-outline-#{$name} {
        background: transparent;
        color: $color;
        border-color: $color;
        
        &:hover {
            background: $color;
            color: white;
        }
    }
}
```

### 10.4 动画系统

```scss
// _animations.scss

// 动画混合
@mixin keyframes($name) {
    @keyframes #{$name} {
        @content;
    }
}

// 淡入动画
@include keyframes(fadeIn) {
    from {
        opacity: 0;
    }
    to {
        opacity: 1;
    }
}

// 滑入动画
@include keyframes(slideInUp) {
    from {
        transform: translateY(100%);
        opacity: 0;
    }
    to {
        transform: translateY(0);
        opacity: 1;
    }
}

// 缩放动画
@include keyframes(zoomIn) {
    from {
        transform: scale(0);
        opacity: 0;
    }
    to {
        transform: scale(1);
        opacity: 1;
    }
}

// 旋转动画
@include keyframes(rotate) {
    from {
        transform: rotate(0deg);
    }
    to {
        transform: rotate(360deg);
    }
}

// 动画工具类
$animations: (
    fadeIn: (
        name: fadeIn,
        duration: 0.3s
    ),
    slideInUp: (
        name: slideInUp,
        duration: 0.5s
    ),
    zoomIn: (
        name: zoomIn,
        duration: 0.3s
    ),
    rotate: (
        name: rotate,
        duration: 1s
    )
);

@each $class, $props in $animations {
    .animate-#{$class} {
        animation-name: map-get($props, name);
        animation-duration: map-get($props, duration);
        animation-fill-mode: both;
    }
}
```

### 10.5 工具类生成器

```scss
// _utilities.scss

// 间距工具类生成器
@each $prop, $abbrev in (margin: m, padding: p) {
    @each $size, $length in $spacers {
        // 全方向
        .#{$abbrev}-#{$size} {
            #{$prop}: $length;
        }
        
        // 垂直方向
        .#{$abbrev}y-#{$size} {
            #{$prop}-top: $length;
            #{$prop}-bottom: $length;
        }
        
        // 水平方向
        .#{$abbrev}x-#{$size} {
            #{$prop}-left: $length;
            #{$prop}-right: $length;
        }
        
        // 单独方向
        @each $side in (top, right, bottom, left) {
            .#{$abbrev}#{str-slice($side, 0, 1)}-#{$size} {
                #{$prop}-#{$side}: $length;
            }
        }
    }
}

// 文本对齐
$text-aligns: left, center, right, justify;

@each $align in $text-aligns {
    .text-#{$align} {
        text-align: $align;
    }
}

// 字体大小
@each $name, $size in $font-sizes {
    .text-#{$name} {
        font-size: $size;
    }
}

// 字体粗细
$font-weights: (
    light: 300,
    normal: 400,
    medium: 500,
    semibold: 600,
    bold: 700
);

@each $name, $weight in $font-weights {
    .font-#{$name} {
        font-weight: $weight;
    }
}

// Display工具类
$displays: none, inline, inline-block, block, flex, inline-flex, grid;

@each $display in $displays {
    .d-#{$display} {
        display: $display;
    }
}

// Flexbox工具类
.flex-row {
    flex-direction: row;
}

.flex-column {
    flex-direction: column;
}

.justify-start {
    justify-content: flex-start;
}

.justify-center {
    justify-content: center;
}

.justify-end {
    justify-content: flex-end;
}

.justify-between {
    justify-content: space-between;
}

.items-start {
    align-items: flex-start;
}

.items-center {
    align-items: center;
}

.items-end {
    align-items: flex-end;
}
```

### 10.6 完整项目示例

```scss
// main.scss - 主入口文件

// 1. 抽象层（变量、函数、混合）
@use 'abstracts/variables' as *;
@use 'abstracts/functions' as *;
@use 'abstracts/mixins' as *;

// 2. 第三方库
@import 'normalize.css';

// 3. 基础层
@use 'base/reset';
@use 'base/typography';

// 4. 布局层
@use 'layout/grid';
@use 'layout/header';
@use 'layout/footer';
@use 'layout/sidebar';

// 5. 组件层
@use 'components/buttons';
@use 'components/cards';
@use 'components/forms';
@use 'components/modals';
@use 'components/navigation';

// 6. 页面层
@use 'pages/home';
@use 'pages/about';
@use 'pages/contact';

// 7. 主题层
@use 'themes/dark';
@use 'themes/light';

// 8. 工具类
@use 'utilities/spacing';
@use 'utilities/text';
@use 'utilities/display';
@use 'utilities/animations';
```

---

## 总结

### 核心要点

1. **变量管理**
   - 使用语义化命名
   - 集中管理配置
   - 建立设计系统

2. **混合使用**
   - 避免过度使用
   - 参数化设计
   - 复用常见模式

3. **嵌套控制**
   - 限制嵌套层级（≤3层）
   - 使用BEM命名
   - 保持代码可读性

4. **模块化**
   - 按功能拆分文件
   - 使用@use/@forward
   - 避免全局污染

5. **性能优化**
   - 减少编译时间
   - 优化输出CSS
   - 按需导入模块

### 学习路径

1. **入门阶段**（1-2周）
   - 掌握变量、嵌套、混合基础
   - 理解编译流程
   - 完成简单项目

2. **进阶阶段**（2-3周）
   - 学习函数和控制指令
   - 掌握模块系统
   - 构建组件库

3. **高级阶段**（3-4周）
   - 性能优化技巧
   - 大型项目架构
   - 最佳实践应用

### 常见问题

**Q1: Sass和Less如何选择？**
A: 大型项目推荐Sass（功能强大、模块系统完善），中小型项目可选Less（简单易学、快速上手）。

**Q2: 如何避免CSS输出过大？**
A: 使用占位符选择器、按需导入、避免过度嵌套、合理使用@extend。

**Q3: 变量命名有什么规范？**
A: 使用语义化命名、遵循BEM规范、建立命名空间、保持一致性。

**Q4: 如何组织大型项目的样式文件？**
A: 采用7-1模式（7个文件夹+1个主文件）、按功能模块拆分、使用@use管理依赖。

**Q5: 编译速度慢怎么办？**
A: 减少嵌套层级、避免复杂计算、使用增量编译、优化导入结构。

### 推荐资源

**官方文档**:
- Sass官方文档: https://sass-lang.com/documentation
- Less官方文档: https://lesscss.org/

**学习资源**:
- Sass Guidelines: https://sass-guidelin.es/
- CSS-Tricks Sass Guide
- Frontend Masters课程

**工具推荐**:
- SassMeister（在线编译器）
- CodePen（在线演示）
- VS Code插件（Live Sass Compiler）

---

**课程总结**: 本教程全面介绍了Sass和Less两大CSS预处理器，涵盖基础语法、进阶特性、构建集成、最佳实践和实战案例。通过学习本教程，你将能够熟练使用预处理器提升CSS开发效率，构建可维护的样式系统。

**下一步学习**: 建议继续学习PostCSS、Tailwind CSS等现代CSS工具链，深入理解CSS工程化体系。

---

**最后更新时间**: 2026-02-27  
**@author**: erik.zhou
