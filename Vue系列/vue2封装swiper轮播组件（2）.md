学习vue的第二篇文章，完成了以下功能。

1、父组件传递给子组件数据；

2、子组件通过props接收数据；

3、v:bind以及v-for的使用；

4、实现了轮播组件。

前一篇我们搭建了一个vue2+webpack2的框架，实现了一个全局导航组件。

今天说说在vue中使用轮播组件的选择（会造轮子的自己写轮播特效）。

### **父组件如何传递数据给子组件？**

官网有这么一段内容来介绍。

```
Prop
使用 Prop 传递数据
camelCase vs. kebab-case
动态 Prop
字面量语法 vs 动态语法
单向数据流
Prop 验证
```
简单来说就是通过props实现父组件的数据流向子组件。


![clipboard.png](/img/bVMwMh)


为什么叫做单向数据流呢？
数据在父组件中可以修改，比如通过http请求动态更新数据，而子组件只负责通过props接收数据，子组件的权限是只读（如果我没理解错的话，那么跟react中是一样的。）

在本例子中，我实现了这样一个父组件Home.vue

```
<template>
  <div>
   <app-banner :listImg="listImg"></app-banner>
  </div>
</template>

<script>
 import Banner from './templates/Banner.vue'
 import a from '../../static/img/home/banner1.png'
 import b from '../../static/img/home/banner2.jpg'
 import c from '../../static/img/home/banner3.jpg'
 import d from '../../static/img/home/banner4.jpg'
 import e from '../../static/img/home/banner5.jpg'
     export default {
        name: 'Home',
        data() {
            return {
                listImg: [{
                    url: a
                }, {
                    url: b
                }, {
                    url: c
                }, {
                    url: d
                }, {
                    url: e
                }]
            }
        },
        components: {
            'app-banner': Banner
        }
    };
</script>
```
第一步：定义我们的数据结构data，data是一个方法，该方法返回一个object或者类数组等各种数据模型。我定义的是listImg的数组结构。每个数组元素对应一个图片路径，图片都保存在根目录下面的static文件夹。你可能想到使用require导入图片，使用import也是类似的，至于直接传入路径，不使用import或者require会有什么后果，可以自己测试。

第二步：在template中使用v-bind绑定listImg的数据到一个和他同名的:listImg的属性上，这个属性名可以在符合vue规范的情况下任意定义，:listImg === v-bind:listImg。在这里我还要说一个特别的情况。如果你这样写:listImg="{listImg}",子组件接收到的就是一个object，相当于改变了原来的数组类型变成了对象。


### **子组件通过props接收数据并绑定到DOM**

我新建了一个子组件叫做Banner.vue，这个子组件自然就是指轮播图组件[swiper][3]（感兴趣的可以去官网看看）。

#### 第一步：安装swiper。

```
npm install --save swiper
```
#### 第二步：写template。

轮播图是一个列表，所以这里使用到了v-for来遍历，轮播的部分是swiper-slide元素。我把图片路径绑定到了style属性上面。请注意绑定语法的缩写是:（冒号），style内部是一个object，所以background-image要写成backgroundImage，而图片地址url采用字符串拼接的方式来做。

```
<template>
    <div class="swiper-container">
        <div class="swiper-wrapper">
            <div class="swiper-slide" v-for="str in listImg" :style="{ backgroundImage: 'url(' + str.url + ')' }"></div>
        </div>
        <div class="swiper-pagination swiper-pagination-white"></div>
    </div>
</template>
```
#### 第三步：编写Banner.vue的JavaScript代码。

根据swiper的官方教程，我们需要实例化swiper。
1、导入swiper；
2、导入swiper的css；
3、通过props获取父组件传递过来的属性listImg；
4、mounted类似react中的componentDidMount方法，实例化swiper必须等到dom渲染完成才能操作。

```
<script>
    import Swiper from 'swiper';
    import 'swiper/dist/css/swiper.min.css';
    export default {
        props: ['listImg'],
        mounted() {
            console.log('mounted', this)
            var swiper = new Swiper('.swiper-container', {
                pagination: '.swiper-pagination',
                paginationClickable: true,
                loop: true,
                speed: 600,
                autoplay: 4000,
                onTouchEnd: function() {
                    swiper.startAutoplay()
                }
            });
        }
    }
</script>
```

#### 第四步：写css样式。

===================================== 分割线 =============================================

最后，到这一步已经完成了一个轮播图组件了。swiper还是挺好用的。贴上完整的Banner.vue代码，一字不差。

```
<template>
    <div class="swiper-container">
        <div class="swiper-wrapper">
            <div class="swiper-slide" v-for="str in listImg" :style="{ backgroundImage: 'url(' + str.url + ')' }"></div>
        </div>
        <div class="swiper-pagination swiper-pagination-white"></div>
    </div>
</template>

<script>
    import Swiper from 'swiper';
    import 'swiper/dist/css/swiper.min.css';
    export default {
        props: ['listImg'],
        mounted() {
            console.log('mounted', this)
            var swiper = new Swiper('.swiper-container', {
                pagination: '.swiper-pagination',
                paginationClickable: true,
                loop: true,
                speed: 600,
                autoplay: 4000,
                onTouchEnd: function() {
                    swiper.startAutoplay()
                }
            });
        }
    }
</script>

<style lang="less">
    .swiper-container {
        width: 100%;
        height: 10rem;
        .swiper-wrapper {
            width: 100%;
            height: 100%;
        }
        .swiper-slide {
            background-position: center;
            background-size: cover;
            width: 100%;
            height: 100%;
            img {
                width: 100%;
                height: 100%;
            }
        }
        .swiper-pagination-bullet {
            width:0.833rem;
            height: 0.833rem;
            display: inline-block;
            background: #7c5e53;
        }
    }
</style>
```

总结：vue的轮播实现并不难，整体过程和react中很相似，途中我也遇到一个小bug，图片路径一直报错，最后我发现图片后缀不对，第一张是banner1.png，后面4张都是bannerx.jpg，图片都是从酷狗下载的，一下子没注意，被这个坑了一下。

运行效果：[vue-酷我demo][1]
![图片描述][2]

项目地址：https://github.com/hyy1115/vue2-web

上一章：[react转vue——vue2-webpack2框架搭建之路（1）][4]

下一章：[react转vue——webpack压缩打包vue项目（3）][5]

**如果文章对你有帮助，请点击一下推荐。**
  [1]: https://hyy1115.github.io/blog/
  [2]: /img/bVMw0u
  [3]: http://www.swiper.com.cn/api/index.html
  [4]: https://segmentfault.com/a/1190000009127162
  [5]: https://segmentfault.com/a/1190000009162193