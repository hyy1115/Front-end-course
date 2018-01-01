因为react是单页应用，所以我们可能需要根据不同的路由改变文档的title，那么，这时候你可能就会用到[react-document-title][1]插件。

这个插件主文件代码41行，主要导入了下面3个依赖包：

    var React = require('react'),
    PropTypes = require('prop-types'),
    withSideEffect = require('react-side-effect');    
react-side-effect是一个类似Connect组件的容器，通常它被称为高阶组件。

但是，实际上，我们可以思考，是否可以不使用这个插件完成不同路由修改title的功能，答案是当然可以。

如果使用原生js，修改title的代码只需要一行：

    document.title = '我是标题'
**在react中，我们可以使用非常少的代码封装出一个公共组件，来修改每个路由的title。**

    import React from 'react'
    import PropTypes from 'prop-types'
    export default class ReactDocumentTitle extends React.Component {
        setTitle() {
            const { title } = this.props
            document.title = title
        }
        componentDidMount() {
            this.setTitle()
        }
        componentDidUpdate() {
            this.setTitle()
        }
        render() {
            return React.Children.only(this.props.children)
        }
    }
    ReactDocumentTitle.propTypes = {
        title: PropTypes.string.isRequired
    }
这份代码是将react-side-effect和react-document-title合并到一起做的事情，我把它叫做精简版。

**使用该组件：**

    import ReactDocumentTitle from 'path/ReactDocumentTitle'
    
    render() {
        return (
            <ReactDocumentTitle title="文档标题">
                //这里仅能有一个唯一的root元素。
            </ReactDocumentTitle>
        )
    }

如果你对高阶组件的写法有兴趣，可以研究一下[react-side-effect][2]。需要注意的是，这个高阶组件的代码是使用了babel编译后的结果，你可能看起来没那么容易理解。

**如果把我上面写的那段代码使用babel编译，你再试着理解一下：**

    'use strict';
    
    exports.__esModule = true;
    
    var _react = require('react');
    
    var _react2 = _interopRequireDefault(_react);
    
    var _propTypes = require('prop-types');
    
    var _propTypes2 = _interopRequireDefault(_propTypes);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
    
    function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }
    
    function _possibleConstructorReturn(self, call) { if (!self) { throw new ReferenceError("this hasn't been initialised - super() hasn't been called"); } return call && (typeof call === "object" || typeof call === "function") ? call : self; }
    
    function _inherits(subClass, superClass) { if (typeof superClass !== "function" && superClass !== null) { throw new TypeError("Super expression must either be null or a function, not " + typeof superClass); } subClass.prototype = Object.create(superClass && superClass.prototype, { constructor: { value: subClass, enumerable: false, writable: true, configurable: true } }); if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass; }
    
    var ReactDocumentTitle = function (_React$Component) {
        _inherits(ReactDocumentTitle, _React$Component);
    
        function ReactDocumentTitle() {
            _classCallCheck(this, ReactDocumentTitle);
    
            return _possibleConstructorReturn(this, _React$Component.apply(this, arguments));
        }
    
        ReactDocumentTitle.prototype.setTitle = function setTitle() {
            var title = this.props.title;
    
            document.title = title;
        };
    
        ReactDocumentTitle.prototype.componentDidMount = function componentDidMount() {
            this.setTitle();
        };
    
        ReactDocumentTitle.prototype.componentDidUpdate = function componentDidUpdate() {
            this.setTitle();
        };
    
        ReactDocumentTitle.prototype.render = function render() {
            return _react2.default.Children.only(this.props.children);
        };
    
        return ReactDocumentTitle;
    }(_react2.default.Component);
    
    exports.default = ReactDocumentTitle;
    
    ReactDocumentTitle.propTypes = {
        title: _propTypes2.default.string.isRequired
    };

**这里就有一个非常有趣的地方，以后你使用ES6写了一个react组件，然后再编译成ES5之后，发布到github上，别人就会觉得你的代码高大上很多。**


  [1]: https://github.com/gaearon/react-document-title
  [2]: https://github.com/gaearon/react-side-effect