# Reconciliation（和解）
React提供了一个声明性的API，以便您不必担心每个更新的确切更改。这使得编写应用程序变得更容易，但在React中如何实现这一点可能并不明显。本文解释了我们在React的“差异化”算法中做出的选择，以便组件更新可预测，同时对于高性能应用程序足够快。
<hr />

### 动机
当你使用React的时候，你可以把render（）函数想象成创建一个React元素的树。在下一个state或props更新时，render（）函数将返回不同的React元素树。然后React需要弄清楚如何有效地更新UI以匹配最近的树。

对于产生将一棵树转换为另一棵树的最小操作次数的算法问题，有一些通用的解决方案。然而，现有技术的算法在O（n3）的顺序中具有复杂度，其中n是树中元素的数量。

如果我们在React中使用它，则显示1000个元素将需要10亿次比较。这代价太昂贵了。相反，React基于两个假设实现了启发式O（n）算法：

 1. 两种不同类型的元素会产生不同的树。

 2. 开发人员可以通过关键props暗示哪些子元素可以在不同的渲染器中保持稳定。

实际上，这些假设对于几乎所有的实际使用情况都是有效的。

<hr />

### Diffing算法
比较两棵树时，React首先比较两个根元素。行为根据根元素的类型而不同。

#### 不同类型的元素

每当根元素具有不同的类型时，React将会拆除旧的树并从头开始构建新的树。从<a>到<img>，或从<Article>到<Comment>，或从<Button>到<div> - 任何这些都将导致完全重建。

当拆除树时，旧的DOM节点被破坏。组件实例接收componentWillUnmount（）。在构建新树时，将新的DOM节点插入到DOM中。组件实例接收componentWillMount（），然后接收componentDidMount（）。任何与旧树相关的状态都将丢失。

根目录下的任何组件也将被卸载，并将其状态销毁。例如，当差异：

```javascript
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```

这将销毁旧的Counter，并重新安装一个新的Counter。

#### 同一类型的DOM元素

比较同一类型的两个React DOM元素时，React查看两者的属性，保持相同的底层DOM节点，并只更新已更改的属性。例如：

```javascript
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```

通过比较这两个元素，React知道只修改底层DOM节点上的className。 

更新样式时，React也知道只更新已更改的属性。例如：

```javascript
<div style={{color: 'red', fontWeight: 'bold'}} />

<div style={{color: 'green', fontWeight: 'bold'}} />
```

在这两个元素之间转换时，React知道只修改颜色样式，而不是fontWeight。 

处理完DOM节点后，React会在子节点上递归。

#### 同一类型的组件元素

当一个组件更新时，这个实例保持不变，所以状态在整个渲染过程中保持不变。 React更新底层组件实例的props以匹配新元素，并在底层实例上调用componentWillReceiveProps（）和componentWillUpdate（）。 

接下来，render（）方法被调用，diff算法在前一个结果和新结果上递归。

#### 在Children上递归

默认情况下，React对DOM节点的子节点进行递归时，React只是同时遍历两个子节点列表，并且每当出现差异时都会生成一个突变节点。 

例如，在子元素的末尾添加元素时，在这两棵树之间转换效果很好：

```javascript
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

React将匹配两个<li>first</li>树，匹配两个<li>second</li>树，然后插入<li>third</li>树。 

如果你天真地实现它，开始插入一个元素的性能会更差。例如，这两棵树之间的转换效果不佳：

```javascript
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

react会改变每个child，而不是意识到它可以保持<li>Duke</li>和<li>Villanova</li>子树完整。这种低效率可能是一个问题。

#### Keys

为了解决这个问题，React支持一个key属性。当children有keys的时候，React使用key来匹配原始树中的children和新树中的children。例如，为我们上面的低效率示例添加一个key可以使树转换高效：

```javascript
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

现在React知道key“2014”的元素是新元素，key“2015”和“2016”的元素刚刚移动。

在实践中，找到一个key通常不难。您要显示的元素可能已经有一个唯一的ID，所以key可以来自您的数据：

```javascript
<li key={item.id}>{item.name}</li>
```

如果情况并非如此，则可以将新的ID属性添加到模型中，或者对内容的某些部分进行散列以生成key。关键只在于它的兄弟姐妹是独一无二的，而不是全局唯一的。

作为最后的手段，您可以传递数组中的项目索引作为关键字。如果项目从未重新排序，这可以很好地工作，但重新排序会很慢。

当索引用作key时，重新排序也会导致组件状态的问题。组件实例基于其key进行更新和重用。如果key是索引，则移动项目会改变它。因此，像受控输入之类的组件状态可能会以意想不到的方式混淆和更新。

在CodePen上有一个使用索引作为key可能导致的问题的[例子][1]，还有同一个例子的[更新版本][2]，展示了如何不使用索引作为key将解决这些重新排序，排序和预先考虑的问题。

<hr />

### 权衡

记住reconciliation(和解)算法是一个很重要的实现细节。 React可以在每个动作上重新渲染整个应用程序;最终的结果将是一样的。为了清楚起见，在这种情况下重新渲染意味着为所有组件调用渲染，并不意味着React将卸载并重新安装它们。它只会按照前面章节所述的规则进行应用。

为了更快地创建通用用例，我们定期改进启发式。在目前的实现中，你可以表达一个事实，就是一个子树已经在兄弟姐妹之间移动了，但是你不能说它已经移动到别的地方了。该算法将重新渲染完整的子树。

因为React依赖于启发式，如果它们背后的假设不符合，性能将受到影响。

 1. 该算法不会尝试匹配不同组件类型的子树。如果您发现自己在输出非常相似的两种组件类型之间交替，则可能需要使其类型相同。实际上，我们并没有发现这是一个问题。

 2. key应该是稳定的，可预测的和独特的。不稳定的key（如Math.random（）生成的key）将导致许多组件实例和DOM节点被不必要地重新创建，这可能会导致子组件中的性能下降和状态丢失。

  [1]: https://codepen.io/pen?&editors=0010
  [2]: https://codepen.io/pen?&editors=0010