---
title: react 开发总结
date: 2016-12-30 19:59:53
tags: react
---

接触 react 开发也有 1 年时间了，从一开始没听说过 react 到开始自己编写和维护 react 组件，从中学到的不仅仅是怎样用 react 进行前端开发，还有**前端模块化**、**前端工程化**、**MVC模式** 等等对前端十分重要的知识和概念。同时也接触了像是**redux**、**webpack**、**eslint**、**autoprefixer**等一些有趣的库和工具。不管怎样，算是真正进入了前端开发这个领域。

<!--more-->

## Thinking in React

刚开始使用 react 编写页面的时候，或许会让人不知从何下手。
一种简单的方式，就是根据下面的步骤来开始：

1. 把要编写的页面划分区域并分割成一个个组件
2. 使用 jsx 语法来编写出一个静态版本的页面（也就是把数据写死）
3. 根据业务需求，标识出需要动态改变的 UI 数据（state）
4. 为了得到需要的 UI 数据和增加需要的事件处理，看看最少还需要增加哪些 state
5. 添加 state 和需要的逻辑处理函数
6. 重构并提取可复用的组件

## 编写通用组件

编写通用组件有三种方式：**属性**、**容器组件**、**高阶组件**

### 属性

根据父组件调用子组件时传入的 prop 的不同来展现不同的组件 内容/样式。

比如编写一个 Button 组件的时候，根据传入的 btnType 来决定 Button 的样式

```js
class Button extends React.Component{
    static defaultProps = {
        text: "按钮",
        btnType: '',
    }

    handleClick(){
        this.props.handleClick();
    }

    render(){
        const { text, btnType } = this.props;
        return  <div>
                    <button className={btnType} onClick={::this.handleClick}>
                        {text}
                    </button>
                </div>
    }
}
```

### 容器组件

如果想提高组件的灵活性，可以利用容器组件嵌套子组件的方式声明组件，让容器组件通过 prop.children 来获取嵌套组件的实例。

例如想要编写一个 tab 切换组件可以这么使用：

```js
<Tabs>
    <TabPane>
        <div>Tab1</div>
    </TabPane>
    <TabPane>
        <div>Tab2</div>
    </TabPane>
    <TabPane>
        <div>Tab3</div>
    </TabPane>
</Tabs>
```

就可以这么来实现组件：

```js
class Tabs extends React.Component {
    render() {
        const panes = React.Children.map(this.props.children,
            child => <div className="tab-pane">{child}</div>);
        return (
            <div>
                {panes}
             </div>
        );
    }
}

function TabPane({ children }) {
    return <div>{children}</div>;
}
```

### 高阶组件

高阶组件（Higher Order Component）是指通过函数和闭包，现有组件类添加逻辑，改变已有组件的行为。

比如上面的 Button 组件可以通过 btnType 来指定要圆形（round）还是方形（square）按钮，我们如果想要一个跟 Button 组件类似，但它的 btnType 永远是方形的组件，和一个永远是圆形的组件，就可以通过高阶组件来实现：

```js
function setType(btnType) {
    return function(WrappedComponent) {
        return class CustomButton extends React.Component {
            render() {
                const customProps = {...this.props, btnType}
                return  <WrappedComponent {...customProps} />
            }

        }
    }
}

const RoundButton = setType('round')(Button)
const SquareButton = setType('square')(Button)
```

## react 组件设计原则

- 职责清晰

    由多个组件协同完成一个复杂功能，而不是一个组件替其他组件完成它该干的事

- 避免信息冗余

    如果一个 state 可以由其他 props/state 计算出来，则删除该 state

- 尽量使用标签嵌套，而不是属性配置

- 避免使用 ref

    使用父组件的 state 来控制子组件状态，而不是使用 ref

- 尽量使用 stateless function component

- 对于每个非必要属性（prop）需要为其定义一个默认值（getDefaultProps）

- 永远不要尝试通过 setState 或者 replaceState 以外的方式去修改 state 对象
