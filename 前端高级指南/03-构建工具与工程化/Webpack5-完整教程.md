# Webpack5 - 完整教程

## 目录
1. [Webpack核心概念](#webpack核心概念)
2. [基础配置](#基础配置)
3. [Loader详解](#loader详解)
4. [Plugin详解](#plugin详解)
5. [代码分割](#代码分割)
6. [性能优化](#性能优化)
7. [模块联邦](#模块联邦)

## Webpack核心概念

### 什么是Webpack
Webpack是一个现代JavaScript应用程序的静态模块打包工具。

### 核心概念
- **Entry（入口）**：构建的起点
- **Output（输出）**：打包后的文件输出位置
- **Loader（加载器）**：转换非JavaScript文件
- **Plugin（插件）**：执行更广泛的任务
- **Mode（模式）**：development、production、none

### 安装Webpack
```bash
npm install --save-dev webpack webpack-cli webpack-dev-server
```

## 基础配置

### 最小配置
```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  mode: 'development',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
    clean: true // Webpack5新特性：自动清理输出目录
  }
};
```

### 多入口配置
```javascript
module.exports = {
  entry: {
    app: './src/app.js',
    admin: './src/admin.js'
  },
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

### 开发服务器配置
```javascript
module.exports = {
  devServer: {
    static: './dist',
    hot: true,
    port: 3000,
    open: true,
    compress: true,
    historyApiFallback: true, // SPA路由支持
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        pathRewrite: { '^/api': '' }
      }
    }
  }
};
```

## Loader详解

### 处理CSS
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.scss$/,
        use: [
          'style-loader',
          'css-loader',
          'sass-loader'
        ]
      },
      {
        test: /\.less$/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: true // CSS Modules
            }
          },
          'less-loader'
        ]
      }
    ]
  }
};
```

### 处理图片和字体
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|svg|jpg|jpeg|gif)$/i,
        type: 'asset/resource', // Webpack5内置资源模块
        generator: {
          filename: 'images/[name].[hash][ext]'
        }
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/i,
        type: 'asset/resource',
        generator: {
          filename: 'fonts/[name].[hash][ext]'
        }
      },
      {
        test: /\.svg$/,
        type: 'asset/inline' // 转为base64
      }
    ]
  }
};
```

### 处理JavaScript
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              ['@babel/preset-env', {
                useBuiltIns: 'usage',
                corejs: 3
              }]
            ],
            plugins: ['@babel/plugin-transform-runtime']
          }
        }
      }
    ]
  }
};
```

### 处理TypeScript
```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/
      }
    ]
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js']
  }
};
```

### 自定义Loader
```javascript
// my-loader.js
module.exports = function(source) {
  // source是文件内容
  const result = source.replace(/console\.log\(/g, 'console.info(');
  return result;
};

// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          {
            loader: path.resolve(__dirname, 'loaders/my-loader.js')
          }
        ]
      }
    ]
  }
};
```

## Plugin详解

### 常用插件配置
```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const webpack = require('webpack');

module.exports = {
  plugins: [
    // 生成HTML文件
    new HtmlWebpackPlugin({
      template: './src/index.html',
      filename: 'index.html',
      chunks: ['app'], // 指定引入的chunk
      minify: {
        removeComments: true,
        collapseWhitespace: true
      }
    }),
    
    // 提取CSS到单独文件
    new MiniCssExtractPlugin({
      filename: 'css/[name].[contenthash].css'
    }),
    
    // 清理输出目录
    new CleanWebpackPlugin(),
    
    // 复制静态资源
    new CopyWebpackPlugin({
      patterns: [
        { from: 'public', to: 'public' }
      ]
    }),
    
    // 定义环境变量
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
      'API_URL': JSON.stringify('https://api.example.com')
    }),
    
    // 模块热替换
    new webpack.HotModuleReplacementPlugin()
  ]
};
```

### 自定义Plugin
```javascript
// my-plugin.js
class MyPlugin {
  apply(compiler) {
    // 在编译开始时执行
    compiler.hooks.compile.tap('MyPlugin', () => {
      console.log('开始编译...');
    });
    
    // 在生成资源到output目录之前
    compiler.hooks.emit.tapAsync('MyPlugin', (compilation, callback) => {
      // compilation包含所有模块和资源信息
      console.log('生成的文件：', Object.keys(compilation.assets));
      
      // 添加自定义文件
      compilation.assets['custom.txt'] = {
        source: () => 'Hello Webpack',
        size: () => 13
      };
      
      callback();
    });
    
    // 编译完成
    compiler.hooks.done.tap('MyPlugin', (stats) => {
      console.log('编译完成！');
    });
  }
}

module.exports = MyPlugin;

// webpack.config.js
const MyPlugin = require('./plugins/my-plugin');

module.exports = {
  plugins: [
    new MyPlugin()
  ]
};
```

## 代码分割

### 入口分割
```javascript
module.exports = {
  entry: {
    app: './src/app.js',
    vendor: './src/vendor.js'
  }
};
```

### 动态导入
```javascript
// 使用import()动态导入
button.addEventListener('click', () => {
  import(/* webpackChunkName: "lodash" */ 'lodash')
    .then(({ default: _ }) => {
      console.log(_.join(['Hello', 'webpack'], ' '));
    });
});
```

### SplitChunksPlugin配置
```javascript
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all', // 对所有chunk进行分割
      cacheGroups: {
        // 提取node_modules中的代码
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10
        },
        // 提取公共代码
        common: {
          minChunks: 2,
          name: 'common',
          priority: 5,
          reuseExistingChunk: true
        },
        // 提取CSS
        styles: {
          name: 'styles',
          test: /\.css$/,
          chunks: 'all',
          enforce: true
        }
      }
    },
    // 运行时代码单独提取
    runtimeChunk: {
      name: 'runtime'
    }
  }
};
```

