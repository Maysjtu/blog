[深入浅出Webpack](http://webpack.wuhaolin.cn/)



# 3-1 使用 ES6 语言

虽然目前部分浏览器和 Node.js 已经支持 ES6，但由于它们对 ES6 所有的标准支持不全，这导致在开发中不敢全面地使用 ES6。

通常我们需要把采用 ES6 编写的代码转换成目前已经支持良好的 ES5 代码，这包含2件事：

1. 把新的 ES6 语法用 ES5 实现，例如 ES6 的 `class` 语法用 ES5 的 `prototype` 实现。
2. 给新的 API 注入 polyfill ，例如项目使用 `fetch` API 时，只有注入对应的 polyfill 后，才能在低版本浏览器中正常运行。

## Babel

[Babel](https://babeljs.io/) 可以方便的完成以上2件事。 Babel 是一个 JavaScript 编译器，能将 ES6 代码转为 ES5 代码，让你使用最新的语言特性而不用担心兼容性问题，并且可以通过插件机制根据需求灵活的扩展。 在 Babel 执行编译的过程中，会从项目根目录下的 `.babelrc` 文件读取配置。`.babelrc` 是一个 JSON 格式的文件，内容大致如下：

```js
{
  "plugins": [
    [
      "transform-runtime",
      {
        "polyfill": false
      }
    ]
   ],
  "presets": [
    [
      "es2015",
      {
        "modules": false
      }
    ],
    "stage-2",
    "react"
  ]
}
```

### Plugins

`plugins` 属性告诉 Babel 要使用哪些插件，插件可以控制如何转换代码。

以上配置文件里的 `transform-runtime` 对应的插件全名叫做 `babel-plugin-transform-runtime`，即在前面加上了 `babel-plugin-`，要让 Babel 正常运行我们必须先安装它：

```
npm i -D babel-plugin-transform-runtime
```

`babel-plugin-transform-runtime` 是 Babel 官方提供的一个插件，作用是减少冗余代码。 Babel 在把 ES6 代码转换成 ES5 代码时通常需要一些 ES5 写的辅助函数来完成新语法的实现，例如在转换 `class extent` 语法时会在转换后的 ES5 代码里注入 `_extent` 辅助函数用于实现继承：

 ```js
function _extent(target) {
  for (var i = 1; i < arguments.length; i++) {
    var source = arguments[i];
    for (var key in source) {
      if (Object.prototype.hasOwnProperty.call(source, key)) {
        target[key] = source[key];
      }
    }
  }
  return target;
}
 ```

这会导致每个使用了 `class extent` 语法的文件都被注入重复的`_extent` 辅助函数代码，`babel-plugin-transform-runtime` 的作用在于不把辅助函数内容注入到文件里，而是注入一条导入语句：

```js
var _extent = require('babel-runtime/helpers/_extent');
```

这样能减小 Babel 编译出来的代码的文件大小。

同时需要注意的是由于 `babel-plugin-transform-runtime` 注入了 `require('babel-runtime/helpers/_extent')` 语句到编译后的代码里，需要安装 `babel-runtime` 依赖到你的项目后，代码才能正常运行。 也就是说 `babel-plugin-transform-runtime` 和 `babel-runtime` 需要配套使用，使用了 `babel-plugin-transform-runtime` 后一定需要 `babel-runtime`。

### Presets

...





# 3-9 为单页应用生成 HTML

## 引入问题

在[3-6 使用 React 框架](http://webpack.wuhaolin.cn/3%E5%AE%9E%E6%88%98/3-6%E4%BD%BF%E7%94%A8React%E6%A1%86%E6%9E%B6.html)中，是用最简单的 `Hello,Webpack` 作为例子让大家理解， 这个例子里因为只输出了一个 `bundle.js` 文件，所以手写了一个 `index.html` 文件去引入这个 `bundle.js`，才能让应用在浏览器中运行起来。

在实际项目中远比这复杂，一个页面常常有很多资源要加载。接下来举一个实战中的例子，要求如下：

1. 项目采用 ES6 语言加 React 框架。
2. 给页面加入 [Google Analytics](https://analytics.google.com/analytics/web/)，这部分代码需要内嵌进 HEAD 标签里去。
3. 给页面加入 [Disqus](https://disqus.com/) 用户评论，这部分代码需要异步加载以提升首屏加载速度。
4. 压缩和分离 JavaScript 和 CSS 代码，提升加载速度。

在开始前先来看看该应用最终发布到线上的代码：

 ```html
<html>
<head>
  <meta charset="UTF-8">
  <!--注入 Chunk app 依赖的 CSS-->
  <style rel="stylesheet">h1{color:red}</style>
  <!--内嵌 google_analytics 中的 JavaScript 代码-->
  <script>
(function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
})(window,document,'script','https://www.google-analytics.com/analytics.js','ga');
ga('create', 'UA-XXXXX-Y', 'auto');
ga('send', 'pageview');
  </script>
  <!--异步加载 Disqus 评论-->
  <script async="" src="https://dive-into-webpack.disqus.com/embed.js"></script>
</head>
<body>
<div id="app"></div>
<!--导入 app 依赖的 JS-->
<script src="app_746f32b2.js"></script>
<!--Disqus 评论容器-->
<div id="disqus_thread"></div>
</body>
</html>
 ```

> HTML 应该是被压缩过的，这里为了方便大家阅读而格式化了 HTML，并且加入了注释。

构建出的目录结构为：

 ```js
dist
├── app_792b446e.js
└── index.html
 ```

可以看到部分代码被内嵌进了 HTML 的 HEAD 标签中，部分文件的文件名称被打上根据文件内容算出的 Hash 值，并且加载这些文件的 URL 地址也被正常的注入到了 HTML 中。 如果你还采用手写 `index.html` 文件去完成以上要求，这就会使工作变得复杂、易错，项目难以维护。 本节教你如何自动化的生成这个符合要求的 `index.html`。

## 解决方案

推荐一个用于方便的解决以上问题的 Webpack 插件 [web-webpack-plugin](https://github.com/gwuhaolin/web-webpack-plugin)。 该插件已经被社区上许多人使用和验证，解决了大家的痛点获得了很多好评，下面具体介绍如何用它来解决上面的问题。

首先，修改 Webpack 配置为如下：

```js
const path = require('path');
const UglifyJsPlugin = require('webpack/lib/optimize/UglifyJsPlugin');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const DefinePlugin = require('webpack/lib/DefinePlugin');
const { WebPlugin } = require('web-webpack-plugin');

module.exports = {
  entry: {
    app: './main.js'// app 的 JavaScript 执行入口文件
  },
  output: {
    filename: '[name]_[chunkhash:8].js',// 给输出的文件名称加上 Hash 值
    path: path.resolve(__dirname, './dist'),
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: ['babel-loader'],
        // 排除 node_modules 目录下的文件，
        // 该目录下的文件都是采用的 ES5 语法，没必要再通过 Babel 去转换
        exclude: path.resolve(__dirname, 'node_modules'),
      },
      {
        test: /\.css/,// 增加对 CSS 文件的支持
        // 提取出 Chunk 中的 CSS 代码到单独的文件中
        use: ExtractTextPlugin.extract({
          use: ['css-loader?minimize'] // 压缩 CSS 代码
        }),
      },
    ]
  },
  plugins: [
    // 使用本文的主角 WebPlugin，一个 WebPlugin 对应一个 HTML 文件
    new WebPlugin({
      template: './template.html', // HTML 模版文件所在的文件路径
      filename: 'index.html' // 输出的 HTML 的文件名称
    }),
    new ExtractTextPlugin({
      filename: `[name]_[contenthash:8].css`,// 给输出的 CSS 文件名称加上 Hash 值
    }),
    new DefinePlugin({
      // 定义 NODE_ENV 环境变量为 production，以去除源码中只有开发时才需要的部分
      'process.env': {
        NODE_ENV: JSON.stringify('production')
      }
    }),
    // 压缩输出的 JavaScript 代码
    new UglifyJsPlugin({
      // 最紧凑的输出
      beautify: false,
      // 删除所有的注释
      comments: false,
      compress: {
        // 在UglifyJs删除没有用到的代码时不输出警告
        warnings: false,
        // 删除所有的 `console` 语句，可以兼容ie浏览器
        drop_console: true,
        // 内嵌定义了但是只用到一次的变量
        collapse_vars: true,
        // 提取出出现多次但是没有定义成变量去引用的静态值
        reduce_vars: true,
      }
    }),
  ],
};
```



















