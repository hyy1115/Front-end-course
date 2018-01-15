# React 基本理论概念
React.js的实际实现充满了实用的解决方案，增量步骤，算法优化，遗留代码，调试工具以及实际使用所需的东西。这些东西更短暂，如果它足够有价值并且具有足够的优先权，它可以随时间变化。实际的执行情况要困难得多。 我喜欢有一个简单的心智模式，我可以把自己放在里面。

### 转换
React的核心前提是UI仅仅是将数据映射到不同形式的数据。相同的输入给出相同的输出。一个简单的纯函数。

```javascript
function NameBox(name) {
  return { fontWeight: 'bold', labelContent: name };
}
```
```javascript
'Sebastian Markbåge' ->
{ fontWeight: 'bold', labelContent: 'Sebastian Markbåge' };
```

### 抽象化
尽管如此，你不能在一个单独的函数中容纳一个复杂的UI。 UI可以被抽象成可重复使用的部分，而不会泄露其实现细节。如从另一个调用一个函数。

```javascript
function FancyUserBox(user) {
  return {
    borderStyle: '1px solid blue',
    childContent: [
      'Name: ',
      NameBox(user.firstName + ' ' + user.lastName)
    ]
  };
}
```
```javascript
{ firstName: 'Sebastian', lastName: 'Markbåge' } ->
{
  borderStyle: '1px solid blue',
  childContent: [
    'Name: ',
    { fontWeight: 'bold', labelContent: 'Sebastian Markbåge' }
  ]
};
```

### 组成
为了实现真正的可重用特性，简单地重用树叶并为其构建新的容器是不够的。您还需要能够从构成其他抽象的容器中构建抽象。我认为“构图”的方式是将两个或更多不同的抽象组合成一个新的抽象。

```javascript
function FancyBox(children) {
  return {
    borderStyle: '1px solid blue',
    children: children
  };
}

function UserBox(user) {
  return FancyBox([
    'Name: ',
    NameBox(user.firstName + ' ' + user.lastName)
  ]);
}
```

### state
UI不仅仅是服务器/业务逻辑状态的复制。实际上有很多状态是特定于一个确切的预测，而不是其他的。例如，如果您开始在文本字段中输入内容。这可能会或可能不会复制到其他选项卡或您的移动设备。滚动位置是一个典型的例子，您几乎从不想在多个投影中进行复制。 

我们倾向于喜欢我们的数据模型是不可变的。我们通过线程函数可以将状态更新为顶部的单个原子。

```javascript
function FancyNameBox(user, likes, onClick) {
  return FancyBox([
    'Name: ', NameBox(user.firstName + ' ' + user.lastName),
    'Likes: ', LikeBox(likes),
    LikeButton(onClick)
  ]);
}

// 实施细节

var likes = 0;
function addOneMoreLike() {
  likes++;
  rerender();
}

// 初始化

FancyNameBox(
  { firstName: 'Sebastian', lastName: 'Markbåge' },
  likes,
  addOneMoreLike
);
```

注：这些示例使用副作用来更新状态。我的实际心理模型是他们在“更新”过程中返回下一个版本的状态。没有这个解释就简单多了，但是我们会在将来改变这些例子。

### 记忆化
如果我们知道是纯函数，那么一次又一次地调用相同的函数是浪费的。我们可以创建一个追踪最后一个参数和最后结果的函数的记忆版本。这样我们不必重新执行它，如果我们继续使用相同的值。

```javascript
function memoize(fn) {
  var cachedArg;
  var cachedResult;
  return function(arg) {
    if (cachedArg === arg) {
      return cachedResult;
    }
    cachedArg = arg;
    cachedResult = fn(arg);
    return cachedResult;
  };
}

var MemoizedNameBox = memoize(NameBox);

function NameAndAgeBox(user, currentTime) {
  return FancyBox([
    'Name: ',
    MemoizedNameBox(user.firstName + ' ' + user.lastName),
    'Age in milliseconds: ',
    currentTime - user.dateOfBirth
  ]);
}
```

### 列表
大多数用户界面都是某种形式的列表，然后为列表中的每个项目生成多个不同的值。这创建了一个自然的层次。 

要管理列表中每个项目的状态，我们可以创建一个包含特定项目状态的Map。

