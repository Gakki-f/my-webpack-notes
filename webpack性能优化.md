# webpack性能优化
## 1.HMR
```
hot module replacement
在devserve：hot：true 即可

样式文件：可以用HMR style-loader 已经实现
js文件：默认无HMR ->修改js代码添加支持HMR
html：{
	在entry：[添加html]  整体全部刷新因为它是入口所以不做hmr
	默认无HMR 同时导致html无法热更新（不做HMR功能）
}
```

## 2.source map
```
一种提供源代码到构建后代码 映射技术
如果构建后代码出错了，可以追踪到源码

[inline-|hidden-|eval-] [nosources-] [cheap-[module-]]source-map

source-map: 外部
	错误代码准确信息 和 源代码的错误位置
inline-source-map：内联 只生成一个内敛source-map
错误代码准确信息 和 源代码的错误位置
hidden-source-map：外部
	不能追踪源代码错误，只能提示到构建后代码错误位置
eval-source-map：内联 每个文件都生成对应的source-map 
	错误代码准确信息 和 源代码的错误位置
nosource-source-map：外部
	错误嗲吗准确信息，但是没有任何源代码信息
cheap-source-map：外部
	只能精确到行
cheap-module-source-map：外部
	错误代码准确信息 和 源代码的错误位置

内联和外部的却比
1.外部生成了文件，内联没有
2.内联构建速度更快

速度快 eval>inline>cheap...
	eval-cheap-source-map
	eval-source-map
调试更友好
	source-map
	cheap-module-source-map
	cheap-source-map
``` 
## 3.oneOf
```
rules:[
	{
		一下loader只会匹配一个
		注意：不能有两个配置处理同一种类型文件（只会生效一个，提取出去）
		oneOf: [

		]
	}
]
```
##	4.缓存
```
babel缓存：构建速度，js代码多

hash:每次打包都给个唯一hash 打包都会改变不会文件变不变
chunkhash： 来自同一个chunk（入口）共享同一个hash ，js影响css
contenthash： 每个文件加上hash 这个最好
```
##	5.tree shaking
```
去除没有用到的代码
使用条件两个{
	1.开启es6模块，
	2.开启production模式 生产模式
}
可以在package.json中配置
不需要tree shaking 的文件
"sideEffects": ['*.css','*.less']
``` 
##	6.code split
```
代码分割
```
## 7.懒加载预加载
```
import引入js
异步
```
## 8.PWA
```
渐进式网络开发应用程序（离线可访问技术）
workbox --> wordbox-webpack-plugin
important WordBoxWebpackPlugin from ‘wordbox-webpack-plugin’
new WordBoxWebpackPlugin.generateSW(

		clientsClaim: true,
		skipWaiting: true
)
入口文件 注册serviceWordk 需要判断是否支持
```
## 9.thread
```
多进程打包
thread-loader
进程启动大改是600ms，进程通信也有开销时间
只有工作消耗时间比较长，才需要多进程打包比如bable（js代码多）
```


##	10.external
```
忽略打包的东西
比如jquery 用cdn引入 npm包的就忽略
和mode同级别
externals: {
	忽略库名：npm包名
	jquery: 'jQuery'
}
```
##	11.DLL
```js
对代码单独打包
使用dll技术对某些库（第三方库：jquery，vue，react。。。）进行单独打包
webpack.dll.js
module-exports = {
	entry: {
		//最终打包生辰的[name] --> jquery
		['jquery] --> 要打包的库是jquery
		jquery: ['jquery]
	},
	output: {
		filename: '[name].js',
		path: resolve(__dirname, 'dll'),
		library: '[name]_[hash]', // 打包的库里面向外暴露的内容名
	},
	plugin: [
	new webpack.DllPlugin({
		name: '[name]_[hash]', // 映射库的暴露的名字
		path: resolve(__dirname, 'dll/manifest.json') // 输出文件路径
	})
}
//在webpack.config.js中
new webpack.DllreferencePlugin({
	//找到单独打包的dll里面依赖，不打包它
	manifest: resolve(__dirname, 'dll/manifest.json')
})
//add-asset-html-webpack-plugin
//将某个文件打包输出，并且在html中自动引入资源
new AddAssetHtmlWebpackPlugin({
	//将dll单独打包的依赖引入到html中去
	filepath: resolve(__dirname, 'dll/jquery.js') // 
})
```
# webpack 性能优化总结

## 开发环境性能优化

* 优化打包构建速度  
  
	1. HMR 一个模块变化只更新这个模块 热更新块开发时候调试舒服 hot module replacement

* 优化代码调试  
  
	1. source-map 很多种，各种组合，调试时候映射源码和构建后代码

## 生产环境性能优化

* 优化打包构建速度 （提升开发者体验）

	1. oneOf

	2. 缓存: { babel缓存：构建速度，js代码多 } 

	3. 多进程打包

	4. externals（不打包某个依赖，通过cdn引入）

	5. dll（先把库单独打包，后面不用打包直接用，防止重复打包）{至少要写两个配置文件}

* 优化代码运行的性能
	1. 缓存

	2. tree shaking

	3. code split

	4. 懒加载预加载

	5. pwa （离线可访问）

