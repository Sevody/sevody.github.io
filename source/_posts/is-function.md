---
title: 如何判断变量是否是函数
date: 2018-10-01 16:24:40
tags:
---


```js
// 使用 instanceof 判断
function isFun(v) {
  return v instanceof Function
}

// 使用 typeof 判断
function isFun2(v) {
  return typeof v === 'function'
}

// 使用 toString 判断
function isFun3(v) {
  return Object.prototype.toString.call(v) === '[object Function]'
}
```

这3种方法都能判断变量是否是函数，但也存在一些的区别和问题，下面来具体看看。

<!--more-->

## 使用 instanceof 判断

`instanceof` 运算符用来检测 `constructor.prototype` 是否存在于参数 `object` 的原型链上。

```js
new Date instanceof Date; // true
 
[1,2,3] instanceof Array; // true
 
function CustomType() {};
new CustomType instanceof CustomType; // true 
```

在开头的 `isFun` 函数中，也就是检测 `Object.getPrototypeOf(v) === Function.prototype`。

然而在浏览器中，我们的代码可能需要在多个窗口之间进行交互。多个窗口意味着多个全局环境，不同的全局环境拥有不同的全局对象，从而拥有不同的内置类型构造函数。这可能会引发一些问题。比如：

```js
let iFrame = document.createElement('IFRAME');
document.body.appendChild(iFrame);
 
let IFrameArray = window.frames[1].Array; 
let array = new IFrameArray();
 
array instanceof Array; // false
array instanceof IFrameArray; // true; 
```

所以使用 `instanceof` 来判断变量是否是函数在这种场景下并不可靠。

## 使用 typeof 判断

`typeof` 操作符返回一个字符串，表示未经计算的操作数的类型。

| Type of val | Result |
| --- | --- |
| Undefined | ‘undefined' |
| Null | 'object' |
| Boolean | 'boolean' |
| Number | 'number' |
| String | 'string' |
| Object(native and not callable) | 'object' |
| Object(native or host and callable) | 'function' |

值得注意的是，在 JavaScript 最初的实现中，JavaScript 中的值是由一个表示类型的标签和实际数据值表示的。对象的类型标签是 `0`。由于 `null` 代表的是空指针（大多数平台下值为 `0x00`），因此，`null` 的类型标签是 `0`，`typeof null` 也因此返回 `'object'`。

在 ECMAScript 2015 之前，`typeof` 总能保证对任何所给的操作数返回一个字符串。即便是没有声明的标识符，`typeof` 也能返回 `'undefined'`。使用 `typeof` 永远不会抛出错误。

但在加入了块级作用域的 `let` 和 `const` 之后，在其被声明之前对块中的 `let` 和 `const` 变量使用 `typeof` 会抛出一个 `ReferenceError`。块作用域变量在块的头部处于“暂存死区”，直至其被初始化，在这期间，访问变量将会引发错误。

```js
typeof undeclaredVariable === 'undefined';

typeof newLetVariable; // ReferenceError
typeof newConstVariable; // ReferenceError
typeof newClass; // ReferenceError

let newLetVariable;
const newConstVariable = 'hello';
class newClass{};
```

回到一开始的问题，在 2018 年的现在，使用 `typeof` 判断变量是否是函数，并没有什么问题。然而我们如果看一下有不短历史的开源仓库的代码，判断变量是否是函数很少会使用 `typeof` 来实现的。究其原因，实际上是因为在一些很久之前的浏览器版本，判断**正则表达式**的时候，`typeof` 会返回 `'function'`，比如 *PhantomJS v1* 和 *Chrome 1-12*。所以，如果你不必兼容这些老版本的浏览器，可以放心使用 `typeof`。


## 使用 toString 判断

使用 `Object.prototype.toString` 来判断变量是否是函数是兼容性最好的，唯一的问题是，可读性比较差，不太好理解。

