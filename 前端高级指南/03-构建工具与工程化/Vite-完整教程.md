# Vite - 完整教程

## 目录
1. [Vite简介](#vite简介)
2. [快速开始](#快速开始)
3. [核心特性](#核心特性)
4. [配置详解](#配置详解)
5. [插件开发](#插件开发)
6. [性能优化](#性能优化)
7. [生产构建](#生产构建)

## Vite简介

### 什么是Vite
Vite是新一代前端构建工具，利用浏览器原生ES模块特性，提供极速的开发体验。

### 核心优势
- **极速冷启动**：无需打包，即时启动
- **即时热更新**：真正的按需编译
- **真正的按需编译**：只编译当前页面
- **开箱即用**：内置常用功能
- **Rollup打包**：生产环境优化

### Vite vs Webpack
```
Webpack开发流程：
启动 → 打包所有模块 → 启动服务器 → 浏览器请求

Vite开发流程：
启动服务器 → 浏览器请求 → 按需编译模块
```

## 快速开始

### 创建项目
```bash
# 使用npm
npm create vite@latest my-app

# 使用yarn
yarn create vite my-app

# 使用pnpm
pnpm create vite my-app

# 指定模板
npm create vite@latest my-app -- --template react
npm create vite@latest my-app -- --template vue
npm create vite@latest my-app -- --template react-ts
```

### 项目结构
```
my-app/
├── public/
│   └── favicon.ico
├── src/
│   ├── assets/
│   ├── components/
│   ├── App.jsx
│   └── main.jsx
├── index.html
├── package.json
└── vite.config.js
```

### 基本命令
```bash
# 开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览生产构建
npm run preview
```

## 核心特性

### 原生ESM支持
```javascript
// 直接使用ES模块导入
import { createApp } from 'vue';
import App from './App.vue';

// 动态导入
const module = await import('./utils.js');

// 导入JSON
import config from './config.json';

// 导入CSS
import './style.css';

// 导入静态资源
import logo from './logo.png';
```

### 热模块替换（HMR）
```javascript
// main.js
import { createApp } from 'vue';
import App from './App.vue';

const app = createApp(App);
app.mount('#app');

// HMR API
if (import.meta.hot) {
  import.meta.hot.accept('./App.vue', (newModule) => {
    // 处理更新
    app.unmount();
    createApp(newModule.default).mount('#app');
  });
  
  // 自定义HMR处理
  import.meta.hot.accept((newModule) => {
    console.log('模块已更新');
  });
  
  // 监听其他模块
  import.meta.hot.accept('./utils.js', (newModule) => {
    // 更新工具函数
  });
  
  // 清理副作用
  import.meta.hot.dispose(() => {
    console.log('模块即将被替换');
  });
}
```

### TypeScript支持
```typescript
// vite-env.d.ts
/// <reference types="vite/client" />

// 环境变量类型
interface ImportMetaEnv {
  readonly VITE_APP_TITLE: string;
  readonly VITE_API_URL: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}

// 使用
console.log(import.meta.env.VITE_APP_TITLE);
```

### CSS处理
```javascript
// 导入CSS
import './style.css';

// CSS Modules
import styles from './style.module.css';
console.log(styles.className);

// CSS预处理器
import './style.scss';
import './style.less';

// PostCSS（自动读取postcss.config.js）
// postcss.config.js
export default {
  plugins: {
    autoprefixer: {},
    'postcss-nested': {}
  }
};
```

### 静态资源处理
```javascript
// 导入资源URL
import imgUrl from './img.png';
console.log(imgUrl); // /assets/img.2d8efhg.png

// 显式URL导入
import assetUrl from './asset.png?url';

// 导入为字符串
import svg from './icon.svg?raw';

// 导入为Worker
import Worker from './worker.js?worker';
const worker = new Worker();

// 导入为WebAssembly
import init from './lib.wasm?init';
init().then((exports) => {
  exports.test();
});
```

## 配置详解

### 基础配置
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  // 项目根目录
  root: process.cwd(),
  
  // 开发服务器配置
  server: {
    host: '0.0.0.0',
    port: 3000,
    open: true,
    cors: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  },
  
  // 构建配置
  build: {
    outDir: 'dist',
    assetsDir: 'assets',
    sourcemap: false,
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true
      }
    },
    rollupOptions: {
      output: {
        chunkFileNames: 'js/[name]-[hash].js',
        entryFileNames: 'js/[name]-[hash].js',
        assetFileNames: '[ext]/[name]-[hash].[ext]'
      }
    }
  },
  
  // 路径别名
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@utils': path.resolve(__dirname, 'src/utils')
    },
    extensions: ['.js', '.jsx', '.ts', '.tsx', '.json']
  },
  
  // 插件
  plugins: [react()],
  
  // 环境变量前缀
  envPrefix: 'VITE_',
  
  // CSS配置
  css: {
    modules: {
      localsConvention: 'camelCase'
    },
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`
      }
    }
  }
});
```

### 多环境配置
```javascript
// vite.config.js
import { defineConfig, loadEnv } from 'vite';

export default defineConfig(({ command, mode }) => {
  // 加载环境变量
  const env = loadEnv(mode, process.cwd(), '');
  
  return {
    define: {
      __APP_ENV__: JSON.stringify(env.APP_ENV)
    },
    server: {
      port: mode === 'development' ? 3000 : 4000
    },
    build: {
      sourcemap: mode === 'development'
    }
  };
});

// .env.development
VITE_API_URL=http://localhost:8080
VITE_APP_TITLE=开发环境

// .env.production
VITE_API_URL=https://api.example.com
VITE_APP_TITLE=生产环境
```

### 条件配置
```javascript
export default defineConfig(({ command, mode }) => {
  if (command === 'serve') {
    // 开发环境配置
    return {
      server: {
        port: 3000
      }
    };
  } else {
    // 生产环境配置
    return {
      build: {
        minify: 'terser'
      }
    };
  }
});
```

## 插件开发

### 插件结构
```javascript
// my-plugin.js
export default function myPlugin(options = {}) {
  return {
    name: 'my-plugin',
    
    // 配置解析钩子
    config(config, { command }) {
      if (command === 'build') {
        config.build.minify = 'terser';
      }
    },
    
    // 配置服务器
    configureServer(server) {
      server.middlewares.use((req, res, next) => {
        if (req.url === '/custom') {
          res.end('Custom response');
        } else {
          next();
        }
      });
    },
    
    // 转换代码
    transform(code, id) {
      if (id.endsWith('.custom')) {
        return {
          code: transformCode(code),
          map: null
        };
      }
    },
    
    // 处理HMR
    handleHotUpdate({ file, server }) {
      if (file.endsWith('.custom')) {
        server.ws.send({
          type: 'custom',
          event: 'update'
        });
      }
    },
    
    // 构建开始
    buildStart() {
      console.log('构建开始');
    },
    
    // 构建结束
    buildEnd() {
      console.log('构建结束');
    }
  };
}

// 使用插件
import myPlugin from './plugins/my-plugin';

export default defineConfig({
  plugins: [
    myPlugin({
      option: 'value'
    })
  ]
});
```

### 实用插件示例

#### 自动导入插件
```javascript
// auto-import-plugin.js
import { parse } from '@babel/parser';
import traverse from '@babel/traverse';

export default function autoImportPlugin() {
  const imports = new Map();
  
  return {
    name: 'auto-import',
    
    transform(code, id) {
      if (!id.endsWith('.jsx') && !id.endsWith('.tsx')) {
        return;
      }
      
      const ast = parse(code, {
        sourceType: 'module',
        plugins: ['jsx', 'typescript']
      });
      
      const usedComponents = new Set();
      
      traverse(ast, {
        JSXElement(path) {
          const name = path.node.openingElement.name.name;
          if (name && /^[A-Z]/.test(name)) {
            usedComponents.add(name);
          }
        }
      });
      
      let importStatements = '';
      usedComponents.forEach(component => {
        importStatements += `import ${component} from '@/components/${component}';\n`;
      });
      
      return {
        code: importStatements + code,
        map: null
      };
    }
  };
}
```

#### 环境变量注入插件
```javascript
export default function envPlugin(envVars) {
  return {
    name: 'env-plugin',
    
    config(config) {
      config.define = config.define || {};
      Object.keys(envVars).forEach(key => {
        config.define[`process.env.${key}`] = JSON.stringify(envVars[key]);
      });
    }
  };
}
```

#### 虚拟模块插件
```javascript
export default function virtualModulePlugin() {
  const virtualModuleId = 'virtual:my-module';
  const resolvedVirtualModuleId = '\0' + virtualModuleId;
  
  return {
    name: 'virtual-module',
    
    resolveId(id) {
      if (id === virtualModuleId) {
        return resolvedVirtualModuleId;
      }
    },
    
    load(id) {
      if (id === resolvedVirtualModuleId) {
        return `export const msg = "来自虚拟模块"`;
      }
    }
  };
}

// 使用
import { msg } from 'virtual:my-module';
console.log(msg);
```

## 性能优化

### 依赖预构建
```javascript
export default defineConfig({
  optimizeDeps: {
    // 强制预构建
    include: ['lodash-es', 'axios'],
    
    // 排除预构建
    exclude: ['your-local-package'],
    
    // 自定义esbuild选项
    esbuildOptions: {
      target: 'es2020'
    }
  }
});
```

### 代码分割
```javascript
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // 将React相关库打包到一起
          'react-vendor': ['react', 'react-dom'],
          
          // 将工具库打包到一起
          'utils': ['lodash-es', 'axios'],
          
          // 按路由分割
          'pages-home': ['./src/pages/Home'],
          'pages-about': ['./src/pages/About']
        }
      }
    }
  }
});

// 或使用函数
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          if (id.includes('node_modules')) {
            return 'vendor';
          }
          if (id.includes('src/components')) {
            return 'components';
          }
        }
      }
    }
  }
});
```

### 资源内联
```javascript
export default defineConfig({
  build: {
    assetsInlineLimit: 4096, // 小于4KB的资源内联为base64
    
    rollupOptions: {
      output: {
        // 自定义资源处理
        assetFileNames: (assetInfo) => {
          if (assetInfo.name.endsWith('.svg')) {
            return 'icons/[name]-[hash][extname]';
          }
          return 'assets/[name]-[hash][extname]';
        }
      }
    }
  }
});
```

### 构建缓存
```javascript
export default defineConfig({
  cacheDir: 'node_modules/.vite',
  
  build: {
    // 生成manifest.json
    manifest: true,
    
    // 启用CSS代码分割
    cssCodeSplit: true
  }
});
```

## 生产构建

### 构建优化
```javascript
export default defineConfig({
  build: {
    // 目标浏览器
    target: 'es2015',
    
    // 压缩
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
        pure_funcs: ['console.log']
      }
    },
    
    // 代码分割阈值
    chunkSizeWarningLimit: 500,
    
    // CSS代码分割
    cssCodeSplit: true,
    
    // 生成sourcemap
    sourcemap: false,
    
    // Rollup配置
    rollupOptions: {
      output: {
        // 分包策略
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['lodash-es']
        }
      }
    }
  }
});
```

### 多页面应用
```javascript
import { resolve } from 'path';

export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        main: resolve(__dirname, 'index.html'),
        admin: resolve(__dirname, 'admin/index.html')
      }
    }
  }
});
```

### 库模式
```javascript
export default defineConfig({
  build: {
    lib: {
      entry: resolve(__dirname, 'src/index.js'),
      name: 'MyLib',
      fileName: (format) => `my-lib.${format}.js`,
      formats: ['es', 'cjs', 'umd']
    },
    rollupOptions: {
      // 确保外部化处理那些你不想打包进库的依赖
      external: ['react', 'react-dom'],
      output: {
        globals: {
          react: 'React',
          'react-dom': 'ReactDOM'
        }
      }
    }
  }
});
```

## 实战案例

### React项目配置
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [
    react({
      // 启用Fast Refresh
      fastRefresh: true,
      
      // Babel配置
      babel: {
        plugins: [
          ['@babel/plugin-proposal-decorators', { legacy: true }]
        ]
      }
    })
  ],
  
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },
  
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true
      }
    }
  },
  
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'ui-vendor': ['antd', '@ant-design/icons']
        }
      }
    }
  }
});
```

### Vue3项目配置
```javascript
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import vueJsx from '@vitejs/plugin-vue-jsx';
import AutoImport from 'unplugin-auto-import/vite';
import Components from 'unplugin-vue-components/vite';
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers';

