<!--
 * @owner: hank.liu
 * @team: 卡鲁秋
-->

# 前端工具 面试知识点总结

本部分主要是笔者在复习 前端工具 相关知识和一些相关面试题时所做的笔记，主要是个人复习使用，如果出现错误，希望并感谢大家指出，如总结答案与标准有出入，请轻喷，谢谢🙏

## 介绍 webpack 是什么？有什么优势？

* webpack 是一款[模块加载器]兼[打包工具]，用于把各种静态资源（js/css/image等）作为模块来使用
* webpack 的优势：
  - webpack 同时支持 commonJS 和 AMD/CMD，方便代码迁移
  - 不仅仅能被模块化 JS ，还包括 CSS、Image 等
  - 能替代部分 grunt/gulp 的工作，如打包、压缩混淆、图片base64
  - 扩展性强，插件机制完善，特别是支持 React 热插拔的功能

## 谈谈你对webpack的看法

> webpack 是一个模块打包工具，你可以使用webpack管理你的模块依赖，并编绎输出模块们所需的静态文件。它能够很好地管理、打包Web开发中所用到的HTML、Javascript、CSS以及各种静态文件（图片、字体等），让开发过程更加高效。对于不同类型的资源，webpack有对应的模块加载器。webpack模块打包器会分析模块间的依赖关系，最后 生成了优化且合并后的静态资源


## webpack hash chunkhash contenthash

Webpack里面有三种hash，分别是：hash, chunkhash, contenthash

我们的浏览器会缓存我们的文件。缓存是把双刃剑，好处是浏览器读取缓存的文件，能带来更佳的用户体验（不需要额外流量，速度更快）；坏处是有时候我们修改了文件内容，但是浏览器依然读取缓存的文件（也就是旧文件），导致用户看到的文件不是最新的。

hash能够帮助我们缓存已经及时的更新缓存文件。

### hash

hash是项目级别的，多个入口文件输出的所有构建的文件是使用同一个hash。

缺点是假如我只改了其中一个文件，但是所有文件的文件名里面的hash都是一样的。这样会导致本来应该被浏览器缓存的文件，强制要去服务器读取一遍，但是这个文件又和之前的旧文件并没有区别，这样就很不好了。那能不能做到只有改变了文件，hash值才变，而没有改变的文件，文件名里面的hash值就不变呢？答案就是chunkhash。

### chunkhash

chunkhash是模块级别的，它根据不同的入口文件(Entry)进行依赖文件解析、构建对应的chunk，生成对应的hash值。我们在生产环境里把一些公共库和程序入口文件区分开，单独打包构建，接着我们采用chunkhash的方式生成hash值，那么只要我们不改动公共库的代码，就可以保证其hash值不会受影响。

缺点就是一般我们会将css单独提取到一个文件中，当我们改变依赖的css文件的js文件的时候，依赖他的其它们文件构建的文件的hash值也会发送改变，即使我们的css文件并没有任何的改变，单独提取出来的css文件的hash值也发生了改变。

### contenthash

contenthash是文件级别的。

contenthash表示由文件内容产生的hash值，内容不同产生的contenthash值也不一样。在项目中，通常做法是把项目中css都抽离出对应的css文件来加以引用。


## Tree-Shaking的工作原理

Tree-shaking （树摇）最早是由Rollup实现，是一种采用删除不需要的额外代码的方式优化代码体积的技术，webpack2借鉴了这个特性也增加了tree-shaking的功能。

tree-shaking 只能在静态modules下工作，在ES6之前我们使用CommonJS规范引入模块，具体采用require()的方式动态引入模块，这个特性可以通过判断条件解决按需记载的优化问题

是CommonJS规范无法确定在实际运行前需要或者不需要某些模块，所以CommonJS不适合tree-shaking机制。

在JavaScript模块话方案中，只有ES6的模块方案：import()引入模块的方式采用静态导入，可以采用一次导入所有的依赖包再根据条件判断的方式，获取不需要的包，然后执行删除操作。

Tree-shaking的实现原理

### 利用ES6模块特性：

1. 只能作为模块顶层的语句出现
2. import的模块名只能是字符串常量
3. 引入的模块不能再进行修改

### 代码删除

uglify：判断程序流，判断变量是否被使用和引用，进而删除代码

### 实现原理可以简单的概况：

1. ES6 Module引入进行静态分析，故而编译的时候正确判断到底加载了那些模块
2. 静态分析程序流，判断那些模块和变量未被使用或者引用，进而删除对应代码

注:

- Tree-shaking不能移除export default方式导出的模块而导入的一个整体的模块，所以应该尽量避免使用export default { A, B, C }导出的方式，而应该替换成 export { A, B, C } 的方式到处

## dev-server是怎么跑起来

webpack-dev-server 可以作为命令行工具使用，核心模块依赖是 webpack 和 webpack-dev-middleware。webpack-dev-server 负责启动一个 express 服务器监听客户端请求；实例化 webpack compiler；启动负责推送 webpack 编译信息的 websocket 服务器；负责向 bundle.js 注入和服务端通信用的 websocket 客户端代码和处理逻辑。webpack-dev-middleware 把 webpack compiler 的 outputFileSystem 改为 in-memory fileSystem；启动 webpack watch 编译；处理浏览器发出的静态资源的请求，把 webpack 输出到内存的文件响应给浏览器。

每次 webpack 编译完成后向客户端广播 ok 消息，客户端收到信息后根据是否开启 hot 模式使用 liveReload 页面级刷新模式或者 hotReload 模块热替换。hotReload 存在失败的情况，失败的情况下会降级使用页面级刷新。

