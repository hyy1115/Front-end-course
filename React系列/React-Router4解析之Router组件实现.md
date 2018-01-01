### Router组件的使用
react-router中有Router和Route两种看起来名字很接近的组件，但他们的作用不一样，Router作为顶级的容器使用，而Route作为一个子路由组件。react-router4的思想是一切皆组件，在使用的时候，可以把Route当做页面组件，也可以当做普通组件。**别再把Router和Route搞混了！**
```javascript
import { Router } from 'react-router'
import createBrowserHistory from 'history/createBrowserHistory'

const history = createBrowserHistory()

<Router history={history}>
  <App/>
</Router>
```

### Router组件的实现
只从上面的使用方法，仅能看出Router的一个功能，提供history API接口，其实更重要的是它的上下文功能。假设我们只考虑history属性的配置，自己写一个简易版的Router组件。这样做是为了从零开始构建一个Router，从而发现它的秘密。

你如果好奇为什么需要上下文，而不是props，可以看一个大神的解释：[React中的“虫洞”][1]

1、带有history属性的Router的实现
首先思考一下，history传入Router之后，要用来做什么？
下面的代码实现了渲染一个子组件的Router，可history要放在那里呢？
```javascript
import React from 'react'
export default class Router extends React.Component {
    render() {
        const { children, history } = this.props  
        return React.Children.only(children)
    }
}
```
2、history在Router组件中的作用揭秘
我们知道Router组件最重要的功能就是提供上下文。在Link和Route这些作为Router的子组件中，需要靠Router的上下文来通信。
所以Router组件有2个作用：
**作为顶层的容器；**
**给它的子组件提供上下文。**

知道了上下文的作用之后，history的作用就很明显了。我们需要了解react上下文如何实现。

和上下文有关的API有3个：contextTypes、childContextTypes、getChildContext

contextTypes： react的静态属性，定义需要接收的上下文对象。

childContextTypes：react的静态属性，定义传递给子组件的上下文对象。

getChildContext：react方法，获取子组件中的某个上下文对象（如果有更新的话）。

**看API很抽象，具体看一下Router的源码。**

首先定义childContextTypes，定义了上下文router对象。
```javascript
import React from 'react'
export default class Router extends React.Component {
    static childContextTypes = {
        router: PropTypes.object.isRequired
      }
    render() {
        const { children } = this.props  
        return React.Children.only(children)
    }
}
```
router对象具体包含哪些属性，就得使用getChildContext()方法。像history、match、location都是经常用到的。
```javascript
import React from 'react'
export default class Router extends React.Component {
    static childContextTypes = {
        router: PropTypes.object.isRequired
      }
    getChildContext() {
      return {
        router: {
          ...this.context.router,
          history: this.props.history,
          route: {
            location: this.props.history.location,
            match: this.state.match
          }
        }
      }
    }
    render() {
        const { children } = this.props  
        return React.Children.only(children)
    }
}
```
在父组件定义了上下文，如果父组件想要接收子组件的上下文更新，就需要contextTypes。
```javascript
import React from 'react'
export default class Router extends React.Component {
    static contextTypes = {
      router: PropTypes.object
    }
    static childContextTypes = {
      router: PropTypes.object.isRequired
    }
    getChildContext() {
      return {
        router: {
          ...this.context.router,
          history: this.props.history,
          route: {
            location: this.props.history.location,
            match: this.state.match
          }
        }
      }
    }
    render() {
        const { children } = this.props  
        return React.Children.only(children)
    }
}
```
在子组件接收Router父组件的上下文很简单，比如Link组件。在Link组件中定义静态属性contextTypes的对象为父组件中的router对象即可用this.context.router读取上下文了。
```javascript
import React from 'react'
import PropTypes from 'prop-types'

export default class Link extends React.Component {
  static defaultProps = {
    replace: false
  }
  //重点
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
      const { history } = this.context.router //重点
      const { replace, to } = this.props
      if (replace) {
        history.replace(to)
      } else {
        history.push(to)
      }
    }
  }
  render() { }
}
```

**上下文在Router组件中的应用学习完之后，下面讲解Router中最后一个重要的功能：match**

match是一个封装好的方法，它返回一个对象，这个对象你肯定很熟悉，它就是Route组件的属性配置。
```javascript
{
  path: '/',
  url: '/',
  params: {},
  isExact: pathname === '/'
}
```
match作为上下文属性中通信的最重要的功能，它可以监听路由的变化，利用了react的state可以实现自动更新组件的特性。
```javascript
//获取取当前的pathname
state = {
    match: this.computeMatch(this.props.history.location.pathname)
  }
computeMatch(pathname) {
    return {
      path: '/',
      url: '/',
      params: {},
      isExact: pathname === '/'
    }
  }
```
当创建Router组件的时候，实现一个监听history改变的方法。history.listen()来自于history插件，想要研究listen的实现请自己查看history源码。
```javascript
componentWillMount() {
    const { children, history } = this.props
    this.unlisten = history.listen(() => {
      this.setState({
        match: this.computeMatch(history.location.pathname)
      })
    })
  }
```
通过上面这几个步骤，就实现了一个完整的Router容器组件。


### Router源码（省去了错误提示的代码） 
```
import React from 'react'
import PropTypes from 'prop-types'

export default class Router extends React.Component {
  static propTypes = {
    history: PropTypes.object.isRequired,
    children: PropTypes.node
  }
  static contextTypes = {
    router: PropTypes.object
  }
  static childContextTypes = {
    router: PropTypes.object.isRequired
  }
  getChildContext() {
    return {
      router: {
        ...this.context.router,
        history: this.props.history,
        route: {
          location: this.props.history.location,
          match: this.state.match
        }
      }
    }
  }
  state = {
    match: this.computeMatch(this.props.history.location.pathname)
  }
  computeMatch(pathname) {
    return {
      path: '/',
      url: '/',
      params: {},
      isExact: pathname === '/'
    }
  }
  componentWillMount() {
    const { children, history } = this.props
    this.unlisten = history.listen(() => {
      this.setState({
        match: this.computeMatch(history.location.pathname)
      })
    })
  }
  componentWillUnmount() {
    this.unlisten()
  }
  render() {
    const { children } = this.props
    return children ? React.Children.only(children) : null
  }
}
```

### 总结
Router组件实现了3个功能：
1、上下文
2、监听history
3、自身封装成一个容器组件

ps：感谢你的阅读，如果感兴趣后续文章，请关注专栏《前端架构经验分享》

  [1]: https://segmentfault.com/a/1190000004636213