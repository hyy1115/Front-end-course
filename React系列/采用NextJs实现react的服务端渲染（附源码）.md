最近心血来潮，想尝试一下react的服务端渲染，说一下我一开始的思路到最后的实现。

### 思路分析
刚开始没有具体的思路，看了react官网的服务端渲染API，看了react-router4官网的服务端渲染DEMO，接着在网上搜索了一些别人写的博客，这方面博客挺少，而且介绍不全面。

我的想法是在现有客户端源码的基础上（https://github.com/hyy1115/react-redux-webpack3），加上服务端渲染的代码。

恩，这看起来似乎很简单，接着我就开始研究，我先是构建了一个server.js和client.js来区分不同的环境，并且在server.js中调用了react/server的API渲染了一个静态页面，这一步很正常，没有任何问题。

接着，在我以为很快就能实现的时候，我把渲染的入口改成了react-router4推荐的静态路由模式，啪啪啪，超多的error扑面而来，染红了terminal。我想了想，伤心了一下。

都是些什么错误呢？

1、css报错，不管是less、css、scss，始终无法解析，原因在于服务端不支持解析。

2、图片报错，图片也无法解析。

3、window对象下的变量报错，服务端是global环境。

4、js的各种报错。

5、路径报错。

7、未知的报错。

一整天我都在修复这些bug，最后太多太多，让我心累，就去咨询了一个大佬，大佬告诉我，使用nextjs，真是一语惊醒梦中人，我还自己配置这些环境太傻了吧。

### NextJS框架
Next地址：https://github.com/zeit/next.js

#### next是什么
react服务端渲染框架

#### next简介
推荐你点击上面的网址看看readme的介绍，非常详细。

## react-next服务端渲染项目搭建

**项目地址：[https://github.com/hyy1115/react-next][1]**

#### 实现的功能
```text
1、服务端渲染，支持react16
2、webpack，包括开发环境下的热更新等功能，以及部署环境下的打包功能
3、babelrc配置
4、支持scss、css、css-jsx
5、支持next-router
6、支持redux
7、支持图片格式文件
8、支持axios
```

### 注意事项
使用next框架，你仍旧可以采用客户端react组件的写法，只是在redux和router的选择上，使用next集成的redux和router功能，项目结构上面，和客户端渲染的写法有些许不同。

**详细教程晚些时候会在项目中更新。**

![图片描述][2]


  [1]: https://github.com/hyy1115/react-next
  [2]: /img/bVWJ5y