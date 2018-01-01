#### 引言
常常有人问我，react如何学起？

我一般会告诉他，github上找个demo下载到本地运行，再结合网上的一些教程看代码。

但那些大部分是react技术栈的demo，无法让你更简单的理解react的组件化思想。

首先推荐一个在线运行react的网站，这也是dan大神推荐的 [http://codepen.io/gaearon/pen/ZpvBNJ?editors=0010][2]，在该网站上面，你可以写任意的react代码，实时显示出来渲染结果。

用react做开发，可以分2大点：

1、react技术栈框架搭建；

2、react编程思想。

技术栈搭建即使对于有一定react开发经验的人来说，都有一些困难，我们先了解react编程思想中组件的实现。现在请你打开 [http://codepen.io/gaearon/pen/ZpvBNJ?editors=0010][3]，跟着我写react代码。


#### hello world react
react很多实践是SPA应用，而SPA通常只有一个入口，ReactDOM.render()会把组件或者jsx渲染在根元素“root”下，比如这个例子中会在root下面渲染 <h1>Hello, world!</h1>
```
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```
![clipboard.png](/img/bVNtQ3)

#### App根组件

我们把 <h1>Hello, world!</h1> 用react组件的形式来写，这里也提到了props的用法，用来从父组件传递属性给子组件。
```
class App extends React.Component {
  render() {
    return (
      <div>
        hello {this.props.name}!
      </div>
    )
  }
}

ReactDOM.render(
  <App name="world" />,
  document.getElementById('root')
);
```

![clipboard.png](/img/bVNtTG)

#### 父组件嵌套子组件，子组件嵌套子组件。
这里面比较绕，但是你可以知道的是父子组件可以任意嵌套，每个组件都像一个盒子，盒子里面可以装下大盒子或者小盒子。
我们增加一个子组件child，在父组件App中调用子组件，child又嵌套了一个表单组件。Forms是Child的子组件，Forms的结构和Child有些不同，比如在App中调用Child是<Child />，在Child中调用Forms是<Forms>你的内容</Forms>。
先来看Child：Child定义了一个state用来保存输入框的值，当onChange事件发生的时候，输入框的值会通过handleClick实时保存到state中，我们把value传递到Forms组件里面同步显示出来。
Forms组件：Forms可以嵌套jsx，因为在Forms组件的内部，我使用了this.props.children这个属性，用来表示Forms中嵌套节点的传递，在这里就是传递input。

```
class Child extends React.Component {
  
  constructor(props) {
    super(props)
    this.state = {
      value: undefined
    }
    this.handleClick = this.handleClick.bind(this)
  }

  handleClick(e) {
    this.setState({
      value: e.target.value
    })
  }
  
  render() {
    return (
      <div>
        子组件
        <Forms value={this.state.value}>
          <input type="text" onChange={(e) => this.handleClick(e)} />
        </Forms>
      </div>
    )
  }
}

class Forms extends React.Component {
  render() {
    return (
      <form>
        {this.props.children}
        您输入的值是：{this.props.value}
      </form>
    )
  }
}
```

![clipboard.png](/img/bVNtZX)

这么一个简单的在线实时编辑教程，教给你react组件的常用写法、嵌套、属性传递、jsx传递、单入口等知识，相信你对react会有个基本的认识。

手痒的就赶紧去在线编辑试试吧，你还可以写入官网demo的代码试试哦。

demo在线编程地址：[http://codepen.io/hyy1115/pen/vmdrgo?editors=1010][4]


  [1]: /img/bVZwRf
  [2]: http://codepen.io/gaearon/pen/ZpvBNJ?editors=0010
  [3]: http://codepen.io/gaearon/pen/ZpvBNJ?editors=0010
  [4]: http://codepen.io/hyy1115/pen/vmdrgo?editors=1010