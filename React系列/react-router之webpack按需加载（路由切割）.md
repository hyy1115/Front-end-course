webpack工具相信用react的同学都比较熟悉了，一个很爽的功能——**热更新**，稍微改个分号都能够在浏览器局部刷新，很厉害有木有。

安静一下，同学们，不要喧哗！

本章内容不讲热更新，我们来看看webpack的另外一个功能——**代码切割（或者叫做路由切割）**。

作为react开发者，你应该用过react-router插件吧？没用过的就不要花时间看下面的内容了。

react-router把每个页面包装成了一个唯一的路由，这样的话，一个页面就对应了一个路由（改变hash的算同一个路由），我们在用webpack打包的时候，通常将主要目录下的所有js打包成一个单独的bundle.js（名字随便取）文件，这样一来，只需要首次加载js，然后通过ajax请求json资源，只要你不刷新网页，就不需要重新请求bundle.js，但是也会带来一些问题，就是首屏渲染太慢、SEO。

一个小小的react应用，用webpack打包之后，都可能达到1M以上，然后大家就从网上搜索各种webpack压缩方案，包括UglifyJsPlugin压缩（很有效）、设置NODE_ENV为production、服务器端压缩GZIP等，想尽各种办法，展现了前端工程师无与伦比的美丽。

上面这些办法对于一些中小型应用来说，压缩方案已经足够了，但是当项目大到了一个限度之后，无论你怎么压缩，都无法做到压缩到合适的大小。那么前端界的大神们也想到了一个办法，**服务端渲染**（说抄java、php，还别不信），服务端渲染很容易理解，就是根据前端请求的路由返回对应的资源，而对于前端来说，这些资源就是一个个切割好的js。

看文字有点累，下面贴上代码给大家看一下基本的react-router长什么样。

```
import React from 'react';
import { Route } from 'react-router';

/* containers */
import { AppContainer } from 'appContainer'
import { HomeContainer } from 'containers/Home/homeContainer'//首页
import { SearchContainer } from 'containers/Search/searchContainer'//搜索页面

export default (
    <Route path="/" component={AppContainer}>
        <Route path="home" component={HomeContainer} />
        <Route path="search" component={SearchContainer} />
    </Route>
)
```

这样的写法用webpack打包出来的是一个完整的bundle.js，现在start 切割。

1、修改你的路由

```
import React from 'react';
import { Route, IndexRoute } from 'react-router';

import { AppContainer } from './appContainer';
import { HomeContainer } from './containers/Home/homeContainer';
//首页不变，搜索页面是子页面，我把他切割出来作为单独的一个js文件，cb里面有一个default，表示导出带有**default**的容器组件。
const searchContainer = (location, cb) => {
    require.ensure([], require => {
        cb(null, require('./containers/Search/searchContainer').default)
    },'search')
}

export default (
    <Route path="/" component={AppContainer}>
        <IndexRoute component={HomeContainer} />
        <Route path="home" component={HomeContainer} />
        <Route path="search" getComponent={searchContainer} />
    </Route>
);


```
2、是不是很简单，你是不是要问，这就可以了？当然不是，还有一个需要注意的地方，就是在webpack的配置文件要加上这样一句话。

```
//入口文件
entry: {
    app: [
      'babel-polyfill',
      './src/index' 
    ],
    vendor: ['react'] //提取react模块作为公共的js文件
  },

//输入文件
output: {
    filename: '[name].js', //注意这里，用[name]可以自动生成路由名称对应的js文件
    path: path.join(__dirname, 'build'),
    publicPath: '/build/',
    chunkFilename: '[name].js' //注意这里，用[name]可以自动生成路由名称对应的js文件
  },

//插件
plugins: [
//必须配置，react的公共模块
    new webpack.optimize.CommonsChunkPlugin({
      names: ['vendor'],
      filename: 'vendor.js'
    })
  ],
```
3、然后呢？然后就没了，这样就配置好了！真的相信我，start切割没那么难，只不过网上很多教程没有说明react-router和webpack的关联，特别是**[name].js**这一点很重要，你要是不这样写，就不能打包成对应的路由js。

4、如果你配置了webpack服务器的话，打开网站首页可以看到只加载了一个js文件**app.js**，你点击搜索页面的路由的时候，可以看到**search.js**加载进来了。哇，是不是很开心！开发过程中使用前端服务器加载静态html需要注意一下**打包的js加载顺序**，写反了会报错！

```
前端静态html只需要这初始化加载这2个文件，search.js会异步加载进来（你可以叫它懒加载）
<script src="/build/vendor.js"></script>
<script src="app.js"></script>
```

5、好了，现在开发完了，准备打包成静态文件。输入**npm run build**刷刷刷的图片、js打包出来了。看一个打包效果图。
![图片描述][1]


  [1]: /img/bVHwha

6、要发布了？no，你需要学会服务端渲染，不送。。
