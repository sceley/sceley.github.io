---
title: 探索webpack
date: 2019-05-16 13:30:25
tags:
---

## 序言

webpack 是一个现代 JavaScript 应用程序的静态模块打包器。

<!-- more -->

## webpack的配置

```
const path = require('path');
module.exports = {
  entry: "./app/entry", // string | object | array
  // Webpack打包的入口
  output: {  // 定义webpack如何输出的选项
    path: path.resolve(__dirname, "dist"), // string
    // 所有输出文件的目标路径
    filename: "[chunkhash].js", // string
    // 「入口(entry chunk)」文件命名模版
    publicPath: "/assets/", // string
    // 构建文件的输出目录
    /* 其它高级配置 */
  },
  module: {  // 模块相关配置
    rules: [ // 配置模块loaders，解析规则
      {
        test: /\.jsx?$/,  // RegExp | string
        include: [ // 和test一样，必须匹配选项
          path.resolve(__dirname, "app")
        ],
        exclude: [ // 必不匹配选项（优先级高于test和include）
          path.resolve(__dirname, "app/demo-files")
        ],
        loader: "babel-loader", // 模块上下文解析
        options: { // loader的可选项
          presets: ["es2015"]
        },
      },
  },
  resolve: { //  解析模块的可选项
    modules: [ // 模块的查找目录
      "node_modules",
      path.resolve(__dirname, "app")
    ],
    extensions: [".js", ".json", ".jsx", ".css"], // 用到的文件的扩展
    alias: { // 模块别名列表
      "module": "new-module"
	  },
  },
  devtool: "source-map", // enum
  // 为浏览器开发者工具添加元数据增强调试
  plugins: [ // 附加插件列表
    // ...
  ],
}

```

### mode

webpack 提供 mode 配置选项，告知 webpack 使用相应模式的内置优化。

选项 | 描述
:--- | :---
development | 会将 process.env.NODE_ENV 的值设为 development。启用 NamedChunksPlugin 和 NamedModulesPlugin。
production | 会将 process.env.NODE_ENV 的值设为 production。启用 FlagDependencyUsagePlugin, FlagIncludedChunksPlugin, ModuleConcatenationPlugin, NoEmitOnErrorsPlugin, OccurrenceOrderPlugin, SideEffectsFlagPlugin 和 UglifyJsPlugin.

> mode 选项尚未设置，webpack 将回退到 production。 将 mode 选项设置为 development 或 production 以启用每个环境的默认值。您还可以将其设置为 none 以禁用任何默认行为。

### entry

#### 单个入口（简写）语法

用法：entry: string|Array<string>

```js
const config = {
  entry: './path/to/my/entry/file.js'
};

module.exports = config;
```

简化写法

```js
const config = {
  entry: {
    main: './path/to/my/entry/file.js'
  }
};
```

> 你向 entry 传入一个数组时会发生什么？向 entry 属性传入「文件路径(file path)数组」将创建“多个主入口(multi-main entry)”。


#### 对象语法

用法：entry: {[entryChunkName: string]: string|Array<string>}

```js
const config = {
  entry: {
    app: './src/app.js',
    vendors: './src/vendors.js'
  }
};
```

### output

用法

- filename 用于输出文件的文件名。
- 目标输出目录 path 的绝对路径。
- publicPath 资源引入链接的前缀(url=`${publicPath}/${filename}`)

### loader

loader 用于对模块的源代码进行转换。loader 可以使你在 import 或"加载"模块时预处理文件。loader 可以将文件从不同的语言（如 TypeScript）转换为 JavaScript，或将内联图像转换为 data URL。loader 甚至允许你直接在 JavaScript 模块中 import CSS文件！

配置

```js
module: {
    rules: [
        {
            test: /\.jsx?$/,
            use: [
                {
                    loader: "babel-loader",
                    options: {
                    	presets: ['env']
                    }
                }
            ]
        }
    ];
}
```
### module

#### webpack模块:

- ES2015 import 语句
- CommonJS require() 语句
- AMD define 和 require 语句
- css/sass/less 文件中的 @import 语句。
- 样式(url(...))或 HTML 文件(<img src=...>)中的图片链接(image url)

### plugins

