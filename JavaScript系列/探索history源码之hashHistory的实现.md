### history简介
我们不探寻它的历史，只关注技术，通常有2种history，分别是hashHistory和browserHistory，本文带领大家从零开始实现一个hashHistory。

```javascript
hashHistory：'#/home'
browserHistory: '/home'
```

下面的实现方案是根据官方history源码来分析的，你可以下载[hashHistory源码][1]结合本文学习。

### 实现方案
**1、创建createHashHistory函数**
```javascript
const createHashHistory = () => { 
    const history = {}
    return history
}
export default createHashHistory

```
**2、先要了解history对象长什么样，接着，我们一个个去实现它**
```javascript
history = {
    length: 1, //Number
    action: "POP", //String
    location: {}, //Object
    createHref, //函数
    push, //函数
    replace, //函数
    go, //函数
    goBack, //函数
    goForward, //函数
    listen //函数
}
```
**3、实现length**
在window下面有一个history对象，可以用来获取length。
```javascript
const globalHistory = window.history
history = {
    length: globalHistory.length
}
```
**4、action默认为POP，它还可能是PUSH或者REPLACE。我们不在这一步实现它，等下面实现push和replace的时候再来实现。**

**5、实现location**
location对象包含下面几个key，这里能用到的是pathname。history.location和window.location是不一样的，history.location是window.location的精简版。你可以在浏览器控制台打印window.location看一下完整的location对象。
```javascript
location = {
    hash:"",
    pathname:"/",
    search:"",
    state:undefined
}
```
定义一个getDOMLocation函数，用来获取封装后的location。
```javascript
const decodePath = path =>
  path.charAt(0) === "/" ? path : "/" + path

const getHashPath = () => {
    //如果url存在#，则去掉#，返回路径
    //比如："http://localhost:8080/#/"，返回'/'
    const href = window.location.href
    const hashIndex = href.indexOf("#")
    return hashIndex === -1 ? "" : href.substring(hashIndex + 1)
}

const getDOMLocation = () => {
    //getHashPath截取url的路由，如果存在#，则去掉#
    let path = decodePath(getHashPath())     
    //创建location
    return createLocation(path)
}
```
这一步的核心就是createLocation()的实现。但是，它不复杂，只是代码有点长，如果要了解，请看源码 [createLocation][2]。

**6、实现createHref**
你可能没有用过history.createHref()，它用来创建一个hash路由，也就是'#/'或者'#/home'这类的。
```javascript
const createPath = location => {
  const { pathname, search, hash } = location
  let path = pathname || "/"
  if (search && search !== "?")
    path += search.charAt(0) === "?" ? search : `?${search}`
  if (hash && hash !== "#") path += hash.charAt(0) === "#" ? hash : `#${hash}`
  return path
}

const createHref = location =>
    "#" + encodePath(createPath(location))
```

**7、实现push方法**
我们在使用push的时候，通常是history.push('/home')这种形式，不需要自己加#。
push实现的原理：判断push传入的路由和当前url的路由是否一样，如果一样，则不更新路由，否则就更新路由。
```javascript
//更新history对象的值，length、location和action
const setState = nextState => {
    Object.assign(history, nextState)    
    history.length = globalHistory.length        
    transitionManager.notifyListeners(history.location, history.action)
}
//notifyListeners函数用来通知history的更新
 const notifyListeners = (...args) => {
    listeners.forEach(listener => listener(...args))
  }
//更新路由
const pushHashPath = path => (window.location.hash = path)