要理解为什么 `Object.prototype.toString` 能判断变量是否是函数，首先要知道的是这个方法的定义是什么。然而需要注意的是，`Object.prototype.toString` 这个方法的定义在 [ES5 spec](http://ecma-international.org/ecma-262/5.1/) 和 [ES6 spec](http://www.ecma-international.org/ecma-262/6.0/) 中是有些不一样的。

以前在[深入理解函数对象](https://sevody.github.io/2017/06/28/function-object/)中提到过 **Object 是一个属性的集合，每个对象对象有着对应的 internal slot 和 internal method**，而在 ES5 spec 中， `[[Class]]` 就是对象里面的一个 `internal slot` (ES spec 一般用双方括号 `[[]]` 来标识内置属性，比如 `[[Get]]` 、`[[Set]]` 、 `[[Prototype]]`  、 `[[Extensible]]` 等)。而根据 ES5 sepc，`[[CLass]]` 的[定义](http://ecma-international.org/ecma-262/5.1/#sec-8.6.2)是：*A String value indicating a specification defined classification of objects.* 也就是每个内建的对象都会有一个对应的不可修改的 `[[Class]]` 属性来表明该对象属于哪种类型的对象。

那么再看下 `Object.prototype.toString` 在 ES5 spec 的[定义](http://ecma-international.org/ecma-262/5.1/#sec-15.2.4.2):

> 1. If the this value is undefined, return "[object Undefined]".
2. If the this value is null, return "[object Null]".
3. Let O be the result of calling ToObject passing the this value as the argument.
4. Let class be the value of the [[Class]] internal property of O.
5. Return the String value that is the result of concatenating the three Strings "[object ", class, and "]".

简单来说，对象默认的 `toString` 方法就是返回 **\`object ${[[Class]]}\`** 这样格式的字符串。

而需要注意的是，内建对象的 `Object.prototype.toString` 方法基本上都会被覆盖：

```js
[1,2,3].toString() // '1, 2, 3'
 
(new Date).toString() // 'Mon Oct 01 2018 22:31:15 GMT+0800 (中国标准时间)'
 
(function f() {}).toString() // 'function f() {}'
```

所以我们必须用 `call` 方法来调用 `Object.prototype.toString`：

```js
Object.prototype.toString.call([1,2,3]) // '[object Array]'
 
Object.prototype.toString.call(new Date) // '[object Date]'
 
Object.prototype.toString.call((function f() {})) // '[object Function]'
```

所以判断变量是否是函数就可以这么实现：

```js
function isFun3(v) {
  return Object.prototype.toString.call(v) === '[object Function]'
}
```

而到了 ES6 spec，`[[Class]]` 内置属性被去掉了，取而代之的是一个名为 `@@toStringTag` 的 `Symbol`。它的具体[定义](http://www.ecma-international.org/ecma-262/6.0/#sec-well-known-symbols)是 *A String valued property that is used in the creation of the default string description of an object. Accessed by the built-in method Object.prototype.toString.* 

在 ES6 spec 中，`Object.prototype.toString` 的[定义](http://www.ecma-international.org/ecma-262/6.0/#sec-object.prototype.tostring)也变成了：

> 1. If the this value is undefined, return "[object Undefined]".
2. If the this value is null, return "[object Null]".
3. Let O be ToObject(this value).
4. Let isArray be IsArray(O).
5. ReturnIfAbrupt(isArray).
6. If isArray is true, let builtinTag be "Array".
7. Else, if O is an exotic String object, let builtinTag be "String".
8. Else, if O has an [[ParameterMap]] internal slot, let builtinTag be "Arguments".
9. Else, if O has a [[Call]] internal method, let builtinTag be "Function".
10. Else, if O has an [[ErrorData]] internal slot, let builtinTag be "Error".
11. Else, if O has a [[BooleanData]] internal slot, let builtinTag be "Boolean".
12. Else, if O has a [[NumberData]] internal slot, let builtinTag be "Number".
13. Else, if O has a [[DateValue]] internal slot, let builtinTag be "Date".
14. Else, if O has a [[RegExpMatcher]] internal slot, let builtinTag be "RegExp".
15. Else, let builtinTag be "Object".
16. Let tag be Get (O, @@toStringTag).
17. ReturnIfAbrupt(tag).
18. If Type(tag) is not String, let tag be builtinTag.
19. Return the String that is the result of concatenating "[object ", tag, and "]".

注意 *16. Let tag be Get (O, @@toStringTag)* ，在 ES5 的时候，自定义的对象调用 `Object.prototype.toString` 的时候只能返回内置的默认类型：

```js
class ValidatorClass {}

Object.prototype.toString.call(new ValidatorClass()); // '[object Object]'
```

而在 ES6，自定义对象的 `Object.prototype.toString` 的返回结果变得可修改了：

```js
class ValidatorClass {
  get [Symbol.toStringTag]() {
    return "Validator";
  }
}

Object.prototype.toString.call(new ValidatorClass()); // '[object Validator]'
```

不过从判断变量是否是函数这个问题上看，ES5 和 ES6 返回的结果都是一样的，所以可以尽情使用 `Object.prototype.toString.call(v) === '[object Function]'` 来实现。
 

Ref

- http://ecma-international.org/ecma-262/5.1/
- http://www.ecma-international.org/ecma-262/6.0/
- https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toStringTag
- https://javascriptweblog.wordpress.com/2011/08/08/fixing-the-javascript-typeof-operator