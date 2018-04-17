# webpack.config.js
## 1. entry
- 单入口： String | Array | Object
- 多入口： Object
- 多个彼此互不依赖的入口: Array
## 2. output
- path: 必须是绝对路径，是所有文件的输入路径，可以用`path.join()`
- publicPath: url相对于HTML页面，用于HTML引入资源的路径补全
- name：可包含[name][hash][chunkhash]，chunkhash不能与HMR一起用
## loader
- 加载文件ing转换
- ```
  module {
    rules: [
      {test, use, exclude} //loader
    ]
  }
  ```
- loader自身可配置，比如`url-loader?limit=1024`
- use的值可为字符串，比如'url-loader'，也可表达式，比如`ExtractTextPlugin.Extract({use: 'css-loader',fallback: 'style-loader'})`
## resolve
- 配置模块解析的规则，比如指定目录，用以提升效率
## target
- 默认为web，服务器渲染可将target设置为node
## devtool
- 开发：source-map
- 生产： cheap-module-source-map
## plugins
- 解决loader无法实现的其他事
- webpack.optimize.CommonsChunkPlugin
  - 多个入口的相同依赖定义成一个新入口
  - name/names entry中已定义则直接提取
  - filename: 提取出的文件名，默认为name
  - chunks： 通过name来提取chunk，默认提取所有
  - minChunks
- HtmlWebpackPlugin
  - 生成HTML文件
  - title: HTML文件的title
  - filename: 默认index.html
  - favicon: 图标
  - template: 可以为多种格式的模板，单必须安装对应的loader
  - inject： 默认`true`，表示<script>在<body>底部，`body`与`true`相同，`false`表示不生成<script>，`head`表示<script>在<head>内
  - minify: 默认false，表示不压缩生成的html
- webpack.optimize.UglifyPlugin()
  - warning: false 去掉删除无用代码时的警告
- ExtractTextWebpackPlugin
  - filename: [name].css [id] [contenthash]
  - allChunks: 配合CommonChunksPlugin使用
- webpack.DefinePlugin
  - 创建在编译时刻配置的全局变量
  - `'process.env': JSON.stringify({NODE_ENV: 'production'})`
- ProviderPlugin
  - HACK不规范的模块，比如$:'jquery'
- OpenBrowserWebpackPlugin({url:xx})
  - url默认为`http://localhost:8080`
- CleanWebpackPlugin(paths, {options})
  - paths: ['dist', 'build']
  - options: root: __dirname; exclude: []; verbose: boolean; dry: false---true表示真的删除
- OccurenceOrderPlugin
  - webpack为每个模块指定唯一的id，改插件可是webpack为最常用的模块分配最小的id
- CopyWebpackPlugin
  - from to
  - force ignore
  - context
  - flatten 忽略文件夹

# 打包原理
- 所有资源统一入口：入口JS直接或间接引入其他文件资源
- 文件管理： 每个入口文件打包所有依赖的资源，每个资源打包一次；多个入口文件互不相干
