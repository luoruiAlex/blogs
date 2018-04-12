# 预备知识
## 1. Node.js
- Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。Node.js诞生后，能使JS代码脱离浏览器，使得JS能像Java、Python一样具有完整的IO操作功能，从此后我们可以用JS开发服务器程序(比如Express)、开发本地桌面应用(Electron，比如微软的Visual Studio Code就是用Electron开发的)，甚至开发移动app(React Native)。
- Node.js中的JS与浏览器中的JS大部分一致，主要差别在于浏览器中有操作网页DOM的API，而Node.js中有IO操作的API。
- Node.js怎样执行JS脚本？ **`node xx.js`**
## 2. npm与package.json
- npm是Node.js 的包管理器，相当于Java中的Maven、Python中的pip
- 怎样管理依赖的包？ 本地缓存 <---> 私有库 <---> 远程仓库，如果不搭建私有库则直接用远程仓库，国外一般用官方地址[https://www.npmjs.com/](https://www.npmjs.com/)，国内一般都用淘宝镜像。
- 常用npm命令
  - `npm init` 在当前目录下生成package.json(相当于将当前目录当做JS工程根目录)。在命令执行后，会有一连串的交互式问题，让你输入工程的配置参数。如果想生成默认的package.json，可执行`npm init -y`
  - `npm install`
    - `npm install`表示安装package.json中dependencies和devDependencies下所有的模块
    - `npm install moduleName`表示安装某个模块，默认为**本地模式**(安装到应用程序的本地node_modules目录下，只能本项目使用)。
    - `npm install moduleName -g`表示使用**全局模式**安装，安装到Node安装目录下的node_modules下。
    - Maven的pom.xml中有`<dependencies>`，当多人共同协作时，只需要同步pom.xml文件就能保证所有人使用了相同的依赖jar包。`npm install moduleName --save`表示在安装到node_modules下的同时，在package.json的`dependencies`下增加一行记录，功能与pom.xml中的`<dependencies>`类似。同样的，`npm install moduleName --save-dev`表示在`devDependencies`下增加一行记录，这里的模块都是开发前端项目需要的开发工具，比如babel可以将ES6代码转换成ES5，使得我们用ES6开发的JS代码可以在不支持ES6的旧浏览器上运行。
  - 其他常用命令 `npm uninstall` `npm update` `npm publish`
  - npm script
    - 什么是npm script? npm 允许在package.json文件里面，使用scripts字段定义脚本命令。比如我们在scripts中加入一行`"dev": "node build/dev-server.js"`, 当我们在命令行下执行`npm run dev`就表示执行了`node build/dev-server.js`。
    - 为什么要多此一举使用npm script？ npm script可以使用通配符、传入参数、使用npm的内部变量，还可以使用`pre` `post`两个钩子进行准备和清理工作。
    - 某些script有简写: `npm start`相当于`npm run start`，`npm test`相当于`npm run test`
## 3. webpack
- 什么webpack? 一个前端资源加载/打包工具.
- webpack有什么用? 如果说npm和package.json为我们解决了JS的模块的依赖的功能，那么webpack就为我们解决了构建打包的问题，npm + package.json + webpack才能与Java中Maven的地位相当。此外，webpack通过插件和loader，能提供HTTP服务和模拟后台接口(以后只有在和后台联调的时候需要启动tomcat了，这个叫做前后端分离)、集成babel功能(将ES2017、ES6代码转换成ES5、将TypeScript、JSX转化成JS)，有HMR(热模块替换，是否厌倦了传统前端开发时每次修改代码都需要刷新页面然后进行恢复操作的繁琐，HMR能让你在修改源码的时候，直接更新浏览器页面中对应的节点而且不用进行恢复操作)，能将前端用到的图片编译进入JS源码(减少http请求的次数，防止进入页面的时候图片处为空白)。总而言之，webpack在手，前端我有。
- webpack也有配置文件，默认为webpack.config.js，一般在开发的时候由架构师写好配置文件，这里不详述。
## 4. 总结
### 有了Node.js npm package.json webpack之后，前端开发的效率大大提升，步骤如下：
1. 架构师通过npm init创建工程，并配置好package.json和webpack.config.js，上传到版本库
2. 开发人员安装Node.js后下载工程代码，进入项目根目录，执行`npm install`下载依赖的包
3. 执行`npm run dev`启动项目，开始编译，如果webpack中配置了html-webpack-plugin会自动在浏览器中打开页面
4. 修改源码，自动重新编译，编译完成后自动在浏览器中更新(HMR)
5. 如果有需要，执行`npm run test`进行JS的单元测试，执行`npm run lint`检测JS代码是否符合代码规范