```javascript
function UserList(users, likesPerUser, updateUserLikes) {
  return users.map(user => FancyNameBox(
    user,
    likesPerUser.get(user.id),
    () => updateUserLikes(user.id, likesPerUser.get(user.id) + 1)
  ));
}

var likesPerUser = new Map();
function updateUserLikes(id, likeCount) {
  likesPerUser.set(id, likeCount);
  rerender();
}

UserList(data.users, likesPerUser, updateUserLikes);
```

注意：我们现在有多个不同的参数传递给FancyNameBox。这打破了我们的记忆，因为我们一次只能记住一个价值。更多的在下面。

### 延续
不幸的是，由于在用户界面中有很多列表，所以明确地管理这些列表变得相当多。

我们可以通过推迟一个函数的执行来将这个样板的一部分移出我们的关键业务逻辑。例如，通过使用“currying”（在JavaScript中绑定）。然后，我们把state从我们的核心职能之外转移到现在没有模板的地方。

这不是在减少样板，而是至少将其移出关键的业务逻辑。

```javascript
function FancyUserList(users) {
  return FancyBox(
    UserList.bind(null, users)
  );
}

const box = FancyUserList(data.users);
const resolvedChildren = box.children(likesPerUser, updateUserLikes);
const resolvedBox = {
  ...box,
  children: resolvedChildren
};
```

### State Map
我们从前面知道，一旦我们看到重复的模式，我们可以使用合成来避免一遍又一遍地重复相同的模式。我们可以将提取和传递状态的逻辑移动到一个我们重用的低级函数。

```javascript
function FancyBoxWithState(
  children,
  stateMap,
  updateState
) {
  return FancyBox(
    children.map(child => child.continuation(
      stateMap.get(child.key),
      updateState
    ))
  );
}

function UserList(users) {
  return users.map(user => {
    continuation: FancyNameBox.bind(null, user),
    key: user.id
  });
}

function FancyUserList(users) {
  return FancyBoxWithState.bind(null,
    UserList(users)
  );
}

const continuation = FancyUserList(data.users);
continuation(likesPerUser, updateUserLikes);
```

### 记忆 Map
一旦我们想记忆列表中的多个项目，记忆就变得更加困难。你必须找出一些复杂的缓存算法来平衡内存使用与频率。 

幸运的是，UI在相同的位置上往往是相当稳定的。树中的相同位置每次获得相同的值。这棵树变成了一个真正有用的记忆策略。 

我们可以使用我们用于状态的相同技巧，并通过可组合函数传递记忆缓存。

```javascript
function memoize(fn) {
  return function(arg, memoizationCache) {
    if (memoizationCache.arg === arg) {
      return memoizationCache.result;
    }
    const result = fn(arg);
    memoizationCache.arg = arg;
    memoizationCache.result = result;
    return result;
  };
}

function FancyBoxWithState(
  children,
  stateMap,
  updateState,
  memoizationCache
) {
  return FancyBox(
    children.map(child => child.continuation(
      stateMap.get(child.key),
      updateState,
      memoizationCache.get(child.key)
    ))
  );
}

const MemoizedFancyNameBox = memoize(FancyNameBox);
```

### 代数效应
事实证明，通过几个抽象层次来传递你可能需要的每一个小小的价值，都是一种PITA。有时候有一个捷径可以在两个抽象之间传递，而不涉及中间件。在React中我们称之为“上下文”。 

有时，数据依赖关系并不整齐地遵循抽象树。例如，在布局算法中，在你完全实现自己的位置之前，你需要知道一些关于你的children的大小。 

现在，这个例子有点“out there”。我将使用ECMAScript提出的[代数效应][1]。如果你熟悉函数式编程，他们会避免记忆强加的中间仪式。

```javascript
function ThemeBorderColorRequest() { }

function FancyBox(children) {
  const color = raise new ThemeBorderColorRequest();
  return {
    borderWidth: '1px',
    borderColor: color,
    children: children
  };
}

function BlueTheme(children) {
  return try {
    children();
  } catch effect ThemeBorderColorRequest -> [, continuation] {
    continuation('blue');
  }
}

function App(data) {
  return BlueTheme(
    FancyUserList.bind(null, data.users)
  );
}
```


  [1]: http://math.andrej.com/eff/