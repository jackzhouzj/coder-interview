# CSS-in-JS - 完整教程

## 课程信息
- **课程名称**: CSS-in-JS完整教程
- **难度级别**: 中高级
- **预计学时**: 8小时
- **核心内容**: Styled Components、Emotion、CSS Modules、性能优化
- **@author**: erik.zhou

---

## 目录
1. [CSS-in-JS概述](#1-css-in-js概述)
2. [Styled Components](#2-styled-components)
3. [Emotion](#3-emotion)
4. [CSS Modules](#4-css-modules)
5. [主题系统](#5-主题系统)
6. [动态样式](#6-动态样式)
7. [服务端渲染](#7-服务端渲染)
8. [性能优化](#8-性能优化)
9. [最佳实践](#9-最佳实践)
10. [方案对比](#10-方案对比)
11. [实战案例](#11-实战案例)

---

## 1. CSS-in-JS概述

### 1.1 什么是CSS-in-JS

CSS-in-JS是一种将CSS样式直接写在JavaScript中的技术方案，通过JavaScript来管理组件的样式。

**核心特点**:
- 样式与组件共存
- 动态样式生成
- 作用域隔离
- 主题切换简单
- TypeScript支持好

### 1.2 为什么使用CSS-in-JS

```javascript
// 传统CSS的问题
const traditionalProblems = {
    globalScope: '全局作用域，容易冲突',
    naming: '命名困难，需要BEM等规范',
    deadCode: '难以删除未使用的样式',
    dynamic: '动态样式实现复杂',
    maintenance: '样式与组件分离，维护困难'
};

// CSS-in-JS的优势
const cssInJsBenefits = {
    scoped: '自动作用域隔离',
    dynamic: '轻松实现动态样式',
    coLocation: '样式与组件共存',
    typescript: 'TypeScript类型支持',
    ssr: '服务端渲染友好',
    theming: '主题系统简单'
};
```

### 1.3 主流方案对比

```javascript
// 主流CSS-in-JS方案
const solutions = {
    styledComponents: {
        stars: '40k+',
        features: ['Tagged Template', 'Props支持', '主题系统'],
        pros: ['API简洁', '社区活跃', '文档完善'],
        cons: ['运行时开销', '包体积较大']
    },
    emotion: {
        stars: '17k+',
        features: ['多种API', '性能优异', 'Source Maps'],
        pros: ['性能好', '灵活性高', '包体积小'],
        cons: ['学习曲线陡']
    },
    cssModules: {
        stars: '内置',
        features: ['编译时处理', '零运行时', 'CSS语法'],
        pros: ['性能最优', '学习成本低', '工具支持好'],
        cons: ['动态样式受限', '需要构建工具']
    },
    jss: {
        stars: '7k+',
        features: ['对象语法', '插件系统', 'Material-UI使用'],
        pros: ['功能强大', '插件丰富'],
        cons: ['API复杂', '社区较小']
    }
};
```

---

## 2. Styled Components

### 2.1 基础使用

```bash
# 安装
npm install styled-components
npm install -D @types/styled-components  # TypeScript支持
```

```javascript
// Button.jsx
/**
 * Styled Components基础示例
 * @author erik.zhou
 */
import React from 'react';
import styled from 'styled-components';

// 基础样式组件
const Button = styled.button`
    padding: 10px 20px;
    background-color: #3b82f6;
    color: white;
    border: none;
    border-radius: 4px;
    font-size: 16px;
    cursor: pointer;
    transition: background-color 0.3s;
    
    &:hover {
        background-color: #2563eb;
    }
    
    &:active {
        transform: translateY(1px);
    }
    
    &:disabled {
        background-color: #9ca3af;
        cursor: not-allowed;
    }
`;

// 使用
function App() {
    return (
        <div>
            <Button>点击我</Button>
            <Button disabled>禁用按钮</Button>
        </div>
    );
}

export default App;
```

### 2.2 Props传递

```javascript
// DynamicButton.jsx
/**
 * 基于Props的动态样式
 * @author erik.zhou
 */
import styled from 'styled-components';

const Button = styled.button`
    padding: ${props => props.size === 'large' ? '15px 30px' : '10px 20px'};
    background-color: ${props => {
        switch (props.variant) {
            case 'primary':
                return '#3b82f6';
            case 'secondary':
                return '#8b5cf6';
            case 'danger':
                return '#ef4444';
            default:
                return '#6b7280';
        }
    }};
    color: white;
    border: none;
    border-radius: 4px;
    font-size: ${props => props.size === 'large' ? '18px' : '16px'};
    cursor: pointer;
    transition: all 0.3s;
    
    &:hover {
        opacity: 0.9;
        transform: translateY(-2px);
    }
`;

// 使用
function App() {
    return (
        <div>
            <Button variant="primary" size="large">
                主要按钮
            </Button>
            <Button variant="secondary">
                次要按钮
            </Button>
            <Button variant="danger">
                危险按钮
            </Button>
        </div>
    );
}
```

### 2.3 样式继承

```javascript
// StyledInheritance.jsx
/**
 * 样式继承示例
 * @author erik.zhou
 */
import styled from 'styled-components';

// 基础按钮
const Button = styled.button`
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    font-size: 16px;
    cursor: pointer;
    transition: all 0.3s;
`;

// 继承并扩展
const PrimaryButton = styled(Button)`
    background-color: #3b82f6;
    color: white;
    
    &:hover {
        background-color: #2563eb;
    }
`;

const OutlineButton = styled(Button)`
    background-color: transparent;
    color: #3b82f6;
    border: 2px solid #3b82f6;
    
    &:hover {
        background-color: #3b82f6;
        color: white;
    }
`;

// 改变标签类型
const LinkButton = styled(Button).attrs({
    as: 'a'
})`
    display: inline-block;
    text-decoration: none;
    background-color: #10b981;
    color: white;
    
    &:hover {
        background-color: #059669;
    }
`;

// 使用
function App() {
    return (
        <div>
            <PrimaryButton>主要按钮</PrimaryButton>
            <OutlineButton>轮廓按钮</OutlineButton>
            <LinkButton href="#">链接按钮</LinkButton>
        </div>
    );
}
```

### 2.4 attrs方法

```javascript
// AttrsExample.jsx
/**
 * attrs方法使用示例
 * @author erik.zhou
 */
import styled from 'styled-components';

// 设置默认属性
const Input = styled.input.attrs(props => ({
    type: props.type || 'text',
    placeholder: props.placeholder || '请输入内容'
}))`
    width: 100%;
    padding: 10px;
    border: 2px solid #e5e7eb;
    border-radius: 4px;
    font-size: 16px;
    transition: border-color 0.3s;
    
    &:focus {
        outline: none;
        border-color: #3b82f6;
    }
    
    &::placeholder {
        color: #9ca3af;
    }
`;

// 动态属性
const PasswordInput = styled(Input).attrs({
    type: 'password'
})`
    letter-spacing: 2px;
`;

// 使用
function App() {
    return (
        <div>
            <Input placeholder="用户名" />
            <PasswordInput placeholder="密码" />
        </div>
    );
}
```

### 2.5 全局样式

```javascript
// GlobalStyles.jsx
/**
 * 全局样式定义
 * @author erik.zhou
 */
import { createGlobalStyle } from 'styled-components';

const GlobalStyle = createGlobalStyle`
    * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }
    
    body {
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', sans-serif;
        font-size: 16px;
        line-height: 1.6;
        color: #1f2937;
        background-color: ${props => props.theme.backgroundColor || '#ffffff'};
    }
    
    a {
        color: #3b82f6;
        text-decoration: none;
        transition: color 0.3s;
        
        &:hover {
            color: #2563eb;
        }
    }
    
    button {
        font-family: inherit;
    }
    
    img {
        max-width: 100%;
        height: auto;
    }
`;

// 使用
function App() {
    return (
        <>
            <GlobalStyle />
            <div>应用内容</div>
        </>
    );
}

export default App;
```

### 2.6 CSS辅助函数

```javascript
// CSSHelpers.jsx
/**
 * CSS辅助函数示例
 * @author erik.zhou
 */
import styled, { css } from 'styled-components';

// 可复用的CSS片段
const flexCenter = css`
    display: flex;
    align-items: center;
    justify-content: center;
`;

const truncate = css`
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
`;

const boxShadow = (level = 1) => css`
    box-shadow: ${level === 1 
        ? '0 1px 3px rgba(0, 0, 0, 0.12)' 
        : level === 2 
        ? '0 4px 6px rgba(0, 0, 0, 0.1)' 
        : '0 10px 15px rgba(0, 0, 0, 0.1)'
    };
`;

// 使用辅助函数
const Card = styled.div`
    ${flexCenter}
    ${boxShadow(2)}
    padding: 20px;
    border-radius: 8px;
    background-color: white;
`;

const Title = styled.h3`
    ${truncate}
    max-width: 300px;
    font-size: 18px;
    font-weight: 600;
`;

// 条件样式
const Button = styled.button`
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    
    ${props => props.primary && css`
        background-color: #3b82f6;
        color: white;
    `}
    
    ${props => props.secondary && css`
        background-color: #8b5cf6;
        color: white;
    `}
    
    ${props => props.disabled && css`
        opacity: 0.5;
        cursor: not-allowed;
    `}
`;
```

### 2.7 动画

```javascript
// Animations.jsx
/**
 * 动画示例
 * @author erik.zhou
 */
import styled, { keyframes } from 'styled-components';

// 定义关键帧动画
const fadeIn = keyframes`
    from {
        opacity: 0;
        transform: translateY(20px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
`;

const rotate = keyframes`
    from {
        transform: rotate(0deg);
    }
    to {
        transform: rotate(360deg);
    }
`;

const pulse = keyframes`
    0%, 100% {
        transform: scale(1);
    }
    50% {
        transform: scale(1.05);
    }
`;

// 使用动画
const FadeInBox = styled.div`
    animation: ${fadeIn} 0.5s ease-in;
`;

const Spinner = styled.div`
    width: 40px;
    height: 40px;
    border: 4px solid #e5e7eb;
    border-top-color: #3b82f6;
    border-radius: 50%;
    animation: ${rotate} 1s linear infinite;
`;

const PulseButton = styled.button`
    padding: 10px 20px;
    background-color: #3b82f6;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    animation: ${pulse} 2s ease-in-out infinite;
`;

// 使用
function App() {
    return (
        <div>
            <FadeInBox>淡入内容</FadeInBox>
            <Spinner />
            <PulseButton>脉冲按钮</PulseButton>
        </div>
    );
}
```

---

## 3. Emotion

### 3.1 基础使用

```bash
# 安装
npm install @emotion/react @emotion/styled
```

```javascript
// EmotionBasic.jsx
/**
 * Emotion基础示例
 * @author erik.zhou
 */
import styled from '@emotion/styled';
import { css } from '@emotion/react';

// styled API
const Button = styled.button`
    padding: 10px 20px;
    background-color: #3b82f6;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    
    &:hover {
        background-color: #2563eb;
    }
`;

// css prop
function App() {
    return (
        <div>
            <Button>Styled API</Button>
            
            <button
                css={css`
                    padding: 10px 20px;
                    background-color: #8b5cf6;
                    color: white;
                    border: none;
                    border-radius: 4px;
                    cursor: pointer;
                    
                    &:hover {
                        background-color: #7c3aed;
                    }
                `}
            >
                CSS Prop
            </button>
        </div>
    );
}

export default App;
```

### 3.2 对象样式

```javascript
// EmotionObject.jsx
/**
 * Emotion对象样式
 * @author erik.zhou
 */
import { css } from '@emotion/react';

const buttonStyles = {
    padding: '10px 20px',
    backgroundColor: '#3b82f6',
    color: 'white',
    border: 'none',
    borderRadius: '4px',
    cursor: 'pointer',
    transition: 'all 0.3s',
    
    '&:hover': {
        backgroundColor: '#2563eb',
        transform: 'translateY(-2px)'
    }
};

function App() {
    return (
        <button css={buttonStyles}>
            对象样式按钮
        </button>
    );
}
```

### 3.3 组合样式

```javascript
// EmotionComposition.jsx
/**
 * Emotion样式组合
 * @author erik.zhou
 */
import { css } from '@emotion/react';

// 基础样式
const baseButton = css`
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    font-size: 16px;
    cursor: pointer;
    transition: all 0.3s;
`;

const primaryButton = css`
    ${baseButton}
    background-color: #3b82f6;
    color: white;
    
    &:hover {
        background-color: #2563eb;
    }
`;

const dangerButton = css`
    ${baseButton}
    background-color: #ef4444;
    color: white;
    
    &:hover {
        background-color: #dc2626;
    }
`;

// 使用
function App() {
    return (
        <div>
            <button css={primaryButton}>主要按钮</button>
            <button css={dangerButton}>危险按钮</button>
        </div>
    );
}
```

### 3.4 性能优化

```javascript
// EmotionPerformance.jsx
/**
 * Emotion性能优化
 * @author erik.zhou
 */
import { css } from '@emotion/react';
import { memo } from 'react';

// 提取静态样式到组件外部
const staticStyles = css`
    padding: 20px;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
`;

// 使用memo避免不必要的重渲染
const Card = memo(({ title, content }) => {
    return (
        <div css={staticStyles}>
            <h3>{title}</h3>
            <p>{content}</p>
        </div>
    );
});

Card.displayName = 'Card';

export default Card;
```

---

## 4. CSS Modules

### 4.1 基础使用

```css
/* Button.module.css */
.button {
    padding: 10px 20px;
    background-color: #3b82f6;
    color: white;
    border: none;
    border-radius: 4px;
    font-size: 16px;
    cursor: pointer;
    transition: background-color 0.3s;
}

.button:hover {
    background-color: #2563eb;
}

.button:disabled {
    background-color: #9ca3af;
    cursor: not-allowed;
}

.primary {
    background-color: #3b82f6;
}

.secondary {
    background-color: #8b5cf6;
}

.danger {
    background-color: #ef4444;
}
```

```javascript
// Button.jsx
/**
 * CSS Modules使用示例
 * @author erik.zhou
 */
import React from 'react';
import styles from './Button.module.css';

function Button({ variant = 'primary', children, ...props }) {
    const className = `${styles.button} ${styles[variant]}`;
    
    return (
        <button className={className} {...props}>
            {children}
        </button>
    );
}

export default Button;
```

### 4.2 组合样式

```css
/* Card.module.css */
.card {
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    overflow: hidden;
}

.header {
    padding: 20px;
    border-bottom: 1px solid #e5e7eb;
}

.body {
    padding: 20px;
}

.footer {
    padding: 20px;
    background-color: #f9fafb;
    border-top: 1px solid #e5e7eb;
}

/* 组合样式 */
.card.elevated {
    box-shadow: 0 10px 15px rgba(0, 0, 0, 0.1);
}
```

```javascript
// Card.jsx
/**
 * CSS Modules组合样式
 * @author erik.zhou
 */
import React from 'react';
import styles from './Card.module.css';
import clsx from 'clsx';

function Card({ elevated, children }) {
    return (
        <div className={clsx(styles.card, elevated && styles.elevated)}>
            {children}
        </div>
    );
}

function CardHeader({ children }) {
    return <div className={styles.header}>{children}</div>;
}

function CardBody({ children }) {
    return <div className={styles.body}>{children}</div>;
}

function CardFooter({ children }) {
    return <div className={styles.footer}>{children}</div>;
}

Card.Header = CardHeader;
Card.Body = CardBody;
Card.Footer = CardFooter;

export default Card;
```

### 4.3 全局样式

```css
/* global.css */
:global {
    * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }
    
    body {
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
        line-height: 1.6;
    }
}

/* 局部样式 */
.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
}

/* 混合使用 */
.button :global(.icon) {
    margin-right: 8px;
}
```

---

## 5. 主题系统

### 5.1 ThemeProvider基础

```javascript
// theme.js
/**
 * 主题配置
 * @author erik.zhou
 */
export const lightTheme = {
    colors: {
        primary: '#3b82f6',
        secondary: '#8b5cf6',
        success: '#10b981',
        danger: '#ef4444',
        warning: '#f59e0b',
        text: '#1f2937',
        textSecondary: '#6b7280',
        background: '#ffffff',
        backgroundSecondary: '#f9fafb',
        border: '#e5e7eb'
    },
    spacing: {
        xs: '4px',
        sm: '8px',
        md: '16px',
        lg: '24px',
        xl: '32px'
    },
    borderRadius: {
        sm: '4px',
        md: '8px',
        lg: '12px',
        full: '9999px'
    },
    shadows: {
        sm: '0 1px 3px rgba(0, 0, 0, 0.12)',
        md: '0 4px 6px rgba(0, 0, 0, 0.1)',
        lg: '0 10px 15px rgba(0, 0, 0, 0.1)'
    }
};

export const darkTheme = {
    colors: {
        primary: '#60a5fa',
        secondary: '#a78bfa',
        success: '#34d399',
        danger: '#f87171',
        warning: '#fbbf24',
        text: '#f9fafb',
        textSecondary: '#d1d5db',
        background: '#1f2937',
        backgroundSecondary: '#111827',
        border: '#374151'
    },
    spacing: lightTheme.spacing,
    borderRadius: lightTheme.borderRadius,
    shadows: {
        sm: '0 1px 3px rgba(0, 0, 0, 0.3)',
        md: '0 4px 6px rgba(0, 0, 0, 0.2)',
        lg: '0 10px 15px rgba(0, 0, 0, 0.2)'
    }
};
```

```javascript
// App.jsx
/**
 * ThemeProvider使用示例
 * @author erik.zhou
 */
import React from 'react';
import { ThemeProvider } from 'styled-components';
import styled from 'styled-components';
import { lightTheme } from './theme';

const Container = styled.div`
    background-color: ${props => props.theme.colors.background};
    color: ${props => props.theme.colors.text};
    padding: ${props => props.theme.spacing.lg};
    min-height: 100vh;
`;

const Button = styled.button`
    padding: ${props => props.theme.spacing.md};
    background-color: ${props => props.theme.colors.primary};
    color: white;
    border: none;
    border-radius: ${props => props.theme.borderRadius.md};
    box-shadow: ${props => props.theme.shadows.md};
    cursor: pointer;
    transition: all 0.3s;
    
    &:hover {
        transform: translateY(-2px);
        box-shadow: ${props => props.theme.shadows.lg};
    }
`;

function App() {
    return (
        <ThemeProvider theme={lightTheme}>
            <Container>
                <h1>主题系统示例</h1>
                <Button>主题按钮</Button>
            </Container>
        </ThemeProvider>
    );
}

export default App;
```

### 5.2 主题切换

```javascript
// ThemeToggle.jsx
/**
 * 主题切换功能
 * @author erik.zhou
 */
import React, { useState, createContext, useContext } from 'react';
import { ThemeProvider as StyledThemeProvider } from 'styled-components';
import styled from 'styled-components';
import { lightTheme, darkTheme } from './theme';

// 创建主题上下文
const ThemeContext = createContext();

export function ThemeProvider({ children }) {
    const [isDark, setIsDark] = useState(false);
    
    const toggleTheme = () => {
        setIsDark(prev => !prev);
    };
    
    const theme = isDark ? darkTheme : lightTheme;
    
    return (
        <ThemeContext.Provider value={{ isDark, toggleTheme }}>
            <StyledThemeProvider theme={theme}>
                {children}
            </StyledThemeProvider>
        </ThemeContext.Provider>
    );
}

export function useTheme() {
    const context = useContext(ThemeContext);
    if (!context) {
        throw new Error('useTheme必须在ThemeProvider内使用');
    }
    return context;
}

// 主题切换按钮
const ToggleButton = styled.button`
    padding: ${props => props.theme.spacing.md};
    background-color: ${props => props.theme.colors.background};
    color: ${props => props.theme.colors.text};
    border: 2px solid ${props => props.theme.colors.border};
    border-radius: ${props => props.theme.borderRadius.full};
    cursor: pointer;
    transition: all 0.3s;
    
    &:hover {
        background-color: ${props => props.theme.colors.backgroundSecondary};
    }
`;

export function ThemeToggle() {
    const { isDark, toggleTheme } = useTheme();
    
    return (
        <ToggleButton onClick={toggleTheme}>
            {isDark ? '🌞 浅色' : '🌙 深色'}
        </ToggleButton>
    );
}
```

### 5.3 TypeScript支持

```typescript
// theme.types.ts
/**
 * 主题类型定义
 * @author erik.zhou
 */
export interface Theme {
    colors: {
        primary: string;
        secondary: string;
        success: string;
        danger: string;
        warning: string;
        text: string;
        textSecondary: string;
        background: string;
        backgroundSecondary: string;
        border: string;
    };
    spacing: {
        xs: string;
        sm: string;
        md: string;
        lg: string;
        xl: string;
    };
    borderRadius: {
        sm: string;
        md: string;
        lg: string;
        full: string;
    };
    shadows: {
        sm: string;
        md: string;
        lg: string;
    };
}

// 扩展styled-components类型
import 'styled-components';

declare module 'styled-components' {
    export interface DefaultTheme extends Theme {}
}
```

```typescript
// Button.tsx
/**
 * TypeScript主题组件
 * @author erik.zhou
 */
import styled from 'styled-components';

interface ButtonProps {
    variant?: 'primary' | 'secondary' | 'danger';
    size?: 'small' | 'medium' | 'large';
}

const Button = styled.button<ButtonProps>`
    padding: ${props => {
        switch (props.size) {
            case 'small':
                return props.theme.spacing.sm;
            case 'large':
                return props.theme.spacing.lg;
            default:
                return props.theme.spacing.md;
        }
    }};
    background-color: ${props => {
        switch (props.variant) {
            case 'secondary':
                return props.theme.colors.secondary;
            case 'danger':
                return props.theme.colors.danger;
            default:
                return props.theme.colors.primary;
        }
    }};
    color: white;
    border: none;
    border-radius: ${props => props.theme.borderRadius.md};
    cursor: pointer;
    transition: all 0.3s;
    
    &:hover {
        opacity: 0.9;
    }
`;

export default Button;
```

---

## 6. 动态样式

### 6.1 条件样式

```javascript
// ConditionalStyles.jsx
/**
 * 条件样式示例
 * @author erik.zhou
 */
import styled, { css } from 'styled-components';

const Button = styled.button`
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.3s;
    
    /* 基于props的条件样式 */
    ${props => props.primary && css`
        background-color: #3b82f6;
        color: white;
        
        &:hover {
            background-color: #2563eb;
        }
    `}
    
    ${props => props.outline && css`
        background-color: transparent;
        color: #3b82f6;
        border: 2px solid #3b82f6;
        
        &:hover {
            background-color: #3b82f6;
            color: white;
        }
    `}
    
    ${props => props.disabled && css`
        opacity: 0.5;
        cursor: not-allowed;
        pointer-events: none;
    `}
    
    ${props => props.loading && css`
        position: relative;
        color: transparent;
        
        &::after {
            content: '';
            position: absolute;
            width: 16px;
            height: 16px;
            top: 50%;
            left: 50%;
            margin-left: -8px;
            margin-top: -8px;
            border: 2px solid white;
            border-radius: 50%;
            border-top-color: transparent;
            animation: spin 0.6s linear infinite;
        }
        
        @keyframes spin {
            to { transform: rotate(360deg); }
        }
    `}
`;
```

### 6.2 响应式样式

```javascript
// ResponsiveStyles.jsx
/**
 * 响应式样式示例
 * @author erik.zhou
 */
import styled from 'styled-components';

// 定义断点
const breakpoints = {
    mobile: '576px',
    tablet: '768px',
    desktop: '992px',
    wide: '1200px'
};

// 媒体查询辅助函数
const media = {
    mobile: `@media (max-width: ${breakpoints.mobile})`,
    tablet: `@media (max-width: ${breakpoints.tablet})`,
    desktop: `@media (max-width: ${breakpoints.desktop})`,
    wide: `@media (min-width: ${breakpoints.wide})`
};

const Container = styled.div`
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 20px;
    
    ${media.desktop} {
        max-width: 960px;
    }
    
    ${media.tablet} {
        max-width: 720px;
        padding: 0 15px;
    }
    
    ${media.mobile} {
        max-width: 100%;
        padding: 0 10px;
    }
`;

const Grid = styled.div`
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;
    
    ${media.desktop} {
        grid-template-columns: repeat(3, 1fr);
    }
    
    ${media.tablet} {
        grid-template-columns: repeat(2, 1fr);
        gap: 15px;
    }
    
    ${media.mobile} {
        grid-template-columns: 1fr;
        gap: 10px;
    }
`;

const Card = styled.div`
    padding: 20px;
    background-color: white;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    
    ${media.mobile} {
        padding: 15px;
    }
`;
```

### 6.3 状态驱动样式

```javascript
// StateDrivenStyles.jsx
/**
 * 状态驱动样式示例
 * @author erik.zhou
 */
import React, { useState } from 'react';
import styled from 'styled-components';

const Input = styled.input`
    width: 100%;
    padding: 10px;
    border: 2px solid ${props => {
        if (props.error) {
            return '#ef4444';
        }
        if (props.success) {
            return '#10b981';
        }
        if (props.focused) {
            return '#3b82f6';
        }
        return '#e5e7eb';
    }};
    border-radius: 4px;
    font-size: 16px;
    transition: all 0.3s;
    
    &:focus {
        outline: none;
    }
`;

const Message = styled.div`
    margin-top: 8px;
    font-size: 14px;
    color: ${props => props.error ? '#ef4444' : '#10b981'};
    opacity: ${props => props.show ? 1 : 0};
    transform: translateY(${props => props.show ? 0 : -10}px);
    transition: all 0.3s;
`;

function FormInput({ value, onChange, validate }) {
    const [focused, setFocused] = useState(false);
    const [error, setError] = useState('');
    const [success, setSuccess] = useState(false);
    
    const handleBlur = () => {
        setFocused(false);
        const validationError = validate(value);
        if (validationError) {
            setError(validationError);
            setSuccess(false);
        } else {
            setError('');
            setSuccess(true);
        }
    };
    
    return (
        <div>
            <Input
                value={value}
                onChange={onChange}
                onFocus={() => setFocused(true)}
                onBlur={handleBlur}
                focused={focused}
                error={!!error}
                success={success}
            />
            <Message show={!!error} error>
                {error}
            </Message>
            <Message show={success && !error}>
                验证通过
            </Message>
        </div>
    );
}

export default FormInput;
```

---

## 7. 服务端渲染

### 7.1 SSR配置

```javascript
// server.js
/**
 * Styled Components SSR配置
 * @author erik.zhou
 */
import express from 'express';
import React from 'react';
import { renderToString } from 'react-dom/server';
import { ServerStyleSheet } from 'styled-components';
import App from './App';

const server = express();

server.get('*', (req, res) => {
    const sheet = new ServerStyleSheet();
    
    try {
        // 渲染应用并收集样式
        const html = renderToString(
            sheet.collectStyles(<App />)
        );
        
        // 获取样式标签
        const styleTags = sheet.getStyleTags();
        
        // 发送完整HTML
        res.send(`
            <!DOCTYPE html>
            <html>
                <head>
                    <meta charset="utf-8">
                    <title>SSR App</title>
                    ${styleTags}
                </head>
                <body>
                    <div id="root">${html}</div>
                    <script src="/bundle.js"></script>
                </body>
            </html>
        `);
    } catch (error) {
        console.error('SSR错误:', error);
        res.status(500).send('服务器错误');
    } finally {
        sheet.seal();
    }
});

server.listen(3000, () => {
    console.log('服务器运行在 http://localhost:3000');
});
```

### 7.2 样式提取

```javascript
// StyleExtraction.jsx
/**
 * 样式提取示例
 * @author erik.zhou
 */
import { ServerStyleSheet, StyleSheetManager } from 'styled-components';

// 提取关键CSS
function extractCriticalCSS(App) {
    const sheet = new ServerStyleSheet();
    
    try {
        const html = renderToString(
            sheet.collectStyles(<App />)
        );
        
        // 获取CSS字符串
        const css = sheet.getStyleElement();
        
        return { html, css };
    } finally {
        sheet.seal();
    }
}

// 使用StyleSheetManager优化
function OptimizedSSR({ children }) {
    return (
        <StyleSheetManager disableVendorPrefixes>
            {children}
        </StyleSheetManager>
    );
}
```

### 7.3 Next.js集成

```javascript
// pages/_document.js
/**
 * Next.js集成Styled Components
 * @author erik.zhou
 */
import Document, { Html, Head, Main, NextScript } from 'next/document';
import { ServerStyleSheet } from 'styled-components';

export default class MyDocument extends Document {
    static async getInitialProps(ctx) {
        const sheet = new ServerStyleSheet();
        const originalRenderPage = ctx.renderPage;
        
        try {
            ctx.renderPage = () =>
                originalRenderPage({
                    enhanceApp: (App) => (props) =>
                        sheet.collectStyles(<App {...props} />)
                });
            
            const initialProps = await Document.getInitialProps(ctx);
            
            return {
                ...initialProps,
                styles: (
                    <>
                        {initialProps.styles}
                        {sheet.getStyleElement()}
                    </>
                )
            };
        } finally {
            sheet.seal();
        }
    }
    
    render() {
        return (
            <Html lang="zh-CN">
                <Head />
                <body>
                    <Main />
                    <NextScript />
                </body>
            </Html>
        );
    }
}
```

```javascript
// next.config.js
/**
 * Next.js配置
 * @author erik.zhou
 */
module.exports = {
    compiler: {
        styledComponents: true
    }
};
```

---

## 8. 性能优化

### 8.1 运行时优化

```javascript
// RuntimeOptimization.jsx
/**
 * 运行时性能优化
 * @author erik.zhou
 */
import React, { memo } from 'react';
import styled from 'styled-components';

// 1. 提取静态样式到组件外部
const staticButtonStyles = `
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.3s;
`;

const Button = styled.button`
    ${staticButtonStyles}
    background-color: ${props => props.color || '#3b82f6'};
    color: white;
    
    &:hover {
        opacity: 0.9;
    }
`;

// 2. 使用memo避免不必要的重渲染
const OptimizedButton = memo(({ children, ...props }) => {
    return <Button {...props}>{children}</Button>;
});

OptimizedButton.displayName = 'OptimizedButton';

// 3. 避免在render中创建样式组件
// 错误示例
function BadExample() {
    // 每次渲染都会创建新的样式组件
    const DynamicButton = styled.button`
        background-color: #3b82f6;
    `;
    
    return <DynamicButton>按钮</DynamicButton>;
}

// 正确示例
const StaticButton = styled.button`
    background-color: #3b82f6;
`;

function GoodExample() {
    return <StaticButton>按钮</StaticButton>;
}

export default OptimizedButton;
```

### 8.2 构建优化

```javascript
// babel.config.js
/**
 * Babel配置优化
 * @author erik.zhou
 */
module.exports = {
    plugins: [
        [
            'babel-plugin-styled-components',
            {
                // 生产环境禁用displayName
                displayName: process.env.NODE_ENV !== 'production',
                // 生产环境禁用SSR
                ssr: process.env.NODE_ENV !== 'production',
                // 启用文件名和行号
                fileName: true,
                // 压缩CSS
                minify: true,
                // 移除无用代码
                pure: true
            }
        ]
    ]
};
```

```javascript
// webpack.config.js
/**
 * Webpack配置优化
 * @author erik.zhou
 */
module.exports = {
    optimization: {
        splitChunks: {
            cacheGroups: {
                // 分离styled-components
                styledComponents: {
                    test: /[\\/]node_modules[\\/]styled-components[\\/]/,
                    name: 'styled-components',
                    chunks: 'all',
                    priority: 10
                }
            }
        }
    }
};
```

### 8.3 最佳实践

```javascript
// PerformanceBestPractices.jsx
/**
 * 性能优化最佳实践
 * @author erik.zhou
 */
import styled, { css } from 'styled-components';

// 1. 使用css辅助函数复用样式
const flexCenter = css`
    display: flex;
    align-items: center;
    justify-content: center;
`;

// 2. 避免深层嵌套
// 不推荐
const BadNesting = styled.div`
    .container {
        .header {
            .title {
                .text {
                    color: red;
                }
            }
        }
    }
`;

// 推荐
const Container = styled.div``;
const Header = styled.div``;
const Title = styled.div``;
const Text = styled.span`
    color: red;
`;

// 3. 使用attrs减少重渲染
const Input = styled.input.attrs(props => ({
    type: props.type || 'text',
    size: props.size || 20
}))`
    padding: 10px;
    border: 1px solid #ccc;
`;

// 4. 条件样式优化
const Button = styled.button`
    padding: 10px 20px;
    
    /* 使用props函数而不是三元表达式 */
    ${props => props.primary && css`
        background-color: #3b82f6;
        color: white;
    `}
`;

// 5. 避免内联函数
// 不推荐
const BadButton = styled.button`
    background-color: ${() => '#3b82f6'};
`;

// 推荐
const primaryColor = '#3b82f6';
const GoodButton = styled.button`
    background-color: ${primaryColor};
`;
```

---

## 9. 最佳实践

### 9.1 命名规范

```javascript
// NamingConventions.jsx
/**
 * 命名规范示例
 * @author erik.zhou
 */
import styled from 'styled-components';

// 1. 组件名使用PascalCase
const Button = styled.button``;
const CardHeader = styled.div``;
const UserAvatar = styled.img``;

// 2. 容器组件添加Container后缀
const PageContainer = styled.div``;
const FormContainer = styled.form``;

// 3. 包装组件添加Wrapper后缀
const ButtonWrapper = styled.div``;
const InputWrapper = styled.div``;

// 4. 样式变量使用camelCase
const primaryColor = '#3b82f6';
const borderRadius = '4px';
const boxShadow = '0 2px 4px rgba(0, 0, 0, 0.1)';

// 5. 主题键名使用camelCase
const theme = {
    colors: {
        primary: '#3b82f6',
        secondary: '#8b5cf6'
    },
    spacing: {
        small: '8px',
        medium: '16px',
        large: '24px'
    }
};
```

### 9.2 组件组织

```javascript
// ComponentOrganization.jsx
/**
 * 组件组织最佳实践
 * @author erik.zhou
 */
import styled from 'styled-components';

// 1. 样式组件放在文件顶部
const Container = styled.div`
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
`;

const Header = styled.header`
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
`;

const Title = styled.h1`
    font-size: 24px;
    font-weight: 600;
    color: #1f2937;
`;

const Content = styled.main`
    background-color: white;
    border-radius: 8px;
    padding: 20px;
`;

// 2. 业务组件放在样式组件之后
function Page() {
    return (
        <Container>
            <Header>
                <Title>页面标题</Title>
            </Header>
            <Content>
                页面内容
            </Content>
        </Container>
    );
}

export default Page;
```

```javascript
// Button/index.js
/**
 * 按钮组件模块化组织
 * @author erik.zhou
 */
import styled from 'styled-components';

// styles.js - 样式定义
export const StyledButton = styled.button`
    padding: 10px 20px;
    background-color: ${props => props.theme.colors.primary};
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: all 0.3s;
    
    &:hover {
        opacity: 0.9;
    }
`;

// Button.jsx - 组件逻辑
export function Button({ children, ...props }) {
    return (
        <StyledButton {...props}>
            {children}
        </StyledButton>
    );
}

// index.js - 导出
export { Button } from './Button';
export { StyledButton } from './styles';
```

### 9.3 代码分割

```javascript
// CodeSplitting.jsx
/**
 * 代码分割示例
 * @author erik.zhou
 */
import React, { lazy, Suspense } from 'react';
import styled from 'styled-components';

// 懒加载样式组件
const HeavyComponent = lazy(() => import('./HeavyComponent'));

const LoadingContainer = styled.div`
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 200px;
`;

const Spinner = styled.div`
    width: 40px;
    height: 40px;
    border: 4px solid #e5e7eb;
    border-top-color: #3b82f6;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    
    @keyframes spin {
        to { transform: rotate(360deg); }
    }
`;

function App() {
    return (
        <Suspense
            fallback={
                <LoadingContainer>
                    <Spinner />
                </LoadingContainer>
            }
        >
            <HeavyComponent />
        </Suspense>
    );
}

export default App;
```

---

## 10. 方案对比

### 10.1 Styled Components vs Emotion

```javascript
// Comparison.jsx
/**
 * Styled Components vs Emotion对比
 * @author erik.zhou
 */

// Styled Components
import styled from 'styled-components';

const StyledButton = styled.button`
    padding: 10px 20px;
    background-color: #3b82f6;
    color: white;
`;

// Emotion - styled API
import styled from '@emotion/styled';

const EmotionStyledButton = styled.button`
    padding: 10px 20px;
    background-color: #3b82f6;
    color: white;
`;

// Emotion - css prop
import { css } from '@emotion/react';

function EmotionCssButton() {
    return (
        <button
            css={css`
                padding: 10px 20px;
                background-color: #3b82f6;
                color: white;
            `}
        >
            按钮
        </button>
    );
}

// 对比总结
const comparison = {
    styledComponents: {
        优点: [
            'API简洁统一',
            '社区活跃，生态丰富',
            '文档完善',
            'TypeScript支持好'
        ],
        缺点: [
            '包体积较大（~15KB）',
            '运行时开销较高',
            '只支持styled API'
        ],
        适用场景: [
            '大型项目',
            '需要完整主题系统',
            '团队熟悉styled API'
        ]
    },
    emotion: {
        优点: [
            '包体积小（~8KB）',
            '性能优异',
            '多种API选择',
            'Source Maps支持'
        ],
        缺点: [
            '学习曲线陡',
            'API较多容易混淆',
            '社区相对较小'
        ],
        适用场景: [
            '性能敏感项目',
            '需要灵活API',
            '包体积要求严格'
        ]
    }
};
```

### 10.2 CSS-in-JS vs CSS Modules

```javascript
// CSSinJSvsCSSModules.jsx
/**
 * CSS-in-JS vs CSS Modules对比
 * @author erik.zhou
 */

// CSS-in-JS优势
const cssInJsAdvantages = {
    动态样式: '轻松实现基于props的动态样式',
    主题系统: '内置主题支持，切换简单',
    作用域隔离: '自动生成唯一类名',
    JavaScript集成: '可以使用JS变量和函数',
    类型安全: 'TypeScript支持好'
};

// CSS Modules优势
const cssModulesAdvantages = {
    性能: '零运行时开销，编译时处理',
    学习成本: '使用标准CSS语法',
    工具支持: 'IDE支持好，调试方便',
    包体积: '无需额外依赖',
    稳定性: '成熟方案，风险低'
};

// 选择建议
const recommendations = {
    选择CSSinJS: [
        '需要复杂的动态样式',
        '需要完整的主题系统',
        '团队熟悉JavaScript',
        '项目规模较大'
    ],
    选择CSSModules: [
        '性能要求极高',
        '团队更熟悉CSS',
        '项目较简单',
        '需要最小包体积'
    ]
};
```

### 10.3 性能对比

```javascript
// PerformanceComparison.js
/**
 * 性能对比数据
 * @author erik.zhou
 */

const performanceMetrics = {
    bundleSize: {
        styledComponents: '15.2 KB (gzipped)',
        emotion: '7.9 KB (gzipped)',
        cssModules: '0 KB (无运行时)'
    },
    runtimeOverhead: {
        styledComponents: '中等（样式注入 + 类名生成）',
        emotion: '较低（优化的样式注入）',
        cssModules: '无（编译时处理）'
    },
    firstPaint: {
        styledComponents: '~50ms 延迟',
        emotion: '~30ms 延迟',
        cssModules: '无延迟'
    },
    reRenderCost: {
        styledComponents: '中等',
        emotion: '较低',
        cssModules: '最低'
    }
};

// 性能优化建议
const optimizationTips = {
    styledComponents: [
        '使用babel插件优化',
        '提取静态样式',
        '避免render中创建组件',
        '使用memo减少重渲染'
    ],
    emotion: [
        '使用css prop减少组件层级',
        '启用自动vendor前缀',
        '使用对象样式提升性能',
        '合理使用缓存'
    ],
    cssModules: [
        '使用CSS压缩',
        '启用Tree Shaking',
        '合并相同样式',
        '使用PostCSS优化'
    ]
};
```

---

## 11. 实战案例

### 11.1 完整组件库

```javascript
// ComponentLibrary/Button.jsx
/**
 * 按钮组件库
 * @author erik.zhou
 */
import styled, { css } from 'styled-components';

const sizeStyles = {
    small: css`
        padding: 6px 12px;
        font-size: 14px;
    `,
    medium: css`
        padding: 10px 20px;
        font-size: 16px;
    `,
    large: css`
        padding: 14px 28px;
        font-size: 18px;
    `
};

const variantStyles = {
    primary: css`
        background-color: ${props => props.theme.colors.primary};
        color: white;
        
        &:hover {
            background-color: ${props => props.theme.colors.primaryDark};
        }
    `,
    secondary: css`
        background-color: ${props => props.theme.colors.secondary};
        color: white;
        
        &:hover {
            background-color: ${props => props.theme.colors.secondaryDark};
        }
    `,
    outline: css`
        background-color: transparent;
        color: ${props => props.theme.colors.primary};
        border: 2px solid ${props => props.theme.colors.primary};
        
        &:hover {
            background-color: ${props => props.theme.colors.primary};
            color: white;
        }
    `,
    ghost: css`
        background-color: transparent;
        color: ${props => props.theme.colors.primary};
        
        &:hover {
            background-color: ${props => props.theme.colors.primaryLight};
        }
    `
};

const StyledButton = styled.button`
    border: none;
    border-radius: ${props => props.theme.borderRadius.md};
    cursor: pointer;
    font-weight: 500;
    transition: all 0.3s;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    
    ${props => sizeStyles[props.size || 'medium']}
    ${props => variantStyles[props.variant || 'primary']}
    
    ${props => props.block && css`
        width: 100%;
    `}
    
    ${props => props.disabled && css`
        opacity: 0.5;
        cursor: not-allowed;
        pointer-events: none;
    `}
    
    ${props => props.loading && css`
        position: relative;
        color: transparent;
        
        &::after {
            content: '';
            position: absolute;
            width: 16px;
            height: 16px;
            border: 2px solid currentColor;
            border-radius: 50%;
            border-top-color: transparent;
            animation: spin 0.6s linear infinite;
        }
    `}
    
    @keyframes spin {
        to { transform: rotate(360deg); }
    }
`;

export function Button({ 
    children, 
    icon, 
    size = 'medium',
    variant = 'primary',
    block = false,
    loading = false,
    disabled = false,
    ...props 
}) {
    return (
        <StyledButton
            size={size}
            variant={variant}
            block={block}
            loading={loading}
            disabled={disabled || loading}
            {...props}
        >
            {icon && <span>{icon}</span>}
            {children}
        </StyledButton>
    );
}
```

```javascript
// ComponentLibrary/Card.jsx
/**
 * 卡片组件
 * @author erik.zhou
 */
import styled from 'styled-components';

const StyledCard = styled.div`
    background-color: white;
    border-radius: ${props => props.theme.borderRadius.lg};
    box-shadow: ${props => props.theme.shadows.md};
    overflow: hidden;
    transition: all 0.3s;
    
    ${props => props.hoverable && `
        cursor: pointer;
        
        &:hover {
            box-shadow: ${props.theme.shadows.lg};
            transform: translateY(-4px);
        }
    `}
`;

const CardHeader = styled.div`
    padding: ${props => props.theme.spacing.lg};
    border-bottom: 1px solid ${props => props.theme.colors.border};
`;

const CardTitle = styled.h3`
    margin: 0;
    font-size: 18px;
    font-weight: 600;
    color: ${props => props.theme.colors.text};
`;

const CardBody = styled.div`
    padding: ${props => props.theme.spacing.lg};
`;

const CardFooter = styled.div`
    padding: ${props => props.theme.spacing.lg};
    border-top: 1px solid ${props => props.theme.colors.border};
    background-color: ${props => props.theme.colors.backgroundSecondary};
`;

export function Card({ children, hoverable = false, ...props }) {
    return (
        <StyledCard hoverable={hoverable} {...props}>
            {children}
        </StyledCard>
    );
}

Card.Header = CardHeader;
Card.Title = CardTitle;
Card.Body = CardBody;
Card.Footer = CardFooter;
```

### 11.2 主题系统实战

```javascript
// ThemeSystem/themes.js
/**
 * 完整主题系统
 * @author erik.zhou
 */

// 基础主题配置
const baseTheme = {
    spacing: {
        xs: '4px',
        sm: '8px',
        md: '16px',
        lg: '24px',
        xl: '32px',
        xxl: '48px'
    },
    borderRadius: {
        sm: '4px',
        md: '8px',
        lg: '12px',
        xl: '16px',
        full: '9999px'
    },
    fontSize: {
        xs: '12px',
        sm: '14px',
        md: '16px',
        lg: '18px',
        xl: '20px',
        xxl: '24px'
    },
    fontWeight: {
        normal: 400,
        medium: 500,
        semibold: 600,
        bold: 700
    },
    lineHeight: {
        tight: 1.2,
        normal: 1.5,
        relaxed: 1.8
    },
    breakpoints: {
        mobile: '576px',
        tablet: '768px',
        desktop: '992px',
        wide: '1200px'
    }
};

// 浅色主题
export const lightTheme = {
    ...baseTheme,
    name: 'light',
    colors: {
        primary: '#3b82f6',
        primaryLight: '#60a5fa',
        primaryDark: '#2563eb',
        secondary: '#8b5cf6',
        secondaryLight: '#a78bfa',
        secondaryDark: '#7c3aed',
        success: '#10b981',
        successLight: '#34d399',
        successDark: '#059669',
        warning: '#f59e0b',
        warningLight: '#fbbf24',
        warningDark: '#d97706',
        danger: '#ef4444',
        dangerLight: '#f87171',
        dangerDark: '#dc2626',
        text: '#1f2937',
        textSecondary: '#6b7280',
        textTertiary: '#9ca3af',
        background: '#ffffff',
        backgroundSecondary: '#f9fafb',
        backgroundTertiary: '#f3f4f6',
        border: '#e5e7eb',
        borderLight: '#f3f4f6',
        borderDark: '#d1d5db'
    },
    shadows: {
        sm: '0 1px 2px rgba(0, 0, 0, 0.05)',
        md: '0 4px 6px rgba(0, 0, 0, 0.1)',
        lg: '0 10px 15px rgba(0, 0, 0, 0.1)',
        xl: '0 20px 25px rgba(0, 0, 0, 0.15)'
    }
};

// 深色主题
export const darkTheme = {
    ...baseTheme,
    name: 'dark',
    colors: {
        primary: '#60a5fa',
        primaryLight: '#93c5fd',
        primaryDark: '#3b82f6',
        secondary: '#a78bfa',
        secondaryLight: '#c4b5fd',
        secondaryDark: '#8b5cf6',
        success: '#34d399',
        successLight: '#6ee7b7',
        successDark: '#10b981',
        warning: '#fbbf24',
        warningLight: '#fcd34d',
        warningDark: '#f59e0b',
        danger: '#f87171',
        dangerLight: '#fca5a5',
        dangerDark: '#ef4444',
        text: '#f9fafb',
        textSecondary: '#d1d5db',
        textTertiary: '#9ca3af',
        background: '#1f2937',
        backgroundSecondary: '#111827',
        backgroundTertiary: '#0f172a',
        border: '#374151',
        borderLight: '#4b5563',
        borderDark: '#1f2937'
    },
    shadows: {
        sm: '0 1px 2px rgba(0, 0, 0, 0.3)',
        md: '0 4px 6px rgba(0, 0, 0, 0.2)',
        lg: '0 10px 15px rgba(0, 0, 0, 0.2)',
        xl: '0 20px 25px rgba(0, 0, 0, 0.25)'
    }
};
```

```javascript
// ThemeSystem/ThemeProvider.jsx
/**
 * 主题提供者组件
 * @author erik.zhou
 */
import React, { createContext, useContext, useState, useEffect } from 'react';
import { ThemeProvider as StyledThemeProvider } from 'styled-components';
import { lightTheme, darkTheme } from './themes';

const ThemeContext = createContext();

export function ThemeProvider({ children }) {
    const [theme, setTheme] = useState(() => {
        const savedTheme = localStorage.getItem('theme');
        return savedTheme === 'dark' ? darkTheme : lightTheme;
    });
    
    useEffect(() => {
        localStorage.setItem('theme', theme.name);
    }, [theme]);
    
    const toggleTheme = () => {
        setTheme(prev => prev.name === 'light' ? darkTheme : lightTheme);
    };
    
    const setLightTheme = () => setTheme(lightTheme);
    const setDarkTheme = () => setTheme(darkTheme);
    
    const value = {
        theme,
        themeName: theme.name,
        toggleTheme,
        setLightTheme,
        setDarkTheme,
        isDark: theme.name === 'dark'
    };
    
    return (
        <ThemeContext.Provider value={value}>
            <StyledThemeProvider theme={theme}>
                {children}
            </StyledThemeProvider>
        </ThemeContext.Provider>
    );
}

export function useTheme() {
    const context = useContext(ThemeContext);
    if (!context) {
        throw new Error('useTheme必须在ThemeProvider内使用');
    }
    return context;
}
```

### 11.3 复杂表单系统

```javascript
// FormSystem/Form.jsx
/**
 * 表单系统实战
 * @author erik.zhou
 */
import React, { useState } from 'react';
import styled from 'styled-components';

const FormContainer = styled.form`
    max-width: 600px;
    margin: 0 auto;
    padding: ${props => props.theme.spacing.xl};
    background-color: ${props => props.theme.colors.background};
    border-radius: ${props => props.theme.borderRadius.lg};
    box-shadow: ${props => props.theme.shadows.md};
`;

const FormGroup = styled.div`
    margin-bottom: ${props => props.theme.spacing.lg};
`;

const Label = styled.label`
    display: block;
    margin-bottom: ${props => props.theme.spacing.sm};
    font-size: ${props => props.theme.fontSize.sm};
    font-weight: ${props => props.theme.fontWeight.medium};
    color: ${props => props.theme.colors.text};
`;

const Input = styled.input`
    width: 100%;
    padding: ${props => props.theme.spacing.md};
    border: 2px solid ${props => {
        if (props.error) {
            return props.theme.colors.danger;
        }
        if (props.success) {
            return props.theme.colors.success;
        }
        return props.theme.colors.border;
    }};
    border-radius: ${props => props.theme.borderRadius.md};
    font-size: ${props => props.theme.fontSize.md};
    color: ${props => props.theme.colors.text};
    background-color: ${props => props.theme.colors.background};
    transition: all 0.3s;
    
    &:focus {
        outline: none;
        border-color: ${props => props.theme.colors.primary};
        box-shadow: 0 0 0 3px ${props => props.theme.colors.primary}20;
    }
    
    &::placeholder {
        color: ${props => props.theme.colors.textTertiary};
    }
    
    &:disabled {
        background-color: ${props => props.theme.colors.backgroundSecondary};
        cursor: not-allowed;
    }
`;

const ErrorMessage = styled.div`
    margin-top: ${props => props.theme.spacing.sm};
    font-size: ${props => props.theme.fontSize.sm};
    color: ${props => props.theme.colors.danger};
`;

const SuccessMessage = styled.div`
    margin-top: ${props => props.theme.spacing.sm};
    font-size: ${props => props.theme.fontSize.sm};
    color: ${props => props.theme.colors.success};
`;

const SubmitButton = styled.button`
    width: 100%;
    padding: ${props => props.theme.spacing.md};
    background-color: ${props => props.theme.colors.primary};
    color: white;
    border: none;
    border-radius: ${props => props.theme.borderRadius.md};
    font-size: ${props => props.theme.fontSize.md};
    font-weight: ${props => props.theme.fontWeight.medium};
    cursor: pointer;
    transition: all 0.3s;
    
    &:hover {
        background-color: ${props => props.theme.colors.primaryDark};
    }
    
    &:disabled {
        background-color: ${props => props.theme.colors.border};
        cursor: not-allowed;
    }
`;

export function Form() {
    const [formData, setFormData] = useState({
        username: '',
        email: '',
        password: ''
    });
    
    const [errors, setErrors] = useState({});
    const [touched, setTouched] = useState({});
    
    const validate = (name, value) => {
        switch (name) {
            case 'username':
                if (!value) {
                    return '用户名不能为空';
                }
                if (value.length < 3) {
                    return '用户名至少3个字符';
                }
                return '';
            case 'email':
                if (!value) {
                    return '邮箱不能为空';
                }
                if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
                    return '邮箱格式不正确';
                }
                return '';
            case 'password':
                if (!value) {
                    return '密码不能为空';
                }
                if (value.length < 6) {
                    return '密码至少6个字符';
                }
                return '';
            default:
                return '';
        }
    };
    
    const handleChange = (e) => {
        const { name, value } = e.target;
        setFormData(prev => ({ ...prev, [name]: value }));
        
        if (touched[name]) {
            const error = validate(name, value);
            setErrors(prev => ({ ...prev, [name]: error }));
        }
    };
    
    const handleBlur = (e) => {
        const { name, value } = e.target;
        setTouched(prev => ({ ...prev, [name]: true }));
        const error = validate(name, value);
        setErrors(prev => ({ ...prev, [name]: error }));
    };
    
    const handleSubmit = (e) => {
        e.preventDefault();
        
        const newErrors = {};
        Object.keys(formData).forEach(key => {
            const error = validate(key, formData[key]);
            if (error) {
                newErrors[key] = error;
            }
        });
        
        if (Object.keys(newErrors).length > 0) {
            setErrors(newErrors);
            setTouched({
                username: true,
                email: true,
                password: true
            });
            return;
        }
        
        console.log('表单提交:', formData);
    };
    
    return (
        <FormContainer onSubmit={handleSubmit}>
            <FormGroup>
                <Label htmlFor="username">用户名</Label>
                <Input
                    id="username"
                    name="username"
                    value={formData.username}
                    onChange={handleChange}
                    onBlur={handleBlur}
                    error={touched.username && errors.username}
                    success={touched.username && !errors.username}
                    placeholder="请输入用户名"
                />
                {touched.username && errors.username && (
                    <ErrorMessage>{errors.username}</ErrorMessage>
                )}
                {touched.username && !errors.username && formData.username && (
                    <SuccessMessage>✓ 用户名可用</SuccessMessage>
                )}
            </FormGroup>
            
            <FormGroup>
                <Label htmlFor="email">邮箱</Label>
                <Input
                    id="email"
                    name="email"
                    type="email"
                    value={formData.email}
                    onChange={handleChange}
                    onBlur={handleBlur}
                    error={touched.email && errors.email}
                    success={touched.email && !errors.email}
                    placeholder="请输入邮箱"
                />
                {touched.email && errors.email && (
                    <ErrorMessage>{errors.email}</ErrorMessage>
                )}
            </FormGroup>
            
            <FormGroup>
                <Label htmlFor="password">密码</Label>
                <Input
                    id="password"
                    name="password"
                    type="password"
                    value={formData.password}
                    onChange={handleChange}
                    onBlur={handleBlur}
                    error={touched.password && errors.password}
                    success={touched.password && !errors.password}
                    placeholder="请输入密码"
                />
                {touched.password && errors.password && (
                    <ErrorMessage>{errors.password}</ErrorMessage>
                )}
            </FormGroup>
            
            <SubmitButton type="submit">
                提交
            </SubmitButton>
        </FormContainer>
    );
}
```

---

## 总结

### 核心要点

1. **CSS-in-JS优势**
   - 样式与组件共存，维护方便
   - 动态样式实现简单
   - 自动作用域隔离
   - 主题系统完善
   - TypeScript支持好

2. **主流方案选择**
   - Styled Components：大型项目，完整生态
   - Emotion：性能敏感，灵活API
   - CSS Modules：零运行时，最佳性能

3. **性能优化关键**
   - 提取静态样式到组件外部
   - 使用memo避免不必要重渲染
   - 避免在render中创建样式组件
   - 合理使用attrs减少重渲染
   - 启用Babel插件优化

4. **最佳实践**
   - 遵循命名规范
   - 合理组织组件结构
   - 使用主题系统统一样式
   - 实现代码分割
   - 注重可维护性

### 学习路径

1. **基础阶段**（1-2周）
   - 掌握Styled Components基础语法
   - 理解Props传递和样式继承
   - 学习全局样式和动画

2. **进阶阶段**（2-3周）
   - 深入主题系统
   - 掌握动态样式和响应式
   - 学习SSR配置

3. **高级阶段**（3-4周）
   - 性能优化技巧
   - 构建完整组件库
   - 实战项目应用

### 推荐资源

1. **官方文档**
   - [Styled Components](https://styled-components.com/)
   - [Emotion](https://emotion.sh/)
   - [CSS Modules](https://github.com/css-modules/css-modules)

2. **学习资源**
   - CSS-in-JS最佳实践
   - 性能优化指南
   - 主题系统设计

3. **工具推荐**
   - babel-plugin-styled-components
   - styled-components VSCode插件
   - Chrome DevTools

---

## 附录

### A. 常用API速查

```javascript
// Styled Components
import styled, { 
    css,                    // CSS辅助函数
    keyframes,              // 动画
    createGlobalStyle,      // 全局样式
    ThemeProvider          // 主题提供者
} from 'styled-components';

// Emotion
import styled from '@emotion/styled';
import { css, keyframes, Global } from '@emotion/react';
```

### B. 配置模板

```javascript
// .babelrc
{
    "plugins": [
        ["babel-plugin-styled-components", {
            "displayName": true,
            "fileName": true,
            "ssr": true,
            "minify": true,
            "pure": true
        }]
    ]
}
```

### C. 故障排查

**问题1：样式不生效**
- 检查ThemeProvider是否正确包裹
- 确认样式组件是否在组件外部定义
- 查看浏览器控制台是否有错误

**问题2：性能问题**
- 避免在render中创建样式组件
- 使用memo优化组件
- 检查是否有不必要的重渲染

**问题3：SSR样式闪烁**
- 确认服务端正确收集样式
- 检查样式标签是否正确注入
- 使用ServerStyleSheet处理

---

**课程结束**

通过本教程，你已经掌握了CSS-in-JS的核心概念、主流方案、性能优化和实战应用。继续实践，不断提升！

**@author**: erik.zhou
