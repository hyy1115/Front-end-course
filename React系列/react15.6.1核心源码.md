# React核心源码
React对象中，你可能会看到一些熟悉的API，比如React.Component。你也可以看 [React官方教程][1] 了解每个API的使用方式。

```javascript
const React = {
  Children: {
    map: ReactChildren.map, //使用方法React.Children.map(this.props.children, children => children)
    forEach: ReactChildren.forEach,
    count: ReactChildren.count,
    toArray: ReactChildren.toArray,
    only: onlyChild //React.Children.only(this.props.children)
  },

  Component: ReactBaseClasses.Component, //最基础的react组件基类
  PureComponent: ReactBaseClasses.PureComponent, //在shouldComponentUpdate执行了浅对比，当数据结构层级不复杂时，你可以使用它替代Component

  createElement, //这是一种ES5创建react组件的写法，更喜欢ES6的就忽略它
  cloneElement, //克隆一个新的react元素，保留原来的props和ref
  isValidElement: ReactElement.isValidElement, //验证是否react元素，返回布尔值

  PropTypes: ReactPropTypes, //该方法已经移除，使用prop-types插件替代
  createClass: createReactClass, //不推荐使用它
  createFactory, //推荐使用createElement替代它
  createMixin, //被遗弃，使用https://fb.me/createmixin-was-never-implemented for more info替代

  DOM: ReactDOMFactories, //提供了各种DOM元素的创建，例如：React.DOM.svg()

  version: ReactVersion, //一个查看react版本号的东东

  __spread //被遗弃
};
```

#### 下面看几个常用API的实现方式

### React.Component核心源码
```javascript
class ReactComponent {
	//基类
    constructor(props, context, updater) {
    	//定义了4个成员
        this.props = props;
        this.context = context;
        this.refs = emptyObject;
        this.updater = updater || ReactNoopUpdateQueue;
    }
    //setState的使用大家已经很熟悉了，推荐this.setState(() => {return {state}})
    setState(partialState, callback) {
        this.updater.enqueueSetState(this, partialState);
        if (callback) {
            this.updater.enqueueCallback(this, callback, 'setState');
        }
    }

    //forceUpdate绕过shouldComponentUpdate，直接更新state，避免使用它
    forceUpdate(callback) {
        this.updater.enqueueForceUpdate(this);
        if (callback) {
            this.updater.enqueueCallback(this, callback, 'forceUpdate');
        }
    }
}
```

**setState:**
有人可能需要在执行完setState之后读取到新的state，比如：
```javascript
state = {count: 0}
handleClick() {
	this.setState((prevState) => {
        return {count: prevState.count + 1}
    })
    console.log('count: ', this.state.count) //读取到的值是0
}
```
你可能希望读到新的值1，其实很简单，使用async。
```javascript
state = {count: 0}
async handleClick() {
	await this.setState((prevState) => {
        return {count: prevState.count + 1}
    })
    console.log('count: ', this.state.count) //读取到的值是1
}
```

### React.PureComponent核心源码
```javascript
function ReactPureComponent(props, context, updater) {
    this.props = props;
    this.context = context;
    this.refs = emptyObject;
    this.updater = updater || ReactNoopUpdateQueue;
}

```


[1]: https://facebook.github.io/react/docs/react-api.html