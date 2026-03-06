# React Testing Library - 完整教程

## 目录
1. [React Testing Library简介](#1-react-testing-library简介)
2. [安装与配置](#2-安装与配置)
3. [查询元素](#3-查询元素)
4. [用户交互](#4-用户交互)
5. [异步测试](#5-异步测试)
6. [测试Hooks](#6-测试hooks)
7. [测试Context](#7-测试context)
8. [测试路由](#8-测试路由)
9. [Mock与Spy](#9-mock与spy)
10. [最佳实践](#10-最佳实践)

---

## 1. React Testing Library简介

### 1.1 核心理念

```typescript
/**
 * React Testing Library核心理念
 * @author erik.zhou
 */

// 核心原则：
// 1. 测试应该尽可能接近用户使用方式
// 2. 避免测试实现细节
// 3. 关注组件的行为而非内部状态
// 4. 使用可访问性查询优先

// ❌ 不好的实践 - 测试实现细节
test('bad practice', () => {
    const { container } = render(<Counter />);
    const button = container.querySelector('.increment-button');
    expect(button.textContent).toBe('Increment');
});

// ✅ 好的实践 - 测试用户行为
test('good practice', () => {
    render(<Counter />);
    const button = screen.getByRole('button', { name: /increment/i });
    expect(button).toBeInTheDocument();
});
```

### 1.2 查询优先级

```typescript
/**
 * 查询优先级指南
 * @author erik.zhou
 */

// 优先级从高到低：

// 1. 可访问性查询（推荐）
screen.getByRole('button', { name: /submit/i });
screen.getByLabelText('Username');
screen.getByPlaceholderText('Enter email');
screen.getByText('Welcome');

// 2. 语义化查询
screen.getByAltText('Profile picture');
screen.getByTitle('Close');

// 3. Test ID（最后选择）
screen.getByTestId('custom-element');
```

---

## 2. 安装与配置

### 2.1 安装依赖

```bash
# 安装React Testing Library
npm install --save-dev @testing-library/react

# 安装用户事件库
npm install --save-dev @testing-library/user-event

# 安装Jest DOM匹配器
npm install --save-dev @testing-library/jest-dom

# TypeScript类型定义
npm install --save-dev @types/jest
```

### 2.2 配置Jest

```javascript
/**
 * Jest配置
 * @author erik.zhou
 */

// jest.config.js
module.exports = {
    testEnvironment: 'jsdom',
    setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
    moduleNameMapper: {
        '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
        '\\.(jpg|jpeg|png|gif|svg)$': '<rootDir>/__mocks__/fileMock.js'
    },
    transform: {
        '^.+\\.(ts|tsx)$': 'ts-jest',
        '^.+\\.(js|jsx)$': 'babel-jest'
    }
};

// jest.setup.js
import '@testing-library/jest-dom';

// 全局配置
global.matchMedia = global.matchMedia || function() {
    return {
        matches: false,
        addListener: jest.fn(),
        removeListener: jest.fn()
    };
};
```

### 2.3 配置Vitest

```typescript
/**
 * Vitest配置
 * @author erik.zhou
 */

// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [react()],
    test: {
        globals: true,
        environment: 'jsdom',
        setupFiles: './test/setup.ts',
        css: true
    }
});

// test/setup.ts
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach } from 'vitest';

afterEach(() => {
    cleanup();
});
```

---

## 3. 查询元素

### 3.1 getBy查询

```typescript
/**
 * getBy查询示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';

function LoginForm() {
    return (
        <form>
            <label htmlFor="username">Username</label>
            <input id="username" type="text" placeholder="Enter username" />
            
            <label htmlFor="password">Password</label>
            <input id="password" type="password" />
            
            <button type="submit">Login</button>
            <img src="logo.png" alt="Company Logo" />
        </form>
    );
}

describe('LoginForm Queries', () => {
    test('getByRole - 通过角色查询', () => {
        render(<LoginForm />);
        
        const button = screen.getByRole('button', { name: /login/i });
        expect(button).toBeInTheDocument();
    });
    
    test('getByLabelText - 通过标签查询', () => {
        render(<LoginForm />);
        
        const usernameInput = screen.getByLabelText(/username/i);
        expect(usernameInput).toBeInTheDocument();
    });
    
    test('getByPlaceholderText - 通过占位符查询', () => {
        render(<LoginForm />);
        
        const input = screen.getByPlaceholderText(/enter username/i);
        expect(input).toBeInTheDocument();
    });
    
    test('getByText - 通过文本查询', () => {
        render(<LoginForm />);
        
        const label = screen.getByText(/username/i);
        expect(label).toBeInTheDocument();
    });
    
    test('getByAltText - 通过alt文本查询', () => {
        render(<LoginForm />);
        
        const logo = screen.getByAltText(/company logo/i);
        expect(logo).toBeInTheDocument();
    });
});
```

### 3.2 queryBy和findBy查询

```typescript
/**
 * queryBy和findBy查询示例
 * @author erik.zhou
 */

import { render, screen, waitFor } from '@testing-library/react';

function AsyncComponent() {
    const [data, setData] = React.useState<string | null>(null);
    
    React.useEffect(() => {
        setTimeout(() => {
            setData('Loaded data');
        }, 1000);
    }, []);
    
    return (
        <div>
            {data ? <p>{data}</p> : <p>Loading...</p>}
        </div>
    );
}

describe('Query Variants', () => {
    // getBy - 立即查询，找不到抛出错误
    test('getBy - throws error if not found', () => {
        render(<div>Hello</div>);
        
        expect(() => screen.getByText(/goodbye/i)).toThrow();
    });
    
    // queryBy - 立即查询，找不到返回null
    test('queryBy - returns null if not found', () => {
        render(<div>Hello</div>);
        
        const element = screen.queryByText(/goodbye/i);
        expect(element).toBeNull();
    });
    
    // findBy - 异步查询，等待元素出现
    test('findBy - waits for element', async () => {
        render(<AsyncComponent />);
        
        const element = await screen.findByText(/loaded data/i);
        expect(element).toBeInTheDocument();
    });
});
```

### 3.3 getAllBy查询

```typescript
/**
 * getAllBy查询示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';

function TodoList() {
    const todos = ['Buy milk', 'Walk dog', 'Write code'];
    
    return (
        <ul>
            {todos.map((todo, index) => (
                <li key={index}>{todo}</li>
            ))}
        </ul>
    );
}

describe('Multiple Elements', () => {
    test('getAllByRole - 查询多个元素', () => {
        render(<TodoList />);
        
        const items = screen.getAllByRole('listitem');
        expect(items).toHaveLength(3);
    });
    
    test('getAllByText - 使用正则匹配', () => {
        render(<TodoList />);
        
        const items = screen.getAllByText(/buy|walk|write/i);
        expect(items).toHaveLength(3);
    });
});
```

### 3.4 within查询

```typescript
/**
 * within查询示例
 * @author erik.zhou
 */

import { render, screen, within } from '@testing-library/react';

function UserCard() {
    return (
        <div data-testid="user-card">
            <h2>John Doe</h2>
            <p>Email: john@example.com</p>
            <button>Edit</button>
        </div>
    );
}

describe('Within Queries', () => {
    test('should query within specific container', () => {
        render(<UserCard />);
        
        const card = screen.getByTestId('user-card');
        const button = within(card).getByRole('button', { name: /edit/i });
        
        expect(button).toBeInTheDocument();
    });
});
```

---

## 4. 用户交互

### 4.1 基础交互

```typescript
/**
 * 基础用户交互示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function Counter() {
    const [count, setCount] = React.useState(0);
    
    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setCount(count + 1)}>Increment</button>
            <button onClick={() => setCount(count - 1)}>Decrement</button>
            <button onClick={() => setCount(0)}>Reset</button>
        </div>
    );
}

describe('User Interactions', () => {
    test('should increment counter', async () => {
        const user = userEvent.setup();
        render(<Counter />);
        
        const incrementButton = screen.getByRole('button', { name: /increment/i });
        
        await user.click(incrementButton);
        
        expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
    });
    
    test('should decrement counter', async () => {
        const user = userEvent.setup();
        render(<Counter />);
        
        const decrementButton = screen.getByRole('button', { name: /decrement/i });
        
        await user.click(decrementButton);
        
        expect(screen.getByText(/count: -1/i)).toBeInTheDocument();
    });
});
```

### 4.2 表单交互

```typescript
/**
 * 表单交互示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function RegistrationForm() {
    const [formData, setFormData] = React.useState({
        username: '',
        email: '',
        password: '',
        agree: false
    });
    
    const handleSubmit = (e: React.FormEvent) => {
        e.preventDefault();
        console.log('Form submitted:', formData);
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <label htmlFor="username">Username</label>
            <input
                id="username"
                type="text"
                value={formData.username}
                onChange={(e) => setFormData({ ...formData, username: e.target.value })}
            />
            
            <label htmlFor="email">Email</label>
            <input
                id="email"
                type="email"
                value={formData.email}
                onChange={(e) => setFormData({ ...formData, email: e.target.value })}
            />
            
            <label htmlFor="password">Password</label>
            <input
                id="password"
                type="password"
                value={formData.password}
                onChange={(e) => setFormData({ ...formData, password: e.target.value })}
            />
            
            <label>
                <input
                    type="checkbox"
                    checked={formData.agree}
                    onChange={(e) => setFormData({ ...formData, agree: e.target.checked })}
                />
                I agree to terms
            </label>
            
            <button type="submit">Register</button>
        </form>
    );
}

describe('Form Interactions', () => {
    test('should fill out form', async () => {
        const user = userEvent.setup();
        render(<RegistrationForm />);
        
        // 输入文本
        await user.type(
            screen.getByLabelText(/username/i),
            'johndoe'
        );
        
        await user.type(
            screen.getByLabelText(/email/i),
            'john@example.com'
        );
        
        await user.type(
            screen.getByLabelText(/password/i),
            'password123'
        );
        
        // 勾选复选框
        await user.click(screen.getByRole('checkbox'));
        
        // 验证输入
        expect(screen.getByLabelText(/username/i)).toHaveValue('johndoe');
        expect(screen.getByLabelText(/email/i)).toHaveValue('john@example.com');
        expect(screen.getByRole('checkbox')).toBeChecked();
    });
    
    test('should submit form', async () => {
        const user = userEvent.setup();
        const handleSubmit = jest.fn();
        
        render(<RegistrationForm />);
        
        // 填写表单
        await user.type(screen.getByLabelText(/username/i), 'johndoe');
        await user.type(screen.getByLabelText(/email/i), 'john@example.com');
        await user.type(screen.getByLabelText(/password/i), 'password123');
        await user.click(screen.getByRole('checkbox'));
        
        // 提交表单
        await user.click(screen.getByRole('button', { name: /register/i }));
    });
});
```

### 4.3 键盘交互

```typescript
/**
 * 键盘交互示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function SearchBox() {
    const [query, setQuery] = React.useState('');
    const [results, setResults] = React.useState<string[]>([]);
    
    const handleKeyDown = (e: React.KeyboardEvent) => {
        if (e.key === 'Enter') {
            setResults(['Result 1', 'Result 2', 'Result 3']);
        }
        if (e.key === 'Escape') {
            setQuery('');
            setResults([]);
        }
    };
    
    return (
        <div>
            <input
                type="text"
                placeholder="Search..."
                value={query}
                onChange={(e) => setQuery(e.target.value)}
                onKeyDown={handleKeyDown}
            />
            <ul>
                {results.map((result, index) => (
                    <li key={index}>{result}</li>
                ))}
            </ul>
        </div>
    );
}

describe('Keyboard Interactions', () => {
    test('should search on Enter key', async () => {
        const user = userEvent.setup();
        render(<SearchBox />);
        
        const input = screen.getByPlaceholderText(/search/i);
        
        await user.type(input, 'test query{Enter}');
        
        expect(screen.getAllByRole('listitem')).toHaveLength(3);
    });
    
    test('should clear on Escape key', async () => {
        const user = userEvent.setup();
        render(<SearchBox />);
        
        const input = screen.getByPlaceholderText(/search/i);
        
        await user.type(input, 'test query{Enter}');
        await user.type(input, '{Escape}');
        
        expect(input).toHaveValue('');
        expect(screen.queryAllByRole('listitem')).toHaveLength(0);
    });
});
```

### 4.4 选择和下拉框

```typescript
/**
 * 选择和下拉框交互示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function CountrySelector() {
    const [country, setCountry] = React.useState('');
    
    return (
        <div>
            <label htmlFor="country">Country</label>
            <select
                id="country"
                value={country}
                onChange={(e) => setCountry(e.target.value)}
            >
                <option value="">Select a country</option>
                <option value="us">United States</option>
                <option value="uk">United Kingdom</option>
                <option value="ca">Canada</option>
            </select>
            {country && <p>Selected: {country}</p>}
        </div>
    );
}

describe('Select Interactions', () => {
    test('should select option', async () => {
        const user = userEvent.setup();
        render(<CountrySelector />);
        
        const select = screen.getByLabelText(/country/i);
        
        await user.selectOptions(select, 'us');
        
        expect(select).toHaveValue('us');
        expect(screen.getByText(/selected: us/i)).toBeInTheDocument();
    });
});
```


---

## 5. 异步测试

### 5.1 waitFor异步等待

```typescript
/**
 * waitFor异步等待示例
 * @author erik.zhou
 */

import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function DataFetcher() {
    const [data, setData] = React.useState<string | null>(null);
    const [loading, setLoading] = React.useState(false);
    const [error, setError] = React.useState<string | null>(null);
    
    const fetchData = async () => {
        setLoading(true);
        setError(null);
        
        try {
            const response = await fetch('/api/data');
            const result = await response.json();
            setData(result.message);
        } catch (err) {
            setError('Failed to fetch data');
        } finally {
            setLoading(false);
        }
    };
    
    return (
        <div>
            <button onClick={fetchData}>Fetch Data</button>
            {loading && <p>Loading...</p>}
            {error && <p role="alert">{error}</p>}
            {data && <p>{data}</p>}
        </div>
    );
}

describe('Async Testing with waitFor', () => {
    test('should fetch and display data', async () => {
        const user = userEvent.setup();
        
        // Mock fetch
        global.fetch = jest.fn(() =>
            Promise.resolve({
                json: () => Promise.resolve({ message: 'Hello from API' })
            })
        ) as jest.Mock;
        
        render(<DataFetcher />);
        
        const button = screen.getByRole('button', { name: /fetch data/i });
        await user.click(button);
        
        // 等待loading消失
        await waitFor(() => {
            expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
        });
        
        // 验证数据显示
        expect(screen.getByText(/hello from api/i)).toBeInTheDocument();
    });
    
    test('should handle fetch error', async () => {
        const user = userEvent.setup();
        
        // Mock fetch error
        global.fetch = jest.fn(() =>
            Promise.reject(new Error('Network error'))
        ) as jest.Mock;
        
        render(<DataFetcher />);
        
        const button = screen.getByRole('button', { name: /fetch data/i });
        await user.click(button);
        
        // 等待错误消息出现
        await waitFor(() => {
            expect(screen.getByRole('alert')).toHaveTextContent(/failed to fetch/i);
        });
    });
});
```

### 5.2 findBy异步查询

```typescript
/**
 * findBy异步查询示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';

function DelayedMessage() {
    const [message, setMessage] = React.useState('');
    
    React.useEffect(() => {
        setTimeout(() => {
            setMessage('Message loaded!');
        }, 1000);
    }, []);
    
    return <div>{message && <p>{message}</p>}</div>;
}

describe('Async Testing with findBy', () => {
    test('should wait for element to appear', async () => {
        render(<DelayedMessage />);
        
        // findBy会自动等待元素出现（默认超时1000ms）
        const message = await screen.findByText(/message loaded/i);
        
        expect(message).toBeInTheDocument();
    });
    
    test('should timeout if element does not appear', async () => {
        render(<div>No message</div>);
        
        // 自定义超时时间
        await expect(
            screen.findByText(/message loaded/i, {}, { timeout: 500 })
        ).rejects.toThrow();
    });
});
```

### 5.3 测试API调用

```typescript
/**
 * 测试API调用示例
 * @author erik.zhou
 */

import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

interface User {
    id: number;
    name: string;
    email: string;
}

function UserList() {
    const [users, setUsers] = React.useState<User[]>([]);
    const [loading, setLoading] = React.useState(false);
    
    const loadUsers = async () => {
        setLoading(true);
        try {
            const response = await fetch('/api/users');
            const data = await response.json();
            setUsers(data);
        } finally {
            setLoading(false);
        }
    };
    
    return (
        <div>
            <button onClick={loadUsers}>Load Users</button>
            {loading && <p>Loading users...</p>}
            <ul>
                {users.map(user => (
                    <li key={user.id}>
                        {user.name} - {user.email}
                    </li>
                ))}
            </ul>
        </div>
    );
}

describe('API Call Testing', () => {
    beforeEach(() => {
        // 清理fetch mock
        jest.clearAllMocks();
    });
    
    test('should load and display users', async () => {
        const user = userEvent.setup();
        
        const mockUsers = [
            { id: 1, name: 'John Doe', email: 'john@example.com' },
            { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
        ];
        
        global.fetch = jest.fn(() =>
            Promise.resolve({
                json: () => Promise.resolve(mockUsers)
            })
        ) as jest.Mock;
        
        render(<UserList />);
        
        const button = screen.getByRole('button', { name: /load users/i });
        await user.click(button);
        
        // 等待用户列表加载
        await waitFor(() => {
            expect(screen.getByText(/john doe/i)).toBeInTheDocument();
        });
        
        expect(screen.getByText(/jane smith/i)).toBeInTheDocument();
        expect(global.fetch).toHaveBeenCalledWith('/api/users');
    });
    
    test('should show loading state', async () => {
        const user = userEvent.setup();
        
        // 创建一个延迟的Promise
        let resolvePromise: (value: any) => void;
        const promise = new Promise(resolve => {
            resolvePromise = resolve;
        });
        
        global.fetch = jest.fn(() => promise) as jest.Mock;
        
        render(<UserList />);
        
        const button = screen.getByRole('button', { name: /load users/i });
        await user.click(button);
        
        // 验证loading状态
        expect(screen.getByText(/loading users/i)).toBeInTheDocument();
        
        // 解析Promise
        resolvePromise!({ json: () => Promise.resolve([]) });
        
        // 等待loading消失
        await waitFor(() => {
            expect(screen.queryByText(/loading users/i)).not.toBeInTheDocument();
        });
    });
});
```

### 5.4 测试定时器

```typescript
/**
 * 测试定时器示例
 * @author erik.zhou
 */

import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function Countdown() {
    const [count, setCount] = React.useState(10);
    const [isRunning, setIsRunning] = React.useState(false);
    
    React.useEffect(() => {
        if (!isRunning || count === 0) return;
        
        const timer = setTimeout(() => {
            setCount(count - 1);
        }, 1000);
        
        return () => clearTimeout(timer);
    }, [count, isRunning]);
    
    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setIsRunning(true)}>Start</button>
            <button onClick={() => setIsRunning(false)}>Stop</button>
            <button onClick={() => { setCount(10); setIsRunning(false); }}>Reset</button>
        </div>
    );
}

describe('Timer Testing', () => {
    beforeEach(() => {
        jest.useFakeTimers();
    });
    
    afterEach(() => {
        jest.useRealTimers();
    });
    
    test('should countdown when started', async () => {
        const user = userEvent.setup({ delay: null });
        render(<Countdown />);
        
        const startButton = screen.getByRole('button', { name: /start/i });
        await user.click(startButton);
        
        expect(screen.getByText(/count: 10/i)).toBeInTheDocument();
        
        // 快进1秒
        jest.advanceTimersByTime(1000);
        
        await waitFor(() => {
            expect(screen.getByText(/count: 9/i)).toBeInTheDocument();
        });
        
        // 快进5秒
        jest.advanceTimersByTime(5000);
        
        await waitFor(() => {
            expect(screen.getByText(/count: 4/i)).toBeInTheDocument();
        });
    });
    
    test('should stop countdown', async () => {
        const user = userEvent.setup({ delay: null });
        render(<Countdown />);
        
        const startButton = screen.getByRole('button', { name: /start/i });
        const stopButton = screen.getByRole('button', { name: /stop/i });
        
        await user.click(startButton);
        
        jest.advanceTimersByTime(3000);
        
        await waitFor(() => {
            expect(screen.getByText(/count: 7/i)).toBeInTheDocument();
        });
        
        await user.click(stopButton);
        
        jest.advanceTimersByTime(3000);
        
        // 计数应该停止在7
        expect(screen.getByText(/count: 7/i)).toBeInTheDocument();
    });
});
```

---

## 6. 测试Hooks

### 6.1 renderHook基础

```typescript
/**
 * renderHook基础示例
 * @author erik.zhou
 */

import { renderHook, act } from '@testing-library/react';

function useCounter(initialValue = 0) {
    const [count, setCount] = React.useState(initialValue);
    
    const increment = () => setCount(c => c + 1);
    const decrement = () => setCount(c => c - 1);
    const reset = () => setCount(initialValue);
    
    return { count, increment, decrement, reset };
}

describe('useCounter Hook', () => {
    test('should initialize with default value', () => {
        const { result } = renderHook(() => useCounter());
        
        expect(result.current.count).toBe(0);
    });
    
    test('should initialize with custom value', () => {
        const { result } = renderHook(() => useCounter(10));
        
        expect(result.current.count).toBe(10);
    });
    
    test('should increment count', () => {
        const { result } = renderHook(() => useCounter());
        
        act(() => {
            result.current.increment();
        });
        
        expect(result.current.count).toBe(1);
    });
    
    test('should decrement count', () => {
        const { result } = renderHook(() => useCounter(5));
        
        act(() => {
            result.current.decrement();
        });
        
        expect(result.current.count).toBe(4);
    });
    
    test('should reset count', () => {
        const { result } = renderHook(() => useCounter(10));
        
        act(() => {
            result.current.increment();
            result.current.increment();
        });
        
        expect(result.current.count).toBe(12);
        
        act(() => {
            result.current.reset();
        });
        
        expect(result.current.count).toBe(10);
    });
});
```

### 6.2 测试异步Hooks

```typescript
/**
 * 测试异步Hooks示例
 * @author erik.zhou
 */

import { renderHook, waitFor } from '@testing-library/react';

interface FetchState<T> {
    data: T | null;
    loading: boolean;
    error: Error | null;
}

function useFetch<T>(url: string): FetchState<T> {
    const [state, setState] = React.useState<FetchState<T>>({
        data: null,
        loading: true,
        error: null
    });
    
    React.useEffect(() => {
        let cancelled = false;
        
        const fetchData = async () => {
            try {
                const response = await fetch(url);
                const data = await response.json();
                
                if (!cancelled) {
                    setState({ data, loading: false, error: null });
                }
            } catch (error) {
                if (!cancelled) {
                    setState({ data: null, loading: false, error: error as Error });
                }
            }
        };
        
        fetchData();
        
        return () => {
            cancelled = true;
        };
    }, [url]);
    
    return state;
}

describe('useFetch Hook', () => {
    beforeEach(() => {
        jest.clearAllMocks();
    });
    
    test('should fetch data successfully', async () => {
        const mockData = { id: 1, name: 'Test' };
        
        global.fetch = jest.fn(() =>
            Promise.resolve({
                json: () => Promise.resolve(mockData)
            })
        ) as jest.Mock;
        
        const { result } = renderHook(() => useFetch('/api/test'));
        
        // 初始状态
        expect(result.current.loading).toBe(true);
        expect(result.current.data).toBeNull();
        
        // 等待数据加载
        await waitFor(() => {
            expect(result.current.loading).toBe(false);
        });
        
        expect(result.current.data).toEqual(mockData);
        expect(result.current.error).toBeNull();
    });
    
    test('should handle fetch error', async () => {
        const mockError = new Error('Network error');
        
        global.fetch = jest.fn(() =>
            Promise.reject(mockError)
        ) as jest.Mock;
        
        const { result } = renderHook(() => useFetch('/api/test'));
        
        await waitFor(() => {
            expect(result.current.loading).toBe(false);
        });
        
        expect(result.current.data).toBeNull();
        expect(result.current.error).toEqual(mockError);
    });
});
```

### 6.3 测试Hook依赖更新

```typescript
/**
 * 测试Hook依赖更新示例
 * @author erik.zhou
 */

import { renderHook } from '@testing-library/react';

function useDebounce<T>(value: T, delay: number): T {
    const [debouncedValue, setDebouncedValue] = React.useState(value);
    
    React.useEffect(() => {
        const timer = setTimeout(() => {
            setDebouncedValue(value);
        }, delay);
        
        return () => {
            clearTimeout(timer);
        };
    }, [value, delay]);
    
    return debouncedValue;
}

describe('useDebounce Hook', () => {
    beforeEach(() => {
        jest.useFakeTimers();
    });
    
    afterEach(() => {
        jest.useRealTimers();
    });
    
    test('should debounce value changes', () => {
        const { result, rerender } = renderHook(
            ({ value, delay }) => useDebounce(value, delay),
            { initialProps: { value: 'initial', delay: 500 } }
        );
        
        expect(result.current).toBe('initial');
        
        // 更新value
        rerender({ value: 'updated', delay: 500 });
        
        // 立即检查，值应该还是initial
        expect(result.current).toBe('initial');
        
        // 快进500ms
        jest.advanceTimersByTime(500);
        
        // 现在值应该更新了
        expect(result.current).toBe('updated');
    });
    
    test('should cancel previous timeout on rapid changes', () => {
        const { result, rerender } = renderHook(
            ({ value, delay }) => useDebounce(value, delay),
            { initialProps: { value: 'first', delay: 500 } }
        );
        
        rerender({ value: 'second', delay: 500 });
        jest.advanceTimersByTime(300);
        
        rerender({ value: 'third', delay: 500 });
        jest.advanceTimersByTime(300);
        
        // 应该还是first，因为timeout被取消了
        expect(result.current).toBe('first');
        
        // 再快进200ms，总共500ms
        jest.advanceTimersByTime(200);
        
        // 现在应该是third
        expect(result.current).toBe('third');
    });
});
```

### 6.4 测试自定义Hook组合

```typescript
/**
 * 测试自定义Hook组合示例
 * @author erik.zhou
 */

import { renderHook, act } from '@testing-library/react';

function useLocalStorage<T>(key: string, initialValue: T) {
    const [storedValue, setStoredValue] = React.useState<T>(() => {
        try {
            const item = window.localStorage.getItem(key);
            return item ? JSON.parse(item) : initialValue;
        } catch (error) {
            return initialValue;
        }
    });
    
    const setValue = (value: T | ((val: T) => T)) => {
        try {
            const valueToStore = value instanceof Function ? value(storedValue) : value;
            setStoredValue(valueToStore);
            window.localStorage.setItem(key, JSON.stringify(valueToStore));
        } catch (error) {
            console.error(error);
        }
    };
    
    return [storedValue, setValue] as const;
}

function useForm<T extends Record<string, any>>(initialValues: T) {
    const [values, setValues] = useLocalStorage('formData', initialValues);
    
    const handleChange = (name: keyof T, value: any) => {
        setValues(prev => ({ ...prev, [name]: value }));
    };
    
    const reset = () => {
        setValues(initialValues);
    };
    
    return { values, handleChange, reset };
}

describe('useForm Hook', () => {
    beforeEach(() => {
        localStorage.clear();
    });
    
    test('should initialize with default values', () => {
        const initialValues = { username: '', email: '' };
        const { result } = renderHook(() => useForm(initialValues));
        
        expect(result.current.values).toEqual(initialValues);
    });
    
    test('should update form values', () => {
        const initialValues = { username: '', email: '' };
        const { result } = renderHook(() => useForm(initialValues));
        
        act(() => {
            result.current.handleChange('username', 'johndoe');
        });
        
        expect(result.current.values.username).toBe('johndoe');
    });
    
    test('should persist values to localStorage', () => {
        const initialValues = { username: '', email: '' };
        const { result } = renderHook(() => useForm(initialValues));
        
        act(() => {
            result.current.handleChange('username', 'johndoe');
            result.current.handleChange('email', 'john@example.com');
        });
        
        const stored = JSON.parse(localStorage.getItem('formData') || '{}');
        expect(stored).toEqual({
            username: 'johndoe',
            email: 'john@example.com'
        });
    });
    
    test('should reset form values', () => {
        const initialValues = { username: '', email: '' };
        const { result } = renderHook(() => useForm(initialValues));
        
        act(() => {
            result.current.handleChange('username', 'johndoe');
        });
        
        expect(result.current.values.username).toBe('johndoe');
        
        act(() => {
            result.current.reset();
        });
        
        expect(result.current.values).toEqual(initialValues);
    });
});
```


---

## 7. 测试Context

### 7.1 基础Context测试

```typescript
/**
 * 基础Context测试示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

interface ThemeContextType {
    theme: 'light' | 'dark';
    toggleTheme: () => void;
}

const ThemeContext = React.createContext<ThemeContextType | undefined>(undefined);

function ThemeProvider({ children }: { children: React.ReactNode }) {
    const [theme, setTheme] = React.useState<'light' | 'dark'>('light');
    
    const toggleTheme = () => {
        setTheme(prev => prev === 'light' ? 'dark' : 'light');
    };
    
    return (
        <ThemeContext.Provider value={{ theme, toggleTheme }}>
            {children}
        </ThemeContext.Provider>
    );
}

function useTheme() {
    const context = React.useContext(ThemeContext);
    if (!context) {
        throw new Error('useTheme must be used within ThemeProvider');
    }
    return context;
}

function ThemedButton() {
    const { theme, toggleTheme } = useTheme();
    
    return (
        <button onClick={toggleTheme} style={{ background: theme === 'light' ? '#fff' : '#000' }}>
            Current theme: {theme}
        </button>
    );
}

describe('Theme Context', () => {
    test('should provide theme context', () => {
        render(
            <ThemeProvider>
                <ThemedButton />
            </ThemeProvider>
        );
        
        expect(screen.getByText(/current theme: light/i)).toBeInTheDocument();
    });
    
    test('should toggle theme', async () => {
        const user = userEvent.setup();
        
        render(
            <ThemeProvider>
                <ThemedButton />
            </ThemeProvider>
        );
        
        const button = screen.getByRole('button');
        
        expect(button).toHaveTextContent(/light/i);
        
        await user.click(button);
        
        expect(button).toHaveTextContent(/dark/i);
    });
});
```

### 7.2 测试多个Context

```typescript
/**
 * 测试多个Context示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';

interface User {
    id: number;
    name: string;
}

const UserContext = React.createContext<User | null>(null);
const SettingsContext = React.createContext({ notifications: true });

function UserProfile() {
    const user = React.useContext(UserContext);
    const settings = React.useContext(SettingsContext);
    
    if (!user) return <div>No user</div>;
    
    return (
        <div>
            <h1>{user.name}</h1>
            <p>Notifications: {settings.notifications ? 'On' : 'Off'}</p>
        </div>
    );
}

describe('Multiple Contexts', () => {
    test('should render with multiple contexts', () => {
        const user = { id: 1, name: 'John Doe' };
        const settings = { notifications: false };
        
        render(
            <UserContext.Provider value={user}>
                <SettingsContext.Provider value={settings}>
                    <UserProfile />
                </SettingsContext.Provider>
            </UserContext.Provider>
        );
        
        expect(screen.getByText(/john doe/i)).toBeInTheDocument();
        expect(screen.getByText(/notifications: off/i)).toBeInTheDocument();
    });
});
```


### 7.3 自定义Wrapper

```typescript
/**
 * 自定义Wrapper示例
 * @author erik.zhou
 */

import { render, screen, RenderOptions } from '@testing-library/react';

interface AuthContextType {
    user: User | null;
    login: (user: User) => void;
    logout: () => void;
}

const AuthContext = React.createContext<AuthContextType | undefined>(undefined);

function AuthProvider({ children }: { children: React.ReactNode }) {
    const [user, setUser] = React.useState<User | null>(null);
    
    const login = (user: User) => setUser(user);
    const logout = () => setUser(null);
    
    return (
        <AuthContext.Provider value={{ user, login, logout }}>
            {children}
        </AuthContext.Provider>
    );
}

// 创建自定义render函数
interface CustomRenderOptions extends Omit<RenderOptions, 'wrapper'> {
    initialUser?: User | null;
}

function customRender(
    ui: React.ReactElement,
    options?: CustomRenderOptions
) {
    const { initialUser, ...renderOptions } = options || {};
    
    function Wrapper({ children }: { children: React.ReactNode }) {
        return (
            <AuthProvider>
                <ThemeProvider>
                    {children}
                </ThemeProvider>
            </AuthProvider>
        );
    }
    
    return render(ui, { wrapper: Wrapper, ...renderOptions });
}

// 导出自定义render
export { customRender as render };

describe('Custom Wrapper', () => {
    test('should use custom render with providers', () => {
        function TestComponent() {
            const { user } = React.useContext(AuthContext)!;
            return <div>{user ? user.name : 'Not logged in'}</div>;
        }
        
        customRender(<TestComponent />);
        
        expect(screen.getByText(/not logged in/i)).toBeInTheDocument();
    });
});
```


---

## 8. 测试路由

### 8.1 React Router基础测试

```typescript
/**
 * React Router基础测试示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';
import userEvent from '@testing-library/user-event';

function Home() {
    return (
        <div>
            <h1>Home Page</h1>
            <Link to="/about">Go to About</Link>
        </div>
    );
}

function About() {
    return (
        <div>
            <h1>About Page</h1>
            <Link to="/">Go to Home</Link>
        </div>
    );
}

function App() {
    return (
        <BrowserRouter>
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/about" element={<About />} />
            </Routes>
        </BrowserRouter>
    );
}

describe('React Router', () => {
    test('should render home page', () => {
        render(<App />);
        
        expect(screen.getByText(/home page/i)).toBeInTheDocument();
    });
    
    test('should navigate to about page', async () => {
        const user = userEvent.setup();
        render(<App />);
        
        const link = screen.getByRole('link', { name: /go to about/i });
        await user.click(link);
        
        expect(screen.getByText(/about page/i)).toBeInTheDocument();
    });
});
```

### 8.2 使用MemoryRouter测试

```typescript
/**
 * MemoryRouter测试示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';
import { MemoryRouter, Routes, Route, useParams } from 'react-router-dom';

function UserProfile() {
    const { userId } = useParams();
    
    return (
        <div>
            <h1>User Profile</h1>
            <p>User ID: {userId}</p>
        </div>
    );
}

describe('MemoryRouter', () => {
    test('should render with initial route', () => {
        render(
            <MemoryRouter initialEntries={['/users/123']}>
                <Routes>
                    <Route path="/users/:userId" element={<UserProfile />} />
                </Routes>
            </MemoryRouter>
        );
        
        expect(screen.getByText(/user id: 123/i)).toBeInTheDocument();
    });
    
    test('should handle multiple routes', () => {
        render(
            <MemoryRouter initialEntries={['/users/456']}>
                <Routes>
                    <Route path="/users/:userId" element={<UserProfile />} />
                    <Route path="/" element={<div>Home</div>} />
                </Routes>
            </MemoryRouter>
        );
        
        expect(screen.getByText(/user id: 456/i)).toBeInTheDocument();
    });
});
```

### 8.3 测试路由守卫

```typescript
/**
 * 路由守卫测试示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';
import { MemoryRouter, Routes, Route, Navigate } from 'react-router-dom';

interface ProtectedRouteProps {
    isAuthenticated: boolean;
    children: React.ReactNode;
}

function ProtectedRoute({ isAuthenticated, children }: ProtectedRouteProps) {
    if (!isAuthenticated) {
        return <Navigate to="/login" replace />;
    }
    
    return <>{children}</>;
}

function Dashboard() {
    return <div>Dashboard</div>;
}

function Login() {
    return <div>Login Page</div>;
}

describe('Protected Routes', () => {
    test('should redirect to login when not authenticated', () => {
        render(
            <MemoryRouter initialEntries={['/dashboard']}>
                <Routes>
                    <Route
                        path="/dashboard"
                        element={
                            <ProtectedRoute isAuthenticated={false}>
                                <Dashboard />
                            </ProtectedRoute>
                        }
                    />
                    <Route path="/login" element={<Login />} />
                </Routes>
            </MemoryRouter>
        );
        
        expect(screen.getByText(/login page/i)).toBeInTheDocument();
    });
    
    test('should render dashboard when authenticated', () => {
        render(
            <MemoryRouter initialEntries={['/dashboard']}>
                <Routes>
                    <Route
                        path="/dashboard"
                        element={
                            <ProtectedRoute isAuthenticated={true}>
                                <Dashboard />
                            </ProtectedRoute>
                        }
                    />
                    <Route path="/login" element={<Login />} />
                </Routes>
            </MemoryRouter>
        );
        
        expect(screen.getByText(/dashboard/i)).toBeInTheDocument();
    });
});
```


---

## 9. Mock与Spy

### 9.1 Mock函数

```typescript
/**
 * Mock函数示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

interface ButtonProps {
    onClick: () => void;
    label: string;
}

function Button({ onClick, label }: ButtonProps) {
    return <button onClick={onClick}>{label}</button>;
}

describe('Mock Functions', () => {
    test('should call onClick handler', async () => {
        const user = userEvent.setup();
        const handleClick = jest.fn();
        
        render(<Button onClick={handleClick} label="Click me" />);
        
        const button = screen.getByRole('button');
        await user.click(button);
        
        expect(handleClick).toHaveBeenCalledTimes(1);
    });
    
    test('should call onClick multiple times', async () => {
        const user = userEvent.setup();
        const handleClick = jest.fn();
        
        render(<Button onClick={handleClick} label="Click me" />);
        
        const button = screen.getByRole('button');
        await user.click(button);
        await user.click(button);
        await user.click(button);
        
        expect(handleClick).toHaveBeenCalledTimes(3);
    });
});
```

### 9.2 Mock模块

```typescript
/**
 * Mock模块示例
 * @author erik.zhou
 */

import { render, screen, waitFor } from '@testing-library/react';

// api.ts
export async function fetchUser(id: number) {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
}

// UserComponent.tsx
function UserComponent({ userId }: { userId: number }) {
    const [user, setUser] = React.useState<any>(null);
    
    React.useEffect(() => {
        fetchUser(userId).then(setUser);
    }, [userId]);
    
    if (!user) return <div>Loading...</div>;
    
    return <div>User: {user.name}</div>;
}

// Mock整个模块
jest.mock('./api', () => ({
    fetchUser: jest.fn()
}));

import { fetchUser } from './api';

describe('Mock Modules', () => {
    test('should fetch and display user', async () => {
        const mockUser = { id: 1, name: 'John Doe' };
        (fetchUser as jest.Mock).mockResolvedValue(mockUser);
        
        render(<UserComponent userId={1} />);
        
        await waitFor(() => {
            expect(screen.getByText(/user: john doe/i)).toBeInTheDocument();
        });
        
        expect(fetchUser).toHaveBeenCalledWith(1);
    });
});
```

### 9.3 Spy函数

```typescript
/**
 * Spy函数示例
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function Logger() {
    const handleLog = () => {
        console.log('Button clicked');
    };
    
    return <button onClick={handleLog}>Log</button>;
}

describe('Spy Functions', () => {
    test('should spy on console.log', async () => {
        const user = userEvent.setup();
        const consoleSpy = jest.spyOn(console, 'log').mockImplementation();
        
        render(<Logger />);
        
        const button = screen.getByRole('button');
        await user.click(button);
        
        expect(consoleSpy).toHaveBeenCalledWith('Button clicked');
        
        consoleSpy.mockRestore();
    });
});
```

### 9.4 Mock定时器

```typescript
/**
 * Mock定时器示例
 * @author erik.zhou
 */

import { render, screen, waitFor } from '@testing-library/react';

function DelayedGreeting() {
    const [greeting, setGreeting] = React.useState('');
    
    React.useEffect(() => {
        const timer = setTimeout(() => {
            setGreeting('Hello!');
        }, 3000);
        
        return () => clearTimeout(timer);
    }, []);
    
    return <div>{greeting || 'Waiting...'}</div>;
}

describe('Mock Timers', () => {
    beforeEach(() => {
        jest.useFakeTimers();
    });
    
    afterEach(() => {
        jest.useRealTimers();
    });
    
    test('should show greeting after delay', async () => {
        render(<DelayedGreeting />);
        
        expect(screen.getByText(/waiting/i)).toBeInTheDocument();
        
        jest.advanceTimersByTime(3000);
        
        await waitFor(() => {
            expect(screen.getByText(/hello/i)).toBeInTheDocument();
        });
    });
});
```


---

## 10. 最佳实践

### 10.1 测试组织结构

```typescript
/**
 * 测试组织结构最佳实践
 * @author erik.zhou
 */

// ❌ 不好的实践 - 测试文件分散
// src/components/Button.tsx
// src/tests/Button.test.tsx
// src/tests/unit/Button.test.tsx

// ✅ 好的实践 - 测试文件与源文件同目录
// src/components/Button.tsx
// src/components/Button.test.tsx
// src/components/__tests__/Button.test.tsx

// 推荐的目录结构
/*
src/
  components/
    Button/
      Button.tsx
      Button.test.tsx
      Button.styles.ts
    UserCard/
      UserCard.tsx
      UserCard.test.tsx
      __tests__/
        UserCard.integration.test.tsx
  hooks/
    useAuth.ts
    useAuth.test.ts
  utils/
    helpers.ts
    helpers.test.ts
*/
```

### 10.2 测试命名规范

```typescript
/**
 * 测试命名规范最佳实践
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function LoginForm() {
    const [username, setUsername] = React.useState('');
    const [password, setPassword] = React.useState('');
    const [error, setError] = React.useState('');
    
    const handleSubmit = (e: React.FormEvent) => {
        e.preventDefault();
        if (!username || !password) {
            setError('请填写所有字段');
            return;
        }
        // 登录逻辑
    };
    
    return (
        <form onSubmit={handleSubmit}>
            <input
                type="text"
                placeholder="用户名"
                value={username}
                onChange={(e) => setUsername(e.target.value)}
            />
            <input
                type="password"
                placeholder="密码"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
            />
            <button type="submit">登录</button>
            {error && <p role="alert">{error}</p>}
        </form>
    );
}

describe('LoginForm', () => {
    // ✅ 好的命名 - 描述行为和预期结果
    test('should display error when submitting empty form', async () => {
        const user = userEvent.setup();
        render(<LoginForm />);
        
        await user.click(screen.getByRole('button', { name: /登录/i }));
        
        expect(screen.getByRole('alert')).toHaveTextContent(/请填写所有字段/i);
    });
    
    // ✅ 使用describe分组相关测试
    describe('when user fills form', () => {
        test('should enable submit button', async () => {
            const user = userEvent.setup();
            render(<LoginForm />);
            
            await user.type(screen.getByPlaceholderText(/用户名/i), 'testuser');
            await user.type(screen.getByPlaceholderText(/密码/i), 'password123');
            
            expect(screen.getByRole('button', { name: /登录/i })).toBeEnabled();
        });
    });
    
    // ❌ 不好的命名 - 不清晰
    test('test1', () => {
        // ...
    });
    
    test('it works', () => {
        // ...
    });
});
```


### 10.3 避免测试实现细节

```typescript
/**
 * 避免测试实现细节
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function Counter() {
    const [count, setCount] = React.useState(0);
    
    return (
        <div>
            <p data-testid="count-display">Count: {count}</p>
            <button onClick={() => setCount(count + 1)}>Increment</button>
        </div>
    );
}

describe('Counter - Bad Practices', () => {
    // ❌ 不好的实践 - 测试state
    test('should update state on click', async () => {
        const user = userEvent.setup();
        const { container } = render(<Counter />);
        
        // 不要直接访问组件state
        // expect(component.state.count).toBe(0);
    });
    
    // ❌ 不好的实践 - 测试CSS类名
    test('should have correct class', () => {
        const { container } = render(<Counter />);
        expect(container.firstChild).toHaveClass('counter-container');
    });
});

describe('Counter - Good Practices', () => {
    // ✅ 好的实践 - 测试用户可见的行为
    test('should increment count when button is clicked', async () => {
        const user = userEvent.setup();
        render(<Counter />);
        
        const button = screen.getByRole('button', { name: /increment/i });
        
        await user.click(button);
        
        expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
    });
});
```

### 10.4 使用测试工具函数

```typescript
/**
 * 测试工具函数最佳实践
 * @author erik.zhou
 */

import { render, screen, RenderOptions } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';

// 创建自定义render函数
interface CustomRenderOptions extends Omit<RenderOptions, 'wrapper'> {
    initialRoute?: string;
}

function renderWithRouter(
    ui: React.ReactElement,
    { initialRoute = '/', ...options }: CustomRenderOptions = {}
) {
    window.history.pushState({}, 'Test page', initialRoute);
    
    function Wrapper({ children }: { children: React.ReactNode }) {
        return <BrowserRouter>{children}</BrowserRouter>;
    }
    
    return render(ui, { wrapper: Wrapper, ...options });
}

// 创建测试数据工厂
function createUser(overrides = {}) {
    return {
        id: 1,
        name: 'Test User',
        email: 'test@example.com',
        ...overrides
    };
}

// 使用工具函数
describe('UserProfile', () => {
    test('should render user information', () => {
        const user = createUser({ name: 'John Doe' });
        
        renderWithRouter(<UserProfile user={user} />, {
            initialRoute: '/profile'
        });
        
        expect(screen.getByText(/john doe/i)).toBeInTheDocument();
    });
});
```


### 10.5 测试覆盖率策略

```typescript
/**
 * 测试覆盖率策略
 * @author erik.zhou
 */

// jest.config.js
module.exports = {
    collectCoverageFrom: [
        'src/**/*.{ts,tsx}',
        '!src/**/*.d.ts',
        '!src/**/*.stories.tsx',
        '!src/index.tsx'
    ],
    coverageThresholds: {
        global: {
            branches: 80,
            functions: 80,
            lines: 80,
            statements: 80
        }
    }
};

// ✅ 优先测试关键路径
describe('PaymentForm', () => {
    test('should process payment successfully', async () => {
        // 测试核心业务逻辑
    });
    
    test('should handle payment failure', async () => {
        // 测试错误处理
    });
    
    test('should validate card number', async () => {
        // 测试输入验证
    });
});

// ✅ 不要为了覆盖率而测试
// 避免测试第三方库、简单的getter/setter等
```

### 10.6 异步测试最佳实践

```typescript
/**
 * 异步测试最佳实践
 * @author erik.zhou
 */

import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

function SearchResults() {
    const [results, setResults] = React.useState<string[]>([]);
    const [loading, setLoading] = React.useState(false);
    
    const search = async (query: string) => {
        setLoading(true);
        try {
            const response = await fetch(`/api/search?q=${query}`);
            const data = await response.json();
            setResults(data);
        } finally {
            setLoading(false);
        }
    };
    
    return (
        <div>
            <button onClick={() => search('test')}>Search</button>
            {loading && <p>Loading...</p>}
            <ul>
                {results.map((result, index) => (
                    <li key={index}>{result}</li>
                ))}
            </ul>
        </div>
    );
}

describe('SearchResults - Async Best Practices', () => {
    // ✅ 使用findBy进行异步查询
    test('should display results after search', async () => {
        const user = userEvent.setup();
        
        global.fetch = jest.fn(() =>
            Promise.resolve({
                json: () => Promise.resolve(['Result 1', 'Result 2'])
            })
        ) as jest.Mock;
        
        render(<SearchResults />);
        
        await user.click(screen.getByRole('button', { name: /search/i }));
        
        // findBy自动等待元素出现
        const result = await screen.findByText(/result 1/i);
        expect(result).toBeInTheDocument();
    });
    
    // ✅ 使用waitFor等待状态变化
    test('should hide loading after search completes', async () => {
        const user = userEvent.setup();
        
        global.fetch = jest.fn(() =>
            Promise.resolve({
                json: () => Promise.resolve([])
            })
        ) as jest.Mock;
        
        render(<SearchResults />);
        
        await user.click(screen.getByRole('button', { name: /search/i }));
        
        // 等待loading消失
        await waitFor(() => {
            expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
        });
    });
    
    // ❌ 不好的实践 - 使用setTimeout
    test('bad practice - using setTimeout', async () => {
        const user = userEvent.setup();
        render(<SearchResults />);
        
        await user.click(screen.getByRole('button', { name: /search/i }));
        
        // 不要这样做
        await new Promise(resolve => setTimeout(resolve, 1000));
    });
});
```


### 10.7 可访问性测试

```typescript
/**
 * 可访问性测试最佳实践
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';

function AccessibleForm() {
    return (
        <form>
            <label htmlFor="email">Email</label>
            <input id="email" type="email" required />
            
            <button type="submit">Submit</button>
        </form>
    );
}

describe('Accessibility Best Practices', () => {
    // ✅ 使用语义化查询
    test('should use semantic queries', () => {
        render(<AccessibleForm />);
        
        // 优先使用role查询
        const button = screen.getByRole('button', { name: /submit/i });
        expect(button).toBeInTheDocument();
        
        // 使用label关联查询
        const input = screen.getByLabelText(/email/i);
        expect(input).toBeInTheDocument();
    });
    
    // ✅ 测试ARIA属性
    test('should have proper ARIA attributes', () => {
        function AlertMessage() {
            return <div role="alert">Error occurred</div>;
        }
        
        render(<AlertMessage />);
        
        const alert = screen.getByRole('alert');
        expect(alert).toHaveTextContent(/error occurred/i);
    });
});
```

### 10.8 性能测试建议

```typescript
/**
 * 性能测试建议
 * @author erik.zhou
 */

import { render, screen } from '@testing-library/react';

describe('Performance Best Practices', () => {
    // ✅ 避免不必要的重新渲染
    test('should minimize re-renders', () => {
        const renderSpy = jest.fn();
        
        function ExpensiveComponent() {
            renderSpy();
            return <div>Expensive Component</div>;
        }
        
        const { rerender } = render(<ExpensiveComponent />);
        
        expect(renderSpy).toHaveBeenCalledTimes(1);
        
        // 相同props不应触发重新渲染
        rerender(<ExpensiveComponent />);
        
        // 如果使用了React.memo，这里应该还是1次
        // expect(renderSpy).toHaveBeenCalledTimes(1);
    });
    
    // ✅ 测试虚拟列表性能
    test('should render large lists efficiently', () => {
        const items = Array.from({ length: 10000 }, (_, i) => ({
            id: i,
            name: `Item ${i}`
        }));
        
        function VirtualList({ items }: { items: typeof items }) {
            // 只渲染可见项
            const visibleItems = items.slice(0, 20);
            
            return (
                <ul>
                    {visibleItems.map(item => (
                        <li key={item.id}>{item.name}</li>
                    ))}
                </ul>
            );
        }
        
        const { container } = render(<VirtualList items={items} />);
        
        // 验证只渲染了可见项
        const listItems = container.querySelectorAll('li');
        expect(listItems).toHaveLength(20);
    });
});
```


### 10.9 测试隔离与清理

```typescript
/**
 * 测试隔离与清理最佳实践
 * @author erik.zhou
 */

import { render, screen, cleanup } from '@testing-library/react';

describe('Test Isolation', () => {
    // ✅ 每个测试后自动清理
    afterEach(() => {
        cleanup();
        jest.clearAllMocks();
        localStorage.clear();
    });
    
    // ✅ 使用beforeEach设置通用状态
    beforeEach(() => {
        // 重置全局状态
        global.fetch = jest.fn();
    });
    
    test('test 1', () => {
        // 测试逻辑
    });
    
    test('test 2', () => {
        // 这个测试不会受test 1影响
    });
});

// ✅ 避免测试间的依赖
describe('Independent Tests', () => {
    // ❌ 不好的实践 - 测试间有依赖
    let sharedState: any;
    
    test('should set state', () => {
        sharedState = { value: 1 };
    });
    
    test('should use state from previous test', () => {
        // 依赖上一个测试的结果
        expect(sharedState.value).toBe(1);
    });
    
    // ✅ 好的实践 - 每个测试独立
    test('should be independent 1', () => {
        const localState = { value: 1 };
        expect(localState.value).toBe(1);
    });
    
    test('should be independent 2', () => {
        const localState = { value: 2 };
        expect(localState.value).toBe(2);
    });
});
```

### 10.10 调试技巧

```typescript
/**
 * 调试技巧
 * @author erik.zhou
 */

import { render, screen, debug } from '@testing-library/react';

function DebugExample() {
    return (
        <div>
            <h1>Title</h1>
            <p>Content</p>
        </div>
    );
}

describe('Debugging Tips', () => {
    test('should use debug utilities', () => {
        const { debug, container } = render(<DebugExample />);
        
        // 打印整个DOM树
        debug();
        
        // 打印特定元素
        debug(screen.getByRole('heading'));
        
        // 使用screen.logTestingPlaygroundURL
        screen.logTestingPlaygroundURL();
        
        // 查看元素的可访问性信息
        console.log(screen.getByRole('heading').outerHTML);
    });
    
    // ✅ 使用screen.debug查看当前DOM
    test('should debug when test fails', () => {
        render(<DebugExample />);
        
        // 如果找不到元素，先debug看看DOM结构
        screen.debug();
        
        // 然后修正查询
        expect(screen.getByText(/title/i)).toBeInTheDocument();
    });
});
```

### 10.11 常见错误处理

```typescript
/**
 * 常见错误处理
 * @author erik.zhou
 */

import { render, screen, waitFor } from '@testing-library/react';

describe('Common Error Handling', () => {
    // ✅ 处理"not wrapped in act"警告
    test('should handle act warnings', async () => {
        function AsyncComponent() {
            const [data, setData] = React.useState('');
            
            React.useEffect(() => {
                setTimeout(() => {
                    setData('loaded');
                }, 100);
            }, []);
            
            return <div>{data}</div>;
        }
        
        render(<AsyncComponent />);
        
        // 使用waitFor等待异步更新
        await waitFor(() => {
            expect(screen.getByText(/loaded/i)).toBeInTheDocument();
        });
    });
    
    // ✅ 处理"Unable to find element"错误
    test('should handle element not found', () => {
        render(<div>Hello</div>);
        
        // 使用queryBy而不是getBy来检查元素不存在
        expect(screen.queryByText(/goodbye/i)).not.toBeInTheDocument();
        
        // 或者使用expect().toThrow()
        expect(() => screen.getByText(/goodbye/i)).toThrow();
    });
});
```


### 10.12 集成测试策略

```typescript
/**
 * 集成测试策略
 * @author erik.zhou
 */

import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { BrowserRouter } from 'react-router-dom';

// 完整的用户流程测试
function App() {
    return (
        <BrowserRouter>
            <Routes>
                <Route path="/" element={<Home />} />
                <Route path="/login" element={<Login />} />
                <Route path="/dashboard" element={<Dashboard />} />
            </Routes>
        </BrowserRouter>
    );
}

describe('Integration Tests', () => {
    // ✅ 测试完整的用户流程
    test('should complete user login flow', async () => {
        const user = userEvent.setup();
        
        // Mock API
        global.fetch = jest.fn((url) => {
            if (url.includes('/api/login')) {
                return Promise.resolve({
                    json: () => Promise.resolve({ token: 'fake-token' })
                });
            }
            return Promise.reject(new Error('Unknown endpoint'));
        }) as jest.Mock;
        
        render(<App />);
        
        // 1. 导航到登录页
        const loginLink = screen.getByRole('link', { name: /login/i });
        await user.click(loginLink);
        
        // 2. 填写登录表单
        await user.type(screen.getByLabelText(/username/i), 'testuser');
        await user.type(screen.getByLabelText(/password/i), 'password123');
        
        // 3. 提交表单
        await user.click(screen.getByRole('button', { name: /submit/i }));
        
        // 4. 验证跳转到dashboard
        await waitFor(() => {
            expect(screen.getByText(/dashboard/i)).toBeInTheDocument();
        });
    });
});
```

### 10.13 测试文档化

```typescript
/**
 * 测试文档化最佳实践
 * @author erik.zhou
 */

/**
 * UserProfile组件测试套件
 * 
 * 测试范围：
 * - 用户信息显示
 * - 编辑功能
 * - 权限控制
 * 
 * 依赖：
 * - @testing-library/react
 * - @testing-library/user-event
 */
describe('UserProfile Component', () => {
    /**
     * 测试场景：显示用户基本信息
     * 
     * 前置条件：用户已登录
     * 测试步骤：
     * 1. 渲染UserProfile组件
     * 2. 验证用户名显示
     * 3. 验证邮箱显示
     * 
     * 预期结果：所有用户信息正确显示
     */
    test('should display user information', () => {
        const user = {
            id: 1,
            name: 'John Doe',
            email: 'john@example.com'
        };
        
        render(<UserProfile user={user} />);
        
        expect(screen.getByText(/john doe/i)).toBeInTheDocument();
        expect(screen.getByText(/john@example.com/i)).toBeInTheDocument();
    });
});
```

### 10.14 持续集成配置

```typescript
/**
 * CI/CD配置示例
 * @author erik.zhou
 */

// .github/workflows/test.yml
/*
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v2
        with:
          files: ./coverage/lcov.info
*/

// package.json scripts
/*
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:ci": "jest --ci --coverage --maxWorkers=2"
  }
}
*/
```

---

## 总结

### 核心要点

1. **查询优先级**
   - 优先使用可访问性查询（getByRole、getByLabelText）
   - 避免使用testId，除非必要
   - 使用语义化查询提高测试可维护性

2. **用户交互**
   - 使用@testing-library/user-event模拟真实用户行为
   - 所有交互都应该是异步的
   - 测试用户可见的行为，而非实现细节

3. **异步处理**
   - 使用findBy进行异步查询
   - 使用waitFor等待状态变化
   - 避免使用setTimeout

4. **测试组织**
   - 测试文件与源文件同目录
   - 使用describe分组相关测试
   - 清晰的测试命名

5. **最佳实践**
   - 避免测试实现细节
   - 保持测试独立性
   - 使用自定义render函数
   - 关注可访问性

### 学习资源

- [React Testing Library官方文档](https://testing-library.com/react)
- [Testing Library查询指南](https://testing-library.com/docs/queries/about)
- [Common mistakes with React Testing Library](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library)
- [Testing Playground](https://testing-playground.com/)

### 下一步

- 学习Playwright进行E2E测试
- 掌握Cypress进行集成测试
- 了解视觉回归测试
- 探索性能测试工具

---

**最后更新时间：** 2026-03-05  
**@author erik.zhou**