开启 hot 模式，即启用 HMR 插件。hot 模式会向服务器请求更新过后的模块，然后对模块的父模块进行回溯，对依赖路径进行判断，如果每条依赖路径都配置了模块更新后所需的业务处理回调函数则是 accepted 状态，否则就降级刷新页面。判断 accepted 状态后对旧的缓存模块和父子依赖模块进行替换和删除，然后执行 accept 方法的回调函数，执行新模块代码，引入新模块，执行业务处理代码。

https://blog.csdn.net/LuckyWinty/article/details/109507412

## 使用过webpack里面哪些plugin和loader

## webpack整个生命周期，loader和plugin有什么区别

Loader，直译为"加载器"。主要是用来解析和检测对应资源，负责源文件从输入到输出的转换，它专注于实现资源模块加载

Plugin，直译为"插件"。主要是通过webpack内部的钩子机制，在webpack构建的不同阶段执行一些额外的工作，它的插件是一个函数或者是一个包含apply方法的对象，接受一个compile对象，通过webpack的钩子来处理资源

### Loader开发思路

- 通过module.exports导出一个函数

- 该函数默认参数一个参数source(即要处理的资源文件)

- 在函数体中处理资源(loader里配置响应的loader后)

- 通过return返回最终打包后的结果(这里返回的结果需为字符串形式)

### Plugin开发思路

- 通过钩子机制实现

- 插件必须是一个函数或包含apply方法的对象

- 在方法体内通过webpack提供的API获取资源做响应处理

- 将处理完的资源通过webpack提供的方法返回该资源

## webpack打包的整个过程

- 初始化参数：根据用户在命令窗口输入的参数以及 webpack.config.js 文件的配置，得到最后的配置。
- 开始编译：根据上一步得到的最终配置初始化得到一个 compiler 对象，注册所有的插件 plugins，插件开始监听 webpack 构建过程的生命周期的环节（事件），不同的环节会有相应的处理，然后开始执行编译。
- 确定入口：根据 webpack.config.js 文件中的 entry 入口，开始解析文件构建 AST 语法树，找出依赖，递归下去。
- 编译模块：递归过程中，根据文件类型和 loader 配置，调用相应的 loader 对不同的文件做不同的转换处理，再找出该模块依赖的模块，然后递归本步骤，直到项目中依赖的所有模块都经过了本步骤的编译处理。
- 编译过程中，有一系列的插件在不同的环节做相应的事情，比如 UglifyPlugin 会在 loader 转换递归完对结果使用 UglifyJs 压缩覆盖之前的结果；再比如 clean-webpack-plugin ，会在结果输出之前清除 dist 目录等等。
- 完成编译并输出：递归结束后，得到每个文件结果，包含转换后的模块以及他们之间的依赖关系，根据 entry 以及 output 等配置生成代码块 chunk。
- 打包完成：根据 output 输出所有的 chunk 到相应的文件目录。

## 一般怎么组织CSS（Webpack）
## 如何配置把js、css、html单独打包成一个文件
## webpack和gulp的优缺点

| | gulp | webpack |
| ---- | ---- | ---- |
| 定位 | 基于任务流的自动化打包工具 | 模块化打包工具 |
| 优点 | 易于学习和理解, 适合多页面应用开发 | 可以模块化的打包任何资源,适配任何模块系统,适合SPA单页应用的开发 |
| 缺点 | 不太适合单页或者自定义模块的开发 | 学习成本低,配置复杂,通过babel编译后的js代码打包后体积过 |

## 使用webpack构建时有无做一些自定义操作
## webpack的热更新是如何做到的？说明其原理？

## loader原理，css-loader与style-loader

1. css-loader 的作用是处理css中的 @import 和 url 这样的外部资源

2. style-loader 的作用是把样式插入到 DOM中，方法是在head中插入一个style标签，并把样式写入到这个标签的 innerHTML里

loader的原理

loader能把源文件翻译成新的结果，一个文件可以链式经过多个loader编译。以处理scss文件为例:

- sass-loader把scss转成css

- css-loader找出css中的依赖，压缩资源

- style-loader把css转换成脚本加载的JavaScript代码

## webpack插件

1. 编写一个JavaScript命名函数。
2. 在它的原型上定义一个apply方法。
3. 指定挂载的webpack事件钩子。
4. 处理webpack内部实例的特定数据。
5. 功能完成后调用webpack提供的回调。

### webpack构建的主要钩子

Compiler暴露了和webpack整个生命周期相关的钩子

#### Compiler钩子

- entryOption: 在 entry 配置项处理过之后，执行插件。

- afterPlugins: 设置完初始插件之后，执行插件。参数：compiler

- afterResolvers: resolver 安装完成之后，执行插件。参数：compiler

- environment: environment 准备好之后，执行插件。

- afterEnvironment: environment 安装完成之后，执行插件。

- beforeRun: compiler.run() 执行之前，添加一个钩子。参数：compiler

- run: 开始读取 records 之前，钩入(hook into) compiler。参数：compiler

- watchRun: 监听模式下，一个新的编译(compilation)触发之后，执行一个插件，但是是在实际编译开始之前。参数：compiler

- watchRun: 监听模式下，一个新的编译(compilation)触发之后，执行一个插件，但是是在实际编译开始之前。参数：compiler

- normalModuleFactory: NormalModuleFactory 创建之后，执行插件。参数：normalModuleFactory

- contextModuleFactory: ContextModuleFactory 创建之后，执行插件。参数：contextModuleFactory

- beforeCompile: 编译(compilation)参数创建之后，执行插件。参数：compilationParams

- compile: 一个新的编译(compilation)创建之后，钩入(hook into) compiler。参数：compilationParams

