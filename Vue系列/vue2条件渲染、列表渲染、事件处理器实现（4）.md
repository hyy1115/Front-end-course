学习vue2的过程中，我基本是按照vue2官网的教程走，并且主要以实现需求为主，如果不需要深入vue2的概念，就用简单的方法去实现。
本章实现了以下功能：
1、列表渲染，v-for；
2、条件渲染，v-if；
3、自适应布局；
4、事件处理器，v-on:click；
5、data数据管理。

运行效果：[vue-酷我demo][1]
![图片描述][2]

### **1、列表渲染，v-for；**

在前面几章已经使用过几次v-for指令了，对此并不陌生，这次更新实现了排行榜列表和歌手列表，以排行榜列表为例，重温一下v-for的用法。

**Bang.vue：排行榜父组件**，bangList是一个数组，for in遍历出来的是一个值属性，根据MDN文档对于for in遍历的解释，通常for in用来遍历对象Object。我们需要给遍历对象指定一个key，然后把遍历出来的属性list传递给子组件。

```
<template>
    <div>
        <ul class="bang-ul">
            <app-bang v-for="list in bangList" :key="list.id" :list="list"></app-bang>
        </ul>
    </div>
</template>

<script>
    import Bang from './template/BangList.vue' //导入排行榜列表子组件
    import { bangList } from '../../static/data/data' //导入数据
    export default {
        name: 'Bang',
        components: {
            'app-bang': Bang
        },
        data() {
            return {
                bangList: bangList
            }
        }
    }
</script>
```

**BangList：排行榜列表子组件，** 子组件是一个li，因为li是可点击的路由，所以使用router-link包裹，:to是绑定路由地址，然后img绑定了图片路径，在dom标签中输出使用双大括号{{list.title}}，而export default是指默认导出的js代码，props表示获取从父组件传递过来的参数，至于参数验证，以后再添加。

```
<template>
    <li>
        <router-link :to="list.pathname">
            <div class="left">
                <img :src="list.url" :alt="list.title">
            </div>
            <div class="right">
                {{list.title}}
            </div>
            <i class="fa fa-angle-right"></i>
        </router-link>
    </li>
</template>

<script>
    export default {
        props: ['list']
    }
</script>
```

### **2、条件渲染，v-if；**

v-if非常好用，在vue2中，你可以享受到条件语句带来的便利，虽然在react中早就有这玩意了。在歌手列表页面，我使用了v-if。

**Artist.vue：歌手列表父组件**，因为artist是一个Object，所以首先由Object.keys()的方式遍历出key，然后使用v-if判断singer等于不同的key时渲染不同的值到模板。如果你看代码有点晕，可以运行项目对着看。

```
<template>
    <div>
        <artist-list v-for="singer in Object.keys(artist)" v-if="singer === 'listOne'" :singer="artist.listOne"></artist-list>
        <artist-list v-for="singer in Object.keys(artist)" v-if="singer === 'listTwo'" :singer="artist.listTwo"></artist-list>
        <artist-list v-for="singer in Object.keys(artist)" v-if="singer === 'listThree'" :singer="artist.listThree"></artist-list>
        <artist-list v-for="singer in Object.keys(artist)" v-if="singer === 'listFour'" :singer="artist.listFour"></artist-list>
    </div>
</template>

<script>
    import SingerList from './templates/SingerList.vue' //导入歌手列表子组件
    import { artist } from '../../static/data/data' //导入歌手列表数据
    export default {
        components: {
            'artist-list': SingerList
        },
        data() {
            return {
                artist: artist
            }
        }
    }
</script>
```

**SingeList.vue：歌手列表子组件**，这个子组件的实现和排行榜的例子非常相似。

```
<template>
    <ul class="singer-ul">
        <li v-for="list in singer">
            <router-link :to="list.pathname">{{list.title}}</router-link>
            <i class="fa fa-angle-right"></i>
        </li>
    </ul>
</template>

<script>
    export default {
        props: ['singer']
    }
</script>
```

### **3、自适应布局；**

