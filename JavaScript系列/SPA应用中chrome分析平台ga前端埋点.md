网站数据分析平台有很多，百度统计、chrome的ga，还有国产其他公司的一些好用的数据分析，但那些是属于商业性质的，费用嘛，**太贵了！**比如神策、GrowingIO等，他们1年的费用对于很多小公司来说都是很难承担的。
但是使用国内这些平台的好处是可以无埋点获取热点图。在一些前端项目中，可能需要埋几百个点，对前端工程师来说既是KPI，也是负担。
我们知道，Google Analytics平台是不收费的，我们不探讨ga是如何实现“无埋点”技术，本文研究的是使用ga的“前端埋点”原理。
### SPA的特点
SPA也叫单页应用网站，它和普通网站不同的是，SPA有一个单独的入口js。而我们要做ga埋点，就需要在这个入口js处理。
### 前端埋点原理
建立一个事件委托模型，统一处理埋点的获取。
比如我们想获取首页用户点击了N个标签，我们只需要给这些标签加上data-ga属性，然后在事件委托函数中获取到当前点击目标的data-ga属性，发送给chrome即可。
### 事件委托函数
SPA的入口js中，我们需要调用这样一个函数。
```javascript
window.onload = function() {
    let doc = document.querySelector('body')
    if (doc) {
        doc.onclick = function(event) {
            let _event = event || window.event
            let target = _event.target || _event.srcElement
            while (!!target && !target.hasAttribute('data-ga')) {
                if ( target.tagName == 'BODY') {
                    break
                }
                target = target.parentNode
            }
            if (target.hasAttribute('data-ga')) {
                let events = target.getAttribute('data-ga').split(/[,，]/)
                if (!!window.ga) {
                    ga('send', 'event', events[0], events[1], events[2] || "")
                }
            }
        }
    }
}

```

ga()方法有5个参数，第一个表示动作，这里使用了send发送，第二个参数是类型，这里是event事件，后面3个分别表示3个说明文字。
```html
<button data-ga="首页，xx按钮，点击事件">xx按钮</button>
```
假设有这么一段HTML代码，在祖先元素埋点，当点击子元素span的时候，获取的event.target是span，那么使用target.hasAttribute('data-ga')获取不到祖先元素的data-ga。
```html
<div data-ga="首页，祖先元素">
    <div>
        <span>子元素</span>
    </div>
</div>
```
解决方法是使用DOM节点冒泡查找法，先查找父元素，没有的话继续往上查找，切记不要查找兄弟元素。
```javascript
while (!!target && target.hasAttribute('data-ga') === false && target.tagName !== 'body') {
    target = target.parentNode
}
```

### 总结
ga前端埋点的思路很简单，仅仅需要一个事件委托函数去管理埋点。这篇文章是从前端的角度来分析的，如果说ga可以实现无埋点技术，那么就需要你研读chrome Analytics平台的文档了。