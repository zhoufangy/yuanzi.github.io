```
title: 深入浅出React和Redux(实战)-程墨
date: 2020-03-01 14:55:27
categories: [React]
tags: [Reading notes]
author: Yuanzi
visitors:
```

### React 相关概念

UI=render(state)

React理念：

1.响应式编程(Reactive Programming)

2.基于组件开发应用

Redux：

Component：React首要思想是通过组件(Component)来开发应用

JSX：JavaScript的语法扩展，在JavaScript中编写像Html一样的代码。

Webpack：进行模块打包

Babel：转译JavaScript代码

ES6语法糖，由Webpack和Babel转译成所有浏览器都支持的ES5语法

Virtual DOM ：每次渲染都只重新渲染最少的DOM元素，比对上次生成的Virtual DOM树，进行差异化修改。

​	Virtual DOM对DOM树的抽象，DOM树：对HTML的抽象。



### React工作方式的优点

JQuery:直观易懂，但代码往往互相纠缠，难以维护，特别是项目逐渐变得庞大时。

React：利用函数式编程思维来解决用户界面渲染的问题，最大的优势是开发者的效率会大大提高，开发出来的代码可维护性和可阅读性也大大增强。

事件

事件    —>    render —>    Virtual DOM —> DOM修改

事件

### 设计高质量的React组件

合理的分而治之，尽量保持一个组件只做一件事。

划分组件边界的原则:高内聚(High Cohesion)，低耦合(Low Couping)

#### React组件的数据种类

​	prop：组件的对外接口

​		propTypes：参数检查，开发时辅助检查，发布时可以使用 babel-react-optimize自动去除。

​		需要引入prop-type插件

​	state：组件的内部状态，赋值需要使用this.setState

​	prop和state的区别

​		prop用于定义外部接口，state用于记录内部状态。

​		prop的赋值在外部世界使用组件时，state的赋值在组件内部。

​		组件不应该改变prop的值，而state存在的目的就是让组件来改变的。

#### 组件的生命周期		

##### 装载过程(Mount)，组件第一次在DOM树中渲染的过程。

​	Constructor

​		初始化state

​		绑定成员函数的this环境

​	getInitialState：返回值用来初始化组件的this.state,但是只有用React.createClass(废弃)方法创造的组件类才会生效。

​	getDefaultProps：返回值可以作为props的初始值，但是只有用React.createClass方法创造的组件类才会生效。

​	componentWillMount：在render之前调用，可以在服务端和浏览器端被调用

​	render：返回JSX描述的结构，最终由React来操作渲染过程。

​	componentDidMount：在render之后调用，只能在浏览器端被调用。可以执行其他UI库代码，比如JQuery

##### 更新过程(Update)，组件被重新渲染的过程。

​	componentWillReceiveProps：只要是父组件的render函数被调用，在render函数里面被渲染的子组件就会经历更新过程，不管父组件传给子组件的props有没有改变，都会触发componentWillReceiveProps函数。

​	shouldComponentUpdate：决定了组件什么时候不需要渲染。返回一个布尔值，告诉React库这个组件在这次更新过程中是否要继续

​	componentWillUpdate

​	render

​	componentDidUpdate：无论更新过程发生在服务器端还是浏览器端，该函数都会被调用。

##### 卸载过程(Unmount)，组件从DOM中删除的过程。

​	componentWillUnmount：处理清理工作，如在componentDidMount中用非React方法创造的DOM元素，需要在componentWillUnmount中清理掉，以避免可能造成的内存泄露。



问题

1.React的PropTypes的string类型没有定义

原因：React从V15.5版本就不支持PorpTypes属性了，改到了prop-types里了。

解决办法：

	1）下载prop-types包
	npm install --save-dev prop-types
	2）引入代码import PropTypes from 'prop-types';



### Redux

Redux是Flux框架的巨大改进，强调单一数据源、保持状态只读和数据改变只能通过纯函数完成的基本原则



store

Context

React-redux

connect：连接容器组件和傻瓜组件

Provider：提供包含store的context

​	subscribe

​	dispatch   组成 store

​	getState

### 代码文件的组织方式

##### 按角色组织 

reducer ：所有redux的reducer

actions：所有action构造函数

components:所有傻瓜组件

containers: 所有容器组件

##### 按功能组织 (Organzied by Feature) (开发redux应用首选)

actionTypes.js定义action类型

actions.js定义action构造函数，决定这个功能模块可以接受的动作

views目录，包含这个功能模块中所有的React组件，包括傻瓜组件和容器组件

index.js这个文件把所有的角色导入然后统一导出

### Store状态树设计

原则：

一个模块控制一个状态节点

避免冗余数据

树形结构扁平

