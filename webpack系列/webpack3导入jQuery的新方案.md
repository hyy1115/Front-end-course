### 本文的目的
#### 拒绝全局导入jQuery！！
#### 拒绝script导入jQuery！！
#### 找到一种只在当前js组件中引入jQuery，并且使用webpack切割打包的方案！

### 测试环境

以下测试在webpack3.8.1，jQuery3.2.1，react16+中进行

### 思路分析

如果说要我在react中全局引入jQuery，我是十分感动，然后拒绝的。

但是，有时候可能react的一些库不够牛逼，还需要用到jQuery的相关插件来辅助完成，这些插件又和jQuery形成了依赖，最终，和我一样，你也可能需要在react中导入jQuery。

这个时候webpack就派上用场了，你也别百度了，网上的方案我试过很多，说句不好听的，大部分都是乐色！

举个例子，很多博客说用下面这种方案，还有其他一堆乱七八糟的辅助方案。

```javascript
new webpack.ProvidePlugin({
    $: 'jquery',
    jQuery: 'jquery',
    'window.jQuery': 'jquery',
    'window.$': 'jquery',
  });
```

一开始的尝试，我以为是成功的，因为$可以打印出来了啊！但是，当我打印jQuery的时候，报错了!!

```javascript
jQuery is not defined
```
接着，就是一个漫长的探索过程，我以为是CMD的锅、我以为是AMD的锅、我还以为是ES6的锅、甚至我坚定的认为是webpack的锅！！

### 最终答案

最终我发现就是webpack的锅，幸好webpack提供了另外一种支持方案。

1、安装expose-loader

```
npm install --save expose-loader
```

2、在webpack.config中加入下面这段loader代码
```
{
   test: require.resolve('jquery'),
   use: [{
      loader: 'expose-loader',
      options: 'jQuery'
   },{
      loader: 'expose-loader',
      options: '$'
   }]
}
```

3、下面该干嘛？放心，你什么都不用干了，接着很轻松的在你的react组件中导入jQuery

```
import React from 'react'

require('jquery')
require('jQuery第三方插件')

class Components extends React.Component {
    constructor(props) {
        super(props)
    }
    componentDidMount() {
         $(document).ready(function() {
            //做爱做的事情
        })
    }
}

```

4、这里可能还存在一个小坑，就是很多jQuery第三方插件的作者写的代码不规范，我就遇到了一些变量没有声明的情况，在那些老程序员眼里，js变量不声明表示全局变量，但在webpack眼里，你不声明就未定义了！如果你遇到jQuery插件未定义的报错，通常给这个变量加上var就行了！

5、最后，我自己写的组件本身已经融入了异步打包功能，所以当前包含jQuery的react组件不会污染其他react组件，不会导致其他组件的体积变大，也不会导致公共js的体积变化，前提是你也实现了react组件的异步加载功能。

6、**关于webpack异步打包组件的方案，请看我的其他文章！**

#### 只要你使用了webpack，无论是react，还是vue开发者也同样适用这种方案
