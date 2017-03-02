---
title: react 组件生命周期
date: 2016-10-23 18:25:26
tags: react
---

在编写 react 组件时，根据需要会在组件生命周期的不同阶段实现不同的逻辑。
下面总结了在每个生命周期的基本用法。

<!--more-->

### getDefaultProps

初始化 props

### getInitailState

初始化 state

### componentWillMount

挂载 DOM 前会调用这个函数，此时可以使用 setState 来覆盖默认的 state

### componentDidMount

第一次 DOM 渲染完后调用，此时经常用于做一些 DOM 操作, 比如 focus


### componentWillReceiveProps

触发二次渲染前会调用，是更新前使用 setState 的唯一 hook

### shouldComponentUpdate

通常用于性能优化，返回 true 才执行 render （ 不能使用 `setState` ）

### componentWillUpdate

实际更新前调用，通常用于设置事件监听 ( 不能使用 `setState` )

```js
componentWillUpdate(nextProps, nextState) {
    if (!this.props.isOpen && nextProps.isOpen) {
        // 当下拉列表展开时可以做一些事件监听
    }

    if (this.props.isOpen && nextProps.isOpen) {
        // 当下拉列表隐藏时取消事件监听
    }
}
```

### componentDidUpdate

更新完成后调用，可以在这里进行一些 DOM 操作

### componentWillUnmount

组件即将移除时调用，可以去除被销毁的 DOM 节点的引用、消除计时器、取消监听等


## thought map

![](http://oj8psq2wh.bkt.clouddn.com/14883644186426.jpg)
