---
title: JavaScript 函数式编程
date: 2018-03-15 18:48:05
tags:
---

## 什么是函数式编程

### 从 *λ演算* 说起

λ演算是图灵完备的，它是一种通用的计算模型，可用于模拟任何一台单带图灵机。图灵完备意味着你的语言可以做到能够用图灵机能做到的所有事情，可以解决所有的可计算问题。

<!--more-->

λ演算是一个极其简单但又十分强大的概念。它的核心主要有两个概念:
1. 函数的抽象，通过引入变量来归纳得出表达式
2. 函数的应用，通过给变量赋值来对已得出的表达式进行计算

举个栗子，我们表示一个一元二次函数是这样子：

```
y = x^2-2*x+1
```

如果用λ演算表示，会变成这样的形式：

```
// λ<变量>.<表达式>
λx.x^2-2*x+1
```

而当我们把1赋值给x时，它的调用过程会是这样：

```
// x=1
(λx.x^2-2*x+1)1=1-2*1+1=0
```

如果是二元一次函数，调用过程会变成这样：

```
// x=1 y=2				
( (λx.λy.2*x+y) 1) 2 = ( λy.2+y) 2 = 4
```

从上面可以看出，λ演算有2个特点：

1. 计算顺序是固定的，从里层到外层一层层归约，如果改变变量的次序也会影响函数应用中的返回值。
2. 函数的返回值也可以是一个*函数*，这与函数式编程语言中函数是一等公民的概念是一致的。

### 定义

说完λ演算，我们就可以来看下函数式编程的定义：

> 函数式编程（functional programming）或称函数程序设计，又称泛函编程，是一种编程典范，它将电脑运算视为数学上的函数计算，并且避免使用程序状态以及易变对象。函数编程语言最重要的基础是λ演算（lambda calculus）。

举一个简单的例子：用函数式编程实现*if else*

```javascript
// IF(cond)(thn)(els)
IF = function(cond) {
  return function(thn) {
    return function(els) {
      return cond(thn)(els)
    }
  }
}
// TRUE
tru = function(thn) {
  return function(els) {
    return thn
  }
}
// FALSE
fls = function(thn) {
  return function(els) {
    return els
  }
}
IF(tru)(1)(2) //=> 1
IF(fls)(1)(2) //=> 2
```

通过上面的例子，从数据的角度看函数式编程，在数据流动过程中，数据的形式是通过函数进行变换的。

### 编程范式

函数式编程是一种范式，大家还可能听说过下面的这些方式：

- Object-oriented (OOP) 面向对象
- Procedural 						面向过程
- Functional   					函数式
- Imperative 						声明式
- ...

而这些编程范式的关系可以参考下面这张图：

