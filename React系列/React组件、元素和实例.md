# React组件、元素和实例
组件，它们的实例和元素之间的差异混淆了许多React初学者。为什么有三个不同的术语？是指画在屏幕上的东西？

### 管理实例
如果您是React新手，那么您可能以前只使用过组件类和实例。例如，您可以通过创建一个类来声明一个Button组件。当应用程序正在运行时，您可能会在屏幕上显示该组件的多个实例，每个实例都有自己的属性和本地状态。这是传统的面向对象的UI编程。为什么引入元素？

在这个传统的UI模型中，由您来负责创建和销毁子组件实例。如果一个表单组件想要呈现一个Button组件，那么它需要创建它的实例，并用任何新的信息手动保持它的最新状态。

```javascript
class Form extends TraditionalObjectOrientedView {
  render() {
    // 读取传递给视图的一些数据
    const { isSubmitted, buttonText } = this.attrs;

    if (!isSubmitted && !this.button) {
      // 表格尚未提交。创建按钮！
      this.button = new Button({
        children: buttonText,
        color: 'blue'
      });
      this.el.appendChild(this.button.el);
    }

    if (this.button) {
      // 该按钮是可见的。更新文本！
      this.button.attrs.children = buttonText;
      this.button.render();
    }

    if (isSubmitted && this.button) {
      // 表格已提交。销毁按钮！
      this.el.removeChild(this.button.el);
      this.button.destroy();
    }

    if (isSubmitted && !this.message) {
      // 表格已提交。显示成功消息！
      this.message = new Message({ text: 'Success!' });
      this.el.appendChild(this.message.el);
    }
  }
}
```
这是伪代码，但是当您编写复合UI代码时，使用像Backbone这样的库，以面向对象的方式保持行为一致，或多或少。

每个组件实例必须保持对其DOM节点和子组件实例的引用，并在时间正确时创建，更新和销毁它们。代码行随着组件可能状态数量的平方而增长，并且父母可以直接访问其子组件实例，使得将来很难将其解耦。

那么，React与众不同呢？

<hr />
### 元素描述树
在React中，这是元素来拯救的地方。元素是一个普通的对象，描述组件实例或DOM节点及其所需的属性。它仅包含有关组件类型（例如Button），其属性（例如其颜色）以及其中的任何子元素的信息。

一个元素不是一个实际的实例。相反，这是一种告诉React你想在屏幕上看到什么的方法。你不能在元素上调用任何方法。它只是一个具有两个字段的不可变描述对象：type：（string | ReactClass）和props：Object

### DOM元素
当一个元素的类型是一个字符串时，它表示一个具有该标签名称的DOM节点，并且props对应于它的属性。这是React将呈现的内容。例如：

```javascript
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```
这个元素只是将下面的HTML表示为一个普通对象的一种方式：
```javascript
<button class='button button-blue'>
  <b>
    OK!
  </b>
</button>
```
注意元素是如何嵌套的。按照惯例，当我们要创建一个元素树时，我们指定一个或多个子元素作为其包含元素的子元素。

重要的是，子元素和父元素都只是描述而不是实际的实例。创建它们时，他们不会在屏幕上引用任何内容。你可以创建它们并把它们扔掉，这并不重要。

React元素很容易遍历，不需要被解析，当然，它们比实际的DOM元素要轻得多 - 它们只是对象！
### 组件元素
但是，元素的类型也可以是与React组件对应的函数或类：
```javascript
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```
这是React的核心思想。

**描述组件的元素也是一个元素，就像描述DOM节点的元素一样。它们可以互相嵌套和混合。**

这个特性可以让你定义一个DangerButton组件作为一个具有特定颜色属性值的Button，而不用担心Button是否呈现给DOM <button>，<div>或者其他东西：
```javascript
const DangerButton = ({ children }) => ({
  type: Button,
  props: {
    color: 'red',
    children: children
  }
});
```
您可以在单个元素树中混合和匹配DOM和组件元素：
```javascript
const DeleteAccount = () => ({
  type: 'div',
  props: {
    children: [{
      type: 'p',
      props: {
        children: 'Are you sure?'
      }
    }, {
      type: DangerButton,
      props: {
        children: 'Yep'
      }
    }, {
      type: Button,
      props: {
        color: 'blue',
        children: 'Cancel'
      }
   }]
});
```
或者，如果您更喜欢JSX：
```javascript
const DeleteAccount = () => (
  <div>
    <p>Are you sure?</p>
    <DangerButton>Yep</DangerButton>
    <Button color='blue'>Cancel</Button>
  </div>
);
```
这种混合和匹配有助于保持组件彼此分离，因为它们可以仅通过组合来表达既是和唯一的关系：

- 按钮是具有特定属性的DOM <按钮>。 
- DangerButton是一个具有特定属性的按钮。 
- DeleteAccount在<div>中包含一个Button和一个DangerButton。

### 组件封装元素树
当React看到一个具有函数或类类型的元素时，它知道要求该元件呈现什么元素，给定相应的props。

当它看到这个元素时：
```javascript
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```
React会询问Button呈现的内容。按钮将返回这个元素：
```javascript
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```
React会重复这个过程，直到它知道页面上每个组件的底层DOM标签元素。

React就像一个小孩，问“Y是什么”，每一个“X是Y”，你解释给他们，直到他们弄清楚世界上的每一件小事情。

