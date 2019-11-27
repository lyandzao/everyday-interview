## 1.`['1', '2', '3'].map(parseInt)` what & why ?

主要是讲**JS的映射与解析** 早在 2013年, 加里·伯恩哈德就在微博上发布了以下代码段:

```js
['10','10','10','10','10'].map(parseInt);
// [10, NaN, 2, 3, 4]
```

### parseInt

`parseInt()` 函数解析一个字符串参数，并返回一个指定基数的整数 (数学系统的基础)。

```js
const intValue = parseInt(string[, radix]);
```

`string` 要被解析的值。如果参数不是一个字符串，则将其转换为字符串(使用 ToString 抽象操作)。字符串开头的空白符将会被忽略。

`radix` 一个介于2和36之间的整数(数学系统的基础)，表示上述字符串的基数。默认为10。 `返回值` 返回一个整数或NaN

```js
parseInt(100); // 100
parseInt(100, 10); // 100
parseInt(100, 2); // 4 -> converts 100 in base 2 to base 10
```

**注意：** 在`radix`为 undefined，或者`radix`为 0 或者没有指定的情况下，JavaScript 作如下处理：

- 如果字符串 string 以"0x"或者"0X"开头, 则基数是16 (16进制).
- 如果字符串 string 以"0"开头, 基数是8（八进制）或者10（十进制），那么具体是哪个基数由实现环境决定。ECMAScript 5 规定使用10，但是并不是所有的浏览器都遵循这个规定。因此，永远都要明确给出radix参数的值。
- 如果字符串 string 以其它任何值开头，则基数是10 (十进制)。

更多详见[parseInt | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseInt)

```javascript
parseInt(’10‘，9)=1*9^1+0*9^0=9;
parseInt('11',10)=11;
```

### map

`map()` 方法创建一个新数组，其结果是该数组中的每个元素都调用一个提供的函数后返回的结果。

```js
var new_array = arr.map(function callback(currentValue[,index[, array]]) {
 // Return element for new_array
 }[, thisArg])
```

可以看到`callback`回调函数需要三个参数, 我们通常只使用第一个参数 (其他两个参数是可选的)。 `currentValue` 是callback 数组中正在处理的当前元素。 `index`可选, 是callback 数组中正在处理的当前元素的索引。 `array`可选, 是callback map 方法被调用的数组。 另外还有`thisArg`可选, 执行 callback 函数时使用的this 值。

```js
const arr = [1, 2, 3];
arr.map((num) => num + 1); // [2, 3, 4]
```

