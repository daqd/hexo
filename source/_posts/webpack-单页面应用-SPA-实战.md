---
title: webpack 单页面应用(SPA)实战
date: 2016-08-04 14:08:50
tags:
	- webpack
	- 前端模块化
---
这篇文章将介绍如何利用 webpack 进行单页面应用的开发，算是我在实际开发中的一些心得和体会,在这里给大家做一个分享。webpack 的介绍这里就不多说了，可以直接去官网查看。 关于这个单页面应用大家可以直接去我的github上查看https://github.com/huangshuwei/webpackForSPA

我将结合这个项目去介绍。如果大家觉得这篇文章有不妥的地方，还请指出。

> 这篇文章的目的是解决我们在开发中会遇到的问题，不是一篇基础教程，还请谅解。

## 项目目录

我将根据这个目录结构进行讲解
<!--more-->

![mulu](/images/2016/08/20160804-1.jpg)

- dist：发布的文件目录，即webpack编译输出的目录
- libs：放置公共的文件，如js、css、img、font等
- mockServer：模拟后端服务，即用webpack开发时模拟调用的后端服务（用nodejs服务模拟）
- node_modules：项目依赖的包
- src：资源文件，里面包含css、font、html、img、js
- package.json：项目配置
- webpack.config.js：webpack的配置文件

## 项目的使用

建议先运行一下这个项目，有一个大致的了解，再往下阅读。使用说明：

```
首先克隆一份到你的本地
$ git clone https://github.com/huangshuwei/webpackForSPA.git

然后 cd 到 ‘webpackForSPA’目录下
$ cd webpackForSPA

接着你可以运行不同的命令查看结果

发布模式：
$ npm run build

开发模式：
$ npm run dev

热更新模式
$ npm run dev-hrm

如果使用了热更新模式，并且想要结合后端服务形式运行，那么cd 到‘mockServer’目录下，并执行node 服务：
$ cd mockServer

$ node server.js

```

## 区分开发、热更新、发布模式

> 一般开发时和发布时是不同的，比如开发时文件的访问目录包含‘dist’目录，但是发布上线时，一般会把‘dist’文件夹去掉。
当然还有其他的一些细节不同。


### 开发模式

- 能看到webpack编译输出的文件
- js、css、html文件不需要压缩
- 可以正确的运行编译输出后的文件
- 这种模式一般只是用来看webpack编译输出后的文件是否正确

### 热更新模式

看不到webpack编译输出的文件
- js、css、html文件不需要压缩
- 更改完文件后无需重新编译并自动刷新浏览器
- 可以结合后端服务开发，避过浏览器同源策略，如结合java、.net服务等

### 发布模式

- 能看到webpack编译输出的文件
- js、css、html文件压缩
- 文件的层级目录不需要包含‘dist’目录

我区分开发、热更新、发布模式是通过配置`package.json`文件的运行命令，有些人是通过创建多个不同的webpack的配置文件来达到想要的效果。

