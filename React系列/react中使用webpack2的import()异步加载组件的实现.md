你还在用require.ensure()？？？

low了！！

import()的大名听过没？？

下面隆重介绍webpack代码切割新方案。

官网教程在这里：[https://doc.webpack-china.org/guides/code-splitting-async/#system-import-][1]

**先别去看官网教程，因为webpack官网教程通常都比较不靠谱，写的云里雾里。**

webpack中，从v1到v2，v3暂且不提，一共有3种代码切割加载的方案。

1、System.import()； 已废除，不推荐

2、require.ensure()； v1和v2均可使用

3、import()；v2支持，v1不支持

#### System.import()

已废除，就不做介绍了

#### require.ensure()

从v1时代，使用过webpack的同学对这种用法应该不会陌生，比较网上可以搜到很多关于require.ensure()的用法的demo。

在百度输入require.ensure()，搜到几个关键的教程。

[webpack2.2文档关于require.ensure使用的介绍][2]

简单介绍下require.ensure的经典使用方法，你若是没用过，别担心，看下面的代码就足够了。
    const Foo = require.ensure([], () => {
        require("a");
    }, err => {
        console.error("We failed to load chunk: " + err);
    }, "chunk-name");
    
    //react-router2 or 3
    <Route path="/xx" getComponent={Foo} />

#### import()

这是我们今天的主角，使用场景是react-router4，你会发现在react-router4中，route的getComponent不见了。

怎么办，这时候你可以去看一下最上面给你的官方代码切割教程，列举了3种使用情况。

1、导入局部模块

    function determineDate() {
      import('moment').then(function(moment) {
        console.log(moment().format());
      }).catch(function(err) {
        console.log('Failed to load moment', err);
      });
    }
    
    determineDate();
    
2、导入整个模块

    import('./component').then(Component => /* ... */);


3、使用await

    async function determineDate() {
      const moment = await import('moment');
      return moment().format('LLLL');
    }
    
    determineDate().then(str => console.log(str));


这3个官方例子都有一个共同点，那就是：import(name) -> Promise

我们发现import()方法返回的是一个Promise，啥？Promise？有人会问，Promise咋了。我直接赋给变量就可以用了啊。

    const Foo = import("./xx") // 错误的写法
    
    <Route path="/xx" component={import("./xxx")} /> //错误的写法
    
少年，别太天真，Promise的返回值只能通过then()来读取，所以你会发现官方的3种使用场景全都是在then()里面操作，那怎么办？我想把import()获取到的组件赋给一个变量或常量。

通过我坚持不懈的chrome，终于发现了一个“惊天函数”。

懂行的人也许会问，react-router4官网文档不是有个[Bundle组件][3]可以实现代码切割吗？

是啊，官方的这个方法我复制过来使用了之后，一堆红色报错啊，被欺骗的❤️。

不废话，直接上可用代码。

1、asyncComponent函数（惊天函数）：函数很好理解，loadComponent参数表示需要代码切割的路径，函数返回值是一个react组件，组件内部帮你做好了then()方法的操作。
    import React from 'react'
    export const asyncComponent = loadComponent => (
        class AsyncComponent extends React.Component {
            state = {
                Component: null,
            }
    
            componentWillMount() {
                if (this.hasLoadedComponent()) {
                    return;
                }
    
                loadComponent()
                    .then(module => module.default)
                    .then((Component) => {
                        this.setState({ Component });
                    })
                    .catch((err) => {
                        console.error(`Cannot load component in <AsyncComponent />`);
                        throw err;
                    });
            }
    
            hasLoadedComponent() {
                return this.state.Component !== null;
            }
    
            render() {
                const { Component } = this.state;
                return (Component) ? <Component {...this.props} /> : null;
            }
        }
    );

2、在react中使用

    import { asyncComponent } from './AsyncComponent'
    
    const Foo = asyncComponent(() => import(/* webpackChunkName: "foo" */ "./foo"))
    
    <Route path="/xx" component={Foo} />

3、好了，这样你就成功了，但是，请注意下面几个问题：

**webpack2的配置文件中，需要配置chunkName。**

    chunkFilename: '[name].js'

**如果你的异步加载组件有导入样式，请把样式移植到全局js文件导入。**

好了，本文到此结束。

“小朋友，等等，你有demo吗？看文字我看不懂啊！”

我回头问道：“你想看demo？那就请看 [二月的import()切割实现][4]，感兴趣可以持续关注，如果webpack出了更好的玩意，我会持续跟进应用到项目中。”


**如果文章对你有帮助，请点击一下推荐。**
  [1]: https://doc.webpack-china.org/guides/code-splitting-async/#system-import-
  [2]: http://www.css88.com/doc/webpack2/guides/code-splitting-require/
  [3]: https://reacttraining.cn/web/guides/code-splitting
  [4]: https://github.com/hyy1115/react-redux-webpack2/blob/master/src/App.js