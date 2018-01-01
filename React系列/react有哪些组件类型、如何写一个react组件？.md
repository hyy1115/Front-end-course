**react组件的分类：**展示型组件和容器型组件

简单理解来说，容器型组件是一个页面容器，用来放置当前页面的所有展示型组件

展示型组件是具体到某一个小的组件模块，比如一个按钮，一个卡片，一个进度条等，我们在用react做组件化开发的时候，先定义好一个个小的展示型组件，然后把这些组件都导入容器型组件，最终组合成一个完整的页面。

记得看到某个社区有人问到展示型组件能不能嵌套容器型组件，我认为可以，比如有这样一个场景，有一个选择地区的容器组件，这个容器是一个公共模块，可能在任何页面的某个小模块触发该地区组件的调用。这个时候就是通过展示型组件嵌套容器组件的真实案例，也算是我在实际项目中遇到的需求。

下面我们来了解一个展示型组件和容器组件的写法有什么区别。

**1、展示型组件**

写法1：函数写法
```
import React, { Component } from 'react' //必须导入
//定义一个header函数，传入参数，返回一个jsx模板,函数名小写字母开头
exports const header = (params) => {
    return (
        <div>{params}</div>
    )
}
```
写法2：react组件写法

```
import React, { Component } from 'react'
//定义一个react组件，组件名大写字母开头
exports class List extends React.Component {
    constructor(props) {
        super(props)
    }
    
    render() {
        const { data } = this.props
        return (
            <ul>
                {
                    data.length > 0 && 
                    data.map((element, index) => {
                       <li key={index}>{element}</li> 
                    })
                }
            </ul>
        )
    }
}
```

**2、容器型组件**

容器组件和展示型组件类似，本质就是react组件，只不过是将各种展示型组件组合起来，只不过函数组件和react组件导入方式有所区别。

```
import React, { Component } from 'react'
import { header, List } from './index'

//定义一个react容器，组件名大写字母开头
exports class Page extends React.Component {
    
    render() {
        let data = [1, 2, 3, 4, 5]
        return (
            <div>
                {header(params)}
                <List data={data} />
            </div>
        )
    }
}
```

我发现很多前端工程师的代码格式乱七八糟，在这里也告诉那些代码格式混乱的同学，平时写代码要注意大小写，该空格的空格，该换行的换行，具体可以看看我上面写的几个组件的例子，好的代码习惯会让代码维护变得更加简单。


