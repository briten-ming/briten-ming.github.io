# React Fiber
最早的Fiber官方解释来源于2016年React团队成员Acdlite的一篇介绍 (opens new window)。

从上一章的学习我们知道：

在React15及以前，Reconciler采用递归的方式创建虚拟DOM，递归过程是不能中断的。如果组件树的层级很深，递归会占用线程很多时间，造成卡顿。

为了解决这个问题，React16将递归的无法中断的更新重构为异步的可中断更新，由于曾经用于递归的虚拟DOM数据结构已经无法满足需要。于是，全新的Fiber架构应运而生。

---

## 双缓存Fiber树
在React中最多会同时存在两棵Fiber树。当前屏幕上显示内容对应的Fiber树称为current Fiber树，正在内存中构建的Fiber树称为workInProgress Fiber树。

current Fiber树中的Fiber节点被称为current fiber，workInProgress Fiber树中的Fiber节点被称为workInProgress fiber，他们通过alternate属性连接。

currentFiber.alternate === workInProgressFiber;
workInProgressFiber.alternate === currentFiber;
React应用的根节点通过使current指针在不同Fiber树的rootFiber间切换来完成current Fiber树指向的切换。

即当workInProgress Fiber树构建完成交给Renderer渲染在页面上后，应用根节点的current指针指向workInProgress Fiber树，此时workInProgress Fiber树就变为current Fiber树。

每次状态更新都会产生新的workInProgress Fiber树，通过current与workInProgress的替换，完成DOM更新。

## 时间切片原理
时间切片的本质是模拟实现requestIdleCallback (opens new window)。

除去“浏览器重排/重绘”，下图是浏览器一帧中可以用于执行JS的时机。

一个task(宏任务) -- 队列中全部job(微任务) -- requestAnimationFrame -- 浏览器重排/重绘 -- requestIdleCallback
requestIdleCallback是在“浏览器重排/重绘”后如果当前帧还有空余时间时被调用的。

浏览器并没有提供其他API能够在同样的时机（浏览器重排/重绘后）调用以模拟其实现。

唯一能精准控制调用时机的API是requestAnimationFrame，他能让我们在“浏览器重排/重绘”之前执行JS。

这也是为什么我们通常用这个API实现JS动画 —— 这是浏览器渲染前的最后时机，所以动画能快速被渲染。

所以，退而求其次，Scheduler的时间切片功能是通过task（宏任务）实现的。

最常见的task当属setTimeout了。但是有个task比setTimeout执行时机更靠前，那就是MessageChannel (opens new window)。

所以Scheduler将需要被执行的回调函数作为MessageChannel的回调执行。如果当前宿主环境不支持MessageChannel，则使用setTimeout。
