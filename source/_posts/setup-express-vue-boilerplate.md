title: 从零开始搭建 Express + Vue 开发环境
date: 2016-02-28 14:02:16
tags: express, vue, webpack, node
---

完整的代码已经上传 Github，[点击访问](https://github.com/southerncross/vue-express-dev-boilerplate)

## 准备工作

### 1. 为前端选择合适的预处理工具和资源管理工具

预处理工具又分为 js 预处理工具和 css 预处理工具。Javascript 一直以来最为人诟病的一点就是缺乏原生的模块机制，所有 js 代码文件在被 html 页面引入后将共用同一个命名空间。所以才出现了各种“标准”尝试解决这个问题，但他们都不是原生的，需要额外的工具对代码进行特殊处理。虽然 ES6 终于引入了模块机制，但以现在的浏览器支持程度，还不足以“毫无顾虑地随拿随用”。所以 js 预处理工具最主要的工作就是帮助解决 js 的模块问题。而 css 预处理工具则很好理解，就是把 sass，less 或者 stylus 代码翻译、合并成 css 代码。

资源管理工具则是帮我们管理前端所需的各种资源文件（比如 css、js、图片、字体等等），便于我们引用。目前常用的解决方法是将他们直接编码进 js 代码中，然后像引用 js 模块一样引用它们。这可比手写各种 url 方便多了。

正如标题所说，我们将采用 Webpack，因为它具备上面所说的所有功能。此外还支持代码热替换，使修改代码后不用刷新页面也能在浏览器中立即看到效果。

![Webpack schedule](http://webpack.github.io/assets/what-is-webpack.png)

### 2. 为后端选择合适的预处理工具

后端面对的都是 js 代码，不需要前端那样的资源管理工具，另外， Node 强迫你使用至少一种模块管理方案（CommonJS 或 ES6 的 import），也不用考虑代码的依赖问题。

所以后端要简单许多，唯一需要考虑的基本上就剩下如何将 ES6 转译成 ES5 了（如果你打算使用 ES6 的话）。目前常用的做法是使用 Babel，你可以用 Babel 命令行工具独立执行编译过程，也可以配置 Babel register 实现代码运行时动态翻译，这对于开发场景而言无疑是最方便的。所以我们选择后一种方式。

![Babel logo](https://babeljs.io/images/logo.svg)

### 3. 为整个项目选择合适的流程控制工具

流程控制工具是为了帮助我们管理诸如代码检查、编译、压缩、移动、部署这些任务的，原本我们是通过手敲命令（或者高级一点写个脚本）的方式做，有了流程控制工具以后，只需要提供配置文件和少量代码就可以完成。

目前最流行的解决方案是 gulp。不过，由于我们这里要搭建的是开发环境，没有移动代码、压缩、部署等需求，所以不需要功能强大的 gulp。我们只要用 nodemon 这个工具监听代码变动然后适时重启 server就够了。

![Nodemon logo](https://camo.githubusercontent.com/fd1ea21338ceeef34920e44e97d099f3c47a78c3/687474703a2f2f6e6f64656d6f6e2e696f2f6e6f64656d6f6e2e737667)

## 正式开始搭建

### 1. 利用 Express 脚手架快速搭建应用

使用 Express 提供的脚手架工具（[Express application generator](http://expressjs.com/en/starter/generator.html)）可以在 1s 之内搭建出最基本的应用。

如果你以前还没试过，首先执行下面的命令安装

`npm install express-generator -g`

然后执行以下命令生成代码，命令执行过程中需要输入一些参数。

`express <myapp>`

完成后的文件结构是这样的

```
.
├── app.js
├── bin
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.jade
    ├── index.jade
    └── layout.jade
```

但是这个文件结构只是后端代码，要想跟前端代码相结合，需要做一些改动。我们计划最终的文件结构应该是这样的

```
.
├── src
│   ├── client
│   └── server
└── ...
```

所以需要将上面自动生成的 Express 代码放到 /src/server/ 路径下。

接下来我们要删除一些用不着的东西以及增加一些缺失的东西。

首先，由于我们打算在前端使用 Vue 框架，并由 Vue-Router 管理大部分路由，后端 Express 仅保留少量的 RESTful API 路由，所以后端不需要复杂的路由设置，那么 routes 文件夹下的内容可以简化成一个 routers.js 文件。

然后，我们需要配置好 Babel register，所以需要在项目跟路径下新增两个文件 `.babelrc` 和 `index.js`，内容分别为：

/.babelrc
```json
{
  "presets": ["es2015"],
}
```
.babelrc 是 Babel 6.0 必须的文件

/.index.js
```
require('babel-register')
require('./src/server')
```
上面两句完成 Bebel 注册，它会爬取所有 `require` 或 `import` 方式依赖的模块并把它们翻译成 ES5。

自动生成的代码里有个 www 文件，他是 Express 应用的入口文件，我们把它放在 server 路径下并改名为 index.js 以便让上面配置的 Babel register 能正确找到它。

> 为什么要改名为 index.js ？这是因为 `require('./src/server')` 在默认情况下会去找 ./src/server/index.js，如果你想用别的名字，那就记得将 Babel register 的配置文件里改为 `require(./src/<your entry file>`。

好了，现在后端的事情先暂时告一段落，接下来看看前端。

### 2. 利用 Vue 脚手架快速搭建应用

同样地，推荐使用 Vue 自带的 [template 工具](https://github.com/vuejs-templates/webpack)，在 1s 内生成基本代码。

虽然利用这个工具生成的代码的后端就是基于 Express 的，但是后端部分的代码结构太简单，不适合做后续开发。所以建议这里先将 Vue 生成的代码放在另外一个地方，然后按需移动到前面用 Express 生成的代码文件夹里。

首先安装 template 工具

`npm install -g vue-cli`

然后执行命令生成代码。命令执行过程中需要输入一些参数

`vue init webpack <my-project>`

生成的代码结构是这样的

```
.
├── build
│   ├── dev-server.js         # development server script
│   ├── karma.conf.js         # unit testing config
│   ├── webpack.base.conf.js  # shared base webpack config
│   ├── webpack.dev.conf.js   # development webpack config
│   ├── webpack.prod.conf.js  # production webpack config
│   └── ...
├── src
│   ├── main.js               # app entry file
│   ├── App.vue               # main app component
│   ├── components            # ui components
│   │   └── ...
│   └── assets                # module assets (processed by webpack)
│       └── ...
├── static                    # pure static assets (directly copied)
├── dist                      # built files ready for deploy
├── test
│   └── unit                  # unit tests
│       ├── index.js          # unit test entry file
│       └── ...
├── .babelrc                  # babel config
├── .eslintrc.js              # eslint config
├── index.html                # main html file
└── package.json              # build scripts and dependencies
```

我们发现这里也有一个 .babelrc，内容跟之前自己创建的基本一致，可以忽略它。此外，它还提供了 .eslintrc.js，是为了配合 eslint 检查代码是否符合规范的。这里面的内容很简单，想要偷懒就直接拿过来，觉得定制的规则不太符合自己的习惯的可以另外配置。

然后看到 build 路径下有 3 个 webpack 有关的配置文件，因为我们是要搭建开发环境，所以挑里面的 webpack.base.conf.js 和 webpack.dev.conf.js 就可以，建议把内容合并到一个 webpack.conf.js 文件里，放在项目的根目录下。

之后，基本上就是把 src 目录移动到之前用 Express 创建的 /src/client/，我习惯将所有 js 的入口文件都改为 index.js 所以，这里也可以将 src/client/main.js 改名为 src/client/index.js。

其他的文件先忽略不管。

到这里，前端部分的代码也基本整理完毕了。

### 3. 配置 Webpack

我们计划让 Webpack 将前端文件打包成一个 build.js 文件，然后放在 /src/server/public/javascripts 中供 jade 模板使用。所以设置好 webpack 的路径部分（其他的保留原来的就好）：

```javascript
{
  ...
  entry: path.join(__dirname, 'src/client/index.js'),
    output: {
      path: path.join(__dirname, 'src/server/public/javascripts/'),
      publicPath: '/javascripts/',
      filename: 'build.js'
    },
  ...
}
```

至于如何启动 Webpack，你可以选择单独用一个 shell 窗口运行它，也可以以 Express 中间件的形式提供代理。采用后一种方式，webpack 并不会把打包好的代码生成在磁盘上，而是保留在内存里。我们选择后一种方式，因为更方便。

> 只应该在开发环境中以 Express 中间件的形式部署 Webpack

所以需要修改 /src/server/index.js，关键是增加这几句

```javascript
import webpack from 'webpack'
import webpackDevMiddleware from 'webpack-dev-middleware'
import webpackHotMiddleware from 'webpack-hot-middleware'
import config from '../../webpack.config'

const compiler = webpack(config)

app.use(webpackDevMiddleware(compiler, {
  publicPath: config.output.publicPath,
  stats: { colors: true },
}))

app.use(webpackHotMiddleware(compiler))
```

这样每次启动 Express 后，Webpack 中间件会拦截 config.output.publicPath 地址的请求并返回正确的结果，同时，如果被 Webpack 监听的文件发生变动，会立即通知前端产生相应变化。

### 4. 配置 Nodemon

之前提到过，计划用 Nodemon 启动 server 并监听代码变动。而 Nodemon 默认会监听除了 .git 和 node_modules 路径外的所有 js 代码，因为我们已经有 Webpack 监听前端代码了，所以得做相关配置让 Nodemon 只监听某一块代码。

在项目根路径下新增文件 nodemon.json，内容为

```
{
  "verbose": true,
  "ignore": ["src/server/public/"],
  "events": {
    "restart": "osascript -e 'display notification \"App restarted due to:\n'$FILENAME'\" with title \"nodemon\"'"
  },
  "watch": ["src/server/"],
  "env": {
    "NODE_ENV": "development"
  },
  "ext": "js jade"
}
```

其中，将 verbose 设置为 true 将打印更丰富的日志信息，对开发很有帮助。

我们选择让 Nodemon 监听 src/server/ 目录，并忽略 src/server/public 目录，因为那里是前端 webpack 生成打包文件的地方。注意我们是以 Express 中间件的形式使用 Webpack，并不会在磁盘上真的产生文件，所以这个 ignore 规则其实可以省略。

别忘了在文件扩展名中增加 jade 类型，因为 Express 使用的是 jade 模板。

### 5. 配置 package.json

首先在 script 中增加一个命令，用来启动整个应用

```javascript
{
  ...
  "scripts": {
    "dev": "nodemon index.js"
  },
  ...
}
```

这样，只需要运行 `npm run dev` 这一个命令就可以启动 server 同时进行开发了。
