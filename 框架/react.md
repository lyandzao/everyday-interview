## 1.redux中间件的原理是什么？

增强dispatch的功能

## 2.你会把数据统一放到redux中管理，还是共享数据放在redux中管理

把所有数据放在redux中，定位问题非常快

## 3.componentWillReceiveProps的调用时机

props改变的时候，第二次

## 4.react性能优化的最佳实践

pureComponent的内部有一个浅比较，shouldComponentUpdate

用pureComponent和Immutable

## 5.虚拟dom是什么？为什么虚拟dom会提升代码性能

真实dom的比较很慢，虚拟dom比较简单，key值相同可以直接复用，不用循环需比对，直接通过key值找到比对

## 6.webpack中，是借助loader完成的jsx代码的转化，还是babel？

babel -- preset-react

## 7.调用setState后，发生了什么？

React 为了优化性能，有可能会将多个 setState() 调用合并为一次更新。

`setState()`方法通过一个队列机制实现`state`更新，当执行`setState()`的时候，会将需要更新的`state`合并之后放入状态队列，而不会立即更新`this.state`(可以和浏览器的事件队列类比)。如果我们不使用`setState`而是使用`this.state.key`来修改，将不会触发组件的`re-render`。如果将`this.state`赋值给一个新的对象引用，那么其他不在对象上的`state`将不会被放入状态队列中，当下次调用`setState()`并对状态队列进行合并时，直接造成了`state`丢失。
作者：云淡风轻的成长链接：https://www.jianshu.com/p/a883552c67de

## 8.ref的作用是什么，你在什么业务场景下使用过refs

获取dom的宽高

触发必要的动画和事件

集成第三方 DOM 库

## 9.refs是一个函数

https://juejin.im/post/5b59287af265da0f601317e3

## 10.高阶组件你是怎么理解的，它本质是一个什么东西？

接受一个组件，返回一个包装过的组件

## 11.受控组件和非受控组件

受控组件更好,数据驱动

example：

```javascript
受控组件<input value={this.state.value}>
非受控组件<input ref={} />,this...=
```

## 12.函数组件和hooks

## 13.this指向问题你一般怎么解决？

箭头函数，bind

## 14.函数式组件怎么优化？

react.memo

## 15.在哪个生命周期里发送ajax？

在componentDidMount()发送

1. componentWillMount()已经废弃
2. SSR项目时，ComponentWillMount会执行两次

https://blog.csdn.net/qq1498982270/article/details/98969259

## 16.ssr的原理是什么？

## 17.redux-saga的设计思想是什么？什么是sideEffects？

## 18.react,jquery,vue是否有可能共存在一个项目中？

可以

## 19.组件是什么？类是什么？类被编译成什么？

组件是页面的一部分
类被编译成构造函数

## 20.你是怎么获取新知识时？

vpn，订阅官方twitter

## 21.如何避免ajax数据重新获取？

redux

## 22.react-router4的核心思想是什么？和3有什么区别？

## 23.reselect是做什么的？

充当computed的作用，做缓存，提升性能

## 24.react-router的基本原理，hashHistory，browserHistory

hashHistory，路由不需要后端配合

browserHistory，需要后端服务器配置

## 25.什么情况下使用异步组件？

reloadble库

按需加载

## 26.xss攻击在react中如何防范？

内部会过滤，慎用dangerouslySetInnerHTML={{}}

## 27.getDerivedStateFromProps,getSnapshotBeforeUpdate

## 28.immutable.js和redux的最佳实践

