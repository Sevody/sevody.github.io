---
title: 字符串与数组切割常用函数
date: 2016-09-02 15:38:34
tags: js
---

使用 js 对字符串和数组进行部分截取操作的时候，我们可以使用 `slice`, `splice`, `substr` 等方法，但有的方法能同时作用于 Array 和 String，有的只能作用于其中之一，而且函数间的参数也都有细微的区别，很容易混淆它们用法。下面把常用的截取函数总结到一起，希望能有个整体的认识。

<!--more-->

**slice（）**

Array 和 String 对象都有

- 在 Array 中： `Array.slice(i [,j])`

```js
/**
 * 截取子数组函数
 * @param  {int} i  [开始截取的索引值,负数代表从末尾算起的索引值，-1为倒数第一个元素]
 * @param  {int} j  [可选，结束的索引值，缺省时则获取从i到末尾的所有元素]
 * @return {array}  [从 i 到 j 的数组，原数组不改变]
 */
slice(i [,j])
```

- 在 String 中： `String.slice(i [,j])`

```js
/**
 * 截取子字符串函数
 * @param  {int} i  [开始截取的索引值,负数代表从末尾算起的索引值，-1为倒数第一个字符]
 * @param  {int} j  [可选，结束的索引值，缺省时则获取从i到末尾的所有字符]
 * @return {array}  [从 i 到 j 的子字符串，原字符串不改变]
 */
slice(i [,j])
```

**splice（）**

- 只存在于 Array 中：`Array.splice(index, howmany [, item1, itemx])`     

```js
/**
 * 向数组添加项目/删除项目，然后返回被删除的项目
 * 该方法会改变原始数组
 * @param  {int} index  [添加/删除元素的位置，使用负数可从数组结尾处开始指定位置]
 * @param  {int} howmany  [要删除的元素数量。如果设置为 0，则不会删除元素]
 * @param  {int} item  [可选，向数组添加的新项目]
 * @return {array}  [被删除项目的新数组（如果有的话）]
 */
splice(index, howmany [, item1, itemx])
```

**split（）**

- 只存在 String 中: `String.split(separator, howmany)`


```js
/**
 * 以指定的分割符分割字符串，并返回分割后的数组
 * @param  {string|Reg} separator  [字符串或正则表达式，从该参数指定的地方分割字符串]
 * @param  {int} howmany  [可选。该参数可指定返回的数组的最大长度。如果设置了该参数，返回的子串不会多于这个参数]
 * @return {array}  [一个字符串数组。该数组是通过在 separator 指定的边界处将字符串分割成子串创建的]
 */
split(separator [, howmany])
```

**substring（）**

- 只存在 String 中 `String.substring(start [, stop])`

```js
/**
 * 截取子字符串函数
 * @param  {int} start  [表示子字符串的开始位置，不能小于 0，否则算作 0]
 * @param  {int} stop  [可选，表示结束的索引值，缺省时则获取从 start 到末尾的所有元素]
 * @return {string}  [从 start 到 stop 的子字符串，原字符串不改变]
 */
substring(start [, stop])
```

**substr（）**

- 只存在 String 中： `String.substr(start [, length])`

```js
/**
 * 截取子数组函数
 * @param  {int} start  [子字符串的开始位置，可以小于 0]
 * @param  {int} length  [可选，表示子字符串的长度]
 * @return {string}  [从 start 开始截取 length 长度的字符串，原字符串不改变]
 */
substr(start [, length])
```


