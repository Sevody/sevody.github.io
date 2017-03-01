---
title: 一起来学学 Promise
date: 2016-10-06 16:03:43
tags: js
---

## 基本概念

Promise 是一种异步解决方案，比传统的回调和事件模型更加合理和强大。

抽象的说，Promise 是一个容器，里面保存着未来将会完成的事件。
具体来说，Promise 是一个对象，通过它可以获取异步操作的消息。

一般来说，Promise 有 3 种状态：

    * Pending （进行中）
    * Resolved / Fulfilled（已完成）
    * Rejected (已失败)

<!--more-->

## 基本用法

```js
let promise = new Promise((resolve, reject) => {
    // do something
    ...

    if(success){
        resolve(value)
    } else {
        reject(error)
    }
})

promise.then((value) => {
    console.log(value)
}, (error) => {
    console.log(error)
})
```

## Fetch

Promise 最常使用的场景之一莫过于处理 ajax 返回的结果。而新的 Fetch API 提供了一个获取资源的接口(包括跨网络)。
`fetch()` 函数接受一个参数——资源的路径。无论请求成功与否，它都返回一个 promise 对象，resolve 对应请求的 Response。

```js
fetch('/users.json')
  .then(function(response) {
    return response.json()
  }).then(function(json) {
    console.log('parsed json', json)
  }).catch(function(ex) {
    console.log('parsing failed', ex)
  })
```

## Ref

- http://es6.ruanyifeng.com/#docs/promise