记得上面的表单示例吗？它可以用React写成如下：
```javascript
const Form = ({ isSubmitted, buttonText }) => {
  if (isSubmitted) {
    // 表格提交！返回一个消息元素。
    return {
      type: Message,
      props: {
        text: 'Success!'
      }
    };
  }

  // 表格仍然可见！返回一个按钮元素。
  return {
    type: Button,
    props: {
      children: buttonText,
      color: 'blue'
    }
  };
};
```
仅此而已！对于React组件，props是输入，元素树是输出。

**返回的元素树可以包含描述DOM节点的元素和描述其他组件的元素。这使您可以在不依赖内部DOM结构的情况下编写UI的独立部分。**

我们让React创建，更新和销毁实例。我们使用从组件返回的元素来描述它们，而React负责管理这些实例。

### 组件可以是类或函数
上面的代码中，Form，Message和Button是React组件。它们既可以像上面那样写成函数，也可以写成从React.Component降序的类。这三种声明组件的方式大部分是等价的：
```javascript
// 作为props的功能
const Button = ({ children, color }) => ({
  type: 'button',
  props: {
    className: 'button button-' + color,
    children: {
      type: 'b',
      props: {
        children: children
      }
    }
  }
});

// 使用React.createClass（）工厂
const Button = React.createClass({
  render() {
    const { children, color } = this.props;
    return {
      type: 'button',
      props: {
        className: 'button button-' + color,
        children: {
          type: 'b',
          props: {
            children: children
          }
        }
      }
    };
  }
});

// 作为从React.Component降序的ES6类
class Button extends React.Component {
  render() {
    const { children, color } = this.props;
    return {
      type: 'button',
      props: {
        className: 'button button-' + color,
        children: {
          type: 'b',
          props: {
            children: children
          }
        }
      }
    };
  }
}
```
当一个组件被定义为一个类时，它比函数组件更强大一点。它可以存储一些本地状态，并在相应的DOM节点被创建或销毁时执行自定义逻辑。

一个函数组件功能不那么强大，但更简单，并且只用一个render（）方法就像一个类组件。除非您只需要在class上提供的功能，否则我们鼓励您使用函数组件。

**但是，无论是函数还是类，从根本上说它们都是React的组件。他们把props作为他们的输入，并把元素作为他们的输出。**

### 自上而下调解
当你这样写时：
```javascript
ReactDOM.render({
  type: Form,
  props: {
    isSubmitted: false,
    buttonText: 'OK!'
  }
}, document.getElementById('root'));
```
React将会询问Form组件返回的元素树，给出这些props。它将逐渐“提炼”对简单原语的理解：

```
// React: 你告诉我这个...
{
  type: Form,
  props: {
    isSubmitted: false,
    buttonText: 'OK!'
  }
}

// React: ...Form告诉我这个...
{
  type: Button,
  props: {
    children: 'OK!',
    color: 'blue'
  }
}

// React: ...Button告诉我这个！我想我已经完成了。
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```
这是React调用调用的过程的一部分，调用在您调用ReactDOM.render（）或setState（）时启动。在调整结束时，React知道结果DOM树，像react-dom或react-native这样的呈现器应用更新DOM节点所必需的最小改变集合（或React Native情况下的平台特定视图） 。

这个逐渐完善的过程也是React应用程序易于优化的原因。如果组件树的某些部分变得太大而无法有效地访问React，则可以告诉它如果相关的props没有改变，就跳过这个“细化”并区分树的某些部分。如果props是不可改变的，计算props是否改变是非常快的，所以react和不变性共同作用，并且可以用最小的努力提供很大的优化。

您可能已经注意到，这个博客条目谈论了很多关于组件和元素，而不是太多关于实例。事实是，React中的实例比大多数面向对象的UI框架中的重要性要少得多。

只有声明为类的组件才有实例，并且您从不直接创建它们：React为您做了这些。虽然父组件实例访问子组件实例的机制存在，但它们仅用于必要的操作（例如设置焦点在字段上），通常应避免。

React负责为每个类组件创建一个实例，因此可以用方法和本地状态以面向对象的方式编写组件，但除此之外，实例在React的编程模型中并不是非常重要，并且由React本身来管理。

<hr />

### 总结
元素是一个普通的对象，用DOM节点或其他组件来描述你想要在屏幕上显示的内容。元素可以在其道具中包含其他元素。创建一个React元素很便宜。一旦创建了一个元素，它就不会发生变异。

一个组件可以用几种不同的方式来声明。它可以是一个带有render（）方法的类。另外，在简单的情况下，它可以被定义为一个函数。无论哪种情况，都需要props作为输入，并返回一个元素树作为输出。

当一个组件接收到一些props作为输入时，这是因为一个特定的父组件返回了一个具有它的类型和这些props的元素。这就是为什么人们说React中的props是单向的：从parents到children

在你写的组件类中，一个实例就是你所说的。这对于存储本地状态和对生命周期事件做出反应非常有用。

函数组件根本没有实例。类组件有实例，但是你永远不需要直接创建一个组件实例--React处理这个。

最后，要创建元素，请使用React.createElement（），JSX或元素工厂帮助程序。不要在真实代码中将元素编写为普通对象，只要知道它们是引擎盖下的普通对象即可。