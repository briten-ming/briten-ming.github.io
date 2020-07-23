## JavaScript 错误
大多数时候我们只关心报错信息，如`x is not defined`,`Cannot read property 'x' of undefined/null` 却忽略了他是什么类型的错误，虽然是个不起眼的细节，但是被人问到了只能靠蒙还是挺难受的。  
下面来盘点一下那些 JavaScript 中常见的错误类型。

#### ReferenceError 引用错误
出现在变量不合法引用的情况下，如 TDZ、变量未声明等。比较常见的有：
```js
console.log(x) // ReferenceError: x is not defined
x()            // ReferenceError: x is not defined
{
  console.log(y);
  let y = 1
}              // ReferenceError: Cannot access 'y' before initialization
```

#### SyntaxError 语法错误
出现在解析代码时，代码有语法错误的情况下，如缺失/多余标点符号 `]` `)` `;`、不合法的赋值语句等等。比较常见的有：
```js
console.log(1)); // SyntaxError: Unexpected token ')'
1++1             // SyntaxError: Invalid left-hand side expression in postfix operation
var 1a = 1;      // SyntaxError: Invalid or unexpected token
/g/x/            // SyntaxError: Invalid regular expression flags
function(){}     // SyntaxError: Function statements require a function name
```

#### TypeError 类型错误
出现在变量、参数的类型或者属性不符合预期的情况，比如对 const 变量进行了赋值、对不是函数的对象或者变量进行函数调用、对不可迭代的对象进行了迭代操作、将不是构造器的对象或者变量来作为构造器使用等等。比较常见的有：
```js
const a = 1;
a = 2;                 // TypeError: Assignment to constant variable.
a();                   // TypeError: a is not a function
for (let tmp of a) {}  // TypeError: a is not iterable
var b = new a();       // TypeError: a is not a constructor
1 instanceof null      // TypeError: Right-hand side of 'instanceof' is not an object
```

#### RangeError 范围错误
出现在和数数值相关的操作中，所操作的数值是非法的（字符串）或者数值不在可用范围之内的情况，大部分时候这种情况会返回 `NaN`，部分时候会抛出`RangeError`。比如给数组的长度赋了一个非法值、
函数调用时传递了一个超出范围的值、栈溢出等等。比较常见的有：
```js
new Array(-1);       // RangeError: Invalid array length
(2).toString("99");  // RangeError: toString() radix argument must be between 2 and 36
'abc'.repeat(-1);    // RangeError: Invalid count value
function loop(x) {
  if (x >= 1000000000000)
    return;
  loop(x + 1);
}
loop(0);             // RangeError: Maximum call stack size exceeded
```

#### URIError URI错误
出现在 URI 编码和解码不成功的情况，比如在调用 `decodeURI` `encodeURI` `encodeURIComponent` `decodeURIComponent`这些函数时解码错误：
```js
decodeURI('%*');     // URIError: URI malformed
```
  
  
> 注：不同浏览器的错误提示可能不同，本文的例子皆是在 Chrome 浏览器下运行
