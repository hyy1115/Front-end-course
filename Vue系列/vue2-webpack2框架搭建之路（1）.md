一直以来，我从事react开发，突然想用vue来搭建一个项目，看看我的踩坑之路。

react、vue、angular代表了3种前端工程化的思想，学习三大框架主要是理解它们的核心概念，比如组件、生命周期、单向数据流、双向绑定等。这些概念在非框架开发中，很少人会去这样系统化的思考，对于新手来说，很多概念都没有接触过，不知道从何入手一个react、vue或者是angular项目，下面我将会从零搭建vue项目，边做项目边学习vue的思想。

### **1、想要使用vue，我首先该怎么做？**

想要学习vue，我第一件事是去vue官网看简介：[https://cn.vuejs.org/v2/guide/installation.html][3] ，仔细一看，vue现在有1.X和2.X的区别，很好，我果断选择2.X。

选中了vue版本，我上知乎搜索了vue框架搭建的方式，看了前辈的各种分享，了解到一个叫做 [cooking][4] 的好玩意，好在哪里？

cooking 的目标是将你从繁琐的构建配置中解放出来，同时还省去每个项目都要安装一堆开发依赖的麻烦。基于 webapck 但更友好的配置项、易用的扩展配置机制，让你专注项目忘掉配置。

哇，看到cooking官网介绍的这么好，我果断按照它的教程去做，瞎搞了一下下，发现用的不爽啊，一键配置环境看起来很高大上，可是还得去学习cooking的使用，而且本地得安装cooking，搞得我头晕，虽然在浏览器成功访问到了网页，但我还是放弃了这个好玩意。

这时候只能自己从0开始搭建项目了。

### **2、在github新建vue2-web项目。**

打开github首页，点击start a project。

接着你会看到Create a new repository，需要你填写项目信息，这个步骤跳过。

然后项目就建好了，clone到本地。

### **3、初始化npm**

用shell或者cmd进入项目根目录，执行下面的命令，选项什么的直接跳过，最后会生成package.json文件。

```
npm init
```

### **4、安装webpack**

没有webpack就活不下去的感觉，但是配置webpack也会让人活不下去，太难记住webpack的配置项了，不过别担心，我已经帮你搞定这一步了，咋们都必须使用webpack2啊。

```
npm install --save-dev webpack
```

还需要前端服务器，做热更新呀，webpack-dev-server登场。

```
npm install --save-dev webpack-dev-server
```

### **5、创建webpack.config.js文件**

和react中的webpack配置文件没什么区别，只是稍微改动一个地方即可移植过来使用。
**千万不要把js和vue放到一起**，不起作用的，必须分开，必须，这个坑我已经踩过了，为了找这个坑，浪费了我好几个小时，最最最隐蔽的一个地方。

```
rules: [{
            test: /\.js$/,
            use: ['babel-loader'],
            exclude: /node_modules/,
            include: resolve('src')
        },{
            test: /\.vue$/,
            use: ['vue-loader'],
            exclude: /node_modules/,
            include: resolve('src')
        },
```

### **6、创建.babelrc文件。**

babel少不了，注意这里不是用react了，而是vue，包括下面几个插件，flow-vue、transform-vue-jsx。

```
{
  "presets": ["es2015", "flow-vue", "stage-0", "stage-2"],
  "plugins": ["transform-vue-jsx"],
  "comments": false,
  "env": {
    "production": {
      "plugins": [
        ["transform-runtime", { "polyfill": false, "regenerator": false }]
      ]
    }
  }
}
```

### **7、在package.json添加start命令**

直接使用webpack-dev-server启动，哇塞，一堆报错，说少了哪个module，这个简单，因为配置文件里面引用的一堆module，还没有安装到项目呢，这时候一个个安装好就行了。

```
"start": "webpack-dev-server",
```

### **8、项目入口main.js文件。**

这个文件名自己喜欢咋取就咋取，代码挺简单的，实例化一个Vue和路由，是不是和react的入口文件很像？当然，我做的是SPA，所以采用单入口的形式，如果是非SPA模式，就不是这种配置方式了。

```
import Vue from 'vue';
import App from './App.vue';
import VueRouter from 'vue-router';
import routes from './routes';
import VueResource from 'vue-resource';

Vue.use(VueResource); //http请求注册
Vue.use(VueRouter); //路由注册

// 实例化路由
const router = new VueRouter({
    // mode: 'history', //H5 路由模式，需要服务端做渲染防止404错误
    base: __dirname,
    linkActiveClass: 'on',
    routes
})

let render = new Vue({
    router,
    el: '#app',
    render: h => h(App)
});

render;

// if (module.hot) {
//     非必须
//     module.hot.accept('./App.vue', () => render);
// }

```

### **9、路由routes.js**

路由和react也非常像（简直一样好不），这里的vue页面采用.vue后缀的方式来写。

```
import Home from './components/home/Home.vue';
import Bang from './components/bang/Bang.vue';

export default [
    {
        path: '/',
        redirect: 'home'
    },
    {
        path: '/home',
        component: Home
    },
    {
        path: '/bang',
        component: Bang
    }
]
```

### **10、单页顶层容器App.vue**

从index进来，就是这个文件，现在开始学习vue的精华。

template：vue的模板语言，也叫作jsx。
transition：过渡动画。
router-view：路由显示容器，通过router-link跳转加载的.vue会在这个容器渲染。router-link被我封装到nav.vue组件里面了。
script：导入了当前顶级容器需要用到的vue组件，包括头部、导航、首页。还有更多丰富的设置我没有研究，后续的学习中会深入下去。
style: 当前组件的样式，我配置了less语法支持。将style改成<style lang="less">即可写less。

```
<template>
    <div>
        <app-header logo="logo" ></app-header>
        <app-nav></app-nav>
        <transition name="fade" mode="out-in">
            <router-view class="view"></router-view>
        </transition>
    </div>
</template>

<script>
    import Header from './components/common/Header.vue';
    import Nav from './components/common/Nav.vue';
    import Home from './components/home/Home.vue';
    export default {
        name: 'App',
        components: {
            "app-header": Header,
            "app-nav": Nav,
            "app-home": Home
        }
    };
</script>

<style>
    body, html {
        font-size: 12px;
        margin: 0;
        padding: 0;
    }
</style>
```

踩坑的过程中，也遇到了好几个报错情况，最后都圆满解决了。
如果你想看更详细的vue组件代码，可以看具体项目：[https://github.com/hyy1115/vue2-web][5]

接下来我会继续完善该项目，探究一个更加灵活的vue架构实现。

### 运行效果图:[vue-酷我demo][1]

![效果图][2]

下一章：[vue2封装swiper轮播组件（2）][6]

**如果文章对你有帮助，请点击一下推荐。**
  [1]: https://hyy1115.github.io/blog/
  [2]: /img/bVMsyF
  [3]: https://cn.vuejs.org/v2/guide/installation.html
  [4]: http://elemefe.github.io/cooking/
  [5]: https://github.com/hyy1115/vue2-web
  [6]: https://segmentfault.com/a/1190000009143923