### 预获取和预加载
```javascript
// 预获取（prefetch）：浏览器空闲时加载
import(/* webpackPrefetch: true */ './utils.js');

// 预加载（preload）：与父chunk并行加载
import(/* webpackPreload: true */ './critical.js');
```

## 性能优化

### 构建速度优化
```javascript
module.exports = {
  // 1. 缩小文件搜索范围
  resolve: {
    modules: [path.resolve(__dirname, 'node_modules')],
    extensions: ['.js', '.jsx', '.json'],
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },
  
  // 2. 使用缓存
  cache: {
    type: 'filesystem', // Webpack5持久化缓存
    cacheDirectory: path.resolve(__dirname, '.webpack_cache')
  },
  
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          // 3. 使用thread-loader多进程构建
          'thread-loader',
          {
            loader: 'babel-loader',
            options: {
              cacheDirectory: true // Babel缓存
            }
          }
        ],
        // 4. 精确匹配，减少搜索
        include: path.resolve(__dirname, 'src'),
        exclude: /node_modules/
      }
    ]
  },
  
  // 5. 优化resolve
  optimization: {
    moduleIds: 'deterministic' // 稳定的模块ID
  }
};
```

### 打包体积优化
```javascript
const TerserPlugin = require('terser-webpack-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const CompressionPlugin = require('compression-webpack-plugin');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');

module.exports = {
  optimization: {
    minimize: true,
    minimizer: [
      // 压缩JavaScript
      new TerserPlugin({
        parallel: true,
        terserOptions: {
          compress: {
            drop_console: true, // 删除console
            drop_debugger: true
          }
        }
      }),
      // 压缩CSS
      new CssMinimizerPlugin()
    ],
    // Tree Shaking
    usedExports: true,
    sideEffects: true
  },
  
  plugins: [
    // Gzip压缩
    new CompressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
      threshold: 10240, // 只压缩大于10KB的文件
      minRatio: 0.8
    }),
    
    // 打包分析
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false
    })
  ],
  
  // 外部化依赖
  externals: {
    react: 'React',
    'react-dom': 'ReactDOM'
  }
};
```

### Source Map优化
```javascript
module.exports = {
  // 开发环境
  devtool: 'eval-cheap-module-source-map',
  
  // 生产环境
  // devtool: 'hidden-source-map' // 或 'nosources-source-map'
};
```

## 模块联邦

### 基本配置
```javascript
// app1/webpack.config.js（提供方）
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'app1',
      filename: 'remoteEntry.js',
      exposes: {
        './Button': './src/components/Button',
        './utils': './src/utils'
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true }
      }
    })
  ]
};

// app2/webpack.config.js（消费方）
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'app2',
      remotes: {
        app1: 'app1@http://localhost:3001/remoteEntry.js'
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true }
      }
    })
  ]
};

// app2中使用
import Button from 'app1/Button';

function App() {
  return <Button>点击我</Button>;
}
```

### 动态远程容器
```javascript
// 动态加载远程模块
const loadComponent = (scope, module) => {
  return async () => {
    await __webpack_init_sharing__('default');
    const container = window[scope];
    await container.init(__webpack_share_scopes__.default);
    const factory = await container.get(module);
    return factory();
  };
};

// 使用
const Button = React.lazy(loadComponent('app1', './Button'));
```

## 完整配置示例

### 开发环境配置
```javascript
// webpack.dev.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const webpack = require('webpack');

module.exports = {
  mode: 'development',
  entry: './src/index.js',
  output: {
    filename: '[name].js',
    path: path.resolve(__dirname, 'dist')
  },
  devtool: 'eval-cheap-module-source-map',
  devServer: {
    static: './dist',
    hot: true,
    port: 3000
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'babel-loader'
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'
    }),
    new webpack.HotModuleReplacementPlugin()
  ]
};
```

### 生产环境配置
```javascript
// webpack.prod.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');

module.exports = {
  mode: 'production',
  entry: './src/index.js',
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist'),
    clean: true
  },
  devtool: 'source-map',
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            cacheDirectory: true
          }
        }
      },
      {
        test: /\.css$/,
        use: [
          MiniCssExtractPlugin.loader,
          'css-loader',
          'postcss-loader'
        ]
      }
    ]
  },
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin(),
      new CssMinimizerPlugin()
    ],
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10
        }
      }
    },
    runtimeChunk: 'single'
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
      minify: {
        removeComments: true,
        collapseWhitespace: true
      }
    }),
    new MiniCssExtractPlugin({
      filename: 'css/[name].[contenthash].css'
    })
  ]
};
```

## 最佳实践

1. **使用Webpack5持久化缓存提升构建速度**
2. **合理配置代码分割，减小首屏加载体积**
3. **开发环境使用eval-cheap-module-source-map**
4. **生产环境开启Tree Shaking和代码压缩**
5. **使用模块联邦实现微前端架构**
6. **定期使用Bundle Analyzer分析打包体积**
7. **合理使用externals外部化大型依赖**

## 常见问题

### Q1: 如何解决Webpack构建慢？
- 使用持久化缓存
- 使用thread-loader多进程构建
- 缩小文件搜索范围
- 使用DllPlugin预编译依赖

### Q2: 如何优化打包体积？
- 开启Tree Shaking
- 使用代码分割
- 压缩代码和资源
- 使用CDN外部化依赖

### Q3: Source Map如何选择？
- 开发：eval-cheap-module-source-map
- 生产：hidden-source-map或不生成

---

**@author erik.zhou**
