---
title: 使用 webpack-dev-server 实现自动刷新
date: 2016-08-13 21:21:34
tags: webpack
---

## 问题背景

我们在开发调试 react 项目的时候，需要使用 webpack 打包并刷新浏览器后才能看到实际效果。为了减少这种重复性的人工操作，我们可以使用`webpack --watch` 来监听文件文件的改变。当我们更改了文件并保存后，webpack 会自动执行打包操作。然而，我们还是需要手动按`F5`(mac 下是`command + r`)来刷新浏览器，而且有时候我们在 webpack 编译完前就刷新了页面，结果没能看到我们预期的改变，于是我们会多次按刷新按钮，直到看到我们预期的改变。而这些在开发过程中的痛点，我们可以用 webpack-dev-server 来解决。

## 概述

webpack-dev-server 是一个小型的 **Node.js Express** 服务器，它使用 **webpack-dev-middleware** 来服务于 webpack 。当 webpack 编译后，它会把编译状态信息发送给浏览器页面，并执行内嵌的刷新脚本。

## 安装

在命令行运行下面的命令：

```bash
$ npm install -g webpack webpack-dev-server
```

## 配置

需要在 `webpack.config.js` 配置文件里加上`publicPath`参数

```javascript
// webpack.config.js
...
output: {
        path: './dist',
        filename: '[name].js',
        publicPath: '/dist' // 路径是项目静态资源的输出路径
    }
...
```

##  启动

在项目的根目录运行下面的命令：

```bash
$ webpack-dev-server --hot --inline
```

## 调试

打开`http://localhost:8080`进行开发调试。
当我们修改了 `index.jsx` 并保存后，webpack 会自动编译代码，并自动刷新浏览器。



## 参考

- http://webpack.github.io/docs/webpack-dev-server.html
