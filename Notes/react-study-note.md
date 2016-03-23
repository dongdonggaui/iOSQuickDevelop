# [React 入门实例教程](http://www.ruanyifeng.com/blog/2015/03/react.html) 学习笔记

## HTML 模板
使用 React 的网页源码，结构大致如下。

~~~html
<!DOCTYPE html>
<html>
  <head>
    <script src="../build/react.js"></script>
    <script src="../build/react-dom.js"></script>
    <script src="../build/browser.min.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <script type="text/babel">
      // ** Our code goes here! **
    </script>
  </body>
</html>
~~~

1. 最后一个 `<script>` 标签的 `type` 属性为 `text/babel` 。这是因为 React 独有的 JSX 语法，跟 JavaScript 不兼容。凡是使用 JSX 的地方，都要加上 `type="text/babel"`
2. 上面代码一共用了三个库： `react.js` 、`react-dom.js` 和 `Browser.js` ，它们必须首先加载。其中，`react.js` 是 React 的核心库，`react-dom.js` 是提供与 DOM 相关的功能，`Browser.js` 的作用是将 JSX 语法转为 JavaScript 语法，这一步很消耗时间，实际上线的时候，应该将它放到服务器完成

~~~bash
$ babel src --out-dir build
~~~

此命令可以将 src 子目录的 js 文件进行语法转换，转码后的文件全部放在 build 子目录

## ReactDOM.render()
ReactDOM.render 是 React 的最基本方法，用于将模板转为 HTML 语言，并插入指定的 DOM 节点

## JSX 语法
1. HTML 语言直接写在 JavaScript 语言之中，不加任何引号，这就是 JSX 的语法，它允许 HTML 与 JavaScript 的混写
2. 遇到 HTML 标签（以 `<` 开头），就用 HTML 规则解析；遇到代码块（以 `{` 开头），就用 JavaScript 规则解析
3. JSX 允许直接在模板插入 JavaScript 变量。如果这个变量是一个数组，则会展开这个数组的所有成员

## 组件
1. React 允许将代码封装成组件（component），然后像插入普通 HTML 标签一样，在网页中插入这个组件。React.createClass 方法就用于生成一个组件类
2. 变量 `HelloMessage` 就是一个组件类。模板插入 `<HelloMessage />` 时，会自动生成 `HelloMessage` 的一个实例
3. 所有组件类都必须有自己的 `render` 方法，用于输出组件（类似于 iOS 中的 drawRect ）
4. 组件类的第一个字母必须大写，否则会报错，比如`HelloMessage`不能写成`helloMessage`
5. 组件类只能包含一个顶层标签，否则也会报错
6. 组件的属性可以在组件类的 `this.props` 对象上获取，比如 `name` 属性就可以通过 `this.props.name` 读取
7. 添加组件属性，有一个地方需要注意，就是 `class` 属性需要写成 `className` ，`for` 属性需要写成 `htmlFor` ，这是因为 `class` 和 `for` 是 JavaScript 的保留字

## this.props.children
1. `this.props` 对象的属性与组件的属性一一对应，但是有一个例外，就是 `this.props.children` 属性。它表示组件的所有子节点
2. `this.props.children` 的值有三种可能：`undefined`、 `object`、`array`
3. React 提供一个工具方法 `React.Children` 来处理 `this.props.children` 。我们可以用 `React.Children.map` 来遍历子节点，而不用担心 `this.props.children` 的数据类型是 undefined 还是 object

## PropTypes
1. 组件类的PropTypes属性，用来验证组件实例的属性是否符合要求，以下例子中，`PropTypes` 告诉 React，这个 `title` 属性是必须的，而且它的值必须是字符串

	~~~js
	var MyTitle = React.createClass({
	  propTypes: {
	    title: React.PropTypes.string.isRequired,
	  },
	
	  render: function() {
	     return <h1> {this.props.title} </h1>;
	   }
	});
	~~~

2. `getDefaultProps` 方法可以用来设置组件属性的默认值

	~~~js
	var MyTitle = React.createClass({
	  getDefaultProps : function () {
	    return {
	      title : 'Hello World'
	    };
	  },
	
	  render: function() {
	     return <h1> {this.props.title} </h1>;
	   }
	});
	~~~

## 获取真实的DOM节点
1. 组件并不是真实的 DOM 节点，而是存在于内存之中的一种数据结构，叫做虚拟 DOM （virtual DOM）
2. 只有当它插入文档以后，才会变成真实的 DOM
3. 根据 React 的设计，所有的 DOM 变动，都先在虚拟 DOM 上发生，然后再将实际发生变动的部分，反映在真实 DOM上，这种算法叫做 [DOM diff](http://calendar.perfplanet.com/2013/diff/) ，它可以极大提高网页的性能表现
4. 从组件获取真实 DOM 的节点，就要用到 `ref` 属性

## this.state
1. `this.props` 表示那些一旦定义，就不再改变的特性，而 `this.state` 是会随着用户互动而产生变化的特性。

## 表单
用户在表单填入的内容，属于用户跟组件的互动，所以不能用 this.props 读取

## 组件的生命周期
> - Mounting：已插入真实 DOM
> - Updating：正在被重新渲染
> - Unmounting：已移出真实 DOM

对应回调

> - componentWillMount()
> - componentDidMount()
> - componentWillUpdate(object nextProps, object nextState)
> - componentDidUpdate(object prevProps, object prevState)
> - componentWillUnmount()

两种特殊状态的处理函数

> - componentWillReceiveProps(object nextProps)：已加载组件收到新的参数时调用
> - shouldComponentUpdate(object nextProps, object nextState)：组件判断是否重新渲染时调用

## Ajax
组件的数据来源，通常是通过 Ajax 请求从服务器获取，可以使用 componentDidMount 方法设置 Ajax 请求，等到请求成功，再用 this.setState 方法重新渲染 UI 

可以把一个Promise对象传入组件

~~~js
ReactDOM.render(
  <RepoList
    promise={$.getJSON('https://api.github.com/search/repositories?q=javascript&sort=stars')}
  />,
  document.body
);
~~~