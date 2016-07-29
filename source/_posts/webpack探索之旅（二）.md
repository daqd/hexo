---
title: Webpack探索之旅（二）
date: 2016-07-25 21:21:46
tags:
	- webpack
  - 前端模块化
categories: 前端工具
---
## 前言
Webpack提供了一套加载器，比如css-loader,less-loader,style-loader，url-loader等，用于将不同的文件加载到js文件中，比如url-loader用于在js中加载png/jpg格式的图片文件，css/style-loader用于加载css文件，less-loader加载器是将less编译成css文件。
<!--more-->
## 入口文件
入口文件是webpack在打包时候首先读取的文件。
例如，`main.js`是一个入口文件。
```
// main.js
document.write('<h1>Webpack Test!</h1>');
```
index.html
```
<html>
  <body>
    <script type="text/javascript" src="bundle.js"></script>
  </body>
</html>
```
通过webpack打包，最终打包为`bundle.js`，编辑`webpack.config.js`文件：
```
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  }
};
```
键入以下命令，启动服务, 打开浏览器访问 `http://127.0.0.1:8080` 
```
$ webpack-dev-server
```
## 多文件入口
webpack也可设置多个入口文件，比如，我们定义下面两个文件：
```
// main1.js
document.write('<h1>Webpack Test!</h1>');

// main2.js
document.write('<h2>Multiple Test!</h2>');
```
index.html
```
<html>
  <body>
    <script src="bundle1.js"></script>
    <script src="bundle2.js"></script>
  </body>
</html>
```
webpack.config.js
```
module.exports = {
  entry: {
    bundle1: './main1.js',
    bundle2: './main2.js'
  },
  output: {
    filename: '[name].js'
  }
};
```
## Babel-loader
转换器可将不同的资源进行转换，babel-loader的作用就是将JSX/ES6的文件转换为普通的JS文件。官方的文档有详细的loader列表及介绍（详见[官方](http://webpack.github.io/docs/list-of-loaders.html)）

`main.jsx` 是一个使用ES6编写的JSX文件。
```
const React = require('react');
const ReactDOM = require('react-dom');

ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.querySelector('#wrapper')
);
```
index.html
```
<html>
  <body>
    <div id="wrapper"></div>
    <script src="bundle.js"></script>
  </body>
</html>
```
webpack.config.js
```
module.exports = {
  entry: './main.jsx',
  output: {
    filename: 'bundle.js'
  },
  module: {
    loaders:[
      {
        test: /\.js[x]?$/,
        exclude: /node_modules/,
        loader: 'babel-loader?presets[]=es2015&presets[]=react'
      },
    ]
  }
};
```
在上面的配置文件中，`module.loaders`用来分配加载器，上面配置文件中的`babel-loader`需要安装`babel-preset-es2015`和`babel-preset-react`插件去转换ES6和React文件。也可通过下面的方式去设置这些匹配项：
```
module: {
  loaders: [
    {
      test: /\.jsx?$/,
      exclude: /node_modules/,
      loader: 'babel',
      query: {
        presets: ['es2015', 'react']
      }
    }
  ]
}
```
## CSS-loader
Webpack也允许在JS中引入CSS，需要安装CSS-loader转换器。
main.js
```
require('./app.css');
```
app.css
```
body {
  background-color: blue;
}
```
index.html
```
<html>
  <head>
    <script type="text/javascript" src="bundle.js"></script>
  </head>
  <body>
    <h1>Hello World</h1>
  </body>
</html>
```
webpack.config.js
```
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'
  },
  module: {
    loaders:[
      { test: /\.css$/, loader: 'style-loader!css-loader' },
    ]
  }
};
```
注：你需要两个转换器去转换CSS文件，css-loader去读取CSS文件，style-loader将CSS文件写入到html文件的style标签内。两个转换器中间用叹号连接。

启动服务之后，页面会出现定义的CSS文件样式。

```
<head>
  <script type="text/javascript" src="bundle.js"></script>
  <style type="text/css">
    body {
      background-color: blue;
    }
  </style>
</head>
```