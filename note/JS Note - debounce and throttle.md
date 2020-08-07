## JavaScript 防抖和节流
在前端开发中有一些事件是会频繁的触发的，如`onmousemove` `onresize` `onscroll`等。如果我们在事件中做一些比较复杂的事情比如请求数据源，由于事件触发的非常频繁，
性能会受到很大的影响。  

目前的解决方案一般有两种：
- 防抖 debounce
- 节流 throttle  

很多人搞不清他俩的区别，网上的说法也是比较绕，我说下我自己的理解。首先他俩都是因事件频繁触发而诞生的，所以事件触发肯定是主语，另外他们都需要指定一个时间段作为判定边界。
- 防抖：禁止在指定时间内的连续触发（连续触发即视为抖动），在事件连续被触发时，除非该事件静默了指定的时间，否则不触发。
- 节流：在事件连续被触发时，事件会按照 **指定时间/每次** 的频率触发  

这样解释好像更加贴和他们的名字，更好理解🤓 ，下面来看下如何实现。

---

首先简单的梳理一下，我们有一个函数，该函数会绑定到某个事件上，它会在事件被触发时执行。
```js
function fn(e){
  // do something...
}
div.addEventListener('mousemove',fn);
```
那么我们会希望有这么一个函数来实现防抖/节流的功能：
- 它接收两个参数，分别是要 do something 的一个函数`fn`和一个时间`wait`作为判定边界
- 它内部有个定时器来处理函数执行前需要等待多久
- 定时器跑完后，它返回一个函数，来提供给事件调用

那么它大概是这样的：
```js
function xxx(func, wait) {
    // ...
    return function () {
        // ...
        setTimeout(func, wait);
    }
}
```
那么我们来顺着这个思路分别实现一下**防抖**和**节流**

### 防抖
根据防抖所要实现的功能，连续触发事件时只在事件静默了**wait**时间后才触发一次，那么每次触发事件时都要重置定时器
```js
function debounce(func, wait) {
    let time = null;
    return function () {
        if(time !== null) clearTimeout(time);
        time = setTimeout(func, wait);
    }
}
```
这就是实现了一个最小版的防抖函数，但是事件当中我们经常要用到事件句柄`evt`的，那么自然就想到使用`apply(this,arguments)`了，顺便也把`this`指向的问题修复一下
```js
function debounce(func, wait){
    let time = null;
    return function () {
      const _this = this;
      if(time !== null) clearTimeout(time);
      time = setTimeout(() => {
        func.apply(_this, arguments);
      }, wait);
    }
}
```

### 节流
有了上面防抖的经验，很快就可以同样实现一个节流函数，思路是在计时阶段直接`return`不做任何处理，计时完毕后执行`func`并删除计时器，重新开始下一次的计时
```js
function throttle(func, wait){
    let time = null;
    return function () {
      const _this = this;
      if(time !== null) return;
      time = setTimeout(() => {
        func.apply(_this, arguments);
        time = null;
      }, wait);
    }
}
```
实例代码[请戳](https://jsbin.com/henizixiga/edit?html,js,output)