像 [这个项目](https://github.com/webpack/react-starter) 就是使用了多个webpack的配置文件。

## 配置命令

这是在 `package.json` 文件中配置的

```
// package.json 文件
...
"scripts": {
    "build": "webpack  --profile --progress --colors --display-error-details",
    "dev": "webpack  --display-modules --profile --progress --colors --display-error-details",
    "dev-hrm": "webpack-dev-server --config"
  },
...
```
- color 输出结果带彩色，比如：会用红色显示耗时较长的步骤
- profile 输出性能数据，可以看到每一步的耗时
- progress 输出当前编译的进度，以百分比的形式呈现
- display-modules 默认情况下 node_modules 下的模块会被隐藏，加上这个参数可以显示这些被隐藏的模块
- display-error-details 输出详细的错误信息
- webpack-dev-server 将会开启热更新

更多请参考官网 [cli](https://webpack.github.io/docs/cli.html)

配置好了`package.json`文件,我们就可以这样运行

```
// 开发模式
npm run dev

// 热更新模式
npm run dev-hrm

// 发布模式
npm run build
```


## 配置变量标识

配置完了命令，当我们运行不同的命令时，我们可以通过`process.env.npm_lifecycle_event`去获取当前运行的命令，根据不同的命令，我们可以按照自己的需要做相应的处理。比如开发模式时，允许开启调试，静态资源不要压缩；发布模式时，不允许调试，静态资源要压缩。具体如下：

```
// webpack.config.js

// 获取当前运行的模式
var currentTarget = process.env.npm_lifecycle_event;

var debug,          // 是否是调试
    devServer,      // 是否是热更新模式
    minimize;       // 是否需要压缩

if (currentTarget == "build") { // 发布模式

    debug = false, devServer = false, minimize = true;
    
} else if (currentTarget == "dev") { // 开发模式

    debug = true, devServer = false, minimize = false;
    
} else if (currentTarget == "dev-hrm") { // 热更新模式

    debug = true, devServer = true, minimize = false;
}
```
## 基础配置
### 配置路径
为了方便我们频繁使用路径，如下配置:
```
// webpack.config.js
var PATHS = {
    // 发布目录
    publicPath: debug ? '/webpackForSPA/dist/' : '/webpackForSPA/',

    // 公共资源目录
    libsPath: path.resolve(process.cwd(), './libs'),
    
    // src 资源目录
    srcPath: path.resolve(process.cwd(), 'src'),
}
```

### 配置别名

webpack的别名的目的就是简化我们的操作，引用资源时直接使用别名即可（和 seajs 里的别名用法一样）。配置如下：

```
// webpack.config.js
...
resolve:{
     alias: {
        // js
        jquery: path.join(PATHS.libsPath, "js/jquery/jquery"),
        underscore: path.join(PATHS.libsPath, "js/underscore/underscore.js"),

        // css
        bootstrapcss: path.join(PATHS.libsPath, "css/bootstrap/bootstrap-3.3.5.css"),
        indexcss: path.join(PATHS.srcPath, "css/index.css"),
    }
}
...
```
### 配置webpack编译入口

```
// webpack.config.js
...
entry:{
    // 入口 js
    index: './src/js/index.js',
    // 公共js包含的文件
    common: [
        path.join(PATHS.libsPath, "js/jquery/jquery.js"),
        path.join(PATHS.libsPath, "js/underscore/underscore.js")
    ],
}
...
```

### 配置webpack编译输出

```
// webpack.config.js
...
output：{
    // 输出目录
    path: path.join(__dirname, 'dist'),

    // 发布后，资源的引用目录
    publicPath: PATHS.publicPath,

    // 文件名称
    filename: 'js/[name].js',

    // 按需加载模块时输出的文件名称
    chunkFilename: 'js/[name].js'
}
...
```

## 提取css到单独的文件

当我们在js文件中通过`require('')`引用js时，webpack 默认会将css文件与当前js文件打包一起，但是这种方式会阻塞页面的加载，因为css的执行要等待js文件加载进来。所以我们会把css从js文件中提取出来，放到一个单独的css文件中。这时我们要使用webpack的插件：`extract-text-webpack-plugin`，配置如下：

### 引入插件

```
// webpack.config.js
var ExtractTextPlugin = require("extract-text-webpack-plugin");
```

### 配置 loader

```
// webpack.config.js
...
loaders: [
    {
        test: /\.css$/,
        loader: ExtractTextPlugin.extract("style-loader", "css-loader!postcss-loader")
    },
    ...
]
...
```

### 配置 plugins

```
// webpack.config.js
...
plugins:[
    new ExtractTextPlugin("css/[name].css", {allChunks: true}),
    ...
]
...
```

## 公共js打包

项目中，我们通常会有公共的js，比如 `jquery`、`bootstrap`、`underscore` 等，那么这时候我们需要将这些公共的js单独打包。这时我们需要用webpack自带的插件：

```
// webpack.config.js
...
plugins:[
    // 会把 ‘entry’ 定义的 common 对应的两个js 打包为 ‘common.js’
    new webpack.optimize.CommonsChunkPlugin("common", 'js/[name].js', Infinity),
]
...
```

## 资源添加版本号

项目上线后，资源的版本号十分重要。资源没有版本号，即使重新发布，客户端浏览器可能会把老的资源缓存下来，导致无法下载最新的资源。webpack 支持给资源添加版本号，不仅仅是js、css,甚至font、img都可以添加版本号。我们可以通过webpack中的`chunkhash`来解决。

首先要了解下webpack 中 `[hash]`、`[chunkhash]`、`[chunkhash:8]`的区别。

- [hash]：webpack编译会产生一个hash值
- [chunkhash]：每个模块的hash值
- [chunkhash:8]：取[chunkhash]的前8位

> 推荐发布模式使用版本号，其他模式无需使用，热更新模式不支持‘chunkhash’，但是支持‘hash’

资源加版本号，那么我们的输出的部分都要做改动，并且要区分当前的命令模式，如下：

```
// webpack.config.js
...
output：{
    // 输出目录
    path: path.join(__dirname, 'dist'),

    // 发布后，资源的引用目录
    publicPath: PATHS.publicPath,

    // 文件名称
    filename: devServer ? 'js/[name].js' : 'js/[name]-[chunkhash:8].js',

    // 按需加载模块时输出的文件名称
    chunkFilename: devServer ? 'js/[name].js' : 'js/[name]-[chunkhash:8].js'
}
...
```

输出公共js的地方也要改动：

```
// webpack.config.js
...
plugins:[
    // 会把 ‘entry’ 定义的 common 对应的两个js 打包为 ‘common.js’
    new webpack.optimize.CommonsChunkPlugin("common", "" + (devServer ? 'js/[name].js' : "js/[name]-[chunkhash:8].js"), Infinity),
]
...
```

## 页面自动引入含有版本号的文件

有个版本号后，我们考虑如何通过html引用这些含有版本号的js、css、font、img。webpack每次编译后的资源 chunkhash 会随着内容的变化而变化，所以我们不可能每次都手动的更改html这些资源的引用路径。这时我们要用到webpack的插件：html-webpack-plugin。这个插件的目的是生成html,也可以根据模板生成html，当然还有其他的功能，具体看插件介绍。下面是的配置：

### 引入插件

```
// webpack.config.js
var HtmlWebpackPlugin = require('html-webpack-plugin');
```

### 配置 plugins，生成需要的html

```
// webpack.config.js
...
plugins:[
    new HtmlWebpackPlugin({
        filename: 'index.html',
        template: __dirname + '/src/index.html',
        inject: 'true'
    }),
    new HtmlWebpackPlugin({
        filename: 'html/hrm.html',
        template: __dirname + '/src/html/hrm.html',
        inject: false,
    }),
    new HtmlWebpackPlugin({
        filename: 'html/home.html',
        template: __dirname + '/src/html/home.html',
        inject: false,
    }),
]
...
```

### Configuration

可以进行一系列的配置，支持如下的配置信息:
- title: 用来生成页面的 title 元素
- filename: 输出的 HTML 文件名，默认是 index.html, 也可以直接配置带有子目录。
- template: 模板文件路径，支持加载器，比如 html!./index.html
- inject: true | 'head' | 'body' | false  ,注入所有的资源到特定的 template 或者 templateContent 中，如果设置为 true 或者 body，所有的 javascript 资源将被放置到 body 元素的底部，'head' 将放置到 head 元素中。
- favicon: 添加特定的 favicon 路径到输出的 HTML 文件中。
- minify: {} | false , 传递 html-minifier 选项给 minify 输出
- hash: true | false, 如果为 true, 将添加一个唯一的 webpack 编译 hash 到所有包含的脚本和 CSS 文件，对于解除 cache 很有用。
- cache: true | false，如果为 true, 这是默认值，仅仅在文件修改之后才会发布文件。
- showErrors: true | false, 如果为 true, 这是默认值，错误信息会写入到 HTML 页面中
- chunks: 允许只添加某些块 (比如，仅仅 unit test 块)
- chunksSortMode: 允许控制块在添加到页面之前的排序方式，支持的值：'none' | 'default' | {function}-default:'auto'

我们前面说过，webpack 默认只识别 js 文件，所以对于html也要使用对应的loader:

```
// webpack.config.js
...
loaders:[
     {test: /\.html$/,loader: "html"},
]
...
```

## 引用图片和字体
引用图片和字体，需要对应的loader,并且可以设置这些资源大小的临界值，当小于临界值的时候，字体或者图片文件会以base64的形式在html引用，否则则是以资源路径的形式引用。如下：

```
// webpack.config.js

// 图片 loader
{
    test: /\.(png|gif|jpe?g)$/,
    loader: 'url-loader',
    query: {
        /*
         *  limit=10000 ： 10kb
         *  图片大小小于10kb 采用内联的形式，否则输出图片
         * */
        limit: 10000,
        name: '/img/[name]-[hash:8].[ext]'
    }
},

// 字体loader
{
    test: /\.(eot|woff|woff2|ttf|svg)$/,
    loader: 'url-loader',
    query: {
        limit: 5000,
        name: '/font/[name]-[hash:8].[ext]'
    }
},
```

## 资源文件的压缩

js、css、html的压缩是少不了的，webpack 自带了压缩插件，如果某些对象名称不想被压缩，可以排除不想要压缩的对象名称。配置如下：

```
// webpack.config.js
...
plugins:[
    new webpack.optimize.UglifyJsPlugin({ 
            mangle: { // 排除不想要压缩的对象名称
                except: ['$super', '$', 'exports', 'require', 'module', '_']
            },
            compress: {
                warnings: false
            },
            output: {
                comments: false,
            }
        })
]
...
```

## 使用jquery、underscore

通过webpack编译输出后的项目中，虽然页面已经引用了jquery、underscore,但是还是无法直接使用‘$’、‘_’对象，我们可以这样：

```
var $ = require('jquery')；
var _ =  require('underscore')；
```

但是这样实在不方便，如果我们就是要使用‘$’、‘_’对象直接操作，webpack 内置的插件可以帮我们解决。具体如下：

```
// webpack.config.js
new webpack.ProvidePlugin({
        $: "jquery",
        jQuery: "jquery",
        "window.jQuery": "jquery",
        "_": "underscore",
    }),
```

## 代码分割，按需加载

在单页面应用中，当我们加载其他的模板文件时，想要引用这个模板文件对应的js。如果我们通过这种方式`require()`,那么webpack会将这个模板文件对应的js也会和当前js打包成一个js。如果项目比较大，那么js文件也将越来越大。我们希望的是加载模板文件的时候动态的引用这个模板文件对应的js。那么我们可以通过` require.ensure()`的方式。

比如现在有两个导航菜单：

```
<ul>
<li><a href="#home">home</a></li>
<li><a href="#hrm">HRM</a></li>
</ul>
```

我们给这两个菜单绑定点击事件，当点击‘home’时引用对应的‘home.js’;当点击‘HRM’时引用对应的‘hrm.js’,那么大致可以这样：

```
function loadJs(jsPath) {
    var currentMod;
    if (jsPath === './home') {
        require.ensure([], function (require) {
            currentMod = require('./home');
        }, 'home');
    }
    else if (jsPath === './hrm') {
        require.ensure([], function (require) {
            currentMod = require('./hrm');
        }, 'hrm');
    }
}

```

## 全局环境变量

有时我们只有在开发过程中，才想输出log日志。可以用以下webpack内置的插件解决：

```
// webpack.config.js
...
plugins:[
      new webpack.DefinePlugin({
        // 全局debug标识
        __DEV__: debug,
    }),

]
...
```

这时代码中就可以这么写了：

```
if (__DEV__) {
    console.log('debug 模式');
}
```

## 清空发布目录
发布前清空发布目录是有必要的，我们可以通过`clean-webpack-plugin`插件解决：

### 引入插件

```
// webpack.config.js
var CleanWebpackPlugin = require('clean-webpack-plugin');
```

### 配置plugins

```
// webpack.config.js
...
plugins:[
    new CleanWebpackPlugin(['dist'], {
        root: '', // An absolute path for the root  of webpack.config.js
        verbose: true,// Write logs to console.
        dry: false // Do not delete anything, good for testing.
    }),
]
...

```

## 热更新结合后端服务

### 热更新

热更新可以在你代码改变的时候即时编译输出，不用每次都要从都重新编译一遍，并且除了第一次编译比较慢，后面的编译都是增量编译，速度很快。有了这个功能，我们就不需要，每次都从头编译一次了。配置如下：

```
// webpack.config.js
...
plugins: [
        // Enable multi-pass compilation for enhanced performance
        // in larger projects. Good default.
        new webpack.HotModuleReplacementPlugin({
            multiStep: true
        }),
],
devServer: {
        // Enable history API fallback so HTML5 History API based
        // routing works. This is a good default that will come
        // in handy in more complicated setups.
        historyApiFallback: true,

        // Unlike the cli flag, this doesn't set
        // HotModuleReplacementPlugin!
        hot: true,
        inline: true,

        // Display only errors to reduce the amount of output.
        stats: 'errors-only',

        host: "localhost", // Defaults to `localhost`   process.env.HOST
        port: "8080",  // Defaults to 8080   process.env.PORT
}
...
```

这时我们只要打开浏览器，输入：`localhost:8080/ `就能看到结果，并且在你修改某些源文件后，浏览器会自动刷新，就能看到webpack 即时编译输出的结果，而不需要重新编译。

## 结合后端服务

我们在使用webpack开发时难免要结合后端服务开发，比如我们用webstorm 编译器开发项目，需要调用java的服务，由于有同源策略问题，这时我们会收到相关报错信息。这时我们可以通过代理的方式绕过同源策略。
这里我用nodejs 模拟一个后端服务，如下：

```
// ~/mockServer/server.js

var http = require('http');

var content = '▍if you see that,It means you have get the correct data by backend server(mock data by nodejs server)!';

var srv = http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'application/text'});
    res.end(content);
});

srv.listen(8888, function() {
    console.log('listening on localhost:8888');
});
```

接下来我们需要这样配置去调用这个nodejs 的服务。
首先将热更新配置的代码修改为：

```
// webpack.config.js
...
plugins: [
        // Enable multi-pass compilation for enhanced performance
        // in larger projects. Good default.
        new webpack.HotModuleReplacementPlugin({
            multiStep: true
        }),
],
devServer: {
        // Enable history API fallback so HTML5 History API based
        // routing works. This is a good default that will come
        // in handy in more complicated setups.
        historyApiFallback: true,

        // Unlike the cli flag, this doesn't set
        // HotModuleReplacementPlugin!
        hot: true,
        inline: true,

        // Display only errors to reduce the amount of output.
        stats: 'errors-only',

        host: "localhost", // Defaults to `localhost`   process.env.HOST
        port: "8080",  // Defaults to 8080   process.env.PORT
        proxy: {
                '/devApi/*': {
                    target: 'http://localhost:8888/',
                    secure: true,
                    /*
                     * rewrite 的方式扩展性更强，不限制服务的名称
                     * */
                    rewrite: function (req) {
                        req.url = req.url.replace(/^\/devApi/, '');
                    }
                }
        }
}
...
```

然后配置一个全局的环境变量，通过`DefinePlugin`：

```
// webpack.config.js
...
plugins: [
 new webpack.DefinePlugin({
        __DEVAPI__: devServer ? "/devApi/" : "''",
    }),
]
...

```

最后在调用服务的地方，只需要在调用地址前添加` __DEVAPI__`全局环境变量即可，如：

```
$.ajax({
        url: __DEVAPI__ + 'http://localhost:8888/',
        data: {},
        type: 'get',
        dataType: 'text',
        success: function (text) {}
    })

```

这样在热更新的模式下，当有`__DEVAPI__` 的地方就会自动识别为`/devApi/`，而这里会通过代理处理帮你重写掉，绕过同源策略。

## 自动打开浏览器

虽然以上的工作几乎已经满足我们对webpack的要求了，但是我们还想懒一点，想在热更新模式下，编译完成后自动打开浏览器。那么我们可以通过这个插件`open-browser-webpack-plugin`解决：

引用插件

```
// webpack.config.js
var OpenBrowserPlugin = require('open-browser-webpack-plugin');
```

配置插件，这个配置要根据项目的具体情况去配置：

```
// webpack.config.js
...
plugins: [
 new OpenBrowserPlugin({url: 'http://localhost:8080' + PATHS.publicPath + 'index.html'})
]
...
```

## 总结

以上就是这篇文章的主要内容，希望通过这篇文章能够给大家带来一些启发。如果有觉得哪里不对，或者不合理的地方，欢迎指出。其实webpack还有一个关于版本号的bug，不知道是不是有人解决了，如果有人已经解决了，还请分享。
