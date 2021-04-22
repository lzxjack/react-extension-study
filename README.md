# 1. setState

使用`setState`更新状态有 2 种写法。

`对象式`是`函数式`的简写方式（语法糖）。

**使用原则（非必须）：**

-   新状态**不依赖于**原状态 ===> 使用`对象式`
-   新状态**依赖于**原状态 ===> 使用`函数式`
-   如果需要在`setState()`执行后获取最新的状态数据，要在`callback`函数中读取

## 1. 对象式

```javascript
setState(stateChange, [callback]);
```

-   `stateChange`为状态改变对象（该对象可以体现出状态的更改）
-   `callback`是可选的回调函数, 它在状态更新完毕、界面也更新后（`render`调用后）才被调用
-   状态的更新是`异步`的，如果想要查看`更新后的状态`，需要写在`callback`中

```javascript
const { count } = this.state;
this.setState({ count: count + 1 }, () => {
    console.log(this.state.count);
});
```

## 2. 函数式

```javascript
setState(updater, [callback]);
```

-   `updater`为返回`stateChange`对象的函数，可以接收到`state`和`props`

-   `callback`是可选的回调函数, 它在状态更新完毕、界面也更新后（`render`调用后）才被调用

```
this.setState((state, props) => ({ count: state.count + 1 }));
```

# 2. 路由组件的lazyLoad

通过 React 的`lazy`函数配合`import()`函数动态加载路由组件，使路由组件代码分开打包。

```javascript
import Loading from './Loading';
const Home = lazy(() => import('./Home'));
const About = lazy(() => import('./About'));
```

通过`<Suspense>`标签指定在加载得到路由打包文件前显示一个自定义`loading界面`。

```javascript
<Suspense fallback={<Loading />}>
    {/* 注册路由 */}
    <Route path="/about" component={About} />
    <Route path="/home" component={Home} />
</Suspense>
```

# 3. Hooks

`Hook`是 React 16.8.0 版本增加的新特性，可以在`函数组件`中使用`state`以及其他的 React 特性。下面介绍三个常用的`Hook`：

-   State Hook：`React.useState()`
-   Effect Hook：`React.useEffect()`
-   Ref Hook：`React.useRef()`

## 1. State Hook

State Hook 让函数组件也可以有`state`状态，并进行状态数据的读写操作。

```javascript
const [xxx, setXxx] = React.useState(initValue); // 解构赋值
```

-   `useState()`

    参数：第一次初始化指定的值在内部作缓存

    返回值：包含 2 个元素的数组，第 1 个为内部当前状态值，第 2 个为更新状态值的函数

-   `setXxx()`2 种写法

    `setXxx(newValue)`：参数为非函数值，直接指定新的状态值，内部用其覆盖原来的状态值

    `setXxx(value => newValue)`：参数为函数，接收原本的状态值，返回新的状态值，内部用其覆盖原来的状态值

```javascript
function Demo() {
    const [count, setCount] = React.useState(0);

    //加的回调
    function add() {
        // 第一种写法
        // setCount(count + 1);
        // 第二种写法
        setCount(count => count + 1);
    }

    return (
        <div>
            <h2>当前求和为：{count}</h2>
            <button onClick={add}>点我+1</button>
        </div>
    );
}
```

## 2. Effect Hook

Effect Hook 可以在函数组件中执行`副作用操作`（用于模拟类组件中的生命周期钩子）。

React 中的`副作用操作`：

-   发`ajax`请求数据获取
-   设置订阅 / 启动定时器
-   手动更改真实 DOM

```javascript
useEffect(() => {
    // 在此可以执行任何带副作用操作
    // 相当于componentDidMount()
    return () => {
        // 在组件卸载前执行
        // 在此做一些收尾工作, 比如清除定时器/取消订阅等
        // 相当于componentWillUnmount()
    };
}, [stateValue]); // 监听stateValue
// 如果省略数组，则检测所有的状态，状态有更新就又调用一次回调函数
// 如果指定的是[], 回调函数只会在第一次render()后执行
```

可以把`useEffect()`看做如下三个函数的组合：

-   `componentDidMount()`
-   `componentDidUpdate()`
-   `componentWillUnmount() `

```javascript
function Demo() {
    const [count, setCount] = React.useState(0);

    React.useEffect(() => {
        let timer = setInterval(() => {
            setCount(count => count + 1);
        }, 500);
        console.log('@@@@');
        return () => {
            clearInterval(timer);
        };
    }, [count]);
    // 检测count的变化，每次变化，都会输出'@@@@'

    //加的回调
    function add() {
        // 第一种写法
        // setCount(count + 1);
        // 第二种写法
        setCount(count => count + 1);
    }

    // 卸载组件的回调;
    function unmount() {
        ReactDOM.unmountComponentAtNode(document.getElementById('root'));
    }

    return (
        <div>
            <h2>当前求和为：{count}</h2>
            <button onClick={add}>点我+1</button>
            <button onClick={unmount}>卸载组件</button>
        </div>
    );
}
```

## 3. Ref Hook

Ref Hook 可以在函数组件中存储/查找组件内的标签或任意其它数据。

保存标签对象，功能与`React.createRef()`一样

