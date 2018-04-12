# 1. 预备知识
## 1.1 Node.js
- Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。Node.js诞生后，能使JS代码脱离浏览器，使得JS能像Java、Python一样具有完整的IO操作功能，从此后我们可以用JS开发服务器程序(比如Express)、开发本地桌面应用(Electron，比如微软的Visual Studio Code就是用Electron开发的)，甚至开发移动app(React Native)。
- Node.js中的JS与浏览器中的JS大部分一致，主要差别在于浏览器中有操作网页DOM的API，而Node.js中有IO操作的API。
- Node.js怎样执行JS脚本？ **`node xx.js`**
## 1.2 npm与package.json
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