![paradigm_tree](http://oj8psq2wh.bkt.clouddn.com/paradigm_tree.jpg)

编程范式的好处在于：易于在学校教学，引发讨论，区分编程语言。

命令式（imperative）语言是指一步步导向目标的一个过程。程序的执行往往限制在某些状态上。状态有global/local variable，它们是可以变化的。还有if else等control flow的结构。命令式语言常见的有C， C++，Java等等。

声明式（declarative）语言描述目标，具体的执行由引擎，系统等等决定。function calls, high order functions, recursion等都是它的特点。比较常见的语言包括：Scala，Haskell，Lisp，Scheme等。

实际上大部分语言都是混合的，代码的风格不是由一小块代码决定，而是要从整体构架的角度出发。而Javascript是非纯函数式编程语言里最早在mainstream里加入函数式编程特性的语言。

![language_style](http://oj8psq2wh.bkt.clouddn.com/language_style.jpg)


## 为什么要函数式编程

### 数据和行为的关系

在计算机中，数据多数指的是存储结构。行为指的多数是计算操作。而在不同编程范式里，它们的关系会有所不同。

我们先来看看面向对象式：

```php
// man.php
class Man {
  function __constructor($sexy){
    $this->sexy = $sexy;
  }
  public function sayHello($string){
    echo "I'm a ".$this->sexy.",I want to say:".$string;
  }
}
// male.php
require_once 'man.php'
$male = new Man("man");
$male->sayHello("my name is jay");
//=> I'm a man, I want to say: my name is jay
```

通过上面的例子可以看出，在面向对象的编程中，我们习惯把对象作为行为的核心，也就是说，先有人，然后，人来执行一个动作。而，对象，其实就是某一种变量，亦或是某一种数据类型。

然后我们再看看函数式：

```js
function Man(sexy){
  return function(string){
    console.log("I'm a "+sexy+",I want to say:"+string);
  }
}
var sayHello = Man('man');
sayHello('my name is jay');
//=> I'm a man, I want to say: my name is jay
```

在函数式编程中，我们去除掉了主语。你不知道这个动作是由谁发出的。相比于在面向对象编程中，数据是对象的属性，在函数式编程中，我们并不在乎这个数据的内容是什么，而更在乎其变化。

在实际的开发过程中，我们有的时候很难抽象出一个对象来描述我们到底要做什么，或者说，我们其实并不在乎这堆数据里面的内容是什么，我们要关心的，只是把数据经过加工，得出结果，仅此而已。至于这个数据，到底是用来干什么的，我们其实可以并不用关心。

### 函数式编程的特性

#### 一等公民

在函数式编程中，函数是一等公民：

> 函数可以存储为变量
> 函数可以成为数组的一个元素
> 函数可以成为对象的成员变量
> 函数可以在使用的时被直接创建
> 函数可以被作为实参传递
> 函数可以被另一个函数返回
> 函数可以返回另一个函数
> 函数可以作为形参

简单地说，所谓一等公民，说的是函数本身可以成为代码构成中的任意部分。

### 函数式编程的特性

#### 纯函数

什么是纯函数，我们可以看下面的例子：

```js
var xs = [1,2,3,4,5];

// 纯的
xs.slice(0,3); //=> [1,2,3]		
xs.slice(0,3); //=> [1,2,3]			
xs.slice(0,3); //=> [1,2,3]

// 不纯的
xs.splice(0,3); //=> [1,2,3]		
xs.splice(0,3); //=> [4,5]	
xs.splice(0,3); //=> []
```

可以看出，所谓的纯函数就是不产生任何的副作用的函数。

#### 可组合

当函数纯化之后，有一个很鲜明的特点是，这个函数变的可以组合了。我们可以像堆乐高积木一样，把各个我们要用的函数堆起来变成一个更大得函数体。

```js
/**
* compose(f, g, h)
* (...args) => f(g(h(...args))).
*/
export default function compose(...funcs) {
  // ...
  const last = funcs[funcs.length - 1]
  const rest = funcs.slice(0, -1)
  return (...args) => rest.reduceRight((composed, f) 
    => f(composed), last(...args))
}
```

#### 高阶函数

```js
function calc(x){
  return function(y){
    return function(method){
      method && method(x)(y);
    }
  }
}
function add(x){
  return function(y){
    console.log(x+y);
  }
}
calc(1)(2)(add)；//=> 3
```

高阶函数就是以一个函数为参数，同时返回一个函数作为函数的返回值。

```js
// redux-thunk
function thunk (_ref) {
  var dispatch = _ref.dispatch,
    getState = _ref.getState;
  return function (next) {
    return function (action) {
      if (typeof action === 'function') {
        return action(dispatch, getState);
      }
      return next(action);
    };
  };
};
```

### 函数式编程的好处

#### 1. 代码简洁，开发快速

函数式编程大量使用函数，减少了代码的重复，因此程序比较短，开发速度较快。

#### 2. 接近自然语言，易于理解

函数式编程的自由度很高，可以写出很接近自然语言的代码。

#### 3. 更方便的代码管理

函数式编程不依赖、也不会改变外界的状态，只要给定输入参数，返回的结果必定相同。因此，每一个函数都可以被看做独立单元，很有利于进行单元测试（unit testing）和除错（debugging），以及模块化组合。 

#### 4. 易于"并发编程"

函数式编程不需要考虑"死锁"（deadlock），因为它不修改变量，所以根本不存在"锁"线程的问题。不必担心一个线程的数据，被另一个线程修改，所以可以很放心地把工作分摊到多个线程，部署"并发编程"（concurrency）。

## 如何进行函数式编程

### 柯里化

#### 定义

> 在计算机科学中，柯里化是把接受多个参数的函数变换成接受一个单一参数的函数，并且返回接受余下的参数而且返回结果的新函数的技术。

Curry化来源于数学家 Haskell Curry的名字 （编程语言 Haskell也是以他的名字命名）。柯里化的过程是逐步传参，逐步缩小函数的适用范围，逐步求解的过程。

一个简单的例子：求和函数

```js
var sum = function (a, b, c) {
  return a + b + c;
};
console.log(sum(1, 2, 3)); //=> 6 
```

```js
var sumCurrying = function(a) {
  return function (b) {
    return function (c) {
      return a + b + c;
    };
  };
};
console.log(sumCurrying(1)(2)(3)); //=> 6
```

柯里化的函数每次调用都返回一个新的函数，该函数接受另一个调用，然后又返回一个新的函数，直至最后返回结果，分布求解，层层递进。

更通用的柯里化：

```js
var currying = function (fn) {
  var _args = [];
  return function () {
    if (arguments.length === 0) {
        return fn.apply(this, _args);
    }
    Array.prototype.push.apply(_args, [].slice.call(arguments));
    return arguments.callee;
  }
};

var multi=function () {
  var total = 0;
  for (var i = 0, c; c = arguments[i++];) {
    total += c;
  }
  return total;
};
var sum = currying(multi);
sum(100,200)(300);
sum(400);
console.log(sum()); //=> 1000
```

#### 柯里化的作用
- 延迟计算

从上面的例子可以看出。

- 参数复用

当在多次调用同一个函数，并且传递的参数绝大多数是相同的，那么该函数可能是一个很好的柯里化候选。

- 动态创建函数

这可以是在部分计算出结果后，在此基础上动态生成新的函数处理后面的业务，这样省略了重复计算。


### 高阶组件

一般的高阶组件使用形式：
```js
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

高阶组件就是一个函数，且该函数接受一个组件作为参数，并返回一个新的组件。

高阶组件: *connect*

```js
import { connect } from 'react-redux'
import { addTodo, deleteTodo } from './actionCreators'
function mapStateToProps(state) {
  return { todos: state.todos }
}
const mapDispatchToProps = {
  addTodo,
  deleteTodo
}
export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

```js
function connect(mapStateToProps, mapDispatchToProps) {
  return function(WrappedComponent) {
    return class Connector extends Component {
      // ...
      render() {
        this.mapState = mapStateToProps(this.state)
        this.mapProps = this.bindActionCreator(mapDispatchToProps, this.context.store.dispatch)
        this.allprops = Object.assign({}, this.props, this.mapState, this.mapProps)
        return <div>
          <WrappedComponent { ...this.allprops} >
          </WrappedComponent>
        </div>
      }
    }
  }
}
```

### *Ramda*

ramdajs是一个更具有函数式代表的JavaScript库。

我们可以快速看下它的使用情景：
						
```js
// 判断第一个参数是否大于第二个参数
R.gt(2)(1) //=> true

// 返回两个值的和
R.add(7)(10) // 17

// 按照指定分隔符将字符串拆成一个数组
R.split('.')('a.b.c.xyz.d') //=> ['a', 'b', 'c', 'xyz', 'd']

// 如果包含某个成员，返回true
R.contains(3)([1, 2, 3]) //=> true

// 将多个函数合并成一个函数，从左到右执行
var negative = x => -1 * x;
var increaseOne = x => x + 1;

var f = R.pipe(Math.pow, negative, increaseOne);
f(3, 4) // -80 => -(3^4) + 1

// ...
```

与 loadash 等函数式工具库不同的是，ramda 把处理的数据放到了第一个参数Ramda 的数据一律放在最后一个参数，理念是"function first，data last"。除了数据放在最后一个参数，Ramda 还有一个特点：所有方法都支持柯里化。由于这两个特点，使得 Ramda 成为 JavaScript 函数式编程最理想的工具库。

Ref :
- https://zh.wikipedia.org/wiki/函數程式語言
- https://zhuanlan.zhihu.com/p/20640353
- http://www.cnblogs.com/zztt/p/4142891.html
- https://mp.weixin.qq.com/s?__biz=MzIwMDgyNzUzNw==&mid=2247483658&idx=1&sn=2dde89033d2fef0cba307db9b8b5489e&scene=0#wechat_redirect