export default defineConfig({
  plugins: [
    vue(),
    vueJsx(),
    
    // 自动导入API
    AutoImport({
      imports: ['vue', 'vue-router', 'pinia'],
      resolvers: [ElementPlusResolver()]
    }),
    
    // 自动导入组件
    Components({
      resolvers: [ElementPlusResolver()]
    })
  ],
  
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  }
});
```

### SSR配置
```javascript
// vite.config.js
export default defineConfig({
  ssr: {
    noExternal: ['vue', 'vue-router']
  }
});

// server.js
import express from 'express';
import { createServer as createViteServer } from 'vite';

async function createServer() {
  const app = express();
  
  const vite = await createViteServer({
    server: { middlewareMode: true },
    appType: 'custom'
  });
  
  app.use(vite.middlewares);
  
  app.use('*', async (req, res) => {
    const url = req.originalUrl;
    
    try {
      const template = await vite.transformIndexHtml(url, 
        fs.readFileSync('index.html', 'utf-8')
      );
      
      const { render } = await vite.ssrLoadModule('/src/entry-server.js');
      const appHtml = await render(url);
      
      const html = template.replace('<!--ssr-outlet-->', appHtml);
      
      res.status(200).set({ 'Content-Type': 'text/html' }).end(html);
    } catch (e) {
      vite.ssrFixStacktrace(e);
      res.status(500).end(e.stack);
    }
  });
  
  app.listen(3000);
}

createServer();
```

## 最佳实践

1. **使用环境变量管理配置**
2. **合理配置依赖预构建**
3. **使用路径别名简化导入**
4. **开发环境使用HMR提升效率**
5. **生产环境开启代码分割和压缩**
6. **使用插件扩展功能**
7. **定期更新Vite版本**

## 常见问题

### Q1: Vite为什么这么快？
- 开发环境使用原生ESM，无需打包
- 使用esbuild预构建依赖
- 按需编译，只编译当前页面

### Q2: 如何处理CommonJS模块？
- Vite会自动将CommonJS转换为ESM
- 可以在optimizeDeps.include中配置

### Q3: 如何迁移Webpack项目到Vite？
- 修改入口HTML文件
- 调整环境变量前缀
- 替换Webpack特有API
- 调整构建配置

---

**@author erik.zhou**
