## 深拷贝和浅拷贝

大部分编程语言存值都有两种方式：存实值、存地址。JS 也不例外。这是因为变量类型分为基本类型和引用类型。  
- 基本类型：值比较简单，单一不易变，体积小，就存实际的值。像 Number、String。
- 引用类型：值比较复杂，包含其他类型，经常改变，体积大，就需要分配一个特定的内存来存储所有的东西，此时变量存的就是所分配内存的内存地址。像 Object、Array。
```js
// 基本类型
const num = 1;
const str = 'hello';

// 引用类型
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

// 引用类型
var a = { word: 'hello' };
var b = a;
a.word = 'hi';
console.log(a, b); // { word: 'hi' }  { word: 'hi' }
```
很多时候这会造成意料之外的 bug，所以深拷贝和浅拷贝应运而生。

### 浅拷贝
浅拷贝指创建一个新的对象，新对象的属性值是原对象的完整复制（基本类型是值的复制，引用类型是地址的复制）  
```js
var a = { 
  name: 'jehu', // 基本类型
  other: { age: 24 }, // 引用类型
};
// 使用Object.assign()实现浅拷贝
var b = Object.assign({}, a);
console.log(b);
// { 
//    name: 'jehu', 
//    other: { age: 24 },
// }

a.name = 'tom';
a.other.age = 14;
console.log(a);
// { 
//    name: 'tom', 
//    other: { age: 14 },
// }
console.log(b);
// { 
//    name: 'jehu', 
//    other: { age: 14 },
// }
```
可以看到浅拷贝对于基本类型已经得到了两份互不干扰的数据，但是对于引用类型因为只是拷贝了地址，所以还是会互相影响。
通俗的讲就是浅拷贝只是浅浅的对对象的第一层的完整拷贝。

### 深拷贝
深拷贝指创建一个新的对象，新对象的属性值是原对象的完整的[递归](about:blank)复制。
```js
var a = { 
  name: 'jehu', // 基本类型
  other: { age: 24 }, // 引用类型
};
// 使用JSON.parse(JSON.stringify())实现深拷贝
var b = JSON.parse(JSON.stringify(a));
console.log(b);
// { 
//    name: 'jehu', 
//    other: { age: 24 },
// }

a.name = 'tom';
a.other.age = 14;
console.log(a);
// { 
//    name: 'tom', 
//    other: { age: 14 },
// }
console.log(b);
// { 
//    name: 'jehu', 
//    other: { age: 24 },
// }
```
可以看到深拷贝得到的是完全不同但是所有值都相同的两个对象。
深拷贝才是真正意义上的 copy 了一份原对象的复制。
