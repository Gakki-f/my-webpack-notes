# webpack配置

```js
const {resolve} = require('path')
const HtmlWebpackPlugin = require('html-webpack-paugin')
module.exports = {
  mode: 'development',
  extry: './src/index.js',
  output: {
    filename: 'built.js',
    path: resolve(__dirname, 'build')
  },
  plugins: [
    new HtmlWebpackPlugin()
  ],
  resolve: {},
  optimization: {
    splitChunks: {
      chunks: 'all'
    },
  }
}
```
## 1.entry

> 入口起点

* 1.string: './src/index.js',  
打包形成一个chunk。输出一个bundle文件。  
此时的chunk的名称默认是main

* 2.array: ['./src/index.js', './src/other.js']  
多入口  
所有入口文件最终只会形成一个chunk，输出出去只有一个bundle文件  
-->只有在HMR功能中让html热更新生效~（作用）
* 3.object: {  
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;index: './src/index/js',  
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;other: './src/other.js'  
   &nbsp;&nbsp;&nbsp;}  
多入口  
有几个入口文件就形成几个chunk，输出几个bundle文件  
此时chunk的名称就是key (index, other)
* 4.特殊用法
```js
{
  
  //所有的入口文件最终只会形成一个chunk，输出出去只有一个bundle文件
  index: ['./src/index.js', './src/count.js']
  //形成一个chunk，输出一个bundle文件
  other: './src/other.js'
}
```
>第一种和第三种用的多

## 2.output
```js
module.exports = {
  output: {
    // 文件名称（指定名称——目录）
    filename: 'js/[name].js',
    // 输出文件目录（将来所有资源输出的公共目录）
    path: resolve(___dirname, 'build'),
    // 所有资源引入的公共路径的前缀
    // 'img/a.jpg' --> '/img/a.jpg'
    publicPath: '/', // 一般用于生产环境
    chunkFilename: 'js/[name]_chunk.js', // 非入口chunk名
    // library: '[name]', 整个库向外暴露的变量名
    // libraryTarget: 'window', 变量名添加到哪个上 browser
    // libraryTarget: 'global', 变量名添加到哪个上 node
    // libraryTarget: 'commonjs',
  }
}
```
## 3.module

```js
module.exports = {
  module: {
    rules: [
      // loader的配置
      {
        test: /\.css$/,
        // 多个loader用use
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.js$/,
        // 单个loader用
        loader: 'eslint-loader',
        // 排除node_modules下的js文件
        exclude: /node_module/,
        //只检查src下的js文件
        include:  resolve(__dirname, 'src'),
        // 优先执行
        enforce: 'pre',
        // 延后执行
        // enforce: 'post',
        options: {}
      },
      {
        // 以下配置只会生效一个
        oneOf: []
      }
    ]
  }
}

```
## 4.resolve
```js
module.exports = {
  resolve: {
    // 路径别名
    alias: {
      @: resolve(__dianame, 'src')
    },
    // 配置省略路径的后缀名
    extensions: ['js', 'json'],
    // 告诉webpack解析模块去找哪个目录
    modules: ['node_modules'], // 一层一层找 可以配置名字 [resolve(__dirname, 'node_module')]
  }
}
```
## 5.devServer (开发环境)
```js
module.exports = {
  devServer: {
    // 运行代码的目录
    contentBase: resolve(__dirname, 'build'),
    // 监视 contentBase目录下的所有文件，一旦文件变化就会reload
    watchContentBase: true,
    watchOptions: {
      // 忽略文件
      ignored: /node_modules/
    },
    // 启动gzip压缩
    compress: true,
    port: 5000,
    host: 'localhost',
    // 自动打开浏览器
    open: true,
    // 开启HMR功能
    hot: true,
    // 不显示启动服务器日志信息
    clientLogLevel: 'none',
    // 除了一些基本启动信息以外，其他内容都不要显示
    quiet: true,
    // 如果出错了，不要全屏提示
    overlay: false,
    // 服务器代理 --> 开发环境跨域问题
    proxy: {
      // 一旦devServe(5000)服务器接收到 /api/xx的请求
      // 就会吧请求转发到另外一个服务器（3000）
      '/api': {
        target: 'http://locaohost:3000',
        // 发送请求时，路径重写，将 /api/xx --> /xx (去掉/api)
        pathRewrite: {
          '^/api': ''
        },
      }
    }
  }
}
```
## 5.optimization
>optimization 必须在生产环境有意义
```js
module.exports = {
  optimization: {
    // 代码分割
    splitChunks: {
      chunks: 'all',
      // 以下都是默认值，可以不用写
      minSize: 30 * 1024, // 分割的chunk最小为30kb
      mixSize: 0, // 分割的chunk最大无限制
      minChunks: 1, // 要提取的chunks最少被引用1次
      maxAsyncRequests: 5, // 按需加载时并行加载文件的最大数量为5
      maxInitalRequests: 3, // 入口js文件最大并行请求数量为3
      automaticNameDelimiter: '~', // 名称连接符
      name: true, // 可以用命名规则
      cacheGroups: { // 可分割chunk的组
      // node_modules文件会被打包到vendors组的chunk中 vendors~xxx.js
      // 满足上面的公共规则，如大小超过30kb至少引用一次
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          // 打包的优先级
          priority: -10
        },
        default: {
          // 要提取的chunk最少被引用2次
          minChunks: 2,
          // 优先级
          priority: -20,
          // 如果当前要打包的模块和之前已经被提取的模块是同一个,
          // 就会复用而不是重新打包模块 (jquery)，
          reuseExistingChunk: true
        }

      }
    },
  }
}
```