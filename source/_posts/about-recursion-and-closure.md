---
title: 递归与闭包的使用
date: 2017-02-05 05:12:47
tags: js
---

关于一道前端题目的思考：

> 写一个mul函数调用时将生成以下输出:

    console.log(mul(2)(3)(4)); // output : 24

    console.log(mul(4)(3)(4)); // output : 48

最简单的实现莫过于

```js
function mul(x) {
    return function(y) {
        return function(z) {
            return x * y * z
        }
    }
}

console.log(mul(2)(3)(4)) // => 24
```

然而这种写法必须调用次数，显然无法接受

如果想要实现更不定次数的调用，可以运用`闭包`的特性，使用`递归`来实现

<!--more-->

```js
// 注意箭头函数不能获取 arguments
function mul(x) {
    return function(y) {
        if (arguments.length) {
            return mul(x * y)
        } else {
            return x
        }
    }
}

console.log(mul(2)(3)(4)()) // => 24
```
不过这种实现，在取值前，必须再调用一次（不传参数）

想要避免这种情况，可以利用 ECMAScript 的隐式转换机制

```js
function mul(x) {
    const re = y => mul(x * y)
    re.valueOf = () => x
    return re
}

console.log(mul(2)(3)(4)) // => 24 (在 chrome 浏览器中)
```
对于 `mul` 函数返回的对象，在取值操作时，会自动调用自身的 [valueOf](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/valueOf) 方法
然而，需要注意的是，在 firefox, safari, node 等环境里，直接调用`console.log` 方法打印`mul(2)(3)(4)`并不会自动调用对象的 `valueOf` 方法

与柯里化的区别：

柯里化函数时，函数的入参个数，或者说，柯里化后的函数调用次数必须是**固定**的，而递归可以**不固定**

Ref :
- https://www.zhihu.com/question/54822257
- https://segmentfault.com/a/1190000005760112
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/valueOf
