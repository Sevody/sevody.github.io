---
title: 深入理解函数对象
date: 2017-06-24 19:43:13
tags: js
---
## 起

由于工作的需要，最近同时学习 go 和 vue。当结合着书上的教程，写了一段时间的 go 代码后，再看回 js 创建 vue 函数对象的代码，突然感觉一脸懵逼(´⊙ω⊙`)。

<!--more-->


在 go 里，对象封装是通过 struct 实现的：

```go
type Idol struct {
  Name string
  Age int
}

func (idol *Idol) sayHello() {
  fmt.Printf("私は%s, %d歳です。", idol.Name, idol.Age)
}

func main() {
  kotori := &Idol{Name:"南小鳥", Age: 17}
	kotori.sayHello()
}
```

嗯，这很好理解，把 struct 想象成 Object 就是了。

然后回到 js :
```js
Vue.config.devtools = true

Vue.component('my-component', {
  // 选项
})

var vm = new Vue({
  el: '#some-element',
  // 选项
})

vm.$watch(
  function () {
    // do something
  }
)
```

什么鬼？Vue 是函数吧？为什么函数里还能设置属性，为什么函数还能调用方法？

.................过了大概 17 秒

哦，赶紧从 go 的世界里回来，这是 js 呢。函数是一种特殊的对象，当然有属性，想必有方法。再仔细看看，config 是静态属性，component 会是静态方法，new 一个函数就是新建一个对象并调用这个构造函数并返回对象，$watch 是在 Vue.prototype 里定义的实例方法。嗯，没毛病。

.................过了大概 17 秒

不行，还是好奇怪，在函数里用点操作符什么的，おかしい。在 js 的函数是对象，那函数体最后是怎么执行？

私、気になる（千反田Geass）

## 承

我们先来看看 [spec](http://www.ecma-international.org/ecma-262/6.0) 关于函数对象的定义：
> ECMAScript function objects encapsulate parameterized ECMAScript code closed over a lexical environment and support the dynamic evaluation of that code. **An ECMAScript function object is an ordinary object and has the same internal slots and the same internal methods as other ordinary objects.** The code of an ECMAScript function object may be either strict mode code (10.2.1) or non-strict mode code. An ECMAScript function object whose code is strict mode code is called a strict function. One whose code is not strict mode code is called a non-strict function.

函数对象是与普通对象有着相同 internal slot 和 internal method 的对象。那么 internal slot 和 internal method 是什么呢？

再来找找[普通对象](http://www.ecma-international.org/ecma-262/6.0/#sec-object-type)的定义：
> An Object is logically a collection of properties. Each property is either a data property, or an accessor property:
  - A data property associates a key value with an ECMAScript language value and a set of Boolean attributes.
  - An accessor property associates a key value with one or two accessor functions, and a set of Boolean attributes. The accessor functions are used to store or retrieve an ECMAScript language value that is associated with the property.
  
> ...

> The actual semantics of objects, in ECMAScript, are specified via algorithms called internal methods. **Each object in an ECMAScript engine is associated with a set of internal methods that defines its runtime behaviour.** These internal methods are not part of the ECMAScript language. They are defined by this specification purely for expository purposes. However, each object within an implementation of ECMAScript must behave as specified by the internal methods associated with it. The exact manner in which this is accomplished is determined by the implementation.

> ...

> **Internal slots correspond to internal state that is associated with objects and used by various ECMAScript specification algorithms.** Internal slots are not object properties and they are not inherited. Depending upon the specific internal slot specification, such state may consist of values of any ECMAScript language type or of specific ECMAScript specification type values. Unless explicitly specified otherwise, internal slots are allocated as part of the process of creating an object and may not be dynamically added to an object. Unless specified otherwise, the initial value of an internal slot is the value undefined. Various algorithms within this specification create objects that have internal slots. However, the ECMAScript language provides no direct way to associate internal slots with an object.

Object 是一个属性的集合。每个属性既可以是一个命名数据属性，也可以是一个命名访问器属性。命名访问器属性也就是具有 **[[Get]]** 和 **[[Set]]** 特性的属性（vue 的响应式变量就是通过它们来实现的）。

而 JS 引擎通过 internal method 来定义对象运行时的行为，通过 internal slot 来获取对象运行时的内部状态。对于 internal method 和 internal slot，用户不能直接访问它们，只能通过 JS 引擎提供的 API 访问。

对象基本的 internal method 有：`[[GetPrototypeOf]]`、`[[SetPrototypeOf]]`、`[[GetOwnProperty]]`、`[[GET]]`、`[[SET]]`、`[[Delete]]` 等等。

对象基本的 internal slot 有：`[[Prototype]]`、`[[Extensible]]`。

而对于函数对象：

> A function object is an object that supports the [[Call]] internal methods. A constructor (also referred to as a constructor function) is a function object that supports the [[Construct]] internal method.

也就是除了对象基本的 internal method 外，函数对象还拥有 `[[Call]]` 和 `[[Construct]]`。

另外，除了对象基本的 internal slot，函数对象还有 `[[Environment]]`、`[[FormalParameters]]`、`[[ECMAScriptCode]]`、`[[Realm]]`、`[[ThisMode]]`、`[[Strict]]` 等 internal slot。

## 转

清楚了函数对象在引擎内部的数据结构，接下来我们就可以看看声明一个函数时，实际上做了什么：
> When the Function function is called with some arguments p1, p2, … , pn, body (where n might be 0, that is, there are no “p” arguments, and where body might also not be provided), the following steps are taken:
  1. Let C be the active function object.
  2. Let args be the argumentsList that was passed to this function by [[Call]] or [[Construct]].
  3. Return CreateDynamicFunction(C, NewTarget, "normal", args).

就结果而言，返回了一个内置了 `[[Call]]` method 和其他  essential internal method 和 slot 的函数对象。
而 `[[Call]]` 方法是就是调用函数时，JS 引擎内部调用的方法：

> The [[Call]] internal method for an ECMAScript function object F is called with parameters thisArgument and argumentsList, a List of ECMAScript language values. The following steps are taken:
  1. Assert: F is an ECMAScript function object.
  2. If F’s [[FunctionKind]] internal slot is "classConstructor", throw a TypeError exception.
  3. Let callerContext be the running execution context.
  4. Let calleeContext be PrepareForOrdinaryCall(F, undefined).
  5. Assert: calleeContext is now the running execution context.
  6. Perform OrdinaryCallBindThis(F, calleeContext, thisArgument).
  7. Let result be **OrdinaryCallEvaluateBody(F, argumentsList)**.
  8. Remove calleeContext from the execution context stack and restore callerContext as the running execution context.
  9. If result.[[type]] is return, return NormalCompletion(result.[[value]]).
  10. ReturnIfAbrupt(result).
  11. Return NormalCompletion(undefined).

继续追踪 OrdinaryCallEvaluateBody 方法就可以看到：
> When the abstract operation OrdinaryCallEvaluateBody is called with function object F and List argumentsList the following steps are taken:
  1. Let status be FunctionDeclarationInstantiation(F, argumentsList).
  2. ReturnIfAbrupt(status)
  3. **Return the result of EvaluateBody of the parsed code that is the value of F's [[ECMAScriptCode]] internal slot passing F as the argument.**

而 [[ECMAScriptCode]] 里存的是：
> Function code is source text that is parsed to supply the value of the [[ECMAScriptCode]] and [[FormalParameters]] internal slots (see 9.2) of an ECMAScript function object. The function code of a particular ECMAScript function does not include any source text that is parsed as the function code of a nested FunctionDeclaration, FunctionExpression, GeneratorDeclaration, GeneratorExpression, MethodDefinition, ArrowFunction, ClassDeclaration, or ClassExpression.

也就是说 JS 引擎把解析好的函数体保存到 `[[ECMAScriptCode]]` internal slot 里，当调用函数的时候，就调用 `[[Call]]` internal method，最终返回解析 `[[ECMAScriptCode]]` 后的结果。而对于如何解析 `[[ECMAScriptCode]]`，这就涉及到编译原理和 JS 引擎的实现了，脱离了 ECMA-262 规范的范畴，就暂时先不去深入了。

## 合

简单的说，我们使用 JavaScript 的时候，实际上是没有所谓的“函数”的。函数就是对象，只是当我们在这个对象名称右边加上一个括号时，JS 引擎会自动帮我们调用这个对象内置的 `[[Call]]` 方法来执行定义在函数体里的内容。

Ref:
- http://www.ecma-international.org/ecma-262/6.0/
- https://stackoverflow.com/questions/29592110/difference-between-accessor-property-and-data-property-in-ecmascript
- https://stackoverflow.com/questions/33075262/what-is-an-internal-slot-of-an-object-in-javascript
- https://www.zhihu.com/question/24804474
- https://www.zhihu.com/question/52561129


