---
title: 浅拷贝和深拷贝
date: 2016-11-18 22:12:03
tags: js
---

深拷贝指的是对象属性所引用的对象**全部**进行新建对象复制，以保证深复制的对象的引用图不包含任何原有对象或对象图上的任何对象，隔离出两个完全不同的对象图。

例如，我们要将 b.obj 复制到 a 中，

浅拷贝仅仅只是复制引用(Reference)，拷贝后，a.obj === b.obj

深拷贝是创建(clone)了一个**一模一样**的对象，并保存在a.obj中，所以 a.obj !== b.obj

<!--more-->

## 代码层面实现深浅复制

**数组拷贝**

```js
// 浅复制, 双向改变, 指向同一块内存区域
let arr = [1, 2, 3]
let arrClone = arr
arrClone[0] = 'test'
console.log(`shallow copy: arr: ${arr} arrClone ${arrClone}`
// shallow copy:    arr: test 2 3    arrClone: test 2 3

// 深复制, 开辟新的内存区

// 方法一: slice (原理: slice返回一个新数组)

let arr = [1, 2, 3]
let arrDeepClone = arr.slice()
arrDeepClone[0] = 'test'
console.log(`shallow copy: arr: ${arr} arrClone ${arrClone}`
// shallow copy: arr: 1 2 3 arrClone: test 2 3

// 方法二: concat (原理: concat返回一个新数组)
let arr = [1, 2, 3]
let arrDeepClone = arr.concat()
arrDeepClone[0] = 'test'
console.log(`shallow copy: arr: ${arr} arrClone ${arrClone}`
// shallow copy: arr: 1 2 3 arrClone: test 2 3
```

**不考虑特殊情况的对象复制**

```javascript
// 知乎看到的深复制方法，这个函数可以深拷贝 对象和数组,很遗憾，对象里的函数会丢失
deepCloneObj = obj => {
    let str, newobj = obj.constructor === Array ? [] : {};
    if(typeof obj !== 'object') {
        return obj;
    }else if(window.JSON) {

        // 好处是非常简单易用，但是坏处也显而易见，会丢失很多东西，这会抛弃对象的constructor，
        // 也就是深复制之后，无论这个对象原本的构造函数是什么，在深复制之后都会变成Object
        // 另外诸如RegExp对象是无法通过这种方式深复制的

        str = JSON.stringify(obj);
        newobj = JSON.parse(str);
        //console.log(JSON.parse(JSON.stringify(/[0-9]/)));
    }else {
        for(let i in obj) {
            newobj[i] = typeof obj[i] === 'object' ? deepCloneObj(obj[i]) : obj[i];
        }
    }
    return newobj;
}

let deepArr4 = { a: 1, b: 'test', c: [1, 2, 3], d: { 'a': 'd:a', 'b': 'd:b' } }
deepArr5 = deepCloneObj(deepArr4);
deepArr5['a'] = 'testa';
console.log('deep copy: ' + JSON.stringify(deepArr4) + " " + JSON.stringify(deepArr5));

// deep copy: {"a":1,"b":"test","c":[1,2,3],"d":{"a":"d:a","b":"d:b"}}
//            {"a":"testa","b":"test","c":[1,2,3],"d":{"a":"d:a","b":"d:b"}}

```

## 第三方库实现深浅复制

### 1. jQuery: `$.extend`

$.extend方法第一个参数可以是布尔值, 用来设置是否深度拷贝

```js
jQuery.extend(true, { a : { a : "a" } }, { a : { b : "b" } } );

jQuery.extend( { a : { a : "a" } }, { a : { b : "b" } } );

```

下面是源码:

```js
jQuery.extend = jQuery.fn.extend = function() {

    var src, copyIsArray, copy, name, options, clone,
        target = arguments[0] || {},
        i = 1,
        length = arguments.length,
        deep = false;

    // Handle a deep copy situation
    if ( typeof target === "boolean" ) {
        deep = target;

        // skip the boolean and the target
        target = arguments[ i ] || {};
        i++;
    }

    // Handle case when target is a string or something (possible in deep copy)
    if ( typeof target !== "object" && !jQuery.isFunction(target) ) {
       target = {};
    }

    // extend jQuery itself if only one argument is passed
    if ( i === length ) {
        target = this;
        i--;
    }

    for ( ; i < length; i++ ) {

        // Only deal with non-null/undefined values
        if ( (options = arguments[ i ]) != null ) {

            // Extend the base object
            for ( name in options ) {
                src = target[ name ];
                copy = options[ name ];

                // Prevent never-ending loop
                if ( target === copy ) {
                    continue;
                }

                // Recurse if we're merging plain objects or arrays
                if ( deep && copy && ( jQuery.isPlainObject(copy) || (copyIsArray = jQuery.isArray(copy)) ) ) {
                    if ( copyIsArray ) {
                        copyIsArray = false;
                        clone = src && jQuery.isArray(src) ? src : [];
                    } else {
                        clone = src && jQuery.isPlainObject(src) ? src : {};
                    }

                    // Never move original objects, clone them
                    target[ name ] = jQuery.extend( deep, clone, copy );

                // Don't bring in undefined values
                } else if ( copy !== undefined ) {
                    target[ name ] = copy;
                }
            }
        }
    }

    // Return the modified object
    return target;

};

```

### 2. Underscore: `_.clone()`

在 Underscore 中有这样一个方法：_.clone()，这个方法实际上是一种浅复制 (shallow-copy)，所有嵌套的对象和数组都是直接复制引用而并没有进行深复制。

### 3. loadsh: `_.clone()` / `_.cloneDeep()`

在lodash中关于复制的方法有两个，分别是`_.clone()`和`_.cloneDeep()`。

其中`_.clone(obj, true)`等价于`_.cloneDeep(obj)`。

使用上，lodash和前两者并没有太大的区别，但看了源码会发现，Underscore 的实现只有30行左右，而 jQuery 也不过60多行。可 lodash 中与深复制相关的代码却有上百行.

这是因为jQuery 无法正确深复制 JSON 对象以外的对象,

而 lodash 花了大量的代码来实现 ES6 引入的大量新的标准对象。


## 参考资料

[知乎: javascript中的深拷贝和浅拷贝](https://www.zhihu.com/question/23031215)

[深入剖析javacript的深复制](http://jerryzou.com/posts/dive-into-deep-clone-in-javascript/)

[js深浅复制](https://segmentfault.com/a/1190000005898146)
