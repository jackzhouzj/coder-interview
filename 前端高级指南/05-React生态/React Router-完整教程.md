# React Router - 完整教程

## 目录
1. [基础概念](#基础概念)
2. [路由配置](#路由配置)
3. [导航与链接](#导航与链接)
4. [路由参数](#路由参数)
5. [嵌套路由](#嵌套路由)
6. [路由守卫](#路由守卫)
7. [数据加载](#数据加载)
8. [高级特性](#高级特性)

## 基础概念

### 安装与配置
```bash
# 安装React Router v6
npm install react-router-dom

# 或使用yarn
yarn add react-router-dom
```

### 基本使用
```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Home from './pages/Home';
import About from './pages/About';
import NotFound from './pages/NotFound';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

### 路由模式
```jsx
import {
  BrowserRouter,  // HTML5 History API
  HashRouter,     // Hash模式
  MemoryRouter    // 内存模式（测试用）
} from 'react-router-dom';

// 1. BrowserRouter（推荐）
// URL: http://example.com/about
<BrowserRouter>
  <App />
</BrowserRouter>

// 2. HashRouter
// URL: http://example.com/#/about
<HashRouter>
  <App />
</HashRouter>

// 3. MemoryRouter（不改变URL）
<MemoryRouter>
  <App />
</MemoryRouter>
```

## 路由配置

### 基础路由配置
```jsx
import { Routes, Route } from 'react-router-dom';

function App() {
  return (
    <Routes>
      {/* 精确匹配 */}
      <Route path="/" element={<Home />} />
      
      {/* 路径匹配 */}
      <Route path="/about" element={<About />} />
      <Route path="/contact" element={<Contact />} />
      
      {/* 动态路由 */}
      <Route path="/users/:id" element={<UserDetail />} />
      
      {/* 通配符路由（404） */}
      <Route path="*" element={<NotFound />} />
    </Routes>
  );
}
```

### 使用配置对象
```jsx
import { useRoutes } from 'react-router-dom';

function App() {
  const routes = useRoutes([
    {
      path: '/',
      element: <Layout />,
      children: [
        { index: true, element: <Home /> },
        { path: 'about', element: <About /> },
        { path: 'contact', element: <Contact /> }
      ]
    },
    {
      path: '/users',
      element: <UsersLayout />,
      children: [
        { index: true, element: <UserList /> },
        { path: ':id', element: <UserDetail /> },
        { path: ':id/edit', element: <UserEdit /> }
      ]
    },
    { path: '*', element: <NotFound /> }
  ]);

  return routes;
}
```

### 路由懒加载
```jsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// 懒加载组件
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <Suspense fallback={<div>加载中...</div>}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </Suspense>
  );
}
```

## 导航与链接

### Link组件
```jsx
import { Link } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      {/* 基础链接 */}
      <Link to="/">首页</Link>
      <Link to="/about">关于</Link>
      
      {/* 带状态的链接 */}
      <Link 
        to="/profile" 
        state={{ from: 'navigation' }}
      >
        个人中心
      </Link>
      
      {/* 相对路径 */}
      <Link to="../parent">返回上级</Link>
      <Link to="child">子页面</Link>
      
      {/* 替换历史记录 */}
      <Link to="/login" replace>登录</Link>
    </nav>
  );
}
```

### NavLink组件
```jsx
import { NavLink } from 'react-router-dom';

function Navigation() {
  return (
    <nav>
      {/* 自动添加active类名 */}
      <NavLink 
        to="/"
        className={({ isActive }) => isActive ? 'active' : ''}
      >
        首页
      </NavLink>
      
      {/* 自定义样式 */}
      <NavLink
        to="/about"
        style={({ isActive }) => ({
          color: isActive ? 'red' : 'black',
          fontWeight: isActive ? 'bold' : 'normal'
        })}
      >
        关于
      </NavLink>
      
      {/* 自定义渲染 */}
      <NavLink to="/profile">
        {({ isActive }) => (
          <span className={isActive ? 'active-link' : ''}>
            个人中心
          </span>
        )}
      </NavLink>
    </nav>
  );
}
```

### 编程式导航
```jsx
import { useNavigate } from 'react-router-dom';

function LoginForm() {
  const navigate = useNavigate();
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      await login();
      
      // 导航到首页
      navigate('/');
      
      // 导航并替换历史记录
      navigate('/dashboard', { replace: true });
      
      // 导航并传递状态
      navigate('/profile', { 
        state: { from: 'login' } 
      });
      
      // 相对导航
      navigate('../parent');
      navigate('child');
      
      // 返回上一页
      navigate(-1);
      
      // 前进
      navigate(1);
    } catch (error) {
      console.error('登录失败', error);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* 表单内容 */}
    </form>
  );
}
```

## 路由参数

### URL参数
```jsx
import { useParams } from 'react-router-dom';

// 路由配置
<Route path="/users/:id" element={<UserDetail />} />
<Route path="/posts/:category/:id" element={<PostDetail />} />

// 组件中获取参数
function UserDetail() {
  const { id } = useParams();
  
  return <div>用户ID: {id}</div>;
}

function PostDetail() {
  const { category, id } = useParams();
  
  return (
    <div>
      分类: {category}
      文章ID: {id}
    </div>
  );
}
```

### 查询参数
```jsx
import { useSearchParams } from 'react-router-dom';

function SearchPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  
  // 获取查询参数
  const query = searchParams.get('q');
  const page = searchParams.get('page') || '1';
  const sort = searchParams.get('sort') || 'date';
  
  // 设置查询参数
  const handleSearch = (keyword) => {
    setSearchParams({ q: keyword, page: '1' });
  };
  
  // 更新单个参数
  const handlePageChange = (newPage) => {
    setSearchParams(prev => {
      prev.set('page', newPage);
      return prev;
    });
  };
  
  // 删除参数
  const clearFilters = () => {
    setSearchParams({});
  };
  
  return (
    <div>
      <p>搜索关键词: {query}</p>
      <p>当前页: {page}</p>
      <p>排序方式: {sort}</p>
      
      <button onClick={() => handlePageChange('2')}>
        下一页
      </button>
      <button onClick={clearFilters}>
        清除筛选
      </button>
    </div>
  );
}
```

### 获取完整URL信息
```jsx
import { useLocation } from 'react-router-dom';

function CurrentPage() {
  const location = useLocation();
  
  console.log('pathname:', location.pathname);  // /users/123
  console.log('search:', location.search);      // ?tab=profile
  console.log('hash:', location.hash);          // #section
  console.log('state:', location.state);        // 传递的状态
  console.log('key:', location.key);            // 唯一标识
  
  return (
    <div>
      <p>当前路径: {location.pathname}</p>
      {location.state?.from && (
        <p>来自: {location.state.from}</p>
      )}
    </div>
  );
}
```

## 嵌套路由

### 基础嵌套
```jsx
import { Routes, Route, Outlet } from 'react-router-dom';

// 布局组件
function Layout() {
  return (
    <div>
      <header>
        <nav>
          <Link to="/">首页</Link>
          <Link to="/about">关于</Link>
        </nav>
      </header>
      
      <main>
        {/* 渲染子路由 */}
        <Outlet />
      </main>
      
      <footer>版权信息</footer>
    </div>
  );
}

// 路由配置
function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="about" element={<About />} />
        <Route path="contact" element={<Contact />} />
      </Route>
    </Routes>
  );
}
```

### 多层嵌套
```jsx
function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        
        {/* 用户模块 */}
        <Route path="users" element={<UsersLayout />}>
          <Route index element={<UserList />} />
          <Route path=":id" element={<UserDetail />} />
          <Route path=":id/edit" element={<UserEdit />} />
          
          {/* 用户设置 */}
          <Route path=":id/settings" element={<SettingsLayout />}>
            <Route index element={<GeneralSettings />} />
            <Route path="security" element={<SecuritySettings />} />
            <Route path="privacy" element={<PrivacySettings />} />
          </Route>
        </Route>
      </Route>
    </Routes>
  );
}

