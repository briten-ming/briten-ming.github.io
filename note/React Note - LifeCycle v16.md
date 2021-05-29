# React v16 生命周期
React v16 已经发布很久了，它废弃了三个旧生命周期函数 `componentWillMount` `componentWillReceiveProps` `componentWillUpdate`，引入了两个钩子函数 `static getDerivedStateFromProps` `getSnapshotBeforeUpdate` 但是随着 Hook 的兴起，我也没有实际的用过这两个函数，恶补一下。文末有在线 Demo

---

## static getDerivedStateFromProps
基本语法
```js
static getDerivedStateFromProps(nextProps, nextState) {
    // do some thing...
    return nextState;
}
```
概念：会在调用 `render` 方法之前调用，并且在**初始挂载**及后续**更新**时都会被调用。它应返回一个对象来更新 state，如果返回 null 则不更新任何内容。


值得一提的是，`getDerivedStateFromProps` 是一个静态函数，所以你不能在其中使用 `this`，我想 React 是想以此来阻止用户在这里做一些副作用，官方文档中也重点提及了此[问题](https://zh-hans.reactjs.org/docs/react-component.html#static-getderivedstatefromprops)。不过也不用担心更新 state 的问题，React 会用此函数的返回值来更新 state。
>Tips:如果你不返回任何值或者返回了 undefined，相当于你返回了 null，但是会有一个⚠️警告  
>Warning: App.getDerivedStateFromProps(): A valid state object (or null) must be returned. You have returned undefined.

## getSnapshotBeforeUpdate
基本语法
```js
getSnapshotBeforeUpdate(preProps, preState) {
    // do some thing...
    return snapRet;
}

componentDidUpdate(props, state, snapRet) {
    console.log(snapRet);
}
```
概念：在最近一次渲染输出（提交到 DOM 节点）`之前`调用。它使得组件能在发生更改之前从 DOM 中捕获一些信息（例如，滚动位置）。此生命周期方法的任何返回值将作为参数传递给 `componentDidUpdate`。

>Tips:与 `getDerivedStateFromProps` 类似，`getSnapshotBeforeUpdate` 不返回任何值或者返回了 undefined 也会有一个相同的⚠️警告，不同的是在 `componentDidUpdate` 中依然可以取到 undefined

[在线测试](https://codesandbox.io/s/react-v16-addlifecycle-xilqz)