```javascript
const refContainer = useRef();
```

```javascript
function Demo() {
    const myRef = React.useRef();

    //提示输入的回调
    function show() {
        alert(myRef.current.value);
    }

    return (
        <div>
            <input type="text" ref={myRef} />
        </div>
    );
}
```

# 4. Fragment

使用`<Fragment><Fragment>`后，可以不用必须有一个真实的 DOM 根标签了。

```javascript
import React, { Component, Fragment } from 'react';

export default class Demo extends Component {
    render() {
        return (
            <Fragment key={1}>
                <input type="text" />
                <input type="text" />
            </Fragment>
        );
    }
}
```

使用空标签`<></>`包裹也可以，他们的区别如下：

-   `<Fragment><Fragment>`：可以接收`key`属性，不能接收其他属性
-   `<></>`：不能接受属性

# 5. Context

一种组件间通信方式，常用于`祖组件`与`后代组件`间通信。

在组件外部创建`Context`容器对象：

```javascript
const XxxContext = React.createContext();
```

渲染子组时，外面包裹`xxxContext.Provider`，通过`value`属性给后代组件传递数据：

```javascript
<XxxContext.Provider value={数据}>子组件</XxxContext.Provider>
```

后代组件读取数据：

方式（1），仅适用于`类组件`：

```javascript
static contextType = xxxContext  // 声明接收context
console.log(this.context); // 读取context中的value数据
```

方式（2），`函数组件`与`类组件`都可以：

```javascript
<XxxContext.Consumer>{value => `${value.username},年龄是${value.age}`}</XxxContext.Consumer>
```

# 6. 组件优化

React 中`Component`组件的2个问题：

1. 只要执行`setState()`，即使**不改变状态数据**，组件也会重新`render()`
2. 只要当前组件重新`render`，就会自动重新`render`子组件，即使子组件没有发生任何变化，这导致页面更新的效率低下

效率高的做法：

- 只有当组件的`state`或`props`数据发生改变时才重新`render()`。

问题的原因：

- `Component`中的`shouldComponentUpdate()`总是返回`true`

解决办法：

- 重写`shouldComponentUpdate()`方法

  比较新旧`state`或`props`数据，如果有变化才返回`true`，否则返回`false`

- 使用`PureComponent`组件代替`Component`组件

  `PureComponent`重写了`shouldComponentUpdate()`，只有`state`或`props`数据有变化才返回`true`

  只是进行`state`和`props`数据的`浅比较`，如果只是数据对象内部数据变了，返回`false`

  所以不要直接修改`state`数据，而是要**产生新数据**

# 7.  render props

向组件内部动态传入带内容的结构（标签）。

## 1. children props

```javascript
<A>
    <B>xxxx</B>
</A>
```

`B`组件通过`this.props.children`获取便签中的数据。

但是`B`组件获得不到`A`组件内的数据。

## 2. render props

```javascript
export default class Parent extends Component {
    render() {
        return (
            <div className="parent">
                <h3>我是Parent组件</h3>
                <A render={name => <B name={name} />} />
            </div>
        );
    }
}

class A extends Component {
    state = { name: 'tom' };
    render() {
        console.log(this.props);
        const { name } = this.state;
        return (
            <div className="a">
                <h3>我是A组件</h3>
                {this.props.render(name)}
            </div>
        );
    }
}

class B extends Component {
    render() {
        console.log('B--render');
        return (
            <div className="b">
                <h3>我是B组件,{this.props.name}</h3>
            </div>
        );
    }
}
```

相当于`A`组件内部写了个插槽，可以在`Parent`组件中任意更改向插槽中插入的组件，并传递`A`组件的数据。

# 8. 错误边界

用来捕获后代组件错误，渲染出备用页面。

只能捕获后代组件`生命周期`产生的错误，不能捕获自己组件产生的错误和其他组件在合成事件、定时器中产生的错误。

`getDerivedStateFromError` + `componentDidCatch`

```javascript
export default class Parent extends Component {
    state = {
        hasError: '', // 用于标识子组件是否产生错误
    };

    //当Parent的子组件出现报错时候，会触发getDerivedStateFromError调用，并携带错误信息
    static getDerivedStateFromError(error) {
        console.log('@@@', error);
        // 返回状态对象
        return { hasError: error };
    }

    componentDidCatch(error, info) {
        // 统计页面的错误。发送请求发送到后台去
        console.log(error, info);
    }

    render() {
        return (
            <div>
                <h2>我是Parent组件</h2>
                {this.state.hasError ? <h2>当前网络不稳定，稍后再试</h2> : <Child />}
            </div>
        );
    }
}
```

# 9. 组件间通信方式总结

通信方式：

1. `props`

   父子间通过`props`

   `children props`

   `render props`

2. 消息订阅-发布

   `pubs-sub`等

3. 集中式管理

   `redux`等

4. `conText`

   生产者-消费者模式

组件之间的关系：

1. 父子组件 —— `props`
2. 兄弟组件（非嵌套组件） —— 消息订阅-发布、集中式管理
3. 祖孙组件（跨级组件） —— 消息订阅-发布、集中式管理、`conText`

***
欢迎在我的博客上访问：

https://lzxjack.top/