如果你运行了我在github上的例子，采用不同的分辨率来看，会发现会自适应，在不同分辨率下保持同样的倍数递增或者递减，这种自适应方式和酷狗移动端的实现是一致的，但是和bootsrap的栅格响应又不一样。

采用这种自适应需要注意以下几点。

**在App.vue添加media属性：**这里的每一个font-size都是我用计算器计算出来的 12*比例 得到的值，而酷狗采用的是百分比 % 的方式，本质是一样的。

```
@media screen and (min-width: 320px) {
        html, body { font-size: 12px !important; }
    }
    @media screen and (min-width: 360px) {
        html, body { font-size: 13.5px !important; }
    }
    @media screen and (min-width: 375px) {
        html, body { font-size: 14px !important; }
    }
    @media screen and (min-width: 414px) {
        html, body { font-size: 15.5px !important; }
    }
    @media screen and (min-width: 720px) {
        html, body { font-size: 27px !important; }
    }
    @media screen and (min-width: 1024px) {
        html, body { font-size: 38.4px !important; }
    }
    @media screen and (min-width: 1080px) {
        html, body { font-size: 40.5px !important; }
    }
    @media screen and (min-width: 1366px) {
        html, body { font-size: 51px !important; }
    }
    @media screen and (min-width: 1440px) {
        html, body { font-size: 54px !important; }
    }
    @media screen and (min-width: 1600px) {
        html, body { font-size: 60px !important; }
    }
    @media screen and (min-width: 1920px) {
        html, body { font-size: 72px !important; }
    }
```

**使用这种自适应布局，你需要将你的样式采用rem或者em的方式来替代px**，具体看我代码的实现。


### **4、事件处理器，v-on:click；**

在首页的SongList.vue中，我写了个绑定事件处理器做测试。
首先，你需要在 methods 中定义方法，我定义了reverseMsg(title){}用来打印当然点击列表的值。
接着，把写好的方法用v-on:click="reverseMsg(song.title)"绑定到li上，这样你就能获取当然点击对象的值，而不需要纠结用this来获取当前点击的dom对象了。

```
<template>
    <li v-if="seen" v-on:click="reverseMsg(song.title)">
        <span class="title" :title="song.singer">{{song.singer}} - {{song.title}}</span>
        <i :style="{background: 'url('+ downLoadIcon +') no-repeat'}"></i>
    </li>
</template>

<script>
    import downLoadIcon from '../../../static/outImg'
    export default {
        props: ['song'],
        data() {
            return {
                downLoadIcon: downLoadIcon,
                seen: true
            }
        },
        methods: {
            reverseMsg(title) {
                console.log('title: ', title)
            }
        }
    }
</script>
```


### **5、data数据管理。**

目前静态数据管理分为2大块，图片管理和静态渲染数据管理，他们都保存在static文件夹里面。

**图片管理：**
你可以这样写，把所有需要导出的图片放到一个js文件里面，使用module.exports的方式定义导出的文件名。

```
// 引用图片路径
let a = require('./img/home/banner1.png')
let b = require('./img/home/banner2.jpg')
let c = require('./img/home/banner3.jpg')
let d = require('./img/home/banner4.jpg')
let e = require('./img/home/banner5.jpg')
let downLoadIcon = require('./img/home/download_icon.png')

let bang1 = require('./img/bang/bang1.png')
let bang2 = require('./img/bang/bang2.png')
let bang3 = require('./img/bang/bang3.png')
let bang4 = require('./img/bang/bang4.png')
let bang5 = require('./img/bang/bang5.png')
let bang6 = require('./img/bang/bang6.png')
let bang7 = require('./img/bang/bang7.png')
let bang8 = require('./img/bang/bang8.png')
let bang9 = require('./img/bang/bang9.png')
let bang10 = require('./img/bang/bang10.png')

// 导出图片
module.exports = {
    a,
    b,
    c,
    d,
    e,
    downLoadIcon,
    bang1,
    bang2,
    bang3,
    bang4,
    bang5,
    bang6,
    bang7,
    bang8,
    bang9,
    bang10
}
```
在项目中引用图片。

```
import { bang1, bang2, bang3, bang4, bang5, bang6, bang7, bang8, bang9, bang10 } from '../outImg'

```
**静态数据管理：**

