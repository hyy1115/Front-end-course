作为一个redux狂热爱好者，我还是第一次尝试剥离redux框架来搭建react项目，我喜欢用最新的版本来研究，比如react15，webpack2，等到react16出来，恐怕大家又得重新适应一些规则了。

学习前端以来，我发现前端框架变化太快，如果不保持持续性的学习能力，很容易就会被新人给替代。

这只是一个小玩意，展示了react和webpack2的基本框架搭建，没有redux，没有mobx，你可以纯粹当做学习如何搭建一个简单的react和webpack2框架，或者用来扩展成一个可管理的项目。

当然，在企业项目中，还是推荐用redux或者mobx来管理state。

![图片描述][1]

看一下主要的代码。

**1、package.json：插件管理，没有配置build，只配置了start启动项目。**

```
{
  "name": "react-webpack2",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "babel-polyfill": "^6.23.0",
    "react": "15.4.2",
    "react-dom": "15.4.2",
    "react-hot-loader": "^3.0.0-beta.6",
    "react-router-dom": "^4.0.0",
    "react-scripts": "0.9.5"
  },
  "devDependencies": {
    "babel-core": "^6.24.0",
    "babel-loader": "^6.4.1",
    "babel-preset-es2015": "^6.24.0",
    "babel-preset-react": "^6.23.0",
    "babel-preset-stage-0": "^6.22.0",
    "babel-preset-stage-2": "^6.22.0",
    "css-loader": "^0.27.3",
    "postcss-loader": "^1.3.3",
    "style-loader": "^0.16.1",
    "webpack": "^2.3.2",
    "webpack-dev-server": "^2.4.2"
  }
}

```

**2、webpack.config.js：webpack配置文件是一个object，你把他看成是一个json数据来理解会容易很多。**


```
const HtmlWebpackPlugin = require('html-webpack-plugin');
const webpack = require('webpack');
const path = require('path');

module.exports = {
    entry: {
        app: [
            'webpack-dev-server/client?http://localhost:3133',
            'webpack/hot/only-dev-server',
            'babel-polyfill',
            'react-hot-loader/patch',
            './src/index'
        ]
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].js',
        publicPath: '/dist'
    },
    module: {
        rules: [
            {test: /\.(js|jsx)$/, use: 'babel-loader', exclude: /node_modules/},
            {test: /\.css$/, use: ['style-loader', 'css-loader?importLoaders=1']},
        ]
    },
    plugins: [
        // new webpack.optimize.UglifyJsPlugin(),
        new HtmlWebpackPlugin({template: './index.html'}),
        new webpack.HotModuleReplacementPlugin(), //热更新
        new webpack.NamedModulesPlugin(), //在控制台打印模块
    ],
    devtool: 'eval'
}
```

**3、server.js：配置webpack-dev-server启动项，还有一种方式是通过express来启动前端项目。**

```
var webpack = require('webpack');
var WebpackDevServer = require('webpack-dev-server');
var config = require('./webpack.config');

new WebpackDevServer(webpack(config), {
    publicPath: config.output.publicPath,
    hot: true,
    historyApiFallback: true,
    stats: {
        colors: true
    }
}).listen(3133, 'localhost', function (err) {
    if (err) {
        console.log(err);
    }

    console.log('Listening at localhost:3133');
});
```

**4、babelrc文件：配置详情请去官网 http://babeljs.io/docs/usage/babelrc/**

```
{
  "presets": [
    ["es2015", {"modules": false, "loose": true}],
    // webpack understands the native import syntax, and uses it for tree shaking
    "stage-0",
    "stage-2",
    // Specifies what level of language features to activate.
    // Stage 2 is "draft", 4 is finished, 0 is strawman.
    // See https://tc39.github.io/process-document/

    "react"
    // Transpile React components to JavaScript
  ],
  "plugins": [
    "react-hot-loader/babel"
    // Enables React code to work with HMR.
  ]
}
```


**5、index.js：在src目录下面的index.js作为网站的入口。我们看到在react-hot-loader提取出了一个AppContainer，官方认为一个网站应用只有一个单一的根元素是比较好的实现方式。关于使用热更新的同学，你配置好了webpack、babel之后，别忘了module.hot.accept()，这个方法是用来调用你需要实现热更新的代码，通常放在网站的入口或者是store的入口。**

```
import React from 'react';
import ReactDOM from 'react-dom';

import { AppContainer } from 'react-hot-loader';
// AppContainer is a necessary wrapper component for HMR

import App from './App';

const render = (Component) => {
    ReactDOM.render(
        <AppContainer>
            <Component/>
        </AppContainer>,
        document.getElementById('root')
    );
};

render(App);

// Hot Module Replacement API
if (module.hot) {
    //这种写法只在webpack2支持，如果是webpack1版本，还是要用require的方式来导入模块。
    module.hot.accept('./App', () => { render(App) });
}
```

**6、App.js：如果说index.js是网站的入口，那么App.js是组件的入口，我们把App.js叫做父组件，下面这种形式是函数式组件的写法，返回一个jsx对象。**

```
import React from 'react';
import styles from './app.css';

import Button from './components/Button'

const App = () => (
    <div className={styles.app}>
        <h2>一个简单的react-webpack计数器....</h2>
        <Button />
    </div>
);

export default App;
```

**7、Button.js：子组件，放在components下面，子组件我采用了class的写法，当你的组件是一个纯函数的时候，就推荐使用6的函数组件写法，当组件有state的时候，采用class的写法比较合适。下面的例子实现了一个计数器的效果，点击按钮，计数器就加1.**

```
import React, { Component } from 'react';

export default class Button extends Component {

    state = {
        count: 1
    }

    render() {
        const { count } = this.state
        return (
            <div>
                <button
                    style={{border: "1px solid #000"}}
                    onClick={() => this.setState({count: count + 1})}>点击计数器</button>
                <div style={{color: "#f60", fontSize: "20px"}}>{count}</div>
            </div>
        )
    }
}
```

**8、现在你可以体验一下热更新了，修改src目录下的css、js，可以在浏览器看到修改的部分更新了。**

**看完了，可以结合配置好的项目学习一下：[react-webpack2][2] **


备注：
react完整项目框架：[目前最新最完整的react技术栈框架在这里，我会持续性跟随各个插件的官方脚步更新版本][3]（新手请先看懂上面的基础框架再入手高级框架，善意的建议）


  [1]: /img/bVLr38
  [2]: https://github.com/hyy1115/react-webpack2/
  [3]: https://github.com/hyy1115/react-redux-webpack2