---
title: 通过 gitlab 和 npm 实现私有模块的发布和安装
date: 2016-12-02 19:53:48
tags:
---

## 问题背景

npm 是 Node 的模块管理器, 功能非常强大. npm 安装公有模块非常简单, 直接一个命令 `npm install 模块名字` 可以了, 之所以可以这样安装, 是因为模块直接发布在 npm 的公有仓库里的 ( npm publish ) , npm 直接从仓库通过网络获取到就好. 但是, 有时候我们不希望把代码直接发布到公有仓库中, 对于企业来说, 内部的非开源模块肯定是不需要发布到公有仓库中的. 所以我们就要寻找一种可以让 npm 安装私有模块的办法.

<!--more-->

## 备选方案

### 购买npm付费账号

  这是最简单的办法, 根据 [npm的价格方案](https://www.npmjs.com/pricing) , 只要是付费用户, 都可以下载和发布不限量的私有模块, 不过代码也是需要上传到npm的服务器的.

### 自建npm仓库

  这个方案的意思就是: 连仓库都是私有的, 模块自然是私有的. 不过这种方案也是需要构建成本的, 需要额外的机器. 具体实现方案可以参考阿里的[cnpm](http://blog.fens.me/nodejs-cnpm-npm/).

### 利用npm安装机制和git仓库

  `npm install` 支持 `npm install <git remote url>` , 其中 *git remote url* 的格式是 *<protocol>://[<user>[:<password>]@]<hostname>[:<port>][:/]<path>[#<commit-ish>]* . 也就是说我们可以通过 npm 直接安装 githug 或 bitbucket 上的代码. 需要注意的是必须确保安装这个私有模块的机器有访问这个私有模块的 git 仓库的权限, 换句话说: 这台机器的公钥必须添加到 git 仓库/账号中.



## gitlab 私有模块方案

由于 npm 支持 git 协议 , 在公司已经建有 gitlab 的情况下, 通过 gitlab 实现私有模块方案是最简单的.

发布私有模块只需直接把代码 push 到 gitlab 仓库, 而安装私有模块也只需要一条命令`git+ssh://git@git.gitlabdomain.com:Username/Repository#{branch|tag}` . 我们可以通过`#` 后面的`branch` 和 `tag` 来进行版本控制.



## 模块实现

如果我们编写的模块用到了 es6 或者 jsx 的语法，在发布之前需要通过 babel 转换成 es5 语法：

```bash
$ babel ./src/main.js -o index.js
```

当然，模块 export 的时候，最好遵循 [UMD 规范](https://github.com/umdjs/umd)

```js
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS-like
        module.exports = factory(require('jquery'));
    } else {
        // Browser globals (root is window)
        root.returnExports = factory(root.jQuery);
    }
}(this, function ($) {
    //    methods
    function myFunc(){};

    //    exposed public method
    return myFunc;
}));
```

为了不用手动处理这些事情，我们还可以利用 webpack 提供的 library 打包功能

```js
// webpack.config.js
...
output: {
    library: "MyLibrary",
    libraryTarget: "umd"
}
...
```
## 参考链接

[如何通过NPM安装私有模块](http://www.jianshu.com/p/7b8771743ee4)

[CNPM搭建私有的NPM服务](http://blog.fens.me/nodejs-cnpm-npm/)

[Install npm module from gitlab private repository](http://stackoverflow.com/questions/22988876/install-npm-module-from-gitlab-private-repository)

[webpack output-library](https://webpack.js.org/configuration/output/#output-library)





