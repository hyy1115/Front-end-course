### 前奏
这篇文章仅对不熟悉在react中使用socket.io的人、以及socket入门者有帮助。

下面这个动态图展示的聊天系统是用react+express+socket搭建的，很模糊吧，要得就是这样的效果，我自己开了2个窗口，创建2个用户自问自答。没有什么高深的技术，对于很多想接触socket的前端工程师来说，不擅长后端的socket代码可能是硬伤。

![图片描述][1]

### 开发环境

服务端：express服务器

客户端：react技术栈，开发环境采用前端服务器的方式，打包后将静态资源放到服务端目录下做测试。

### 基本介绍
想要实现一种实时的双向通信聊天系统，你可能会想到ajax轮询（长或短），但你最想要的还是socket的实现。

在写测试代码之前，我纠结于前端用什么，后端用什么，后来后端选择了express、前端是react。

1、服务端使用到的js库

```javascript
express

socket.io
```

2、前端使用到的js库

```javascript
"react": "^16.2.0",
"react-dom": "^16.2.0",
"socket.io-client": "^2.0.4"
```

### express服务端的实现
服务端的实现我不想多讲，你可以去看官方demo，代码很详细：[socket官方demo实现][2]

服务端的核心代码：

```javascript
io.on('connection', function (socket) {
    // 当客户端发出“new message”时，服务端监听到并执行相关代码
    socket.on('new message', function (data) {
        // 广播给用户执行“new message”
        socket.broadcast.emit('new message', {});
    });
    
    // 当客户端发出“add user”时，服务端监听到并执行相关代码
    socket.on('add user', function (username) {
        socket.username = username;
        //服务端告诉当前用户执行'login'指令
        socket.emit('login', {});
    });
    
    // 当用户断开时执行此指令
    socket.on('disconnect', function () {});
});
```

socket和mongodb有点像，它需要创建一个socket服务，创建成功之后，就可以通过on()去监听一个action，action在这里表示的是 'new message'、'add user'、'login'等指令，这些指令是可以自己命名的。

这些指令有什么作用呢？

当客户端和服务端建立socket通信之后，服务端可以向客户端发出指令，客户端也可以向服务端发出指令，开发者需要先给双方的通信约定一套指令系统。

比如服务端创建了一个 'new message'的指令，但是客户端没有创建这个指令，就会导致客户端无法接收到服务端发出的这个指令。客户端心里可能在想：服务端老兄在瞎bb什么？

客户端也需要 ’new message’指令，这样双方就能达成一套通信的协议，双方可以互相发出这条指令告诉对方最新的状态。

上面代码提到了socket.emit()和socket.broadcast.emit()2种用法，可以看看下面关于emit用法的详细解释。

```javascript
// 发送到当前请求socket通信的客户端
socket.emit('message', "this is a test");

// 发送给所有客户端，除了发件人
socket.broadcast.emit('message', "this is a test");

// 发送给“游戏”房间（频道）中的所有客户，发件人除外
socket.broadcast.to('game').emit('message', 'nice game');

// 发送给所有的客户，包括发件人
io.sockets.emit('message', "this is a test");

// 发送给“游戏”房间（频道）的所有客户，包括发件人
io.sockets.in('game').emit('message', 'cool game');

// 发送给指定的socketid
io.sockets.socket(socketid).emit('message', 'for your eyes only');
```

**socket的这种行为更像是redux，但是redux是单向数据流，而socket是双向。**

### React客户端的实现

React端的实现，才是我们应该关注的重点。

作为一个前端工程师，往往只需要和后端大神配合即可（全栈除外）。

**1、在react组件中导入socket.io-client**

前端使用的是socket.io-client库，这个库使用非常简单。下面的代码中，直接导入socket.io-client并且指向服务端的ip+端口即可。

```javascript
import React, { Component } from 'react'

//require('socket.io-client')('服务端ip+端口')
const socket = require('socket.io-client')('http://localhost:3077');

class App extends Component {
    
}
```

**2、在componentDidMount写socket的接收指令action**

socket.on()设置了服务端约定好的指令，当服务端发出这些指令时，客户端就能接收到。这时候，你可以在回调函数里面根据后端返回的数据 data 做前端的处理，比如设置state的状态，渲染服务端推送的数据。

```javascript
componentDidMount() {
        socket.on('login', (data) => {
            console.log(data)
        });
        socket.on('add user', (data) => {
            console.log(data)
        });
        socket.on('new message', (data) => {
            console.log(data)
        });
    }
```

**3、客户端推送数据到服务端**

很多时候，客户端也需要告诉服务端有新的数据更新，当你在聊天界面发了一条新消息，这时候要告诉服务端，就通过socket.emit()方法，和服务端推送的方法是一样的。

```javascript
socket.emit('new message', value)
```

### 总结

1、当你想要告诉对方一些话时，使用socket.emit()。

2、当你想接收对方的话时，使用socket.on()。

3、emit还有点对点、广播等用法。

4、最后说一句，这些都是基础知识。

  [1]: /img/bV0eV1
  [2]: https://socket.io/demos/chat/