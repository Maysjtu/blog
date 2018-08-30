[深入浅出Webpack](http://webpack.wuhaolin.cn/)



# 第2章 配置

第1章只是粗略讲解了 Webpack 的基础核心功能，本章会列举 Webpack 的常用功能所提供的配置选项，可以作为速查表使用。

配置 Webpack 的方式有两种：

1. 通过一个 JavaScript 文件描述配置，例如使用 `webpack.config.js` 文件里的配置；
2. 执行 Webpack 可执行文件时通过命令行参数传入，例如 `webpack --devtool source-map`。

这两种方式可以相互搭配，例如执行 Webpack 时通过命令 `webpack --config webpack-dev.config.js`指定配置文件，再去 `webpack-dev.config.js` 文件里描述部分配置。

按照**配置方式**来划分，可分为：

- 只能通过命令行参数传入的选项，这种最为少见；
- 只能通过配置文件配置的选项；
- 通过两种方式都可以配置的选项。

按照配置**所影响的功能**来划分，可分为：

- [2-1 Entry](http://webpack.wuhaolin.cn/2%E9%85%8D%E7%BD%AE/2-1Entry.html) 配置模块的入口；
- [2-2 Output](http://webpack.wuhaolin.cn/2%E9%85%8D%E7%BD%AE/2-2Output.html) 配置如何输出最终想要的代码；
- [2-3 Module](http://webpack.wuhaolin.cn/2%E9%85%8D%E7%BD%AE/2-3Module.html) 配置处理模块的规则；
- [2-4 Resolve](http://webpack.wuhaolin.cn/2%E9%85%8D%E7%BD%AE/2-4Resolve.html) 配置寻找模块的规则；
- [2-5 Plugins](http://webpack.wuhaolin.cn/2%E9%85%8D%E7%BD%AE/2-5Plugins.html) 配置扩展插件；
- [2-6 DevServer](http://webpack.wuhaolin.cn/2%E9%85%8D%E7%BD%AE/2-6DevServer.html) 配置 DevServer；
- [2-7 其它配置项](http://webpack.wuhaolin.cn/2%E9%85%8D%E7%BD%AE/2-7%E5%85%B6%E5%AE%83%E9%85%8D%E7%BD%AE%E9%A1%B9.html) 其它零散的配置项；
- [2-8 整体配置结构](http://webpack.wuhaolin.cn/2%E9%85%8D%E7%BD%AE/2-8%E6%95%B4%E4%BD%93%E9%85%8D%E7%BD%AE%E7%BB%93%E6%9E%84.html) 整体地描述各配置项的结构；
- [2-9 多种配置类型](http://webpack.wuhaolin.cn/2%E9%85%8D%E7%BD%AE/2-9%E5%A4%9A%E7%A7%8D%E9%85%8D%E7%BD%AE%E7%B1%BB%E5%9E%8B.html) 配置文件不止可以返回一个 Object，还有其他返回形式；
- [2-10 配置总结](http://webpack.wuhaolin.cn/2%E9%85%8D%E7%BD%AE/2-10%E9%85%8D%E7%BD%AE%E6%80%BB%E7%BB%93.html) 寻找配置 Webpack 的规律，减少思维负担。

# 2-1 Entry

`entry`是配置模块的入口，可抽象成输入，Webpack 执行构建的第一步将从入口开始搜寻及递归解析出所有入口依赖的模块。

`entry` 配置是**必填的**，若不填则将导致 Webpack 报错退出。

## context

Webpack 在寻找相对路径的文件时会以 `context` 为根目录，`context` 默认为执行启动 Webpack 时所在的当前工作目录。 如果想改变 `context` 的默认配置，则可以在配置文件里这样设置它：

```js
module.exports = {
  context: path.resolve(__dirname, 'app')
}
```

注意， `context` 必须是一个绝对路径的字符串。 除此之外，还可以通过在启动 Webpack 时带上参数 `webpack --context` 来设置 `context`。

之所以在这里先介绍 `context`，是因为 Entry 的路径和其依赖的模块的路径可能采用相对于 `context` 的路径来描述，`context` 会影响到这些相对路径所指向的真实文件。

## Entry 类型

Entry 类型可以是以下三种中的一种或者相互组合：

| 类型   | 例子                                                         | 含义                                 |
| ------ | ------------------------------------------------------------ | ------------------------------------ |
| string | `'./app/entry'`                                              | 入口模块的文件路径，可以是相对路径。 |
| array  | `['./app/entry1', './app/entry2']`                           | 入口模块的文件路径，可以是相对路径。 |
| object | `{ a: './app/entry-a', b: ['./app/entry-b1', './app/entry-b2']}` | 配置多个入口，每个入口生成一个 Chunk |

如果是 `array` 类型，则搭配 `output.library` 配置项使用时，只有数组里的最后一个入口文件的模块会被导出。

## Chunk 名称

Webpack 会为每个生成的 Chunk 取一个名称，Chunk 的名称和 Entry 的配置有关：

- 如果 `entry` 是一个 `string` 或 `array`，就只会生成一个 Chunk，这时 Chunk 的名称是 `main`；
- 如果 `entry` 是一个 `object`，就可能会出现多个 Chunk，这时 Chunk 的名称是 `object` 键值对里键的名称。



## 配置动态 Entry

假如项目里有多个页面需要为每个页面的入口配置一个 Entry ，但这些页面的数量可能会不断增长，则这时 Entry 的配置会受到到其他因素的影响导致不能写成静态的值。其解决方法是把 Entry 设置成一个函数去动态返回上面所说的配置，代码如下：

```js
// 同步函数
entry: () => {
  return {
    a:'./pages/a',
    b:'./pages/b',
  }
};
// 异步函数
entry: () => {
  return new Promise((resolve)=>{
    resolve({
       a:'./pages/a',
       b:'./pages/b',
    });
  });
};
```



# 2-2 Output

`output` 配置如何输出最终想要的代码。`output` 是一个 `object`，里面包含一系列配置项，下面分别介绍它们。

###  filename

`output.filename` 配置输出文件的名称，为string 类型。 如果只有一个输出文件，则可以把它写成静态不变的：

```
filename: 'bundle.js'
```

但是在有多个 Chunk 要输出时，就需要借助模版和变量了。前面说到 Webpack 会为每个 Chunk取一个名称，可以根据 Chunk 的名称来区分输出的文件名：

```
filename: '[name].js'

```

 代码里的 `[name]` 代表用内置的 `name` 变量去替换`[name]`，这时你可以把它看作一个字符串模块函数， 每个要输出的 Chunk 都会通过这个函数去拼接出输出的文件名称。

| 变量名    | 含义                       |
| --------- | -------------------------- |
| id        | Chunk 的唯一标识，从0开始  |
| name      | Chunk 的名称               |
| hash      | Chunk 的唯一标识的 Hash 值 |
| chunkhash | Chunk 内容的 Hash 值       |

其中 `hash` 和 `chunkhash` 的长度是可指定的，`[hash:8]` 代表取8位 Hash 值，默认是20位。

> 注意 [ExtractTextWebpackPlugin](https://github.com/webpack-contrib/extract-text-webpack-plugin) 插件是使用 `contenthash` 来代表哈希值而不是 `chunkhash`， 原因在于 ExtractTextWebpackPlugin 提取出来的内容是代码内容本身而不是由一组模块组成的 Chunk。

## chunkFilename

`output.chunkFilename` 配置无入口的 Chunk 在输出时的文件名称。 chunkFilename 和上面的 filename 非常类似，但 chunkFilename 只用于指定在运行过程中生成的 Chunk 在输出时的文件名称。 常见的会在运行时生成 Chunk 场景有在使用 CommonChunkPlugin、使用 `import('path/to/module')` 动态加载等时。 chunkFilename 支持和 filename 一致的内置变量。

## path

`output.path` 配置输出文件存放在本地的目录，必须是 string 类型的绝对路径。通常通过 Node.js 的 `path` 模块去获取绝对路径：

```
path: path.resolve(__dirname, 'dist_[hash]')
```

## publicPath

在复杂的项目里可能会有一些构建出的资源需要异步加载，加载这些异步资源需要对应的 URL 地址。

`output.publicPath` 配置发布到线上资源的 URL 前缀，为string 类型。 默认值是空字符串 `''`，即使用相对路径。

这样说可能有点抽象，举个例子，需要把构建出的资源文件上传到 CDN 服务上，以利于加快页面的打开速度。配置代码如下：

```
filename:'[name]_[chunkhash:8].js'
publicPath: 'https://cdn.example.com/assets/'

```

这时发布到线上的 HTML 在引入 JavaScript 文件时就需要：

```
<script src='https://cdn.example.com/assets/a_12345678.js'></script>

```

使用该配置项时要小心，稍有不慎将导致资源加载404错误。

`output.path` 和 `output.publicPath` 都支持字符串模版，内置变量只有一个：`hash` 代表一次编译操作的 Hash 值。

## crossOriginLoading

 Webpack 输出的部分代码块可能需要异步加载，而异步加载是通过 [JSONP](https://zh.wikipedia.org/wiki/JSONP) 方式实现的。 JSONP 的原理是动态地向 HTML 中插入一个 `<script src="url"></script>` 标签去加载异步资源。`output.crossOriginLoading` 则是用于配置这个异步插入的标签的 `crossorigin` 值。

script 标签的 crossorigin 属性可以取以下值：

- `anonymous`(默认) 在加载此脚本资源时不会带上用户的 Cookies；
- `use-credentials` 在加载此脚本资源时会带上用户的 Cookies。

通常用设置 crossorigin 来获取异步加载的脚本执行时的详细错误信息。

## libraryTarget 和 library

当用 Webpack 去构建一个可以被其他模块导入使用的库时需要用到它们。

- `output.libraryTarget` 配置以何种方式导出库。
- `output.library` 配置导出库的名称。

它们通常搭配在一起使用。

`output.libraryTarget` 是字符串的枚举类型，支持以下配置。

### var (默认)

编写的库将通过 `var` 被赋值给通过 `library` 指定名称的变量。

假如配置了 `output.library='LibraryName'`，则输出和使用的代码如下：

```
// Webpack 输出的代码
var LibraryName = lib_code;

// 使用库的方法
LibraryName.doSomething();

```

假如 `output.library` 为空，则将直接输出：

```
lib_code

```

> 其中 `lib_code` 代指导出库的代码内容，是有返回值的一个自执行函数。

### commonjs

编写的库将通过 CommonJS 规范导出。

假如配置了 `output.library='LibraryName'`，则输出和使用的代码如下：

```
// Webpack 输出的代码
exports['LibraryName'] = lib_code;

// 使用库的方法
require('library-name-in-npm')['LibraryName'].doSomething();
```

> 其中 `library-name-in-npm` 是指模块发布到 Npm 代码仓库时的名称。 

### commonjs2

编写的库将通过 CommonJS2 规范导出，输出和使用的代码如下：

```js
// Webpack 输出的代码
module.exports = lib_code;

// 使用库的方法
require('library-name-in-npm').doSomething();
```

CommonJS2 和 CommonJS 规范很相似，差别在于 CommonJS 只能用 `exports` 导出，而 CommonJS2 在 CommonJS 的基础上增加了 `module.exports` 的导出方式。

在 `output.libraryTarget` 为 `commonjs2` 时，配置 `output.library` 将没有意义。

### this

编写的库将通过 `this` 被赋值给通过 `library` 指定的名称，输出和使用的代码如下：

```js
// Webpack 输出的代码
this['LibraryName'] = lib_code;

// 使用库的方法
this.LibraryName.doSomething();
```

### window

编写的库将通过 `window` 被赋值给通过 `library` 指定的名称，即把库挂载到 `window` 上，输出和使用的代码如下：

```js
// Webpack 输出的代码
window['LibraryName'] = lib_code;

// 使用库的方法
window.LibraryName.doSomething();
```

### global

编写的库将通过 `global` 被赋值给通过 `library` 指定的名称，即把库挂载到 `global` 上，输出和使用的代码如下：

```js
// Webpack 输出的代码
global['LibraryName'] = lib_code;

// 使用库的方法
global.LibraryName.doSomething();
```



......

## Externals

Externals 用来告诉 Webpack 要构建的代码中使用了哪些不用被打包的模块，也就是说这些模版是外部环境提供的，Webpack 在打包时可以忽略它们。

有些 JavaScript 运行环境可能内置了一些全局变量或者模块，例如在你的 HTML HEAD 标签里通过以下代码：

```
<script src="path/to/jquery.js"></script>
```

引入 jQuery 后，全局变量 `jQuery` 就会被注入到网页的 JavaScript 运行环境里。

如果想在使用模块化的源代码里导入和使用 jQuery，可能需要这样：

```
import $ from 'jquery';
$('.my-element');
```

构建后你会发现输出的 Chunk 里包含的 jQuery 库的内容，这导致 jQuery 库出现了2次，浪费加载流量，最好是 Chunk 里不会包含 jQuery 库的内容。

Externals 配置项就是为了解决这个问题。

通过 `externals` 可以告诉 Webpack JavaScript 运行环境已经内置了那些全局变量，针对这些全局变量不用打包进代码中而是直接使用全局变量。 要解决以上问题，可以这样配置 `externals`：

```js
module.export = {
  externals: {
    // 把导入语句里的 jquery 替换成运行环境里的全局变量 jQuery
    jquery: 'jQuery'
  }
}
```

...









