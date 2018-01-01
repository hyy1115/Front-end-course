### Link组件的用法
React-Router4中的Link是一个react组件，先从组件的用法入手，让你更好的理解他的实现。

1、基本用法
```javascript
<Link to={`/user`}>
</Link>
```
经过Link组件的处理后：
```javascript
<a href="#/user">
</a>
```
2、增加其他配置
```javascript
<Link 
    to={`/user`}
    innerRef={(refLink) => this.refLink = refLink}
    className={`item`}
>
</Link>
```
经过Link组件的处理后：
```javascript
<a 
    href="#/user"
    ref={(refLink) => this.refLink = refLink} //ref实际上不会显示在DOM上面，而是在js中绑定
    classname="item"
>
</a>
```
你还可以给Link设置id等其他自定义的属性，下面我们就来探索Link组件的源码，看看它是怎么实现的。

### Link组件的实现
从上面的例子中，我们知道了Link组件最重要的属性to，还有就是自定义属性的功能。我们先不看源码，假设就这2个需求，自己去写一个Link组件。

1、to的实现
```javascript
import React from 'react'
export default class Link extends React.Component {
    render() {
        const { to } = this.props
        return <a href={to} />
    }
}
```
2、其他属性配置的实现
要想实现自定义属性配置，只需要用到es6的解构。
```javascript
import React from 'react'
export default class Link extends React.Component {
    render() {
        const { to, ...props } = this.props
        return <a href={to} {...props} />
    }
}
```
如果你拿着这个7行代码的Link组件去使用，通常情况下也行，但是并不完美，因为官方还考虑到了其他需求。

**Link官方源码**

```
import React from 'react'
import PropTypes from 'prop-types'
import invariant from 'invariant'
import { createLocation } from 'history'

const isModifiedEvent = (event) =>
  !!(event.metaKey || event.altKey || event.ctrlKey || event.shiftKey)

class Link extends React.Component {
  static propTypes = {
    onClick: PropTypes.func,
    target: PropTypes.string,
    replace: PropTypes.bool,
    to: PropTypes.oneOfType([
      PropTypes.string,
      PropTypes.object
    ]).isRequired,
    innerRef: PropTypes.oneOfType([
      PropTypes.string,
      PropTypes.func
    ])
  }

  static defaultProps = {
    replace: false
  }

  static contextTypes = {
    router: PropTypes.shape({
      history: PropTypes.shape({
        push: PropTypes.func.isRequired,
        replace: PropTypes.func.isRequired,
        createHref: PropTypes.func.isRequired
      }).isRequired
    }).isRequired
  }

  handleClick = (event) => {
    if (this.props.onClick)
      this.props.onClick(event)

    if (
      !event.defaultPrevented && // onClick prevented default
      event.button === 0 && // ignore everything but left clicks
      !this.props.target && // let browser handle "target=_blank" etc.
      !isModifiedEvent(event) // ignore clicks with modifier keys
    ) {
      event.preventDefault()

      const { history } = this.context.router
      const { replace, to } = this.props

      if (replace) {
        history.replace(to)
      } else {
        history.push(to)
      }
    }
  }

  render() {
    const { replace, to, innerRef, ...props } = this.props

    invariant(
      this.context.router,
      'You should not use <Link> outside a <Router>'
    )

    const { history } = this.context.router
    const location = typeof to === 'string' ? createLocation(to, null, null, history.location) : to

    const href = history.createHref(location)
    return <a {...props} onClick={this.handleClick} href={href} ref={innerRef}/>
  }
}

export default Link
```
**官方源码还做了下面几样事情：**

