# React 生命周期
React 是一个声明式的组件化的框架，每一个 React 组件都有它自己的生命周期，而该组件处于不同的生命周期阶段时会执行不同的方法。

---

## 初始化阶段：constructor
首先一个组件的产生要从它的构造方法开始。
```js
import React, { Component } from 'react';

class Test extends Component {
  constructor(props) {
    super(props);
    this.state = {
        age: 23
    };
  }
}
```
React 中的组件是类式的写法(class)，我们创建一个 Test 组件，Test 组件继承了 React 的 Component，也就继承了 React 的体系，接下来才会有 render() 和生命周期。

一般在 constructor 中会做一些初始化的工作 (props,state)，比如继承父组件的 props ，设置 state 的初始值。

---

## 组件挂载前：componentWillMount
componentWillMount() 会在组件 render() 之前执行，并且永远都只执行一次。由于这个方法始终只执行一次，所以如果在这里使用了setState方法之后，组件不会重新渲染。一般会把此处的处理提前到 constructor。

---

## 组件挂载后：componentDidMount
componentDidMount() 会在组件 render() 之后立即执行。此时组件已经在 DOM 中挂载完毕，可以通过 this.getDOMNode() 来进行访问。

如果你想和其他 JavaScript 框架一起使用，可以在这个方法中执行 setTimeout , setInterval 或者发送 Ajax 请求等操作(防止异部操作阻塞UI)。
```JS
componentDidMount(){
    setTimeout(function() {
        this.setState({
            age: 24
        })
    }, 1000);
} 
```
---

## 组件接收新 props：componentWillReceiveProps
componentWillReceiveProps(nextProps) 只在组件由于 props 的变化而更新时，它接收一个参数 nextProps 是传入的新 props ，旧的 props 此时依然使用 this.props 来获取。

componentWillReceiveProps(nextProps) 在初始化时不会被调用。
```JS
componentWillReceiveProps(nextProps) {
    this.setState({
        age: nextProps.sth
    });
}
```
---

## 组件接收新 props、state：shouldComponentUpdate
shouldComponentUpdate(nextProps, nextState) 在组件 props 或 state 变化时执行，它接收两个参数，nextProps 是传入的新 props，nextState 是传入的新 state。它返回一个布尔值，返回 false 则不再进行组件的更新，包括后续的 componentWillUpdate 和 componentDidUpdate 都会完全跳过不再执行，返回 true 则继续进行组件的更新。

shouldComponentUpdate 默认实现总是返回 true。
```JS
shouldComponentUpdate(nextProps, nextState) {
    return true;
}
```
>Tips:  React.PureComponent 用对新的 nextProps、nextState 和旧的 this.props、this.state 的浅比较覆写了 shouldComponentUpdate()。

---

## 组件更新前：componentWillUpdate
componentWillUpdate(nextProps, nextState) 在组件即将更新但是还没有 render 之前执行。它接收两个参数，nextProps 是传入的新 props，nextState 是传入的新 state。

---

## 组件更新后：componentDidUpdate
componentDidUpdate(prevProps, prevState) 在组件更新后执行，此时 render() 已经完毕，DOM 已经生成，this.props、this.state 已经更新。它接收的两个参数 prevProps 和 prevState 指的是组件更新前的 props 和 state。

---

## 组件卸载：componentWillUnmount
componentWillUnmount() 在组件被卸载之前立即执行，一般用来执行必要的清理，如 clearInterval、clearTimeout。