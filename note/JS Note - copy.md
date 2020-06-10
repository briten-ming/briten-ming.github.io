## Deep Copy and Shallow Copy
深拷贝和浅拷贝

大部分编程语言存值都有两种方式：存实值、存地址。JS 也不例外。这是因为变量类型分为基本类型和引用类型。  
- 基本类型：值比较简单，单一不易变，体积小，就存实际的值。像 Number、String。
- 引用类型：值比较复杂，包含其他类型，经常改变，体积大，就需要分配一个特定的内存来存储所有的东西，此时变量存的就是所分配内存的内存地址。像 Object、Array。
```js
//基本类型
const num = 1;
const str = 'hello';

//引用类型
const obj = { word: 'hello' };
const arr = [1, 2];
```
基本类型的变量赋值就是实实在在的赋值给了另一个变量。  
引用类型的变量赋值则是把源数据的内存地址赋值给了另一个变量。两个变量仍然共用该地址所指向的内存。所以两个变量的更改会互相影响。
```js
// 基本类型
var a = 1;
var b = a;
a = 2;
console.log(a, b); // 2 1

//引用类型
var a = { word: 'hello' };
var b = a;
a.word = 'hi';
console.log(a, b); // { word: 'hi' }  { word: 'hi' }
```
