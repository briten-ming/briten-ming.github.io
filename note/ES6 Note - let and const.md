let 和 const 是 ES6 的入门语法。

---
在 ES6 之前，声明变量总是使用 var ，这给编写 js 带来了极大的便利，根据能量守恒定律，这也带来了一些问题：

```javascript
var p = document.querySelectorAll('p');
for (var i = 0; i < p.length; i++) {
  p[i].addEventListener('click', function(e) {
    console.log(i)
  });
}
```
看上去这段代码好像是要给所有的 **p**  标签添加 click 事件：点击第 i 个 **p** 标签打印 i 到控制台。

可是实际上你运行之后就会发现，无论你点击第几个 **p** 标签，打印到控制台的总是 **p** 标签的总数，也就是 i 最后一次循环的值。

这是因为 var 声明的变量 i 在全局都有效，并且全局只有一个 i，每次循环 i 的值都会加 1，而循环中每次给 **p** 标签绑定的点击事件 `console.log(i)` 都指向全局的 i，所以点击 **p** 标签时打印到控制台的总是最后一次循环后修改的 i 值，也就是页面上 **p** 标签的数量。这就是作用域的污染。

在 ES6 之前，这个问题可以通过闭包解决：
```javascript
var p = document.querySelectorAll('p');
for (var i = 0; i < p.length; i++) {
  (function (j){ 
      p[j].addEventListener('click', function(e) {
        console.log(j)
      });
  }(i));
}
```
每次循环都使用一个立即执行的函数来创建一个闭包，来保证每次循环给 **p** 标签绑定事件所打印的值都是当前循环传进函数里的函数内局部变量 j。

ES6 中 let 几乎为此而生：
```javascript
var p = document.querySelectorAll('p');
for (let i = 0; i < p.length; i++) {
  p[i].addEventListener('click', function(e) {
    console.log(i)
  });
}
```
for 循环中使用 let 声明变量，那么该变量就只在本次循环中有效，并且每次循环都会创建一个新的变量，他们之间互不影响。

与其对立，const 用来声明一个不能重新赋值的常量：
```javascript
const age = 14;
age = 20 //Uncaught TypeError: Assignment to constant variable.
```
不能重新赋值，所以声明时必须初始化：
```javascript
const age;//Uncaught SyntaxError: Missing initializer in const declaration
```
这样就可以一定程度上避免某些不该变化的常量因为一些错误代码导致变化，从而发生代码异常的情况。

值得一提的是 const 所绑定的常量只是控制该常量本身不能被重新赋值，但是如果常量保存的是一个对象，则此对象的值仍然是可以改变的：
```javascript
const me = {
  name: 'Jehu',
  age: 24
};
me.age = 18;
console.log(me.age);
//18
me = {};
//Uncaught TypeError: Assignment to constant variable.
```
这也是值传递和引用传递的范例。对于像字符串、数值等简单的数据类型，变量保存的内存地址就指向它的值，而对于像对象、数组等复杂的数据类型，变量保存的内存地址指向的是对于对象的的引用。const 只是保证变量保存的内存地址不变，而无法保证这个内存地址的指向不可改变。

---
 使用 let 和 const 声明变量时不会存在**变量提升**而是会出现**暂时性死区**（Temporal Dead Zone，简称 TDZ）。

 JS 引擎在扫描代码时，var 声明的变量会将其提升至顶部作用域，即扫描代码时就已经定义了这个变量，只是值是 undefined :
```javascript
console.log(jehu);
//Uncaught ReferenceError: jehu is not defined
```
```javascript
console.log(jehu);
var jehu = 'me';
//undefined
```
而使用 let 和 const 声明的变量，在此变量声明之前，在其作用域中该变量都是不可用的，也就是 TDZ ：
```javascript
console.log(hair);
const hair = 'black';
//Uncaught ReferenceError: Cannot access 'hair' before initialization
```
```javascript
let gender = 'male';
{
    console.log(gender);
    let gender = 'female';
}
//Uncaught ReferenceError: Cannot access 'gender' before initialization
```