// UsersLayout组件
function UsersLayout() {
  return (
    <div className="users-layout">
      <aside>
        <nav>
          <Link to="/users">用户列表</Link>
        </nav>
      </aside>
      <div className="users-content">
        <Outlet />
      </div>
    </div>
  );
}
```

### 传递Context给子路由
```jsx
import { Outlet, useOutletContext } from 'react-router-dom';

// 父组件
function Dashboard() {
  const [user, setUser] = useState(null);
  
  return (
    <div>
      <h1>Dashboard</h1>
      <Outlet context={{ user, setUser }} />
    </div>
  );
}

// 子组件
function Profile() {
  const { user, setUser } = useOutletContext();
  
  return (
    <div>
      <h2>个人资料</h2>
      <p>用户名: {user?.name}</p>
    </div>
  );
}
```

## 路由守卫

### 认证守卫
```jsx
import { Navigate, useLocation } from 'react-router-dom';

// 认证守卫组件
function RequireAuth({ children }) {
  const isAuthenticated = useAuth();
  const location = useLocation();
  
  if (!isAuthenticated) {
    // 重定向到登录页，并保存当前位置
    return <Navigate to="/login" state={{ from: location }} replace />;
  }
  
  return children;
}

// 使用守卫
function App() {
  return (
    <Routes>
      <Route path="/login" element={<Login />} />
      
      {/* 需要认证的路由 */}
      <Route
        path="/dashboard"
        element={
          <RequireAuth>
            <Dashboard />
          </RequireAuth>
        }
      />
      
      <Route
        path="/profile"
        element={
          <RequireAuth>
            <Profile />
          </RequireAuth>
        }
      />
    </Routes>
  );
}
```

### 权限守卫
```jsx
function RequirePermission({ children, permission }) {
  const userPermissions = usePermissions();
  
  if (!userPermissions.includes(permission)) {
    return <Navigate to="/403" replace />;
  }
  
  return children;
}

