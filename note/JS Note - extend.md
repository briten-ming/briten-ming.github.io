## JavaScript 继承
JavaScript 语言最初只是一门脚本语言，是没有类`class`和继承`extends`的概念的。不过这些都可以依靠 JavaScript 基于 **原型** 这一特性来模拟实现。  
> ES6中引入了`class`和`extends`关键字，但那只是语法糖，JavaScript 仍然是基于原型的。  

本文介绍几种 JavaScript 继承的实现方式。首先我们来创建一个"父类"。
```js
function Animal(name = 'Animal', voice){
  this.name = name;
  this.voice = voice;
  this.action = ['say'];
}
Animal.prototype.isAnimal = true;
Animal.prototype.say = function (){ 
  console.log(`The ${this.name} saying: ${this.voice}!`) 
}
```
### 1.原型继承
```js
function Dog(name){
  this.name = name;
}
Dog.prototype = new Animal();
Dog.prototype.constructor = Dog; //修复子类构造函数

var tom = new Dog('tom');
console.log(tom.name);          // "tom"
console.log(tom.voice);         // undefined 
console.log(tom.action);        // ["say"]
tom.say();                      // The tom saying: undefined!

var bob = new Dog('bob');
bob.action.push('sleep');
console.log(bob.action);        // ["say", "sleep"]
console.log(tom.action);        // ["say", "sleep"]
```
- 核心：将父类的实例作为子类的原型，子类便可以顺着原型链去到父类的属性。
- 特点：
  - 创建对象时无法向父类的构造函数传参
  - 继承父类的所有属性是浅拷贝，引用类型会相互有副作用
  - 无法实现多继承

### 2.构造函数继承
```js
function Dog(name){
  Animal.call(this, name, 'WONNG')
  this.name = name;
}

var tom = new Dog('tom');
console.log(tom.name);          // "tom"
console.log(tom.voice);         // "WONNG" 
console.log(tom.action);        // ["say"]
console.log(tom.say);           // undefined

var bob = new Dog('bob');
bob.action.push('sleep');
console.log(bob.action);        // ["say", "sleep"]
console.log(tom.action);        // ["say"]
```
- 核心：在子类的构造函数中执行父类的构造函数
- 特点：
  - 可以实现多继承
  - 无法继承父类原型上的属性
  - 子类的每个属性都是父类的深拷贝，互相没有副作用
 
### 3.组合继承
```js
function Dog(name){
  Animal.call(this, name, 'WONNG')
  this.name = name;
}
Dog.prototype = new Animal();
Dog.prototype.constructor = Dog; //修复子类构造函数

var tom = new Dog('tom');
console.log(tom.name);          // "tom"
console.log(tom.voice);         // "WONNG" 
console.log(tom.action);        // ["say"]
tom.say();                      // The tom saying: WONNG!

var bob = new Dog('bob');
bob.action.push('sleep');
console.log(bob.action);        // ["say", "sleep"]
console.log(tom.action);        // ["say"]
```
- 核心：在子类的构造函数中执行父类的构造函数
- 特点：
  - 可以实现多继承
  - 继承了父类原型上的属性
  - 子类的每个属性都是父类的深拷贝，互相没有副作用
  - 子类在构造函数和原型上分别创建了两份父类实例，子类构造函数上的父类实例覆盖了子类原型上的父类实例
---
至此组合继承已经近乎完美的实现了 JavaScript 的继承，但是还是不够优雅，因为在组合继承的实现中，子类原型上的父类实例是多余的。
```js
console.log(bob);           // {name: "bob", voice: "WONNG", action: ["say", "sleep"]}
console.log(bob.__proto__); // {name: "Animal", voice: undefined, action: ["say"], constructor: ƒ Dog(name)}
```
可以看到`bob.__proto__`上的`name` `voice` `action`属性都是多余的，甚至当我们删除`bob`的某个属性时，可能会从子类原型上得到意料之外的父类实例属性。
```js
delete bob.name
console.log(bob.name);      // "Animal"
```
所以下面介绍另一个继承方式来解决这个问题。

### 4.寄生组合继承
```js
function Dog(name){
  Animal.call(this, name, 'WONNG')
  this.name = name;
}
function Mid(){}                 // 没有实例属性的中间类
Mid.prototype = Animal.prototype;
Dog.prototype = new Mid();
Dog.prototype.constructor = Dog; // 修复子类构造函数

var tom = new Dog('tom');
console.log(tom.name);          // "tom"
console.log(tom.voice);         // "WONNG"
console.log(tom.action);        // ["say"]
tom.say();                      // The tom saying: WONNG!

var bob = new Dog('bob');
bob.action.push('sleep');
console.log(bob.action);        // ["say", "sleep"]
console.log(tom.action);        // ["say"]

console.log(bob);               // {name: "bob", voice: "WONNG", action: ["say", "sleep"]}
console.log(bob.__proto__);     // {constructor: ƒ Dog(name)}
```
- 核心：用一个没有实例属性的中间类抽离掉原型上的父类实例属性
- 特点：
  - 可以实现多继承
  - 继承了父类原型上的属性
  - 子类的每个属性都是父类的深拷贝，互相没有副作用
  - 原型上干净，只有来自父类的原型属性
  
这种方式就是现阶段很完美，也是最常用的继承方式了。代码上还可以根据各种情况优化一下，比如将 call 改成 apply，我们就可以用原生的 arguments 数组更方便的向父类的构造函数传参，
定义中间类的时候也可以用一个立即执行的匿名函数包裹一下，让中间类处在函数的局部作用域，防止全局作用域下的同名变量或者函数污染。
