先附上antd源码地址：[https://github.com/ant-design/ant-design/tree/master/components/alert][1]


昨天写了一篇分析antd之button组件的分析，今晚继续讲antd组件篇，这篇文章主要介绍的是alert实现原理，以及我们可以从antd的组件思想中学习到的react组件开发知识。
ps：antd用的是typescript，如果是纯ES写法稍微有些不同。

下面这张图是alert组件的主要结构图。
![alert][2]


有这么几个部分：
1、demo：alert组件的使用方法
2、style：组件内部可能用到的初始化样式
3、2个.md说明文档，一个是英文版，一个是中文版
4、index.tsx：alert组件（关于这个组件，我是有话要说的，这个命名应该用alert，然后index通常是用来导出alert组件，antd每个组件都不是同一个人写的，估计写alert组件的人也没考虑那么多。）

大概知道了alert项目文件的构成之后，如何去分析组件怎样实现的呢？
先别看代码，看一下?提供的中文文档。
![图片描述][3]


  [1]: https://github.com/ant-design/ant-design/tree/master/components/alert
  [2]: /img/bVHksO
  [3]: /img/bVHks8

主要看API部分，这些api就是组件内部需要定义的接口，一共有8个参数，包括类型、事件等可能需要用到的功能。假设你们公司也打算用react来封装自己的组件，首先要考虑的是制定这样一份API方案，确定需要实现的功能以及保留的功能。

看完文档之后，对alert组件的数据模型有了一个大概的了解，那么接下来就要看看代码是如何实现的。
react组件其实就是一个JSX语法组成的模板，给dom绑定事件，从外部传入需要的参数等。

下面这个是index.tsx的源码：

```
import React from 'react'; //react都认识了
import ReactDOM from 'react-dom'; //用来获取当前的dom节点，这里只有这一个用途
import Animate from 'rc-animate'; //动画组件，好吧，原来antd的组件内部是这么不纯净，导入这么多额外的插件，难怪有人觉得antd太庞大了。
import Icon from '../icon'; //icon又出现了，这小子几乎在好几个antd组件都会用到
import classNames from 'classnames'; //定义样式对象

//构造函数，干啥的呢，现在还不知道，往下看吧。
function noop() {}

//这些就是可以调用的API
export interface AlertProps {
  /**
   * Type of Alert styles, options:`success`, `info`, `warning`, `error`
   */
  type?: 'success' | 'info' | 'warning' | 'error';
  /** Whether Alert can be closed */
  closable?: boolean;
  /** Close text to show */
  closeText?: React.ReactNode;
  /** Content of Alert */
  message: React.ReactNode;
  /** Additional content of Alert */
  description?: React.ReactNode;
  /** Callback when close Alert */
  onClose?: React.MouseEventHandler<any>;
  /** Whether to show icon */
  showIcon?: boolean;
  style?: React.CSSProperties;
  prefixCls?: string;
  className?: string;
  banner?: boolean;
}

//组件的入口在这里，一个继承于React.Component的Alert子类。
export default class Alert extends React.Component<AlertProps, any> {
//defaultProps是react组件的一个参数
  static defaultProps = {
    type: 'info',
  };
//从类的思想来看，constructor是子类Alert的构造函数，这个组件和button组件的写法有所不同，可能是出自2个工程师之手，我们可以看到在构造函数里面初始化了state的2个参数closing、closed。
  constructor(props) {
    super(props);
    this.state = {
      closing: true,
      closed: false,
    };
  }
//组件内部的点击关闭事件
  handleClose = (e) => {
    e.preventDefault();
    let dom = ReactDOM.findDOMNode(this) as HTMLElement;
    dom.style.height = `${dom.offsetHeight}px`;
    // Magic code
    // 重复一次后才能正确设置 height
    dom.style.height = `${dom.offsetHeight}px`;
    
    //设置完高度之后通过setState来更新状态，关闭alert。
    this.setState({
      closing: false,
    });
    //关闭时触发的回调函数，onClose可以在外部定义，至于noop，在这个组件并没有实现任何功能。
    (this.props.onClose || noop)(e);
  }
//动画结束时触发的回调函数，是动画插件提供的功能，不能算作本组件自己定义的函数。该回调只做了一件事，更新state。
  animationEnd = () => {
    this.setState({
      closed: true,
      closing: true,
    });
  }
//终于到了render方法了，每个react组件都有一个render方法，然后必然又一个return dom。
  render() {
//从外部传入的参数，通过this.props传入，一般用const来定义，这里用let不太合适，但不是个错误。
    let {
      closable, description, type, prefixCls = 'ant-alert', message, closeText, showIcon, banner,
      className = '', style,
    } = this.props;

    // banner模式默认有 Icon，如果传入了showIcon，就显示showIcon，否则显示banner，那要是banner也没有传入呢，那就啥都不显示了。
    showIcon = showIcon || banner;
    // banner模式默认为警告，想要使用其他类型success、info、error，就不要传入banner，然后传入type即可。
    type = banner ? 'warning' : type;
    
    //根据传入的type类型来判断icon要显示那种类型样式。注意，icon也是一个小组件。
    let iconType = '';
    switch (type) {
      case 'success':
        iconType = 'check-circle';
        break;
      case 'info':
        iconType = 'info-circle';
        break;
      case 'error':
        iconType = 'cross-circle';
        break;
      case 'warning':
        iconType = 'exclamation-circle';
        break;
      default:
        iconType = 'default';
    }

    // use outline icon in alert with description
    if (!!description) {
      iconType += '-o';
    }
    //classNames用法很简单，冒号左边是类名，右边是bool，true就显示当前样式，false就不显示当前样式，而close、description、icon、banner的样式通过外部是否传入参数或者state的状态来判断，type的样式就默认显示。
    let alertCls = classNames(prefixCls, {
      [`${prefixCls}-${type}`]: true,
      [`${prefixCls}-close`]: !this.state.closing,
      [`${prefixCls}-with-description`]: !!description,
      [`${prefixCls}-no-icon`]: !showIcon,
      [`${prefixCls}-banner`]: !!banner,
    }, className);

    // 当closeText传入为true时，将closable设置为true，我很好奇closable不也是一个可以外部传入的值吗，为什么还需要通过closeText来判断呢，感觉这3行代码有点不合理。
    if (closeText) {
      closable = true;
    }
    //如果closable为true，则closeIcon等于a标签，否则等于空。
    const closeIcon = closable ? (
      <a onClick={this.handleClose} className={`${prefixCls}-close-icon`}>
        {closeText || <Icon type="cross" />}
      </a>
    ) : null;
    //如果closed是true，就return null，false则return下面的组件。
    return this.state.closed ? null : (
      <Animate
        component=""
        showProp="data-show"
        transitionName={`${prefixCls}-slide-up`}
        onEnd={this.animationEnd}
      >
        <div data-show={this.state.closing} className={alertCls} style={style}>
          {showIcon ? <Icon className={`${prefixCls}-icon`} type={iconType} /> : null}
          <span className={`${prefixCls}-message`}>{message}</span>
          <span className={`${prefixCls}-description`}>{description}</span>
          {closeIcon}
        </div>
      </Animate>
    );
  }
}
```

根据alert组件的模型，我们可以总结出其他react组件的开发模式。
1、写好API文档，这些API将作为组件的参数。
2、写一个基本的react组件架构，比如import、export class、render()、constructor()、interface。
3、接着就在render()方法里面写需要外部传入的参数，通过this.props来控制。
4、在return里面写好你的dom结构，你还可能在render方法定义可变的样式，类似上面的alert组件。
5、给dom绑定事件，然后在alert组件内部写这些事件的逻辑。
6、写逻辑这部分是最难的，要花多点心思去组织你的代码。

赶紧去自己尝试些一个类似的组件吧。
