---
title: call 和 bind 的关系
date: 2016-09-20 21:01:17
tags: js
---

我们写 js 代码的时候，有时会把 this 传入其他的函数作用域来获取需要的对象属性。

- 有时候会用到 `call`
比如：

```js
class A {
    constructor() {
        this.value = 'a'
    }
}

const print = function() {
    console.log(this.value)
}

const a = new A()
print.call(a) // => a
```

- 有时候会用到 `bind`
比如：

```js
class A {
    constructor() {
        this.value = 'a'
    }
}

const print = function() {
    console.log(this.value)
}

const a = new A()
const print_a = print.bind(a)
print_a() // => a
```

既然 `call` 和 `bind` 都会替换函数作用域的 this，那使用它们传入不同的 this，最后函数使用的是哪个 this 呢？如果传入 undefined 又会怎样呢？

<!--more-->

下面来做下实验：

#### 1. 只使用 `call`，不使用 `bind` 的情况：

```js
const value = 'global'
class A {
    constructor() {
        this.value = 'A'
    }
    printA(){
        console.log(this.value)
    }
    child(){
        const printA = this.printA
        return class B {
            constructor(){
                this.value = 'B'
            }
            printB(){
                printA.call(this)
            }
        }
    }
}
var a = new A()
var b = new (a.child())
b.printB() // => B
```

#### 2. 同时使用 `call` 和 `bind` 的情况：

```js
const value = 'global'
class A {
    constructor() {
        this.value = 'A'
    }
    printA(){
        console.log(this.value)
    }
    child(){
        const printA = this.printA.bind(this) // 这里使用了 bind
        return class B {
            constructor(){
                this.value = 'B'
            }
            printB(){
                printA.call(this)
            }
        }
    }
}
var a = new A()
var b = new (a.child())
b.printB() // => A
```

#### 3. 使用 `call` 传入 undefined 的情况：

```js
const value = 'global'
class A {
    constructor() {
        this.value = 'A'
    }
    printA(){
        console.log(this.value)
    }
    child(){
        const printA = this.printA
        return class B {
            constructor(){
                this.value = 'B'
            }
            printB(){
                printA.call(undefined) // 这里传入 undefined
            }
        }
    }
}
var a = new A()
var b = new (a.child())
b.printB() //=> Uncaught TypeError: Cannot read property 'value' of undefined
```

由上面的实验可知：`bind` 绑定的对象覆盖了 `call` 传入的 this.

为什么会这样呢？这时候就需要祭出 **[ecma-262 标准](http://www.ecma-international.org/ecma-262/6.0/#sec-function.prototype.call)** 了。

![](https://phantom-1256176746.cos.ap-chengdu.myqcloud.com/14883585623983.jpg)

由上图可以看到，调用 `call` 时，如果函数是箭头函数或者已经使用 `bind` 绑定了 this，那 `call` 传入的 this 就会被忽略掉。

### 总结

**`bind` 绑定的 this 会覆盖 `call` 传入的 this，`call` 传入 null | undefined 时，this 仍是  null | undefined 不会自动转换成全局对象。**

*遇到问题，记得多~~听长者的话~~看看标准*
