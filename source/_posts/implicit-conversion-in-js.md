---
title: js 中的隐式类型转换
date: 2017-03-02 14:36:23
tags: js
---

As we all know，js 作为一门动态语言，~~maybe~~为了编写代码时能更简单和自由，于是有了隐式类型转换这个机制的存在。

such as:

![](http://oj8psq2wh.bkt.clouddn.com/14884378226469.jpg)

然而，方便的同时也带来的一个问题，就是一不小心就可能掉进隐式类型转换的陷阱。

为了尽量避免掉进隐式类型转换的坑，下面就来看看隐式转换的~~ひみつ~~秘密。

<!--more-->

## 类型转换

### 基本类型

js（es5） ~~规定了~~有 5 中原始类型和 1 种复杂类型：

*primitive*： **undefined**, **null**, **boolean**, **number**, **string**


*non-primitive*： **object** (包括 Array, Function, Date, Reg)

类型转换的操作有：**ToPrimitive**、**toBoolean**、**toNumber**、**toString**

### ToPrimitive

ToPromitive 是指转换为基本类型，JavaScript引擎内部的抽象操作 `ToPrimitive()` 有着这样的声明:

```
ToPrimitive(input [,PreferredType])
```

可选参数 PreferredType 可以是 Number 或者 String，它只代表了一个转换的偏好，转换结果不一定必须是这个参数所指的类型，但转换结果一定是一个原始值。如果 PreferredType 被标志为 Number,则会进行下面的操作来转换输入的值

1. 如果输入的值已经是个原始值,则直接返回它
2. 否则，如果输入的值是一个对象。则调用该对象的 `valueOf()` 方法。如果 `valueOf()` 方法的返回值是一个原始值,则返回这个原始值
3. 否则，调用这个对象的 `toString()` 方法。如果 `toString()` 方法的返回值是一个原始值，则返回这个原始值
4. 否则，抛出 TypeError 异常

如果 PreferredType 被标志为 String，则转换操作的第二步和第三步的顺序会调换。
如果 PreferredType 被省略，除了 Date 类型的对象进行 ToPrimitive 时 PreferredType 会被设置为 String，其它类型的值会被设置为 Number。

### toBoolean

| 参数 | 结果 |
| --- | --- |
|  undefined | false  |
| null | false |
| boolean | 无需转换 |
| number | 0 / Nan => false; else true  |
| string | ‘’ => false; else true  |
| object | true |


### toNumber

| 参数 | 结果 |
| --- | --- |
|  undefined | NaN  |
| null | 0 |
| boolean | 1 / 0 |
| number | 无需转换  |
| string | ‘123’ => 123; ‘12a’ => NaN; ‘’ => 0  |
| object | 先 ToPrimitive(object) |

### toString

| 参数 | 结果 |
| --- | --- |
|  undefined | 'undefined'  |
| null | 'null' |
| boolean | 'true' / 'false' |
| number | 123 => '123'; 1.23 => '1.23'  |
| string | 无需转换 |
| object | 先 ToPrimitive(object) |

## **'=='** 运算符的隐式转换

js 中可以使用 `==` 和 `===` 运算符进行相等性判断，其中 `==` 运算符会进行隐式转换。

`==` 计算规则如下：

1. 如果两个值类型一致，则使用 `===` 进行判断
2. 如果一个值是 null，另一个值是 undefined，则两者相等
3. 如果一个值是 number，另一个值是 string，则把 string 转换为 number 再进行比较
4. 如果其中一个值为 boolean，则把它转换为 number，在比较
5. 如果一个值 object，另一个值是 number 或者 string，则把 object 转换为 primitive 在进行比较
6. 其他类型之间比较均不相等

![](http://oj8psq2wh.bkt.clouddn.com/14883389389148.jpg)

![](http://oj8psq2wh.bkt.clouddn.com/14883389522106.jpg)


## **'<'**、**'>'** 运算符的隐式转换

进行 `<` 、 `>` 判断时，通常只有两种情况：数字和数字比较，字符串和字符串比较。其他所有类型比较都会返回 **false**。

1. 将两个操作数转换为 primitive
    prim1 = ToPrimitive(value1)
    prim2 = ToPrimitive(value2)
2. 如果 prim1 和 prim2 中都为 string，逐个比较每个字符的 code value，返回比较结果
3. 否则将 prim1 和 prim2 都转换为 number 类型，返回比较结果

伪代码如下：

```
a < b:
    pa = ToPrimitive(a)
    pb = ToPrimitive(b)
    if(pa is string && pb is string)
       return compareUnitCodeValue(pa, pb）
    else
       return compare(ToNumber(pa), ToNumber(pb))
```

例子：

```
'' < 'a' // => true
[] < 'a' // => true
'1' < 'a' // => true
1 < 'a' // => false
NaN < 1 // => false
```

## **'+'** 运算符的隐式转换

在 js 中，`+` 运算符通常只有两种情况：数字和数字相加，字符串和字符串相加。其他所有类型都会被转换成这两种类型的值。

其内部操作步骤如下：

1. 将两个操作数转换为 primitive
    prim1 = ToPrimitive(value1)
    prim2 = ToPrimitive(value2)
2. 如果 prim1 或者 prim2 中任意一个为 string，则另一个也转换为 string，然后返回两个 string 连接操作后的结果
3. 否则将 prim1 和 prim2 都转换为 number 类型，返回两者之和

伪代码如下:

```
a + b:
    pa = ToPrimitive(a)
    pb = ToPrimitive(b)*
    if(pa is string || pb is string)
       return concat(ToString(pa), ToString(pb))
    else
       return add(ToNumber(pa), ToNumber(pb))
```

## **'-'**、**'*'**、**'/'** 运算符的隐式转换

与 `+` 运算符类似，但只要 number 类型的操作，其他所有类型都会转换为 number 后才进行操作，无法转换为 number 的，操作结果为 NaN

## 总结

在进行相等判断的时候，空值判断可以使用 `==` 来进行判断，其他情况优先使用 `===`；其他运算符需要注意操作数最终是转换为 number 进行运算还是转换为 string 进行运算。

## Ref

- http://www.ecma-international.org/ecma-262/6.0
- https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Equality_comparisons_and_sameness
- http://www.2ality.com/2012/01/object-plus-object.html