1、增加PropTypes确保传入的值是合法的，这一步有人可能觉得没什么必要，包括我自己写react组件的时候，也不是每次都会写PropTypes，实在是每个组件都写这些太累人了，当然，最好你还是写上。当传入值格式不对的时候，会有报错提示，可以快速排查错误。
```javascript
static propTypes = {
    onClick: PropTypes.func, //点击事件得是个函数
    target: PropTypes.string, //target属性也是a标签常用到的
    replace: PropTypes.bool, //replace是个布尔值，作用下面会讲到
    to: PropTypes.oneOfType([
      PropTypes.string,
      PropTypes.object
    ]).isRequired, //to可以是字符串、对象
    innerRef: PropTypes.oneOfType([
      PropTypes.string,
      PropTypes.func
    ]) //innerRef可以是字符串、函数
  }
```
2、设置默认的replace值
至今我们还不知道replace用来做什么，你可以从他的中文意思或者是location的API去猜测：取代
```javascript
static defaultProps = {
    replace: false
  }
```
3、上下文设置
react中不推荐使用上下文，但在react-router，使用上下文是为了父子组件的通信。我们知道，react-router4中，必须使用<Router></Router>作为容器，在容器内部使用Link，我们先不管Router是如何实现的，在这里需要知道router作为上下文使用即可。
```javascript
static contextTypes = {
    router: PropTypes.shape({
      history: PropTypes.shape({
        push: PropTypes.func.isRequired, //push新增一个浏览器记录
        replace: PropTypes.func.isRequired, //替换当前的浏览器记录
        createHref: PropTypes.func.isRequired //创建href
      }).isRequired
    }).isRequired
  }
```

4、render方法的实现

a、和文章开头自己写的Link不同的是，官方Link增加了invariant()来判断是否存在上下文router，推断外层有无<Router></Router>。

```javascript
render() {
    const { replace, to, innerRef, ...props } = this.props
    invariant(
      this.context.router,
      'You should not use <Link> outside a <Router>'
    )

    const { history } = this.context.router
    const location = typeof to === 'string' ? createLocation(to, null, null, history.location) : to

    const href = history.createHref(location)
    return <a {...props} onClick={this.handleClick} href={href} ref={innerRef}/>
  }
```

b、如果存在上下文router，即读取history对象，那么history有什么方法呢？本文讲的是hashHistory，
browserHistory中的history实现有所不同，但他们的API是完全一样的。

```javascript
//hashHistory和browserHistory的API
history = {
    length: globalHistory.length,
    action: 'POP',
    location: initialLocation,
    createHref: createHref,
    push: push,
    replace: replace,
    go: go,
    goBack: goBack,
    goForward: goForward,
    block: block,
    listen: listen
  }
```
c、用typeof判断传入的to是否是字符串，如果是就使用createLocation()方法处理，createLocation有4个参数，路径、状态、键名、当前的location，createLocation(path, state, key, currentLocation)。
currentLocation函数也挺有意思的，这里提高该函数处理后，返回的location格式如下：
```javascript
location = {
  pathname: '',
  hash: '',
  state: *
}
```
记得to可以传字符串或者对象吗？如果传入的是对象，就不使用createLocation函数处理。也就是下面这样定义的情况：
```javascript
<Link to={{
  pathname: '/courses',
  search: '?sort=name',
  hash: '#the-hash',
  state: { fromDashboard: true }
}}/>
```
d、处理后的location可以直接绑定给href属性吗？不行，因为他是个对象，而实际上href得是个字符串，所以官方代码使用了history.createHref()方法转换了location。
```javascript
const href = history.createHref(location)
    return <a href={href} />
```
5、现在还剩最后一个环节，onClick事件的处理。有2个关键点，isModifiedEvent()和replace揭秘。
isModifiedEvent和if条件里面的几个event的属性，你可以看这篇文章：[event属性介绍][1]
replace如果设置为true，那么就使用history.replace(to)替换当前页面。

```javascript
const isModifiedEvent = (event) =>
  !!(event.metaKey || event.altKey || event.ctrlKey || event.shiftKey)

handleClick = (event) => {
    if (this.props.onClick)
    //如果存在自定义的onClick，则执行此自定义回调函数
      this.props.onClick(event)

    if (
      !event.defaultPrevented && // 事件的默认动作没有被禁用
      event.button === 0 && // 当鼠标左键被点击
      !this.props.target && // 没有传入类似target=_blank的属性
      !isModifiedEvent(event) // 点击操作时忽略其他几个键盘的点击
    ) {
      event.preventDefault()

      const { history } = this.context.router
      const { replace, to } = this.props

      if (replace) {
        history.replace(to)
      } else {
        history.push(to)
      }
    }
  }
```

### 总结
一个只有81行的组件被我讲的那么长篇大论，前半部分的实现已经够用，后半部分主要是react-router自身的架构需求。理解了就不难，你也可以自己尝试写一个简化版的Link，不需要上下文那种。

  [1]: http://www.w3school.com.cn/jsref/dom_obj_event.asp