更多详见[Array.prototype.map() | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

### 回到真实的事例上

回到我们真实的事例上

```js
['1', '2', '3'].map(parseInt)
```

对于每个迭代`map`, `parseInt()`传递两个参数: **字符串和基数**。 所以实际执行的的代码是：

```js
['1', '2', '3'].map((item, index) => {
	return parseInt(item, index)
})
```

即返回的值分别为：

```js
parseInt('1', 0) // 1
parseInt('2', 1) // NaN
parseInt('3', 2) // NaN, 3 不是二进制
```

所以：

```js
['1', '2', '3'].map(parseInt)
// 1, NaN, NaN
```

由此，加里·伯恩哈德例子也就很好解释了，这里不再赘述

```js
['10','10','10','10','10'].map(parseInt);
// [10, NaN, 2, 3, 4]
```

### 如何在现实世界中做到这一点

如果您实际上想要循环访问字符串数组, 该怎么办？ `map()`然后把它换成数字？使用编号!

```js
['10','10','10','10','10'].map(Number);
// [10, 10, 10, 10, 10]
```

## 2.什么防抖和节流？有什么区别？如何实现？

1. **防抖**

> 触发高频事件后n秒内函数只会执行一次，如果n秒内高频事件再次被触发，则重新计算时间

- 思路：

> 每次触发事件时都取消之前的延时调用方法

```js
function debounce(fn) {
      let timeout = null; // 创建一个标记用来存放定时器的返回值
      return function () {
        clearTimeout(timeout); // 每当用户输入的时候把前一个 setTimeout clear 掉
        timeout = setTimeout(() => { // 然后又创建一个新的 setTimeout, 这样就能保证输入字符后的 interval 间隔内如果还有字符输入的话，就不会执行 fn 函数
          fn.apply(this, arguments);
        }, 500);
      };
    }
    function sayHi() {
      console.log('防抖成功');
    }

    var inp = document.getElementById('inp');
    inp.addEventListener('input', debounce(sayHi)); // 防抖
```

2.  **节流**

> 高频事件触发，但在n秒内只会执行一次，所以节流会稀释函数的执行频率

- 思路：

> 每次触发事件时都判断当前是否有等待执行的延时函数

```js
function throttle(fn) {
      let canRun = true; // 通过闭包保存一个标记
      return function () {
        if (!canRun) return; // 在函数开头判断标记是否为true，不为true则return
        canRun = false; // 立即设置为false
        setTimeout(() => { // 将外部传入的函数的执行放在setTimeout中
          fn.apply(this, arguments);
          // 最后在setTimeout执行完毕后再把标记设置为true(关键)表示可以执行下一次循环了。当定时器没有执行的时候标记永远是false，在开头被return掉
          canRun = true;
        }, 500);
      };
    }
    function sayHi(e) {
      console.log(e.target.innerWidth, e.target.innerHeight);
    }
    window.addEventListener('resize', throttle(sayHi));
```

[参考防抖](https://github.com/mqyqingfeng/Blog/issues/22)

[参考节流](https://github.com/mqyqingfeng/Blog/issues/26)

## 3.介绍下 Set、Map、WeakSet 和 WeakMap 的区别？

- Set
  - 成员唯一、无序且不重复
  - [value, value]，键值与键名是一致的（或者说只有键值，没有键名）
  - 可以遍历，方法有：add、delete、has
- WeakSet
  - 成员都是对象
  - 成员都是弱引用，可以被垃圾回收机制回收，可以用来保存DOM节点，不容易造成内存泄漏
  - 不能遍历，方法有add、delete、has
- Map
  - 本质上是键值对的集合，类似集合
  - 可以遍历，方法很多可以跟各种数据格式转换
- WeakMap
  - 只接受对象作为键名（null除外），不接受其他类型的值作为键名
  - 键名是弱引用，键值可以是任意的，键名所指向的对象可以被垃圾回收，此时键名是无效的
  - 不能遍历，方法有get、set、has、delete

##  4.ES5/ES6 的继承除了写法以外还有什么区别？

1. `class` 声明会提升，但不会初始化赋值。`Foo` 进入暂时性死区，类似于 `let`、`const` 声明变量。

```javascript
const bar = new Bar(); // it's ok
function Bar() {
  this.bar = 42;
}

const foo = new Foo(); // ReferenceError: Foo is not defined
class Foo {
  constructor() {
    this.foo = 42;
  }
}
```

2. `class` 声明内部会启用严格模式。

```javascript
// 引用一个未声明的变量
function Bar() {
  baz = 42; // it's ok
}
const bar = new Bar();

class Foo {
  constructor() {
    fol = 42; // ReferenceError: fol is not defined
  }
}
const foo = new Foo();
```

3. `class` 的所有方法（包括静态方法和实例方法）都是不可枚举的。

```javascript
// 引用一个未声明的变量
function Bar() {
  this.bar = 42;
}
Bar.answer = function() {
  return 42;
};
Bar.prototype.print = function() {
  console.log(this.bar);
};
const barKeys = Object.keys(Bar); // ['answer']
const barProtoKeys = Object.keys(Bar.prototype); // ['print']

class Foo {
  constructor() {
    this.foo = 42;
  }
  static answer() {
    return 42;
  }
  print() {
    console.log(this.foo);
  }
}
const fooKeys = Object.keys(Foo); // []
const fooProtoKeys = Object.keys(Foo.prototype); // []
```

4. `class` 的所有方法（包括静态方法和实例方法）都没有原型对象 prototype，所以也没有`[[construct]]`，不能使用 `new` 来调用。

```javascript
function Bar() {
  this.bar = 42;
}
Bar.prototype.print = function() {
  console.log(this.bar);
};

const bar = new Bar();
const barPrint = new bar.print(); // it's ok

class Foo {
  constructor() {
    this.foo = 42;
  }
  print() {
    console.log(this.foo);
  }
}
const foo = new Foo();
const fooPrint = new foo.print(); // TypeError: foo.print is not a constructor
```

5. 必须使用 `new` 调用 `class`。

```javascript
function Bar() {
  this.bar = 42;
}
const bar = Bar(); // it's ok

class Foo {
  constructor() {
    this.foo = 42;
  }
}
const foo = Foo(); // TypeError: Class constructor Foo cannot be invoked without 'new'
```

6. `class` 内部无法重写类名。

```javascript
function Bar() {
  Bar = 'Baz'; // it's ok
  this.bar = 42;
}
const bar = new Bar();
// Bar: 'Baz'
// bar: Bar {bar: 42}  

class Foo {
  constructor() {
    this.foo = 42;
    Foo = 'Fol'; // TypeError: Assignment to constant variable
  }
}
const foo = new Foo();
Foo = 'Fol'; // it's ok
```

## 5.有以下 3 个判断数组的方法，请分别介绍它们之间的区别和优劣

Object.prototype.toString.call() 、 instanceof 以及 Array.isArray()

#### 1. Object.prototype.toString.call()

每一个继承 Object 的对象都有 `toString` 方法，如果 `toString` 方法没有重写的话，会返回 `[Object type]`，其中 type 为对象的类型。但当除了 Object 类型的对象外，其他类型直接使用 `toString` 方法时，会直接返回都是内容的字符串，所以我们需要使用call或者apply方法来改变toString方法的执行上下文。

```js
const an = ['Hello','An'];
an.toString(); // "Hello,An"
Object.prototype.toString.call(an); // "[object Array]"
```

这种方法对于所有基本的数据类型都能进行判断，即使是 null 和 undefined 。

```js
Object.prototype.toString.call('An') // "[object String]"
Object.prototype.toString.call(1) // "[object Number]"
Object.prototype.toString.call(Symbol(1)) // "[object Symbol]"
Object.prototype.toString.call(null) // "[object Null]"
Object.prototype.toString.call(undefined) // "[object Undefined]"
Object.prototype.toString.call(function(){}) // "[object Function]"
Object.prototype.toString.call({name: 'An'}) // "[object Object]"
```

`Object.prototype.toString.call()` 常用于判断浏览器内置对象时。

更多实现可见 [谈谈 Object.prototype.toString](https://juejin.im/post/591647550ce4630069df1c4a)

#### 2. instanceof

`instanceof`  的内部机制是通过判断对象的原型链中是不是能找到类型的 `prototype`。

使用 `instanceof`判断一个对象是否为数组，`instanceof` 会判断这个对象的原型链上是否会找到对应的 `Array` 的原型，找到返回 `true`，否则返回 `false`。

```js
[]  instanceof Array; // true
```

但 `instanceof` 只能用来判断对象类型，原始类型不可以。并且所有对象类型 instanceof Object 都是 true。

```js
[]  instanceof Object; // true
```

#### 3. Array.isArray()

- 功能：用来判断对象是否为数组

- instanceof 与 isArray

  当检测Array实例时，`Array.isArray` 优于 `instanceof` ，因为 `Array.isArray` 可以检测出 `iframes`

  ```js
  var iframe = document.createElement('iframe');
  document.body.appendChild(iframe);
  xArray = window.frames[window.frames.length-1].Array;
  var arr = new xArray(1,2,3); // [1,2,3]
  
  // Correctly checking for Array
  Array.isArray(arr);  // true
  Object.prototype.toString.call(arr); // true
  // Considered harmful, because doesn't work though iframes
  arr instanceof Array; // false
  ```

- `Array.isArray()` 与 `Object.prototype.toString.call()`

  `Array.isArray()`是ES5新增的方法，当不存在 `Array.isArray()` ，可以用 `Object.prototype.toString.call()` 实现。

  ```js
  if (!Array.isArray) {
    Array.isArray = function(arg) {
      return Object.prototype.toString.call(arg) === '[object Array]';
    };
  }
  ```

## 6.介绍模块化发展历程

可从IIFE、AMD、CMD、CommonJS、UMD、webpack(require.ensure)、ES Module、`<script type="module">` 这几个角度考虑。

https://www.processon.com/view/link/5c8409bbe4b02b2ce492286a#map

模块化主要是用来抽离公共代码，隔离作用域，避免变量冲突等。

**IIFE**： 使用自执行函数来编写模块化，特点：**在一个单独的函数作用域中执行代码，避免变量冲突**。

```js
(function(){
  return {
	data:[]
  }
})()
```

**AMD**： 使用requireJS 来编写模块化，特点：**依赖必须提前声明好**。

```js
define('./index.js',function(code){
	// code 就是index.js 返回的内容
})
```

**CMD**： 使用seaJS 来编写模块化，特点：**支持动态引入依赖文件**。

```js
define(function(require, exports, module) {  
  var indexCode = require('./index.js');
});
```

**CommonJS**： nodejs 中自带的模块化。

```js
var fs = require('fs');
```

**UMD**：兼容AMD，CommonJS 模块化语法。

**webpack(require.ensure)**：webpack 2.x 版本中的代码分割。

**ES Modules**： ES6 引入的模块化，支持import 来引入另一个 js 。

```js
import a from 'a';
```

## 7.全局作用域中，用 const 和 let 声明的变量不在 window 上，那到底在哪里？如何去获取？

在ES5中，顶层对象的属性和全局变量是等价的，var 命令和 function 命令声明的全局变量，自然也是顶层对象。

```js
var a = 12;
function f(){};

console.log(window.a); // 12
console.log(window.f); // f(){}
```

但ES6规定，var 命令和 function 命令声明的全局变量，依旧是顶层对象的属性，但 let命令、const命令、class命令声明的全局变量，不属于顶层对象的属性。

```js
let aa = 1;
const bb = 2;

console.log(window.aa); // undefined
console.log(window.bb); // undefined
```

在哪里？怎么获取？通过在设置断点，看看浏览器是怎么处理的：

![letandconst](https://user-images.githubusercontent.com/20290821/53854366-2ec1a400-4004-11e9-8c62-5a1dd91b8a5b.png)

通过上图也可以看到，在全局作用域中，用 let 和 const 声明的全局变量并没有在全局对象中，只是一个块级作用域（Script）中

怎么获取？在定义变量的块级作用域中就能获取啊，既然不属于顶层对象，那就不加 window（global）呗。

```js
let aa = 1;
const bb = 2;

console.log(aa); // 1
console.log(bb); // 2
```

## 8.下面的代码打印什么内容，为什么？

```js
var b = 10;
(function b(){
    b = 20;
    console.log(b); 
})();
```

几个例子：

```js
var b = 10;
(function b() {
   // 内部作用域，会先去查找是有已有变量b的声明，有就直接赋值20，确实有了呀。发现了具名函数 function b(){}，拿此b做赋值；
   // IIFE的函数无法进行赋值（内部机制，类似const定义的常量），所以无效。
  // （这里说的“内部机制”，想搞清楚，需要去查阅一些资料，弄明白IIFE在JS引擎的工作方式，堆栈存储IIFE的方式等）
    b = 20;
    console.log(b); // [Function b]
    console.log(window.b); // 10，不是20
})();
```

所以严格模式下能看到错误：`Uncaught TypeError: Assignment to constant variable`

```js
var b = 10;
(function b() {
  'use strict'
  b = 20;
  console.log(b)
})() // "Uncaught TypeError: Assignment to constant variable."
```

其他情况例子：

有`window`：

```js
var b = 10;
(function b() {
    window.b = 20; 
    console.log(b); // [Function b]
    console.log(window.b); // 20是必然的
})();
```

有`var`:

```js
var b = 10;
(function b() {
    var b = 20; // IIFE内部变量
    console.log(b); // 20
   console.log(window.b); // 10 
})();
```

## 9.简单改造下面的代码，使之分别打印 10 和 20。

```js
var b = 10;
(function b(){
    b = 20;
    console.log(b); 
})();
```

解：

1）打印10

> ```js
> var b = 10;
> (function b(b) {
>  window.b = 20;
>  console.log(b)
> })(b)
> ```

或者

> ```js
> var b = 10;
> (function b(b) {
>  b.b = 20;
>  console.log(b)
> })(b)
> ```

2）打印20

> ```js
> var b = 10;
> (function b(b) {
>  b = 20;
>  console.log(b)
> })(b)
> ```

或

> ```js
> var b = 10;
> (function b() {
>  var b = 20;
>  console.log(b)
> })()
> ```

## 10.下面代码输出什么

```js
var a = 10;
(function () {
    console.log(a)
    a = 5
    console.log(window.a)
    var a = 20;
    console.log(a)
})()
```

依次输出：undefined -> 10 -> 20

解析：

在立即执行函数中，`var a = 20;` 语句定义了一个局部变量 `a`，由于js的变量声明提升机制，局部变量`a`的声明会被提升至立即执行函数的函数体最上方，且由于这样的提升并不包括赋值，因此第一条打印语句会打印`undefined`，最后一条语句会打印`20`。

由于变量声明提升，`a = 5;` 这条语句执行时，局部的变量`a`已经声明，因此它产生的效果是对局部的变量`a`赋值，此时`window.a` 依旧是最开始赋值的`10`，

## 11.使用 sort() 对数组 [3, 15, 8, 29, 102, 22] 进行排序，输出结果

我的答案：

```
[102, 15, 22, 29, 3, 8]
```

解析：

根据MDN上对`Array.sort()`的解释，默认的排序方法会将数组元素转换为字符串，然后比较字符串中字符的UTF-16编码顺序来进行排序。所以`'102'` 会排在 `'15'` 前面。以下是MDN中的解释原文：

> The sort() method sorts the elements of an array in place and returns the array. The default sort order is built upon converting the elements into strings, then comparing their sequences of UTF-16 code units values.

## 12.输出以下代码执行的结果并解释为什么

```js
var obj = {
    '2': 3,
    '3': 4,
    'length': 2,
    'splice': Array.prototype.splice,
    'push': Array.prototype.push
}
obj.push(1)
obj.push(2)
console.log(obj)
```

`push` 方法有意具有通用性。该方法和 [`call()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call) 或 [`apply()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 一起使用时，可应用在类似数组的对象上。`push` 方法根据 `length` 属性来决定从哪里开始插入给定的值。如果 `length` 不能被转成一个数值，则插入的元素索引为 0，包括 `length` 不存在时。当 `length` 不存在时，将会创建它。

**涉及知识点：**

- 类数组（ArrayLike）：

一组数据，由数组来存，但是如果要对这组数据进行扩展，会影响到数组原型，ArrayLike的出现则提供了一个中间数据桥梁，ArrayLike有数组的特性， 但是对ArrayLike的扩展并不会影响到原生的数组。

- push方法：

push 方法有意具有通用性。该方法和 call() 或 apply() 一起使用时，可应用在类似数组的对象上。push 方法根据 length 属性来决定从哪里开始插入给定的值。如果 length 不能被转成一个数值，则插入的元素索引为 0，包括 length 不存在时。当 length 不存在时，将会创建它。 唯一的原生类数组（array-like）对象是 Strings，尽管如此，它们并不适用该方法，因为字符串是不可改变的。

- 对象转数组的方式：

Array.from()、splice()、concat()等 **题分析**： 这个obj中定义了两个key值，分别为splice和push分别对应数组原型中的splice和push方法，因此这个obj可以调用数组中的push和splice方法，调用对象的push方法：push(1)，因为此时obj中定义length为2，所以从数组中的第二项开始插入，也就是数组的第三项（下表为2的那一项），因为数组是从第0项开始的，这时已经定义了下标为2和3这两项，所以它会替换第三项也就是下标为2的值，第一次执行push完，此时key为2的属性值为1，同理：第二次执行push方法，key为3的属性值为2。此时的输出结果就是： Object(4) [empty × 2, 1, 2, splice: ƒ, push: ƒ]----> [ 2: 1, 3: 2, length: 4, push: ƒ push(), splice: ƒ splice() ]

因为只是定义了2和3两项，没有定义0和1这两项，所以前面会是empty。 如果讲这道题改为：

```js
var obj = {
    '2': 3,
    '3': 4,
    'length': 0,
    'splice': Array.prototype.splice,
    'push': Array.prototype.push
}
obj.push(1)
obj.push(2)
console.log(obj)
```

此时的打印结果就是： Object(2) [1, 2, 2: 3, 3: 4, splice: ƒ, push: ƒ]----> [ 0: 1, 1: 2, 2: 3, 3: 4, length: 2, push: ƒ push(), splice: ƒ splice() ]

原理：此时length长度设置为0，push方法从第0项开始插入，所以填充了第0项的empty 至于为什么对象添加了splice属性后并没有调用就会变成类数组对象这个问题，这是控制台中 DevTools 猜测类数组的一个方式： https://github.com/ChromeDevTools/devtools-frontend/blob/master/front_end/event_listeners/EventListenersUtils.js#L330

## 13.call 和 apply 的区别是什么，哪个性能更好一些

1. Function.prototype.apply和Function.prototype.call 的作用是一样的，区别在于传入参数的不同；
2. 第一个参数都是，指定函数体内this的指向；
3. 第二个参数开始不同，apply是传入带下标的集合，数组或者类数组，apply把它传给函数作为参数，call从第二个开始传入的参数是不固定的，都会传给函数作为参数。
4. call比apply的性能要好，平常可以多用call, call传入参数的格式正是内部所需要的格式，参考[call和apply的性能对比](https://github.com/noneven/__/issues/6)

以前看jQuery源码的时候有看到在源码的注释中有些过call的性能会比apply好，在lodash的源码中也同样的发现有call比apply性能更好的注释，这里我在[jsperf](https://jsperf.com/)上写了几个test case，验证了一下确实call比apply的性能更好。

- 0、lodash源码apply方法重写
  [![img](https://raw.githubusercontent.com/coderwin/__/master/src/asserts/callapply.png)](https://raw.githubusercontent.com/coderwin/__/master/src/asserts/callapply.png)
- 1、无指向无参数对比:
  [![img](https://raw.githubusercontent.com/coderwin/__/master/src/asserts/callapplyperf.png)](https://raw.githubusercontent.com/coderwin/__/master/src/asserts/callapplyperf.png)
- 2、有指向无参数对比:
  [![img](https://raw.githubusercontent.com/coderwin/__/master/src/asserts/callapplyperf1.png)](https://raw.githubusercontent.com/coderwin/__/master/src/asserts/callapplyperf1.png)
- 3、无参数有指向:
  [![img](https://raw.githubusercontent.com/coderwin/__/master/src/asserts/callapplyperf2.png)](https://raw.githubusercontent.com/coderwin/__/master/src/asserts/callapplyperf2.png)
- 4、有参数有指向对比:
  [![img](https://raw.githubusercontent.com/coderwin/__/master/src/asserts/callapplyperf3.png)](https://raw.githubusercontent.com/coderwin/__/master/src/asserts/callapplyperf3.png)

总结: 在我们平时的开发中其实不必关注call和apply的性能问题，但是可以尽可能的去用call，特别是es6的reset解构的支持，call基本可以代替apply，可以看出lodash源码里面并没有直接用Function.prototype.apply，而是在参数较少(1-3)个时采用call的方式调用(因为lodash里面没有超过4个参数的方法，PS如果一个函数的设计超过4个入参，那么这个函数就要考虑重构了)

## 14.输出以下代码的执行结果并解释为什么

```js
var a = {n: 1};
var b = a;
a.x = a = {n: 2};

console.log(a.x) 	
console.log(b.x)
```

结果: undefined {n:2}

首先，a和b同时引用了{n:2}对象，接着执行到a.x = a = {n：2}语句，尽管赋值是从右到左的没错，但是.的优先级比=要高，所以这里首先执行a.x，相当于为a（或者b）所指向的{n:1}对象新增了一个属性x，即此时对象将变为{n:1;x:undefined}。之后按正常情况，从右到左进行赋值，此时执行a ={n:2}的时候，a的引用改变，指向了新对象{n：2},而b依然指向的是旧对象。之后执行a.x = {n：2}的时候，并不会重新解析一遍a，而是沿用最初解析a.x时候的a，也即旧对象，故此时旧对象的x的值为{n：2}，旧对象为 {n:1;x:{n：2}}，它被b引用着。 后面输出a.x的时候，又要解析a了，此时的a是指向新对象的a，而这个新对象是没有x属性的，故访问时输出undefined；而访问b.x的时候，将输出旧对象的x的值，即{n:2}。

## 15.箭头函数与普通函数（function）的区别是什么？构造函数（function）可以使用 new 生成实例，那么箭头函数可以吗？为什么？

箭头函数是普通函数的简写，可以更优雅的定义一个函数，和普通函数相比，有以下几点差异：

1、函数体内的 this 对象，就是定义时所在的对象，而不是使用时所在的对象。

2、不可以使用 arguments 对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。

3、不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数。

4、不可以使用 new 命令，因为：

- 没有自己的 this，无法调用 call，apply。
- 没有 prototype 属性 ，而 new 命令在执行时需要将构造函数的 prototype 赋值给新的对象的 __proto__

new 过程大致是这样的：

```js
function newFunc(father, ...rest) {
  var result = {};
  result.__proto__ = father.prototype;
  var result2 = father.apply(result, rest);
  if (
    (typeof result2 === 'object' || typeof result2 === 'function') &&
    result2 !== null
  ) {
    return result2;
  }
  return result;
}
```

## 16.`a.b.c.d` 和 `a['b']['c']['d']`，哪个性能更高

应该是 `a.b.c.d` 比 `a['b']['c']['d']` 性能高点，后者还要考虑 `[ ]` 中是变量的情况，再者，从两种形式的结构来看，显然编译器解析前者要比后者容易些，自然也就快一点。 下图是两者的 AST 对比： ![image](https://user-images.githubusercontent.com/9009389/56872978-501d9a00-6a61-11e9-9e69-85ff00c031fc.png)

## 17.ES6 代码转成 ES5 代码的实现思路是什么

那么 Babel 是如何把 ES6 转成 ES5 呢，其大致分为三步：

- 将代码字符串解析成抽象语法树，即所谓的 AST
- 对 AST 进行处理，在这个阶段可以对 ES6 代码进行相应转换，即转成 ES5 代码
- 根据处理后的 AST 再生成代码字符串

基于此，其实我们自己就可以实现一个简单的“编译器”，用于把 ES6 代码转成 ES5。

比如，可以使用 `@babel/parser` 的 `parse` 方法，将代码字符串解析成 AST；使用 `@babel/core` 的 `transformFromAstSync` 方法，对 AST 进行处理，将其转成 ES5 并生成相应的代码字符串；过程中，可能还需要使用 `@babel/traverse` 来获取依赖文件等。对此感兴趣的可以看看[这个](https://github.com/FishPlusOrange/easy-webpack)。

## 18.为什么普通 `for` 循环的性能远远高于 `forEach` 的性能，请解释其中的原因。

循环条件中不涉及计算时确实如此，但是若设计计算，forEach的性能反而好一些

## 19.数组里面有10万个数据，取第一个元素和第10万个元素的时间相差多少

数组可以直接根据索引取的对应的元素，所以不管取哪个位置的元素的时间复杂度都是 O(1)

得出结论：**消耗时间几乎一致，差异可以忽略不计**

JavaScript 没有真正意义上的数组，所有的数组其实是对象，其“索引”看起来是数字，其实会被转换成字符串，作为属性名（对象的 key）来使用。所以无论是取第 1 个还是取第 10 万个元素，都是用 key 精确查找哈希表的过程，其消耗时间大致相同。 @lvtraveler 请帮忙测试下稀松数组。

## 20.输出以下代码运行结果

```js
// example 1
var a={}, b='123', c=123;  
a[b]='b';
a[c]='c';  
console.log(a[b]);

---------------------
// example 2
var a={}, b=Symbol('123'), c=Symbol('123');  
a[b]='b';
a[c]='c';  
console.log(a[b]);

---------------------
// example 3
var a={}, b={key:'123'}, c={key:'456'};  
a[b]='b';
a[c]='c';  
console.log(a[b]);
```

这题考察的是对象的键名的转换。

- 对象的键名只能是字符串和 Symbol 类型。
- 其他类型的键名会被转换成字符串类型。
- 对象转字符串默认会调用 toString 方法。

```js
// example 1
var a={}, b='123', c=123;
a[b]='b';

// c 的键名会被转换成字符串'123'，这里会把 b 覆盖掉。
a[c]='c';  

// 输出 c
console.log(a[b]);
// example 2
var a={}, b=Symbol('123'), c=Symbol('123');  

// b 是 Symbol 类型，不需要转换。
a[b]='b';

// c 是 Symbol 类型，不需要转换。任何一个 Symbol 类型的值都是不相等的，所以不会覆盖掉 b。
a[c]='c';

// 输出 b
console.log(a[b]);
// example 3
var a={}, b={key:'123'}, c={key:'456'};  

// b 不是字符串也不是 Symbol 类型，需要转换成字符串。
// 对象类型会调用 toString 方法转换成字符串 [object Object]。
a[b]='b';

// c 不是字符串也不是 Symbol 类型，需要转换成字符串。
// 对象类型会调用 toString 方法转换成字符串 [object Object]。这里会把 b 覆盖掉。
a[c]='c';  

// 输出 c
console.log(a[b]);
```

## 21.input 搜索如何防抖，如何处理中文输入

### 简易防抖

```js
<div>
    <input type="text" id="ipt">
  </div>

  <script>
    let ipt = document.getElementById('ipt');
    let dbFun = debounce()
    ipt.addEventListener('keyup', function (e) {
      dbFun(e.target.value);
    })

    function debounce() {
      let timer;
      return function (value) {
        clearTimeout(timer);
        timer = setTimeout(() => {
          console.log(value)
        }, 500);
      }
    }
  </script>
```

处理中文:[参考](https://segmentfault.com/a/1190000012490380)
事件：compositionstart & compositionend

在 `MDN` 上找到了关于他们的描述，[compositionstart](https://developer.mozilla.org/en-US/docs/Web/Events/compositionstart) 和 [compositionend](https://developer.mozilla.org/en-US/docs/Web/Events/compositionend)。简单点描述如下：

- compositionstart：在输入中文或者语音等需要等待一连串的输入的操作之前，`compositionstart` 事件会触发。
- compositionend：在输入中文或者语音等完毕或取消时，`compositionend` 事件会触发。

## 22.var、let 和 const 区别的实现原理是什么

> 变量生命周期：声明（作用域注册一个变量）、初始化（分配内存，初始化为undefined）、赋值

- var：遇到有var的作用域，**在任何语句执行前都已经完成了声明和初始化**，也就是变量提升而且拿到undefined的原因由来
- function： 声明、初始化、赋值一开始就全部完成，所以函数的变量提升优先级更高
- let：解析器进入一个块级作用域，发现let关键字，变量只是先完成**声明**，并没有到**初始化**那一步。此时如果在此作用域提前访问，则报错xx is not defined，这就是暂时性死区的由来。等到解析到有let那一行的时候，才会进入**初始化**阶段。如果let的那一行是赋值操作，则初始化和赋值同时进行
- const、class都是同let一样的道理

比如解析如下代码步骤：

```javascript
{
// 没用的第一行
// 没用的第二行
console.log(a) // 如果此时访问a报错 a is not defined
let a = 1
}
```

步骤：

1. 发现作用域有let a，先注册个a，仅仅注册
2. 没用的第一行
3. 没用的第二行
4. a is not defined，暂时性死区的表现
5. 假设前面那行不报错，a初始化为undefined
6. a赋值为1

对比于var，let、const只是解耦了声明和初始化的过程，var是在任何语句执行前都已经完成了声明和初始化，let、const仅仅是在任何语句执行前只完成了声明

## 23.介绍下前端加密的常见场景和方法

首先，加密的目的，简而言之就是将明文转换为密文、甚至转换为其他的东西，用来隐藏明文内容本身，防止其他人直接获取到敏感明文信息、或者提高其他人获取到明文信息的难度。<br />通常我们提到加密会想到密码加密、HTTPS 等关键词，这里从场景和方法分别提一些我的个人见解。

<a name="867e6c89"></a>

#### 场景-密码传输

前端密码传输过程中如果不加密，在日志中就可以拿到用户的明文密码，对用户安全不太负责。<br />这种加密其实相对比较简单，可以使用 PlanA-前端加密、后端解密后计算密码字符串的MD5/MD6存入数据库；也可以 PlanB-直接前端使用一种稳定算法加密成唯一值、后端直接将加密结果进行MD5/MD6，全程密码明文不出现在程序中。

**PlanA**<br />使用 Base64 / Unicode+1 等方式加密成非明文，后端解开之后再存它的 MD5/MD6 。

**PlanB**<br />直接使用 MD5/MD6 之类的方式取 Hash ，让后端存 Hash 的 Hash 。

<a name="4687d461"></a>

#### 场景-数据包加密

应该大家有遇到过：打开一个正经网站，网站底下蹦出个不正经广告——比如X通的流量浮层，X信的插入式广告……（我没有针对谁）<br />但是这几年，我们会发现这种广告逐渐变少了，其原因就是大家都开始采用 HTTPS 了。<br />被人插入这种广告的方法其实很好理解：你的网页数据包被抓取->在数据包到达你手机之前被篡改->你得到了带网页广告的数据包->渲染到你手机屏幕。<br />而 HTTPS 进行了包加密，就解决了这个问题。严格来说我认为从手段上来看，它不算是一种前端加密场景；但是从解决问题的角度来看，这确实是前端需要知道的事情。

**Plan**<br />全面采用 HTTPS

<a name="H4e5B"></a>

#### 场景-展示成果加密

经常有人开发网页爬虫爬取大家辛辛苦苦一点一点发布的数据成果，有些会影响你的竞争力，有些会降低你的知名度，甚至有些出于恶意爬取你的公开数据后进行全量公开……比如有些食谱网站被爬掉所有食谱，站点被克隆；有些求职网站被爬掉所有职位，被拿去卖信息；甚至有些小说漫画网站赖以生存的内容也很容易被爬取。

**Plan**<br />将文本内容进行展示层加密，利用字体的引用特点，把拿给爬虫的数据变成“乱码”。<br />举个栗子：正常来讲，当我们拥有一串数字“12345”并将其放在网站页面上的时候，其实网站页面上显示的并不是简单的数字，而是数字对应的字体的“12345”。这时我们打乱一下字体中图形和字码的对应关系，比如我们搞成这样：

> 图形：1 2 3 4 5 字码：2 3 1 5 4

这时，如果你想让用户看到“12345”，你在页面中渲染的数字就应该是“23154”。这种手段也可以算作一种加密。<br />具体的实现方法可以看一下《[Web 端反爬虫技术方案](https://juejin.im/post/5b6d579cf265da0f6e51a7e0)》。

<a name="97ac239a"></a>

#### 参考

[HTTPS 到底加密了什么？](https://zhuanlan.zhihu.com/p/38278311) <br />[Web 端反爬虫技术方案](https://juejin.im/post/5b6d579cf265da0f6e51a7e0) <br />[可以说的秘密-那些我们该讨论的前端加密方法](https://juejin.im/entry/5bc93545e51d450e5f3dceff) <br />[前端加密那点事](https://juejin.im/post/5c452021518825242062979f) <br />[关于反爬虫，看这一篇就够了](https://segmentfault.com/a/1190000005840672)

## 24.写出如下代码的打印结果

```js
function changeObjProperty(o) {
  o.siteUrl = "http://www.baidu.com"
  o = new Object()
  o.siteUrl = "http://www.google.com"
} 
let webSite = new Object();
changeObjProperty(webSite);
console.log(webSite.siteUrl);
```

```javascript
// 这里把o改成a
// webSite引用地址的值copy给a了
function changeObjProperty(a) {
  // 改变对应地址内的对象属性值
  a.siteUrl = "http://www.baidu.com"
  // 变量a指向新的地址 以后的变动和旧地址无关
  a = new Object()
  a.siteUrl = "http://www.google.com"
  a.name = 456
} 
var webSite = new Object();
webSite.name = '123'
changeObjProperty(webSite);
console.log(webSite); // {name: 123, siteUrl: 'http://www.baidu.com'}
```

## 25.请写出如下代码的打印结果

```js
function Foo() {
Foo.a = function() {
console.log(1)
}
this.a = function() {
console.log(2)
}
}
Foo.prototype.a = function() {
console.log(3)
}
Foo.a = function() {
console.log(4)
}
Foo.a();
let obj = new Foo();
obj.a();
Foo.a();
```

输出顺序是 4 2 1 .

```javascript
function Foo() {
    Foo.a = function() {
        console.log(1)
    }
    this.a = function() {
        console.log(2)
    }
}
// 以上只是 Foo 的构建方法，没有产生实例，此刻也没有执行

Foo.prototype.a = function() {
    console.log(3)
}
// 现在在 Foo 上挂载了原型方法 a ，方法输出值为 3

Foo.a = function() {
    console.log(4)
}
// 现在在 Foo 上挂载了直接方法 a ，输出值为 4

Foo.a();
// 立刻执行了 Foo 上的 a 方法，也就是刚刚定义的，所以
// # 输出 4

let obj = new Foo();
/* 这里调用了 Foo 的构建方法。Foo 的构建方法主要做了两件事：
1. 将全局的 Foo 上的直接方法 a 替换为一个输出 1 的方法。
2. 在新对象上挂载直接方法 a ，输出值为 2。
*/

obj.a();
// 因为有直接方法 a ，不需要去访问原型链，所以使用的是构建方法里所定义的 this.a，
// # 输出 2

Foo.a();
// 构建方法里已经替换了全局 Foo 上的 a 方法，所以
// # 输出 1
```

答案为：4, 2, 1 解析：

1. `Foo.a()` 这个是调用 Foo 函数的静态方法 a，虽然 Foo 中有优先级更高的属性方法 a，但 Foo 此时没有被调用，所以此时输出 Foo 的静态方法 a 的结果：**4**
2. `let obj = new Foo();` 使用了 new 方法调用了函数，返回了函数实例对象，此时 Foo 函数内部的属性方法初始化，原型方法建立。
3. `obj.a();` 调用 obj 实例上的方法 a，该实例上目前有两个 a 方法：一个是内部属性方法，另一个是原型方法。当这两者重名时，前者的优先级更高，会覆盖后者，所以输出：**2**
4. `Foo.a();` 根据第2步可知 Foo 函数内部的属性方法已初始化，覆盖了同名的静态方法，所以输出：**1**

## 26.106 题：分别写出如下代码的返回值

```js
String('11') == new String('11');
String('11') === new String('11');
```

```javascript
var str1 = String('11')
var str2 = new String('11')
str1 == str2 // true
str1 === str2 // false
typeof str1  // "string"
typeof str2 // "object"
```

总结：

1. `==`时做了隐式转换，调用了toString
2. 2者类型不一样，一个是`string`，一个是`object`

## 27.请写出如下代码的打印结果

```js
var name = 'Tom';
(function() {
if (typeof name == 'undefined') {
  var name = 'Jack';
  console.log('Goodbye ' + name);
} else {
  console.log('Hello ' + name);
}
})();
```

```js
var name = 'Tom';
(function() {
    console.info('name', name);
    console.info('typeof name', typeof name);
    if (typeof name == 'undefined') {
        var name = 'Jack';
        console.log('Goodbye ' + name);
    } else {
        console.log('Hello ' + name);
    }
})();
name undefined
typeof name undefined
Goodbye Jack
```

## 28.扩展题，请写出如下代码的打印结果

```js
var name = 'Tom';
(function() {
if (typeof name == 'undefined') {
  name = 'Jack';
  console.log('Goodbye ' + name);
} else {
  console.log('Hello ' + name);
}
})();
```

hello Tom 1、首先在进入函数作用域当中，获取name属性 2、在当前作用域没有找到name 3、通过作用域链找到最外层，得到name属性 4、执行else的内容，得到Hello Tom

## 29.输出以下代码运行结果

```js
1 + "1"

2 * "2"

[1, 2] + [2, 1]

"a" + + "b"
```

## 30.为什么 for 循环嵌套顺序会影响性能？

```js
var t1 = new Date().getTime()
for (let i = 0; i < 100; i++) {
  for (let j = 0; j < 1000; j++) {
    for (let k = 0; k < 10000; k++) {
    }
  }
}
var t2 = new Date().getTime()
console.log('first time', t2 - t1)

for (let i = 0; i < 10000; i++) {
  for (let j = 0; j < 1000; j++) {
    for (let k = 0; k < 100; k++) {

    }
  }
}
var t3 = new Date().getTime()
console.log('two time', t3 - t2)
```

两个循环的次数的是一样的，但是 j 与 k 的初始化次数是不一样的

- 第一个循环的 j 的初始化次数是 100 次，k 的初始化次数是 10w 次
- 第二个循环的 j 的初始化次数是 1w 次， k 的初始化次数是 1000w 次

所以相同循环次数，外层越大，越影响性能

## 31.输出以下代码执行结果

```js
function wait() {
  return new Promise(resolve =>
    setTimeout(resolve, 10 * 1000)
  )
}

async function main() {
  console.time();
  const x = wait();
  const y = wait();
  const z = wait();
  await x;
  await y;
  await z;
  console.timeEnd();
}
main();
```

`Promise` 内部代码块是按序执行。和普通的代码执行顺序没区别。只是在调用`then()`时 返回了代码块中`resolve()`返回的结果，其他没有任何buf加成。 `async function`是为了处理`Promise.resolve()`以便即时获取到返回结果,`await`获得`resolve()`或`reject()`返回的结果后会继续执行(所以`resolve()`或`reject()`在哪，决定了它什么时候执行完，继续往下执行) 示例1： 将`resolve()` 放在同步代码块中

```js
function wait() {
  return new Promise(resolve =>{
    console.log(1);
    resolve();
    setTimeout(function(){
      console.log(3);
      // resolve();
      console.log(4);
    }, 3* 1000);
     console.log(2);
  }
 )
}

async function main() {
  console.time();
  const x = wait();
  const y = wait();
  const z = wait();
  await x;
  await y;
  await z;
  console.timeEnd();
}
main();
```

![image](https://user-images.githubusercontent.com/23008329/63481802-5086e880-c4c9-11e9-8ac3-f9c1a107729e.png)

示例2：将`resolve()`放到异步回调中

```js
function wait() {
  return new Promise(resolve =>{
    console.log(1);
    // resolve();
    setTimeout(function(){
      console.log(3);
      resolve();
      console.log(4);
    }, 3* 1000);
     console.log(2);
  }
 )
}

async function main() {
  console.time();
  const x = wait();
  const y = wait();
  const z = wait();
  await x;
  await y;
  await z;
  console.timeEnd();
}
main();
```

![image](https://user-images.githubusercontent.com/23008329/63481972-acea0800-c4c9-11e9-9b68-502ea53751ba.png)

`await`要得到结果才会继续执行，没有就一直等待，不会执行后面的代码。

## 32.输出以下代码执行结果，大致时间就好（不同于上题）

```js
function wait() {
  return new Promise(resolve =>
    setTimeout(resolve, 10 * 1000)
  )
}

async function main() {
  console.time();
  await wait();
  await wait();
  await wait();
  console.timeEnd();
}
main();
```

先说结果，大概30秒多点，30秒是因为每个等待10秒，同步执行。 其实还有一个变种：

```javascript
function wait() {
  return new Promise(resolve =>
    setTimeout(resolve, 10 * 1000)
  )
}

async function main() {
  console.time();
  let a = wait();
  let b = wait();
  let c = wait();
  await a;
  await b;
  await c;
  console.timeEnd();
}
main();
```

这个的运行时间是10s多一点，这是因为：a，b，c的异步请求会按顺序发起。而这个过程是不需要互相依赖等待的。等到wait的时候，其实是比较那个异步耗时最多。就会等待最长。最长的耗时就是整体的耗时。

如果在业务中，两个异步没有依赖关系。应该是后面这种写法。