- thisCompilation: 触发 compilation 事件之前执行（查看下面的 compilation）。参数：compilation

- compilation: 编译(compilation)创建之后，执行插件。参数：compilation

- make: 分析模块依赖。参数：compilation

- afterCompile:

- shouldEmit: 此时返回 true/false。参数：compilation

- needAdditionalPass:

- emit: 生成资源到 output 目录之前。参数：compilation

- afterEmit: 生成资源到 output 目录之后。参数：compilation

- done: 编译(compilation)完成。参数：stats

- failed: 编译(compilation)失败。参数：error

- invalid: 监听模式下，编译无效时。参数：fileName, changeTime

- watchClose: 监听模式停止

#### Compilation钩子

Compilation暴露了与模块和依赖有关的粒度更小的事件钩子。

在编译阶段，模块会被加载(loaded)、封存(sealed)、优化(optimized)、分块(chunked)、哈希(hashed)和重新创建(restored)。

从上面的示例可以看到，compilation是Compiler生命周期中的一个步骤，使用compilation相关钩子的通用写法为:

- buildModule: 在模块构建开始之前触发。参数：module

- rebuildModule: 在重新构建一个模块之前触发。参数：module

...

- seal: 编译(compilation)停止接收新模块时触发。

- unseal: 编译(compilation)开始接收新模块时触发。

...

- optimize: 优化阶段开始时触发。

## webpack 打包优化的四种方法（多进程打包，多进程压缩，资源 CDN，动态 polyfill）

### 打包分析

#### 1. 速度分析

我们的目的是优化打包速度，那肯定需要一个速度分析插件，此时 `speed-measure-webpack-plugin` 就派上用场了。它的作用如下：

- 分析整个打包总耗时
- 每个 plugin 和 loader 的耗时情况

安装插件

``` shell
npm install --save-dev speed-measure-webpack-plugin
```

使用插件

修改配置webpack.config.js文件

``` js
// 导入速度分析插件
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");

// 实例化速度分析插件
const smp = new SpeedMeasurePlugin();

const webpackConfig = smp.wrap({
  entry: {
    // ...
  },
  output: {
    // ...
  },
  resolve: {
    // ...
  },
  module: {
    rules: [
      // ....
    ]
  },
  plugins: [new MyPlugin(), new MyOtherPlugin()],
});

module.exports = webpackConfig;
```

运行打包命令之后，可以看到，打包总耗时为 `15.48 secs`

效果如下所示:

