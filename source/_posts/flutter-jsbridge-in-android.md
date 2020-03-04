---
title: Flutter webview navigation delegate for subframes
date: 2020-03-04 10:15:23
tags:
---

## 问题
安卓的 Flutter APP 在 webview 容器内无法使用 jsbridge

## 详细描述
在 Flutter APP 里，通过 `webview_flutter` 库创建 webview 容器，加载 H5 页面，通过 `navigationDelegate` 接口拦截 H5 页面的 url 跳转来实现 H5 跟 Flutter APP 的通信。但是，因为一般的 jsbridge 底层是使用生成 iframe 的方式来加载一个自定义协议来跟 APP 通信，这个在安卓上存在问题：Flutter 的 webview 容器没办法拦截 iframe 请求。

<!--more-->

## 底层原因
要理解为什么会出现这个问题，我们首先要了解一些背景。在安卓底层，有个叫 `shouldOverrideUrlLoading` 的方法，主要是给 webview 提供时机，让其选择是否对 urlLoading 进行拦截。如果这个接口的返回值是 `true`，则拦截 webview 加载 url，如果是 `false` 则允许 webview 加载 url。

我们先来看下正常情况下，安卓的 webview 自定义拦截是怎么做的，下面是安卓 APP 对自定义协议请求的拦截：

![](https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/15813952216851.jpg)

可以看到，如果检测到 url 带有自定义协议标识（ `BridgeUtil.YY_RETURN_DATA` 或者 `BridgeUtil.YY_OVERRIDE_SCHEMA`），`shouldOverrideUrlLoading` 就会返回 true 来拦截 url 加载.

我们再来看下 `webview_flutter` 库是怎么实现 `shouldOverrideUrlLoading` 的方法的：

![](https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/15813956028660.jpg)

可以看出，`shouldOverrideUrlLoading` 是否返回 true，取决于当前请求是否是 mainFrame 请求，而 jsbridge 使用的是生成 iframe 的方式也就是 subframe 请求，自然就没办法拦截 jsbridge 的 url 通信。

那为什么 `webview_flutter` 库的 `shouldOverrideUrlLoading` 只拦截 mainFrame 请求呢？

在安卓上， `shouldOverrideUrlLoading` 是跑在 platform thread 的方法，它必须 **synchronously** 同步返回结果，而安卓底层跟 Flutter 之间的通信需要通过 methodChannel 进行 **asynchronously** 异步通信。所以，如果是 mainFrame 的请求 （或者说通过 location.href = 'xxx' 请求），`shouldOverrideUrlLoading` 方法可以先拦截，然后异步分发到 Flutter 的`navigationDelegate` 接口，让用户代码来判断是否自己处理这个请求，如果用户不处理这个请求，安卓底层就通过 `WebView.loadUrl(String)` 重新让当前 webview 加载这个 url。可是如果是 subframe 的请求（也就是生成 iframe 的方式请求），一旦安卓拦截了这个请求，而用户又不需要自己处理这个请求，安卓底层也没办法通过类似 `WebView.loadUrl` 的方式来重载这类 subframe 请求。所以，这就决定了，在 Flutter APP 上，webview 容器没办法拦截 jsbridge 的 iframe 请求。

## 解决思路

1. Flutter 的 webview 容器没办法拦截 iframe 请求主要原因是没能在安卓层就确定是否要拦截 subframe 的请求，所以可以修改 `webview_flutter` 库安卓部分的代码，在 `shouldOverrideUrlLoading` 中拦截 url 为自定义协议的请求，并返回给 Flutter 的 `navigationDelegate` 方法。这种解决办法的主要问题是，需要自己维护一份 `webview_flutter` 库，想要合并官方的更新，就需要进行手动合并，不大好管理。

2. 改造 jsbridge，放弃生成 iframe 的通信方式，检测到当前的 webview 环境是 Flutter webview 就使用 window.location.href = 'xxx' 的方式进行通信。问题是使用 window.location.href 重载后 js 引擎是否还会执行后面的代码有待验证。

3. 通过使用 javascriptChannels 另外建立一个 channel 发送消息。那样的话就不再依赖于 `webview_flutter` 的 `navigationDelegate` 方法进行拦截，旧有的 jsbridge 协议需要改造，同时整个 jsbridge 库也需要重新设计，如果还要兼容多个 APP 的 jsbridge 调用，处理起来会比较困难。