和图片管理的方式一样，也是通过module.exports的方式。

```
/ 新歌列表
const songList = [
    {_id: 'songs_CC156F3A7186FB12768B2C1B48FDAA74', singer: '庄心妍', title: '屋檐下的浪漫'},
    {_id: 'songs_9AA99B44B77C50737A42E921EF51C937', singer: '童可可', title: '薛定谔的猫'},
    {_id: 'songs_9154AD94E915A6A6CDD6AF57EEFE06A6', singer: '阿杜', title: '一诺千年'},
    {_id: 'songs_53C3F2DBE2790BE8884E6D8F5553FE50', singer: '黄子韬', title: 'Promise'},
    {_id: 'songs_39224F3C618C955352142DB989737B9D', singer: '刘恺威、蒋欣', title: '明明爱【继承人插曲】'},
    {_id: 'songs_2B0BAE72AFC013419B2D21D86C0BA515', singer: '林忆莲', title: '我不能忘记你【记忆大师记忆主题曲】'},
    {_id: 'songs_A350CE1A5B0769D3E39C311C933F1234', singer: '高夫', title: '青春去哪了'},
    {_id: 'songs_5184FAAD32883D73533714BACAA98A1F', singer: '华晨宇', title: ' 寻【花儿与少年3·冒险季主题曲】'},
    {_id: 'songs_7A5B2CC635273C4B81167F19E8897773', singer: '鹏泊', title: '春来鸟'},
    {_id: 'songs_4BCA5B307683B83A6BC2F34A93E7DBC2', singer: '阿兰', title: '兰之乐光'},
    {_id: 'songs_76448B8D90609F4657782A2F4ACE3C1C', singer: '星月组合', title: '今夜的你又在和谁约会'}
]

// 排行榜
const bangList = [
    {url: bang1, title: '酷狗飙升榜', pathname: ''},
    {url: bang2, title: '酷狗TOP500', pathname: ''},
    {url: bang3, title: '网络红歌榜', pathname: ''},
    {url: bang4, title: 'DJ热歌榜', pathname: ''},
    {url: bang5, title: '华语新歌榜', pathname: ''},
    {url: bang6, title: '欧美新歌榜', pathname: ''},
    {url: bang7, title: '韩国新歌榜', pathname: ''},
    {url: bang8, title: '日本新歌榜', pathname: ''},
    {url: bang9, title: '粤语新歌榜', pathname: ''},
    {url: bang10, title: '原创音乐榜', pathname: ''}
]

// 歌手
const artist = {
    'listOne': [
        {pathname: '', title: '热门歌手'}
    ],
    'listTwo': [
        {pathname: '', title: '华语男歌手'},
        {pathname: '', title: '华语女歌手'},
        {pathname: '', title: '华语组合'},
    ],
    'listThree': [
        {pathname: '', title: '日韩男歌手'},
        {pathname: '', title: '日韩女歌手'},
        {pathname: '', title: '日韩组合'},
    ],
    'listFour': [
        {pathname: '', title: '欧美男歌手'},
        {pathname: '', title: '欧美女歌手'},
        {pathname: '', title: '欧美组合'}
    ]
}

module.exports = {
    songList,
    bangList,
    artist
}
```
哎，这就是没有构建mock server的办法，不过哪些数据需要mock server呢？显然我现在使用的都是静态数据，是不需要mock来实现的。

==================================分割线============================================

这一天我把vue2官网的基础篇看了几遍，也运用了部分功能到项目中，其实生命周期在轮播图那章就已经涉及到了，只不过我没有细说，后面会挑出一章专门讲生命周期每个时间点的实际用途。

这次还更新了很多细微的东西，就不细说了，从项目中慢慢体会，打铁还需自身硬，多看看vue2官网的教程，会更有启发。

项目地址：https://github.com/hyy1115/vue2-web (对你学习vue2有帮助的话，别吝啬给个star支持一下)

上一章：[react转vue——webpack压缩打包vue项目（3）][3]


  [1]: https://hyy1115.github.io/blog/
  [2]: /img/bVMG28
  [3]: https://segmentfault.com/a/1190000009162193