// 使用
<Route
  path="/admin"
  element={
    <RequireAuth>
      <RequirePermission permission="admin">
        <AdminPanel />
      </RequirePermission>
    </RequireAuth>
  }
/>
```

### 路由拦截器
```jsx
function RouteGuard({ children }) {
  const location = useLocation();
  const navigate = useNavigate();
  
  useEffect(() => {
    // 页面访问统计
    trackPageView(location.pathname);
    
    // 页面标题
    document.title = getPageTitle(location.pathname);
    
    // 滚动到顶部
    window.scrollTo(0, 0);
  }, [location]);
  
  return children;
}

// 应用守卫
function App() {
  return (
    <BrowserRouter>
      <RouteGuard>
        <Routes>
          {/* 路由配置 */}
        </Routes>
      </RouteGuard>
    </BrowserRouter>
  );
}
```

## 数据加载

### Loader（React Router v6.4+）
```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

// 定义loader
async function userLoader({ params }) {
  const response = await fetch(`/api/users/${params.id}`);
  if (!response.ok) {
    throw new Response('Not Found', { status: 404 });
  }
  return response.json();
}

// 路由配置
const router = createBrowserRouter([
  {
    path: '/users/:id',
    element: <UserDetail />,
    loader: userLoader,
    errorElement: <ErrorPage />
  }
]);

// 组件中使用数据
import { useLoaderData } from 'react-router-dom';

function UserDetail() {
  const user = useLoaderData();
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// 应用
function App() {
  return <RouterProvider router={router} />;
}
```

### Action（表单提交）
```jsx
async function createUserAction({ request }) {
  const formData = await request.formData();
  const user = {
    name: formData.get('name'),
    email: formData.get('email')
  };
  
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(user)
  });
  
  if (!response.ok) {
    return { error: 'Failed to create user' };
  }
  
  return redirect('/users');
}

