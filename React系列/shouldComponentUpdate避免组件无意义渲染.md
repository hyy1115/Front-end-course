state有时候很不听话，在某些时候，我不想他渲染，偏偏react非常智能的帮我们重复渲染。

比如最常见的就是传递的对象为空，组件依旧渲染了一次或者多次。

更多场景不举例了，对症下药。

shouldComponentUpdate是react提供的生命周期函数，他发生在接收到新的props的时候。
简单介绍一下各个生命周期函数。

    componentWillMount：组件挂载之前执行，只执行一次
    
    componentDidMount: 组件渲染完成，只执行一次
    
    =======================================================
    
    componentWillRecevieProps: 组件将要接收新的props执行
    
    shouldComponentUpdate: 判断组件是否应该重新渲染，默认是true
    
    componentWillUpdate: 组件将要重新渲染
    
    componentDidUpdate: 组件重新渲染完成
    
    =======================================================
    
    componentWillUnmount: 卸载组件

组件生命周期是有顺序的，首先挂载组件，挂载成功完成第一次渲染，然后传递新的props，则会触发componentWillRecevieProps，执行重新渲染的周期，直至渲染完成。

### 简单版本
在你的组件内部加上这段代码
  
component.js  
    shouldComponentUpdate(nextProps, nextState) {
        if (_.isEqual(this.props, nextProps) || !_.isEmpty(this.props)) {
            return false
        }
        return true
    }
    
这里用到了_.isEqual和_.isEmpty，_.isEqual判断当前传进来的值和下一次传递的值是不是相等，是则返回true，_.isEmpty判断当前传递进来的对象是不是为空，为空则返回true。

_.isEqual和_.isEmpty是 [lodash][1] 插件里面的函数，这是个轻巧的JavaScript函数插件，可以处理多种常见的数据操作，当然还有一个更多功能的插件。

在你的react项目的入口js导入lodash，因为lodash函数是全局的，所以只需要在入口导入一次即可。

安装

```
npm install --save lodash
```

app.js
```
import ‘lodash’
```

### 高级版本

还有一种更加高级好用的写法，除了比较props之外，我们同时需要比较state是否更新。
```javascript
if (!_.isEqual(this.props, nextProps) || !_.isEqual(this.state, nextState)) {
   return true
} else {
   return false
}
```

### 其他写法
如果组件结构比较简单，你可以使用React.PureComponent的方式创建组件。


  [1]: http://lodashjs.com/docs/#_isemptyvalue