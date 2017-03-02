---
title: react 模块仓库基础设施
date: 2017-02-28 20:21:57
tags:
---

最近几年，前端的工具层出不穷，**grunt**、**gulp**、**browserify**、**webpack**、**autoprefix**...
前端er们为了~~偷懒~~前端工程化也是拼了（> _ <）

我在编写和维护 react 模块的时候，也尝试过不少优化工作流的工具和配置。于是把比较基础的通用配置给总结下，方便在新的项目参考使用。

<!--more-->

## 使用 webpack 自动打包

### 基本配置

下面是使用 webpack 打包的基本配置

```javascript
// webpack.config.babel.js
import webpack from 'webpack'
import path from 'path'
const commonsPlugin = webpack.optimize,CommonsChunkPlugin({name: 'common', minChunks: 2})

export default {
    entry: './app.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js',
        publicPath: 'assets'
    }
    module: {
        loader: [
            {test: /\.jsx?$/, exclude: /node_module/, loader: 'babel-loader'},
            {test: /\.less$/, exclude: /node_module/, loader: 'style!css!less'},
            {test: /\.(png|jpg|svg)$/, exclude: /node_module/, loader: 'url-loader'}
        ]
    },
    plugins: [commonsPlugin]
    resolve: {
        extensions: ['js', 'jsx', 'less']
    }
}
```

注意，这里 webpack 配置的文件名多加了个 babel 后缀，是因为 webpack 会根据 webpack.config[.loader].js 的命令规则，会先使用文件名声明的 loader 来处理配置文件。那样，config 文件就可以使用 es6 语法了。

```javascript
// .babelrc
{
  "presets": [ "es2015", "stage-0", "react"]
}
```

上面的 babel 配置可以让我们的项目使用 es6 和 jsx 语法。


### 热更新 & 自动刷新

为了~~节省换键盘的钱~~少按 F5 / Command + r , 我们最好配置代码的热更新和浏览器自动刷新。这样，当我们改完一行代码并保存后，就能马上在浏览器上看到结果。

全局安装 `webpack-dev-server`

```js
// webpack.config.js

...
output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js',
        publicPath: 'http://localhost:8080/'
    }
...
```

命令行运行： `webpack-dev-server --hot --inline`

之后，打开 `http://localhost:8080` 就能看到我们的页面啦o(￣▽￣)""

### sourceMap

sourceMap 能让我们调试的时候能直接通过源代码调试

- less sourceMap:

```js
// webpack.config.js
...
{ test: /\.less$/, loader: 'style!css?sourceMap!less-loader?sourceMap=true' }
...
```

- jsx sourceMap:

    - `webpack-dev-server` 加上 `-d` 参数

    ```bash
    webpack-dev-server --hot --inline -d
    ```

    - 或者修改 `webpack.config.js`

    ```js
    // webpack.config.js
    ...
    devtool: "inline-source-map",
    ...
    ```

### autoprefix

配置了 autoprefix，再也不用手动敲 `-webkit-` 啦

```js
// webpack.config.js
...
import autoprefixer from 'autoprefixer';
...

...
{test: /\.less$/, loader: "style!css?sourceMap!postcss!less?sourceMap=true"}
...

...
postcss: [
    autoprefixer({
        browsers: ['last 5 versions', '> 1%']
    })
],
...
```

### 分离开发环境和生产环境

*nix:

`export NODE_ENV=prod && npm run webpack`

windows:

`set NODE_ENV=prod && npm run webpack`

之后在 `webpack.config.js` 中，通过 `process.env.NODE_ENV` 来读取环境变量

为了不用手动处理平台的区别，更简单的方式是使用 [better-npm-run](https://www.npmjs.com/package/better-npm-run) 来设置 NODE_ENV

### webpack 性能优化

- 配置 alias

```js
// webpack.config.js
...
resolve: {
    extensions: ['', '.js', '.jsx', '.less', '.css'],
    alias: {
        'react': path.join(nodeModulesPath, 'react/react.js'),
        'react-dom': path.join(nodeModulesPath, 'react-dom/dist/react-dom.js')
    }
}
...
```

- happypack
> https://segmentfault.com/a/1190000007891318#articleHeader6

- 配置 externals
> https://segmentfault.com/a/1190000007891318#articleHeader4

- dllPlugin
> https://segmentfault.com/a/1190000007891318#articleHeader5

## Eslint

### 安装
> https://www.npmjs.com/package/eslint-config-airbnb

### 命令行执行 lint

`node node_modules/eslint/bin/eslint.js src; exit 0`

> exit 0 是为了不让 npm 输入 exit 1 的错误
> https://github.com/eslint/eslint/issues/2409

### 配置 Sublime 自动校验
> https://segmentfault.com/a/1190000008068415

### Style Guide

- [airbnb style](https://github.com/airbnb/javascript)

- [eslint rule details](http://eslint.org/docs/rules)

## 制作 library

### webpack 配置

> https://webpack.js.org/guides/author-libraries
> 需要注意的是不能使用 CommonsPlugin

### 批量 export 组件

利用 require.context 读取组件目录下的所有文件

```js
const req = require.context('.', true, /.*\.js/);

req.keys().forEach((mod) => {
  const v = req(mod);
  const match = mod.match(/\.\/([\w]+)\//);
  if (match && match[1] && match[1]) {
      exports[match[1]] = v.default;
  }
});
```

> https://webpack.js.org/guides/dependency-management/#require-context

## 单元测试
TODO

## Example

- https://github.com/Sevody/re-tabs
- https://github.com/Sevody/re-feedback