![Webpack优化_speed-measure-webpack-plugin 打包速度](https://user-images.githubusercontent.com/8088864/126053799-541740e0-922b-45c0-949d-6a91fc64f759.png)


#### 2. 体积分析

分析完打包速度之后，接着我们来分析打包之后每个文件以及每个模块对应的体积大小。使用到的插件为 `webpack-bundle-analyzer`，构建完成后会在 8888 端口展示大小。

安装插件

``` shell
npm install --save-dev webpack-bundle-analyzer
```

使用插件

修改配置webpack.config.js文件

``` js
// 导入速度分析插件
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");

// 导入体积分析插件
const BundleAnalyzerPlugin = require("webpack-bundle-analyzer").BundleAnalyzerPlugin;

// 实例化速度分析插件
const smp = new SpeedMeasurePlugin();

const webpackConfig = smp.wrap({
  entry: {
    // ...
  },
  output: {
    // ...
  },
  resolve: {
    // ...
  },
  module: {
    rules: [
      // ....
    ]
  },
  plugins: [
    // 实例化体积分析插件
    new BundleAnalyzerPlugin(),
    new MyPlugin(),
    new MyOtherPlugin(),
  ],
});

module.exports = webpackConfig;
```

构建之后可以看到，其中黄色块 `chunk-vendors` 文件占比最大，为 `1.34MB`

效果如下所示:

![Webpack优化_webpack-bundle-analyzer 打包体积分析](https://user-images.githubusercontent.com/8088864/126053872-dca114ab-7a88-44ad-8456-6be4a4ce5510.png)

### 打包优化

#### 1. 多进程多实例构建，资源并行解析

多进程构建的方案比较知名的有以下三个：

- thread-loader (推荐使用这个)
- parallel-webpack
- HappyPack

这里以 `thread-loader` 为例配置多进程多实例构建

安装 loader

``` shell
npm install --save-dev thread-loader
```

使用 loader

修改配置webpack.config.js文件

``` js
const path = require("path");
// 导入速度分析插件
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");

// 实例化速度分析插件
const smp = new SpeedMeasurePlugin();

const webpackConfig = smp.wrap({
  entry: {
    // ...
  },
  output: {
    // ...
  },
  resolve: {
    // ...
  },
  module: {
    rules: [
      rules: [
        {
          test: /\.js$/,
          include: path.resolve('src'),
          use: [
            'thread-loader',
            // your expensive loader (e.g babel-loader)
          ],
        }
      ]
    ]
  },
  plugins: [
    new MyPlugin(),
    new MyOtherPlugin(),
  ],
});

module.exports = webpackConfig;
```

#### 2. 公用代码提取，使用 CDN 加载

以vue.js构建的项目为例，里面很多的第三方库只要不升级对应的版本其内容是不变的，我们可以将这些内用文件不通过webpack打包到模块里面，而是使用CDN加载，例如对于vue，vuex，vue-router，axios，echarts，swiper等第三方库我们可以利用webpack的externals参数来配置，这里我们设定只需要在生产环境中才需要使用。

这里需要使用 `html-webpack-plugin` 和 `webpack-cdn-plugin` 两个插件

安装插件

``` shell
npm install --save-dev html-webpack-plugin, webpack-cdn-plugin
```

使用插件

修改配置webpack.config.js文件

``` js
const path = require("path");
// 导入速度分析插件
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");

// 导入体积分析插件
const BundleAnalyzerPlugin = require("webpack-bundle-analyzer").BundleAnalyzerPlugin;

const HtmlWebpackPlugin = require('html-webpack-plugin');

//判断是否为生产环境
const isProduction = process.env.NODE_ENV === 'production';

// 实例化速度分析插件
const smp = new SpeedMeasurePlugin();

//定义 CDN 路径，这里采用 bootstrap 的 cdn
const cdn = {
    css: [
        'https://cdn.bootcss.com/Swiper/4.5.1/css/swiper.min.css'
    ],
    js: [
        'https://cdn.bootcss.com/vue/2.6.10/vue.min.js',
        'https://cdn.bootcss.com/vue-router/3.1.3/vue-router.min.js',
        'https://cdn.bootcss.com/vuex/3.1.1/vuex.min.js',
        'https://cdn.bootcss.com/axios/0.19.0/axios.min.js',
        'https://cdn.bootcss.com/echarts/4.3.0/echarts.min.js',
        'https://cdn.bootcss.com/Swiper/4.5.1/js/swiper.min.js',
    ]
}

const webpackConfig = smp.wrap({
  entry: {
    // ...
  },
  output: {
    // ...
  },
  resolve: {
    // ...
  },
  //生产环境注入 cdn
  externals: isProduction && {
    'vue': 'Vue',
    'vuex': 'Vuex',
    'vue-router': 'VueRouter',
    'axios': 'axios',
    'echarts': 'echarts',
    'swiper': 'Swiper'
  } || {},
  module: {
    rules: [
      rules: [
        {
          test: /\.js$/,
          include: path.resolve('src'),
          use: [
            'thread-loader',
            // your expensive loader (e.g babel-loader)
          ],
        }
      ]
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({ filename: '../index.html' }), // output file relative to output.path
    new WebpackCdnPlugin({
      modules: [
        {
          name: 'vue',
          var: 'Vue',
          path: 'vue.min.js'
        },
        {
          name: 'vuex',
          var: 'Vuex',
          path: 'vuex.min.js'
        }
        {
          name: 'vue-router',
          var: 'VueRouter',
          path: 'vue-router.min.js'
        },
        {
          name: 'axios',
          var: 'axios',
          path: 'axios.min.js'
        }
        {
          name: 'echarts',
          var: 'echarts',
          path: 'echarts.min.js'
        },
        {
          name: 'swiper',
          var: 'Swiper',
          path: 'swiper.min.js'
        },
      ],
      prod: isProduction,
      prodUrl: '//cdn.bootcdn.net/ajax/libs/:name/:version/:path' // => https://cdn.bootcdn.net/ajax/libs/xxx/xxx/xxx(`:name`,`:version`和`:path`为模板变量)
      publicPath: '/node_modules/dist', // override when prod is false
    }),
    new MyPlugin(),
    new MyOtherPlugin(),
  ],
});

module.exports = webpackConfig;
```

最终生成的inde.html文件如下所示:

``` html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Webpack App</title>
    <link href="https://cdn.bootcdn.net/ajax/libs/Swiper/6.7.5/swiper-bundle.min.css" rel="stylesheet">
  </head>
  <body>
  <script type="text/javascript" src="https://cdn.bootcdn.net/ajax/libs/vue/2.6.13/vue.min.js"></script>
  <script type="text/javascript" src="https://cdn.bootcdn.net/ajax/libs/vue-router/3.1.3/vue-router.min.js"></script>
  <script type="text/javascript" src="https://cdn.bootcdn.net/ajax/libs/vuex/3.1.1/vuex.min.js"></script>
  <script type="text/javascript" src="https://cdn.bootcdn.net/ajax/libs/axios/0.19.0/axios.min.js"></script>
  <script type="text/javascript" src="https://cdn.bootcdn.net/ajax/libs/echarts/4.3.0/echarts.min.js"></script>
  <script type="text/javascript" src="https://cdn.bootcdn.net/ajax/libs/Swiper/6.7.5/js/swiper.min.js"></script>
  <script type="text/javascript" src="/assets/app.js"></script>
  </body>
</html>
```

#### 3. 多进程多实例并行压缩

并行压缩主流有以下三种方案

- 使用 parallel-uglify-plugin 插件
- uglifyjs-webpack-plugin 开启 parallel 参数
- terser-webpack-plugin 开启 parallel 参数 （推荐使用这个，支持 ES6 语法压缩）

安装插件

``` shell
npm install --save-dev terser-webpack-plugin
```

使用插件

修改配置webpack.config.js文件

``` js
const path = require("path");
// 导入速度分析插件
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");

// 导入代码压缩插件
const TerserPlugin = require("terser-webpack-plugin");

// 导入体积分析插件
const BundleAnalyzerPlugin = require("webpack-bundle-analyzer").BundleAnalyzerPlugin;

const HtmlWebpackPlugin = require('html-webpack-plugin');

//判断是否为生产环境
const isProduction = process.env.NODE_ENV === 'production';

// 实例化速度分析插件
const smp = new SpeedMeasurePlugin();

//定义 CDN 路径，这里采用 bootstrap 的 cdn
const cdn = {
    css: [
        'https://cdn.bootcss.com/Swiper/4.5.1/css/swiper.min.css'
    ],
    js: [
        'https://cdn.bootcss.com/vue/2.6.10/vue.min.js',
        'https://cdn.bootcss.com/vue-router/3.1.3/vue-router.min.js',
        'https://cdn.bootcss.com/vuex/3.1.1/vuex.min.js',
        'https://cdn.bootcss.com/axios/0.19.0/axios.min.js',
        'https://cdn.bootcss.com/echarts/4.3.0/echarts.min.js',
        'https://cdn.bootcss.com/Swiper/4.5.1/js/swiper.min.js',
    ]
}

const webpackConfig = smp.wrap({
  entry: {
    // ...
  },
  output: {
    // ...
  },
  resolve: {
    // ...
  },
  module: {
    rules: [
      rules: [
        {
          test: /\.js$/,
          include: path.resolve('src'),
          use: [
            'thread-loader',
            // your expensive loader (e.g babel-loader)
          ],
        }
      ]
    ]
  },
  //生产环境注入 cdn
  externals: isProduction && {
    'vue': 'Vue',
    'vuex': 'Vuex',
    'vue-router': 'VueRouter',
    'axios': 'axios',
    'echarts': 'echarts',
    'swiper': 'Swiper'
  } || {},
  optimization: {
    minimizer: [
      new TerserPlugin({
        parallel: 4
      })
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({ filename: '../index.html' }), // output file relative to output.path
    new WebpackCdnPlugin({
      modules: [
        {
          name: 'vue',
          var: 'Vue',
          path: 'vue.min.js'
        },
        {
          name: 'vuex',
          var: 'Vuex',
          path: 'vuex.min.js'
        }
        {
          name: 'vue-router',
          var: 'VueRouter',
          path: 'vue-router.min.js'
        },
        {
          name: 'axios',
          var: 'axios',
          path: 'axios.min.js'
        }
        {
          name: 'echarts',
          var: 'echarts',
          path: 'echarts.min.js'
        },
        {
          name: 'swiper',
          var: 'Swiper',
          path: 'swiper.min.js'
        },
      ],
      prod: isProduction,
      prodUrl: '//cdn.bootcdn.net/ajax/libs/:name/:version/:path' // => https://cdn.bootcdn.net/ajax/libs/xxx/xxx/xxx(`:name`,`:version`和`:path`为模板变量)
      publicPath: '/node_modules/dist', // override when prod is false
    }),
    new MyPlugin(),
    new MyOtherPlugin(),
  ],
});

module.exports = webpackConfig;
```

#### 4. 使用 polyfill 动态服务

Polyfill 可以为旧浏览器提供和标准 API 一样的功能。比如你想要 IE 浏览器实现 Promise 和 fetch 功能，你需要手动引入 es6-promise、whatwg-fetch。而通过 Polyfill.io，你只需要引入一个 JS 文件。

Polyfill.io 通过分析请求头信息中的 UserAgent 实现自动加载浏览器所需的 polyfills。 Polyfill.io 有一份默认功能列表，包括了最常见的 polyfills：document.querySelector、Element.classList、ES5 新增的 Array 方法、Date.now、ES6 中的 Object.assign、Promise 等。

动态 `polyfill` 指的是根据不同的浏览器，动态载入需要的 `polyfill`。 `Polyfill.io` 通过尝试使用 `polyfill` 重新创建缺少的功能，可以更轻松地支持不同的浏览器，并且可以大幅度的减少构建体积。

Polyfill Service 原理

识别 User Agent，下发不同的 Polyfill

![Webpack polyfill 服务](https://user-images.githubusercontent.com/8088864/126054825-2e1a0e44-2eb7-4668-b044-de846427e577.png)

使用方法：

在 index.html 中引入如下 script 标签

``` html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Document</title>
</head>
<body>
</body>
<script src="https://cdn.polyfill.io/v2/polyfill.min.js?callback=main" async defer></script>
<script>
function main () {
  var node=document.createElement("script");
  node.src="index.js";
  document.body.appendChild(node);
}
</script>
</html>
```

## webpack 做过哪些优化，开发效率方面、打包策略方面等等

### 1）优化 Webpack 的构建速度

- 使用高版本的 Webpack （使用webpack4）
- 多线程/多实例构建：HappyPack(不维护了)、thread-loader
- 缩小打包作用域：
  - exclude/include (确定 loader 规则范围)
  - resolve.modules 指明第三方模块的绝对路径 (减少不必要的查找)
  - resolve.extensions 尽可能减少后缀尝试的可能性
  - noParse 对完全不需要解析的库进行忽略 (不去解析但仍会打包到 bundle 中，注意被忽略掉的文件里不应该包含 import、require、define 等模块化语句)
  - IgnorePlugin (完全排除模块)
  - 合理使用alias
- 充分利用缓存提升二次构建速度：
  - babel-loader 开启缓存
  - terser-webpack-plugin 开启缓存
  - 使用 cache-loader 或者 hard-source-webpack-plugin
    注意：thread-loader 和 cache-loader 兩個要一起使用的話，請先放 cache-loader 接著是 thread-loader 最後才是 heavy-loader
- DLL：
  - 使用 DllPlugin 进行分包，使用 DllReferencePlugin(索引链接) 对 manifest.json 引用，让一些基本不会改动的代码先打包成静态资源，避免反复编译浪费时间。

### 2）使用webpack4-优化原因

1. V8带来的优化（for of替代forEach、Map和Set替代Object、includes替代indexOf）
2. 默认使用更快的md4 hash算法
3. webpacks AST可以直接从loader传递给AST，减少解析时间
4. 使用字符串方法替代正则表达式

#### noParse

- 不去解析某个库内部的依赖关系
- 比如jquery 这个库是独立的， 则不去解析这个库内部依赖的其他的东西
- 在独立库的时候可以使用

``` js
module.exports = {
  module: {
    noParse: /jquery/,
    rules:[]
  }
}
```

#### IgnorePlugin

- 忽略掉某些内容 不去解析依赖库内部引用的某些内容
- 从moment中引用 ./locol 则忽略掉
- 如果要用local的话 则必须在项目中必须手动引入 import 'moment/locale/zh-cn'

``` js
module.exports = {
  plugins: [
    new Webpack.IgnorePlugin(/./local/, /moment/),
  ]
}
```

#### dllPlugin

- 不会多次打包， 优化打包时间
- 先把依赖的不变的库打包
- 生成 manifest.json文件
- 然后在webpack.config中引入
- webpack.DllPlugin Webpack.DllReferencePlugin

#### happypack -> thread-loader

- 大项目的时候开启多线程打包
- 影响前端发布速度的有两个方面，一个是构建，一个就是压缩，把这两个东西优化起来，可以减少很多发布的时间。

#### thread-loader

thread-loader 会将您的 loader 放置在一个 worker 池里面运行，以达到多线程构建。
把这个 loader 放置在其他 loader 之前（如下图 example 的位置）， 放置在这个 loader 之后的 loader 就会在一个单独的 worker 池(worker pool)中运行。

``` js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        include: path.resolve("src"),
        use: [
          "thread-loader",
          // 你的高开销的loader放置在此 (e.g babel-loader)
        ]
      }
    ]
  }
}
```

每个 worker 都是一个单独的有 600ms 限制的 node.js 进程。同时跨进程的数据交换也会被限制。请在高开销的loader中使用，否则效果不佳

#### 压缩加速——开启多线程压缩

- 不推荐使用 webpack-paralle-uglify-plugin，项目基本处于没人维护的阶段，issue 没人处理，pr没人合并。
  Webpack 4.0以前：uglifyjs-webpack-plugin，parallel参数

``` js
module.exports = {
  optimization: {
    minimizer: [
      new UglifyJsPlugin({
        parallel: true,
      }),
    ],
  },
};
```

- 推荐使用 terser-webpack-plugin

``` js
module.exports = {
  optimization: {
    minimizer: [
      new TerserPlugin({
        parallel: true   // 多线程
      })
    ],
  },
};
```

### 2）优化 Webpack 的打包体积

- 压缩代码
  - webpack-paralle-uglify-plugin
  - uglifyjs-webpack-plugin 开启 parallel 参数 (不支持ES6)
  - terser-webpack-plugin 开启 parallel 参数
  - 多进程并行压缩
  - 通过 mini-css-extract-plugin 提取 Chunk 中的 CSS 代码到单独文件，通过optimize-css-assets-webpack-plugin插件 开启 cssnano 压缩 CSS。
- 提取页面公共资源
  - 使用 html-webpack-externals-plugin，将基础包通过 CDN 引入，不打入 bundle 中
  - 使用 SplitChunksPlugin 进行(公共脚本、基础包、页面公共文件)分离(Webpack4内置) ，替代了 CommonsChunkPlugin 插件
  - 基础包分离：将一些基础库放到cdn，比如vue，webpack 配置 external是的vue不打入bundle
- Tree shaking
  - purgecss-webpack-plugin 和 mini-css-extract-plugin配合使用(建议)
  - 打包过程中检测工程中没有引用过的模块并进行标记，在资源压缩时将它们从最终的bundle中去掉(只能对ES6 Modlue生效) 开发中尽可能使用ES6 Module的模块，提高tree shaking效率
  - 禁用 babel-loader 的模块依赖解析，否则 Webpack 接收到的就都是转换过的 CommonJS 形式的模块，无法进行 tree-shaking
  - 使用 PurifyCSS(不在维护) 或者 uncss 去除无用 CSS 代码
- Scope hosting
  - 构建后的代码会存在大量闭包，造成体积增大，运行代码时创建的函数作用域变多，内存开销变大。Scope hosting 将所有模块的代码按照引用顺序放在一个函数作用域里，然后适当的重命名一些变量以防止变量名冲突
  - 必须是ES6的语法，因为有很多第三方库仍采用 CommonJS 语法，为了充分发挥 Scope hosting 的作用，需要配置 mainFields 对第三方模块优先采用 jsnext:main 中指向的ES6模块化语法
- 图片压缩
  - 使用基于 Node 库的 imagemin (很多定制选项、可以处理多种图片格式)
  - 配置 image-webpack-loader
- 动态Polyfill
  - 建议采用 polyfill-service 只给用户返回需要的polyfill，社区维护。(部分国内奇葩浏览器UA可能无法识别，但可以降级返回所需全部polyfill)
  - @babel-preset-env 中通过useBuiltIns: 'usage参数来动态加载polyfill。

### 3）speed-measure-webpack-plugin

简称 SMP，分析出 Webpack 打包过程中 Loader 和 Plugin 的耗时，有助于找到构建过程中的性能瓶颈。

### 4）开发阶段常用的插件

#### 开启多核压缩

插件：**terser-webpack-plugin**

``` js
const TerserPlugin = require('terser-webpack-plugin')
module.exports = {
  optimization: {
    minimizer: [
      new TerserPlugin({
        parallel: true,
        terserOptions: {
          ecma: 6,
        },
      }),
    ]
  }
}
```

#### 监控面板

插件：**speed-measure-webpack-plugin**

在打包的时候显示出每一个loader,plugin所用的时间，来精准优化

``` js
// webpack.config.js文件
const SpeedMeasurePlugin = require('speed-measure-webpack-plugin');
const smp = new SpeedMeasurePlugin();
//............
// 用smp.warp()包裹一下合并的config
module.exports = smp.wrap(merge(_mergeConfig, webpackConfig));
```

#### 开启一个通知面板

插件：**webpack-build-notifier**

``` js
// webpack.config.js文件
const WebpackBuildNotifierPlugin = require('webpack-build-notifier');
const webpackConfig= {
  plugins: [
    new WebpackBuildNotifierPlugin({
      title: '我的webpack',
      // logo: path.resolve('./img/favicon.png'),
      suppressSuccess: true
    })
  ]
}
```

#### 开启打包进度

插件：**progress-bar-webpack-plugin**

``` js
// webpack.config.js文件
const ProgressBarPlugin = require('progress-bar-webpack-plugin');
const webpackConfig= {
  plugins: [
    new ProgressBarPlugin(),
  ]
}
```

#### 开发面板更清晰

插件：**webpack-dashboard**

``` js
// webpack.config.js文件
const DashboardPlugin = require('webpack-dashboard/plugin');
const webpackConfig= {
  plugins: [
    new DashboardPlugin()
  ]
}
```

``` json
// package.json文件
{
  "scripts": {
    "dev": "webpack-dashboard webpack --mode development",
  },
}
```

#### 开启窗口的标题

第三方库: **node-bash-title**

这个包mac的item用有效果，windows暂时没看到效果

``` js
// webpack.config.js文件
const setTitle = require('node-bash-title');
setTitle('server');
```

#### friendly-errors-webpack-plugin

插件：**friendly-errors-webpack-plugin**

``` js
const webpackConfig= {
  plugins: [
    new FriendlyErrorsWebpackPlugin({
      compilationSuccessInfo: {
        messages: ['You application is running here http://localhost:3000'],
        notes: ['Some additionnal notes to be displayed unpon successful compilation']
      },
      onErrors: function (severity, errors) {
        // You can listen to errors transformed and prioritized by the plugin
        // severity can be 'error' or 'warning'
      },
      // should the console be cleared between each compilation?
      // default is true
      clearConsole: true,

      // add formatters and transformers (see below)
      additionalFormatters: [],
      additionalTransformers: []
    }),
  ]
}
```

## Chrome打开一个页面需要启动多少进程

最新的Chrome浏览器包括；1个浏览器主进程（Browser）、1个GPU进程、一个网络（NetWork）进程和多个插件进程。

- 浏览器进程：主要负责界面显示、用户交互、子进程管理，同时提供存储功能；

- 渲染进程：核心人物是将HTML、CSS和JavaScript引擎V8都是及逆行在该进程中，漠然情况下，Chrome会为每个Ta标签创建一个渲染进程。出于安全考虑；显然进程都是运行在沙箱模式下。

- GPU进程：其实，Chrome刚开始发布的时候是没有GPU进程的。而GPU的使用初衷是为了实现3DCSS的效果，只是随后网页、Chrome的UI界面都选择采取GPU来绘制，这使得GPU成为浏览器普遍的需求。最后，Chrome在奇多进程架构上也引入了GPU进程。

- 网络进程：主要负责网页的网络资源加载，之前是作为一个模块运行在浏览器进程在里面的，直至最近才独立出来，成为一个单独的进程。

- 插件进程：主要是负责插件的运行，因插件容易崩溃，所以需要通过插件进程来隔离，以保证插件进程崩溃不会对浏览器和页面造成影响

## 了解v8引擎吗，一段js代码如何执行的

在执行一段代码时，JS 引擎会首先创建一个执行栈

然后JS引擎会创建一个全局执行上下文，并push到执行栈中, 这个过程JS引擎会为这段代码中所有变量分配内存并赋一个初始值（undefined），在创建完成后，JS引擎会进入执行阶段，这个过程JS引擎会逐行的执行代码，即为之前分配好内存的变量逐个赋值(真实值)。

如果这段代码中存在function的声明和调用，那么JS引擎会创建一个函数执行上下文，并push到执行栈中，其创建和执行过程跟全局执行上下文一样。但有特殊情况，即当函数中存在对其它函数的调用时，JS引擎会在父函数执行的过程中，将子函数的全局执行上下文push到执行栈，这也是为什么子函数能够访问到父函数内所声明的变量。

还有一种特殊情况是，在子函数执行的过程中，父函数已经return了，这种情况下，JS引擎会将父函数的上下文从执行栈中移除，与此同时，JS引擎会为还在执行的子函数上下文创建一个闭包，这个闭包里保存了父函数内声明的变量及其赋值，子函数仍然能够在其上下文中访问并使用这边变量/常量。当子函数执行完毕，JS引擎才会将子函数的上下文及闭包一并从执行栈中移除。

最后，JS引擎是单线程的，那么它是如何处理高并发的呢？即当代码中存在异步调用时JS是如何执行的。比如setTimeout或fetch请求都是non-blocking的，当异步调用代码触发时，JS引擎会将需要异步执行的代码移出调用栈，直到等待到返回结果，JS引擎会立即将与之对应的回调函数push进任务队列中等待被调用，当调用(执行)栈中已经没有需要被执行的代码时，JS引擎会立刻将任务队列中的回调函数逐个push进调用栈并执行。这个过程我们也称之为事件循环。


## 介绍chrome 浏览器的几个版本

### 1）Chrome 浏览器提供 4 种发布版本，即稳定版(Stable)、测试版(Beta)、开发者版(Dev)和金丝雀版(Canary)。

虽然 Chrome 这几个版本名称各不相同，但都沿用了相同的版本号，只是更新早晚的区别。就好比 iOS 等系统，Beta 版可以率先更新到 iOS 12 并进行测试，不断改进稳定后，正式版才升级到 12 版本。
Chrome 也是如此，更新最快的 Canary 会领先正式版 1-2 个版本。

#### 1. Canary（金丝雀） 版

只限用于测试，Canary 是 Chrome 的未来版本，是功能、代码最先进的Chrome 版本，一方面软件本身没有足够时间测试，另一方面网页也不一定支持这些全新的功能，因此极不稳定。好在，谷歌将其设定为可独立安装、与其他版本的 Chrome 程序共存，因此适合进阶用户安装备用，尝鲜最新功能。这种不稳定性使得 Canary 版目前并不适合日常使用。
Chrome Canary 是更新速度最快的 Chrome 版本，几乎每天更新。它相当于支持自动更新、并添加了谷歌自家服务与商业闭源插件（Flash 等）的 Chromium，更加强大好用。

#### 2. 开发者版(Dev)

Chrome Dev 最初是以 Chromium 为基础、更新最快的 Chrome，后来则被 Canary 取代。Dev 版每周更新一次，虽然仍不太稳定，但已经可以勉强满足日常使用，适合 Web 开发者用来测试新功能和网页。
让 IT 人员使用开发者版，开发者可以通过开发者版测试自己公司的应用，确保这些应用能与Chrome 最新的 API 更改及功能更改兼容。注意：开发者版并非百分之百稳定，但开发者可以提前 9 至 12 周体验即将添加到 Chrome 稳定版的功能。

#### 3. 测试版(Beta)

Chrome Beta 以 Dev 为基础，每月更新一次。它是正式发布前的最后测试版本，所有功能都已在前面几个版本中得到测试并改进，因此已经十分稳定，普通用户也可以用来日常使用
让 5% 的用户使用测试版，测试版用户可以提前 4-6 周体验即将在 Chrome 稳定版中推出的功能。测试版用户可以发现特定版本可能存在的问题，让您可以先解决问题，然后再向所有用户推出该版本。

#### 4. 稳定版(Stable)

最后的 Chrome Stable 就是我们熟知的正式版，它以 Beta 为基础，几个月更新一次。由于所有的功能都已经过数个月反复测试，是稳定性最高的 Chrome 版本。
让大多数用户使用稳定版，稳定版是已进行充分测试的版本，稳定版每 2-3 周会进行一次小幅更新，并且每 6 周会进行一次重大更新。
所以要定期下载开发者版，体验Chrome 最新的 API和新功能 ，发现自己的应用跟新API和新功能的是否有兼容问题，找到开发亮点。

### 2）对于Chrome的历史版本测试

可以使用Docker Selenium 做分布式自动化测试，部署多个重点关注的版本，进行自动化测试，对比差异。


## 网页移动端调试工具

### vConsole

引入vconsole到项目中：

``` html
<script src="path/to/vconsole.min.js"></script>
<script>
// init vConsole
var vConsole = new VConsole();
console.log('Hello world');
</script>
```

或者通过import 初试化：
``` js
import VConsole from 'vconsole/dist/vconsole.min.js' //import vconsole
let vConsole = new VConsole() // 初始化
```

项目运行，点击页面右下角vconsole图标，即可看到debug内容
如果想要查看接口调用情况，和浏览器一样直接点击network按钮即可

### AlloyLever

AlloyLever是腾讯AlloyTeam团队开源的一款Web 开发调试工具。和vConsole类似

通过npm安装：
``` shell
npm install alloylever
```

使用：
在你的页面任何地方引用下面脚本就可以，但是该js必须引用在其他脚本之前。
``` html
<script src="alloylever.js"></script>
```
务必记住，该js必须引用在其他脚本之前！

### Eruda

使用手机端网页的调试工具Eruda在你的代码里面，加入下面两行代码，就可以轻轻松松实现手机调试了
``` html
<script src="//cdn.jsdelivr.net/npm/eruda"></script>
<script>
eruda.init();
console.log('控制台打印信息');
</script>
```

ps：想要在手机上查看，可以使手机跟你的电脑在同一个局域网内，然后访问电脑的ip，然后就能查看你做的h5页面了

### spy-debugger

spy-debugger 是一个一站式页面调试、抓包工具。远程调试任何手机浏览器页面，任何手机移动端webview（如：微信，HybridApp等）。支持HTTP/HTTPS，无需USB连接设备。

特性

1. 页面调试＋抓包
2. 操作简单，无需USB连接设备
3. 支持HTTPS。
4. spy-debugger内部集成了weinre、node-mitmproxy、AnyProxy。
5. 自动忽略原生App发起的https请求，只拦截webview发起的https请求。对使用了SSL pinning技术的原生App不造成任何影响。
6. 可以配合其它代理工具一起使用(默认使用AnyProxy) (设置外部代理)

### DevTools

android&Html5混合开发WebView调试必备神器DevTools，chrome浏览器调试手机端WebView

DevTools能在浏览器上调试手机中的webview代码，给手机端调试带来了极大的便利!

操作步骤:

1. 打开手机开发者选项，开启USB调试
2. 打开待调试的webview
3. 手机通过USB数据线跟电脑连接
4. 打开chrome浏览器，输入:chrome://inspect/#devices
5. 点击inspect，进入调试页面进行调试，之后就可以直接在电脑上操作手机啦

DevTools需要外网环境才能访问，在内网环境测试的要保证电脑与外网联通；

## 列举IE与其他浏览器不一样的特性？

* IE 的渲染引擎是 Trident 与 W3C 标准差异较大：例如盒子模型的怪异模式
* JS 方面有很多独立的方法，例如事件处理不同：绑定/删除事件，阻止冒泡，阻止默认事件等
* CSS 方面也有自己独有的处理方式，例如设置透明，低版本IE中使用滤镜的方式