webpack 插件是一个具有 apply 属性的 JavaScript 对象。apply 属性会被 webpack compiler 调用，并且 compiler 对象可在整个编译生命周期访问。

```js
const pluginName = 'ConsoleLogOnBuildWebpackPlugin';

class ConsoleLogOnBuildWebpackPlugin {
    apply(compiler) {
        compiler.hooks.run.tap(pluginName, compilation => {
            console.log("webpack 构建过程开始！");
        });
    }
}
```

## babel

### babel生态(eg. babel7)

- @babel/core
- @babel/preset-env : 包含 ES6、7 等版本的语法转化规则 
- @babel/plugin-transform-runtime : 避免 polyfill 污染全局变量，减小打包体积 
- @babel/polyfill : ES6 内置方法和函数转化垫片 
- babel-loader: 负责 ES6 语法转化

> 注意!!! 如果是用 babel7 来转译，需要安装 @babel/core、@babel/preset-env 和 @babel/plugin-transform-runtime，而不是 babel-core、babel-preset-env 和 babel-plugin-transform-runtime，它们是用于 babel6 的

> 使用 @babel/plugin-transform-runtime 的原因
> 默认情况下，这将添加到需要它的每个文件中。这种重复有时是不必要的，尤其是当你的应用程序分布在多个文件上的时候。
> transform-runtime 可以重复使用 Babel 注入的程序代码来节省代码，减小体积。

> 使用 @babel/polyfill 的原因
> Babel 默认只转换新的 JavaScript 句法（syntax），而不转换新的 API，比如 Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，以及一些定义在全局对象上的方法（比如 Object.assign）都不会转码。必须使用 @babel/polyfill，为当前环境提供一个垫片。

.babelrc 的文件配置 Babel

```json
{
  "presets": ["@babel/preset-env"],
  "plugins": ["@babel/plugin-transform-runtime"]
}
```

全局引入 @babel/polyfill

```js
// 全局引入
import '@babel/polyfill'

```

全局引入 @babel/polyfill 的这种方式可能会导入代码中不需要的 polyfill，从而使打包体积更大

更改 .babelrc，只转译我们使用到的

```js
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage"
      }
    ]
  ],
  "plugins": ["@babel/plugin-transform-runtime"]
}
```

同时，将全局引入这段代码注释掉

体积就减小了很多，但是更多的情况是我们并不确切的知道项目中引发兼容问题的具体原因，所以还是全局引入比较好

## 实战

### 编写loader

loader是一个函数，函数接受一个参数，参数是匹配test的文件的文件内容，return转换后的内容

loader把console.log('Hello World')转换为console.log('Change World');

```js
module.exports = (source) => {
    return source.replace(/hello/ig, 'Change');
};
```

配置webpack.config.js

```js
const path  = require('path');
const replaceLoader = path.join(__dirname, './loaders/replace-loader');
module.exports = {
    mode: 'development',
    entry: './src/index.js',
    module: {
        rules: [{
            test: /\.jsx?/,
            use: [replaceLoader]
        }]
    },
    output: {
        path: path.join(__dirname, './dist'),
        filename: '[chunkhash].bundle.js'
    }
};
```

### 编写plugin

plugin定义的是一个类

```js
module.exports = class CopyrightWebpackPlugin {
    constructor() {
        console.log("插件被使用了");
    }

    apply(compiler) {
        // compiler.plugin('emit', (compilation, cb) => {

        // });
        compiler.hooks.emit.tapAsync(
            "CopyrightWebpackPlugin",
            (compilation, cb) => {
                // 生成一个 copyright.txt 文件
                let text = "HelloWorld -> ChangeWorld";
                compilation.assets["copyright.txt"] = {
                    source() {
                        return text;
                    },
                    size() {
                        return text.length;
                    }
                };
                cb();
            }
        );
    }
};
```

在webpack.config.js配置使用，每个plugin都要new出一个实例放到plugins数组中去

```js
const path  = require('path');
const CopyrightWebpackPlugin = require('./plugins/copyright-webpack-plugin');
module.exports = {
    mode: 'development',
    entry: './src/index.js',
    output: {
        path: path.join(__dirname, './dist'),
        filename: '[chunkhash].bundle.js'
    },
    plugins: [
        new CopyrightWebpackPlugin()
    ]
}
```