// 路由配置
const router = createBrowserRouter([
  {
    path: '/users/new',
    element: <CreateUser />,
    action: createUserAction
  }
]);

// 组件
import { Form, useActionData } from 'react-router-dom';

function CreateUser() {
  const actionData = useActionData();
  
  return (
    <Form method="post">
      {actionData?.error && (
        <div className="error">{actionData.error}</div>
      )}
      
      <input type="text" name="name" required />
      <input type="email" name="email" required />
      <button type="submit">创建用户</button>
    </Form>
  );
}
```

### 自定义数据加载
```jsx
function UserDetail() {
  const { id } = useParams();
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let cancelled = false;
    
    async function fetchUser() {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${id}`);
        const data = await response.json();
        
        if (!cancelled) {
          setUser(data);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err.message);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }
    
    fetchUser();
    
    return () => {
      cancelled = true;
    };
  }, [id]);
  
  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误: {error}</div>;
  if (!user) return <div>用户不存在</div>;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

## 高级特性

### 路由匹配
```jsx
import { useMatch, useResolvedPath } from 'react-router-dom';

function CustomNavLink({ to, children }) {
  const resolved = useResolvedPath(to);
  const match = useMatch({ path: resolved.pathname, end: true });
  
  return (
    <Link
      to={to}
      className={match ? 'active' : ''}
    >
      {children}
    </Link>
  );
}
```

### 阻止导航
```jsx
import { useBlocker } from 'react-router-dom';

function EditForm() {
  const [formData, setFormData] = useState({});
  const [isDirty, setIsDirty] = useState(false);
  
  // 阻止导航
  useBlocker(
    ({ currentLocation, nextLocation }) => {
      return (
        isDirty &&
        currentLocation.pathname !== nextLocation.pathname
      );
    }
  );
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    await saveData(formData);
    setIsDirty(false);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.name}
        onChange={(e) => {
          setFormData({ ...formData, name: e.target.value });
          setIsDirty(true);
        }}
      />
      <button type="submit">保存</button>
    </form>
  );
}
```

### 路由过渡动画
```jsx
import { useLocation } from 'react-router-dom';
import { CSSTransition, TransitionGroup } from 'react-transition-group';

function AnimatedRoutes() {
  const location = useLocation();
  
  return (
    <TransitionGroup>
      <CSSTransition
        key={location.key}
        classNames="fade"
        timeout={300}
      >
        <Routes location={location}>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
        </Routes>
      </CSSTransition>
    </TransitionGroup>
  );
}

// CSS
.fade-enter {
  opacity: 0;
}

.fade-enter-active {
  opacity: 1;
  transition: opacity 300ms;
}

.fade-exit {
  opacity: 1;
}

.fade-exit-active {
  opacity: 0;
  transition: opacity 300ms;
}
```

### 滚动恢复
```jsx
import { useEffect } from 'react';
import { useLocation } from 'react-router-dom';

function ScrollToTop() {
  const { pathname } = useLocation();
  
  useEffect(() => {
    window.scrollTo(0, 0);
  }, [pathname]);
  
  return null;
}

// 使用
function App() {
  return (
    <BrowserRouter>
      <ScrollToTop />
      <Routes>
        {/* 路由配置 */}
      </Routes>
    </BrowserRouter>
  );
}
```

## 最佳实践

1. **使用BrowserRouter而不是HashRouter**
2. **合理使用路由懒加载**
3. **统一管理路由配置**
4. **使用路由守卫保护敏感页面**
5. **正确处理404页面**
6. **使用Loader预加载数据**
7. **避免在组件中硬编码路径**
8. **使用相对路径简化导航**

## 常见问题

### Q1: v5升级到v6的主要变化？
- Switch改为Routes
- Route的component/render改为element
- 移除了exact属性
- 嵌套路由使用Outlet
- useHistory改为useNavigate

### Q2: 如何实现路由级别的代码分割？
使用React.lazy和Suspense实现懒加载

### Q3: 如何处理路由权限？
使用路由守卫组件包裹需要权限的路由

---

**@author erik.zhou**
