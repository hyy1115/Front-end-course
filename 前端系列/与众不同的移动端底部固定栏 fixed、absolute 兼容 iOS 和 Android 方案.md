**相信我，我分享的和你在其他博客上看到的终极方案是如此的与众不同！**

做过移动端开发的同学，对底部DOM定位出现的各种奇葩情况已经深恶痛绝了吧，底部DOM设置不同的position，在Android和ios上表现都不一样。

为了兼容Android和ios，很多人都煞费苦心，也包括我。

**打开你做的H5，尤其是在微信上打开看看，是不是觉得很恶心，如果自我感觉很恶心，那么请往下看这篇文章，不恶心说明你成功了，可以走了！**

最终还是成功解决了，这篇文章记录一下兼容2种设备的方案。

### 第一种情况

**据我所知，网上还找不到一个能够真正解决这个问题的教程，因为大多数人都是只考虑在body scroll的情况下，设置底部为fixed或者absolute，然后设置滚动区域padding-bottom的值，这种做法反正我是无法接受的，体验太不爽了，也没有兼容Android和ios。**

下图是第一种情况，滚动区域有表单，底部一个固定栏，当填写表单的时候，我们看看ios和Android的表现情况：

1、底部定位为fixed的情况下

ios：激活输入框时，底部不会弹出来（合理）。
Android：激活输入框时，底部会跟着输入框弹出来（不合理）。


2、底部定位为absolute的情况下

ios：激活输入框时，底部不会弹出来（合理）。
Android：激活输入框时，底部会跟着输入框弹出来（不合理）。

android后遗症：输入框失焦的时候，可能导致底部显示在浏览器中间某个位置，回不到原位置。

absolute后遗症：底部按钮和输入框区域一起随着body滚动，不再置顶独立。当滚动区域超过一屏幕时，底部输入框定位出现错乱。

![clipboard.png](/img/bVZN04)

**传统解决办法**：

通常将底部设置为fixed，当激活输入框的时候，将底部定位改为relative，即可兼容ios和Android。

### 第二种情况

底部如果是个输入框的情况下，我们肯定需要输入框在激活的时候弹出来，和第一种情况是相反的。

1、底部定位为fixed的情况下

ios：激活输入框时，底部不会弹出来（不合理）。
Android：激活输入框时，底部会跟着输入框弹出来（合理）。


2、底部定位为absolute的情况下

ios：当滚动区域超过一屏幕时，底部输入框定位出现错乱（不合理）。
Android：当滚动区域超过一屏幕时，底部输入框定位出现错乱（不合理）。


![clipboard.png](/img/bVZOcw)

**传统解决办法：**

仍旧是采用fixed定位

ios：在激活输入框的时候，执行下面代码

```javascript
setTimeout(() => document.body.scrollTop = document.body.scrollHeight, 500)
```
android: 表现正常

### 传统解决方案的后遗症

**除了抖动问题，还有就是微信端滚动body会显示微信浏览器背景，也就是超出滚动边界回弹效应，还有一个恶心的问题是当有弹框的时候，弹框和body滚动累加的双重滚动会有点击穿透造成的卡顿问题。**

**由此，如果你还寄希望于body滚动，那么你的移动端开发体验真的一塌糊涂。**

### 搭建真正的移动端滚动架构

看到这里，你可以暂时把上面的传统解决方案统统忘记。

下面我将会分享移动端最舒适的架构方案。

1、你可能听过Iscroll，这个东西是我们今天要用到的框架的鼻祖，但我们不是用它，而是我曾经另外一篇文章介绍到的**JRoll框架**（比IScroll更加轻量和兼容的移动端滚动框架）。

2、使用这款框架对我们解决底部定位问题还有优化弹框体验有什么帮助呢？他可以完美解决传统解决方案的后遗症，因为他并不是使用body滚动，而是使用css3滚动，采用GPU加速，在ios和Android上测试并不卡顿。如果你想做出像app一样流程的H5，别再用那恶心的body滚动了。



### 源码（复制查看效果，别忘了导入js插件）

下面的源码你可以直接复制到一个html文件上测试，代码中我提供了多种功能的解决方案：

1、采用滚动框架时，何时获取滚动区域的高度（看源码）

2、输入框底部固定时，在该框架中兼容ios和Android的方法（看源码）

3、采用DocumentFragment动态渲染5000个列表元素，说到这个有点意思，记得腾讯某部门的社招面试题就是考察这个知识点，一般人可能采用的是for循环加innerHTML的方法（看源码）

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=0.5, minimum-scale=0.5, maximum-scale=0.5, user-scalable=no">
    <title>Title</title>
    <style>
        * {
            padding: 0;
            margin: 0;
        }
        body, html {
            font-size: 24px;
            height: 100%;
        }
        ul {
            padding-bottom: 1rem;
        }
        ul li {
            list-style: none;
        }
        .bottom {
            position: fixed;
            bottom: 0;
            left: 0;
            width: 100%;
            height: 4.0833rem;
        }
        .bottom > input {
            width: 100%;
            border: 0;
            outline: 0;
            background: rgb(246, 246, 246);
            color: rgb(255, 255, 255);
            text-align: center;
            line-height: 4.0833rem;
            font-size: 1.25rem;
        }
    </style>
</head>
<body>
<div id="scroll-body">
    <ul></ul>
</div>
<div class="bottom"><input type="text" placeholder="请输入内容" onfocus="evocation()"></div>
<script src="./js/jroll.js"></script>
<script>
    function getClientHeight() {
//        获取移动端屏幕高度
        var winHeight
        if (window.innerHeight) {
            winHeight = window.innerHeight;
        } else if ((document.body) && (document.body.clientHeight)) {
            winHeight = document.body.clientHeight;
        } else if (document.documentElement && document.documentElement.clientHeight && document.documentElement.clientWidth) {
            winHeight = document.documentElement.clientHeight;
        }
        return winHeight
    }

    var scrollBody = document.querySelector('#scroll-body') //获取滚动区域的DOM
    var bottom = document.querySelector('.bottom') //获取底部DOM
    function renderLi() {
        //渲染li列表，采用DocumentFragment方案
        var ul = document.querySelector('ul')
        var dFrag = document.createDocumentFragment()
        var startTime = new Date().getTime()
        for (var i = 0; i < 5000; i++) {
            var li = document.createElement("li")
            li.textContent = i
            dFrag.appendChild(li)
        }
        ul.appendChild(dFrag)
        var endTime = new Date().getTime()
        console.log('渲染耗时：', endTime-startTime, 'ms')
    }
    function evocation() {
        //ios唤出弹框，Android的不需要
        setTimeout(() => document.body.scrollTop = document.body.scrollHeight, 500)
    }
    renderLi()
    document.addEventListener('DOMContentLoaded', function() {
        var height = getClientHeight() - bottom.offsetHeight //获取滚动区域高度
        scrollBody.style.height = height + 'px' //计算出实际的滚动区域的高度，然后设置
        new JRoll(scrollBody) //实例化JRoll插件
    })
</script>
</body>
</html>
```

### 总结

使用上面提供的框架，你在移动端开发中，不再需要担心底部固定的问题，不再需要担心滚动造成的一系列问题，不再需要担心弹框滚动以及点击弹框造成的穿透问题等。

而且，不知道你发现没有，底部固定栏你现在可以尝试使用fixed、absolute、relative等设置，不再局限于只能使用fixed了。感兴趣就好好研究一下代码吧！

但是

在IOS11版本中，我发现了document.body.scrollTop = document.body.scrollHeight无效的bug，目前还没找到原因，小于IOS11一切正常。