//push核心代码
const push = (path, state) => {
    //更新action为'PUSH'
    const action = "PUSH"
    //更新location对象
    const location = createLocation(
       path,
       undefined,
       undefined,
       history.location
    )   
    //更新路由前的确认操作，confirmTransitionTo函数内部会处理好路由切换的状态判断，如果ok，则执行最后一个参数，它是回调函数。
    transitionManager.confirmTransitionTo(
       location,
       action,
       getUserConfirmation,
       ok => {
           //如果不符合路由切换的条件，就不更新路由
           if (!ok) return               
           //获取location中的路径pathname，比如'/home'
           const path = createPath(location)
           const encodedPath = encodePath(path)
           //比较当前的url中的路由和push函数传入的路由是否相同，不相同则hashChanged为true。
           const hashChanged = getHashPath() !== encodedPath          
           if (hashChanged) {
               //路由允许更新
               ignorePath = path
               //更新路由
               pushHashPath(encodedPath)              
               const prevIndex = allPaths.lastIndexOf(createPath(history.location))
               const nextPaths = allPaths.slice(0, prevIndex === -1 ? 0 : prevIndex + 1)                    
               nextPaths.push(path)
               allPaths = nextPaths     
               //setState更新history对象。               
               setState({ action, location })
          } else {
              //push的路由和当前路由一样，会发出一个警告“Hash history cannot PUSH the same path; a new entry will not be added to the history stack”
              setState()
          }
       }
    )
}
```
**8、实现replace**
replace和push都能更新路由，但是replace是更新当前路由，而push是增加一个历史记录。
```javascript
//更新路由
const replaceHashPath = path => {
    const hashIndex = window.location.href.indexOf("#")   
    window.location.replace(
        window.location.href.slice(0, hashIndex >= 0 ? hashIndex : 0) + "#" + path
    )
}
    //replace核心代码
    const replace = (path, state) => {
        const action = "REPLACE"
        const location = createLocation(
            path,
            undefined,
            undefined,
            history.location
        )      
        transitionManager.confirmTransitionTo(
            location,
            action,
            getUserConfirmation,
            ok => {
                if (!ok) return              
                const path = createPath(location)
                const encodedPath = encodePath(path)
                const hashChanged = getHashPath() !== encodedPath  
                //到这里为止，前面的代码和push函数的实现都是一样的
                             
                if (hashChanged) {
                    ignorePath = path
                    //更新路由
                    replaceHashPath(encodedPath)
                }                
                const prevIndex = allPaths.indexOf(createPath(history.location))                
                if (prevIndex !== -1) allPaths[prevIndex] = path               
                setState({ action, location })
            }
        )
    }
```

**9、实现go**
go方法的使用是history.go(-1)这种形式
```javascript
//globalHistory是window.history
const go = n => globalHistory.go(n)
```
**10、实现goBack**
这个应该能一眼看懂了
```javascript
const goBack = () => go(-1)
```
**11、实现goForward**
这个应该也能一眼看懂了
```javascript
const goForward = () => go(1)
```
**12、实现listen**
```javascript
const listen = listener => {
    const unlisten = transitionManager.appendListener(listener)
    checkDOMListeners(1)    
    return () => {
        checkDOMListeners(-1)
        unlisten()
    }
}

//监听hashchange的改变，handleHashChange函数用来判断是哪种类型的路由更新，replace、push等各种hash改变都实现了一个函数，具体看源码。
const checkDOMListeners = delta => {
    listenerCount += delta    
    if (listenerCount === 1) {
        //注册监听函数
        window.addEventListener('hashchange', handleHashChange)
    } else if (listenerCount === 0) {
        //移除监听函数
        window.removeEventListener('hashchange', handleHashChange)
    }
}

  //appendListener函数实现
  let listeners = []
  const appendListener = fn => {
    let isActive = true
    const listener = (...args) => {
      if (isActive) fn(...args)
    }
    listeners.push(listener)
    return () => {
      isActive = false
      listeners = listeners.filter(item => item !== listener)
    }
  }
```

### 总结
history对象的所有属性和方法都实现了一遍，在react-router中，将history对象封装进了Router、Route等组件中，使得你可以在react组件中通过this.props.history读取。

看完源码，你会发现history的实现真的不复杂，找准思路，一个个函数去实现，再考虑兼容性，就非常完美了，以后你在其他博客上看到有人宣传自己搞了个自己的路由插件，不要觉得很牛逼，重构history换个新姿势就是一个插件了。

  [1]: https://github.com/ReactTraining/history/blob/master/modules/createHashHistory.js
  [2]: https://github.com/ReactTraining/history/blob/master/modules/LocationUtils.js