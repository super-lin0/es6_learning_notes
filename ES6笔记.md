---
[MENU]
typora-copy-images-to: imgs
---

# ES6标准入门

[TOC]

## 1、let 和const

-----

### 1.1、let命令

--------

- **基本用法**

```
{
  let a = 10;
  var b = 1;
 }

 console.log(a);	//ReferenceError
 console.log(b);	// 1
```



```
var a = [];
for(let i = 0; i < 10; i++) {
  a[i] = function() {
    console.log(i);		 
  }
}

a[6]();		//6
```

​	在上面代码中,如果使用var关键字，最后的结果输出10，原因是：var关键字声明为全局变量，因此全局只有一个变量i，而循环内部，被赋给数组a的函数内部的i指向的是全局的i 。即所有数组a的成员中的i指向的都是同一个。如果使用的是let，声明变量仅在快作用域内有效，因此最后输出6。

- **FAQ**

  Q：如果每一轮的循环变量的i都是重新声明的，那它怎么知道上一轮循环的值从而计算出本轮循环的值呢？

  A：JavaScript引擎内部会记住上一轮循环的值，初始化本地变量i时，就在上一轮循环的基础上进行计算

  for循环的特性：设置循环变量的那部分是一个副作用域，而循环体内部是一个单独的子作用域

  ```
  for (let i = 0; i < 3; i++) {
    let i = 'abc';
    console.log(i);
  }
  // abc
  // abc
  // abc
  ```

  正确运行以上代码将输出3次abc，这表示函数内部的变量i与循环变量i不在同一个作用域，而是有各自单独的作用域。

- **不存在变量提升**

  var命令会存在“变量提升”现象，即变量可以在声明之前使用，值为undefined。

  let命令改变了语法行为，它所声明的变量一定要在声明之后使用：

  ![变量提升](imgs/变量提升.jpg)

 - **暂时性死区**

   ES6明确规定，如果区块中存在let和const命令，则这个区块对这些命令声明的变量从一开始就形成封闭的作用域。

   只要在声明之前就使用这些变量，就会报错。



   ```
   if(true) {
     // TDZ开始
     tmp = 'abc';  // ReferenceError
     console.log(tmp);   // ReferenceError
   
     let tmp; // TDZ结束
     console.log(tmp); // undefined
   
     tmp = 123;
     console.log(tmp);   // 123
   }
   ```

   总之，在代码块内，只用let命令声明变量之前，该变量都是不可用的。这在语法上称为“暂时性死区”（TDZ）。

   **本质**

   ​	只要进入当前作用域，所要使用的变量就已经存在，但是不可获取，只有等到声明变量的那一行代码出现，

   才可以获取和使用该变量。

- **不允许重复声明**

  let不允许在相同的作用域内重复声明同一个变量。

--------

### 1.2、块级作用域

- **为什么需要**

  1、内层变量可能会覆盖外层变量

  ```
  var test = new Date();
  
  function f() {
    console.log(test);
    if (false) {
      var test = 'Hello World';
    }
  }
  
  f();  // undefined
  ```

  2、用来计数的循环变量泄漏为全局变量

  ```
  var s = "Hello World!";
  for(var i = 0;  i < s.length; i++) {
    console.log(s[i]);
  }
  
  console.log(i); // 12
  ```

- **ES6的块级作用域**

  let实际上为JavaScript新增了块级作用域。

  ```
  (function f1() {
    let c = 5;
    if(true) {
      let c = 10;
    }
    console.log(c);   // 5
  })();
  ```

  1、ES6允许块级作用域的任意嵌套

  2、外层作用域无法读取内层作用域的变量

  3、内层作用域可以定义外层作用域的同名变量

- **块级作用域与函数声明**

  ES6引入了块级作用域，明确允许在块级作用域中声明函数。ES6规定，在块级作用域之中，函数声明语句的行为

  类似于let，在块级作用域之外不可引用。

  ```
  function f() {
    console.log('I am outside');
  }
  
  (function () {
    if(false) {
      function f() {
        console.log('I am inside');
      }
    }
    f();
  }());
  ```

  ES5:  I am inside

  ES6:  I am outside(理论上)

  实际上ES6运行以上代码报错：TypeError: f is not a function

  原因：为了浏览器兼容问题，浏览器的实现可以不遵照上面的规定，可以有自己的行为方式，具体如下：

  1、允许在块级作用域内声明函数

  2、函数声明类似于var，即会提升到全局作用域或函数作用域的头部

  3、函数声明还会提升到所在块级作用域的头部。

  **Notes**

  1、块级作用域允许声明函数的规则只在使用大括号的情况下成立，

  2、考虑到环境导致的行为差异太大，应该避免在块级作用域内声明函数。如果确实需要，也应该写成函数表达式的形式，而不是函数声明语句。

  ```
  //  函数声明语句
  {
    var a = 'secret';
    function f() {
      return a;
    }
  }
  
  // 函数表达式
  {
    let a = 'secret';
    let f = function() {
      return a;
    }
  }
  ```

- **do表达式**

  ```
  let x = do {
    let t = f();
    t * t + 1;
  }
  ```

  变量x会得到整个块级作用域的返回值。（提案）

----------------------

### 1.3、const命令

- **基本用法**

  const声明一个只读的常量。一旦声明，常量的值就不能改变。

  1、const声明的常量不得改变值

  2、一旦声明常量，就必须立即初始化，不能留到以后赋值

  3、作用域与let命令相同：只在声明所在的块级作用域内有效（不会提升，同样存在暂时性死区，只能在声明后使用；不可重复声明）

  ```
  const PI = 3.1415;
  console.log(PI); // 3.1415
  ```

- **本质**

  const实际上保证的并不是变量的值不得改动，而是变量指向的那个内存地址不得改动。

  简单类型（数值、字符串、布尔值），值就保存在变量指向的内存地址中，因此等同于常量

  复合类型（对象和数组）：变量指向的内存地址保存的是一个指针，const只能保证这个指针是固定的，它指向的数据结构是可变的。

  ```
  const obj = {};
  
  obj.name = 'Jack';
  console.log(obj);   //{ name: 'Jack' }
  
  obj = [];   // TypeError: Assignment to constant variable.
  ```

  将对象本身以及对象的属性冻结的例子：

  ```
  var constantize = (obj) => {
    Object.freeze(obj);
    Object.keys(obj).forEach( (key, i) => {
      if ( typeof obj[key] === 'object' ) {
        constantize( obj[key] );
      }
    });
  };
  ```

- **ES6声明变量的6种方法**

  var、function、let、const、import、class

-------------------------------

### 1.4、顶层对象的属性

从ES6开始，全局变量将逐步与顶层对象的属性隔离

```
var a = 1;
console.log(window.a);  // 1

let b = 1;
console.log(window.b);  // undefined
```

**Notes**

​	ES6一方面规定，为了保持兼容性，var命令和function命令声明的全局变量依旧是顶层对象的属性；另一方面规定，let、const、class命令声明的全局变量不属于顶层对象的属性。

----------------------

## 2、变量的解构赋值

----------------------------

### 2.1、数组的解构赋值

- **基本用法**

解构赋值：按照一定模式从数组和对象中提取值，然后对变量进行赋值

```
let [a, b, c] = [1, 2, 3];
```

```
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```

不完全解构的例子：

```
let [x, y] = [1, 2, 3];
console.log(x);   // 1
console.log(y);   // 2

let [a, [b], d] = [1, [2, 3], 4];
console.log(a);   // 1
console.log(b);   // 2
console.log(d);   // 4
```

**Notes**

只要某种数据结构具有Iterator接口，都可以采用解构赋值

```
function* fibs() {
  let a = 0;
  let b = 1;
  while(true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

let [first, second, third, fourth, fifth, sixth] = fibs();
console.log(sixth);   // 5
```

Generator函数具有原生的Iterator接口。

- **默认值**

  解构赋值允许指定默认值

  ```
  let [foo = true] = [];
  console.log(foo);   // true
  
  let [x, y = 'b'] = ['a'];
  console.log(x, y);    // a b
  
  let [e, f = 'b'] = ['a', undefined];
  console.log(e, f);    // a b
  
  let [g = 1] = [null];
  console.log(g);   // null
  
  function baz() {
    console.log('hahah');
  }
  
  let [h = baz()] = [1];
  console.log(h);   // 1
  
  let [j = 1, k = j] = [];
  console.log(j, k);  // 1 1
  
  let [m = n, n = 1] = [];
  console.log(m, n);  // ReferenceError: n is not defined
  ```

  **Notes**

  1、ES6内部使用严格相等运算符（===）判断一个位置是否有值，所以，如果一个数组成员不严格等于undefined,默认值是不会生效的。

  2、如果默认值是一个表达式，那么这个表达式是惰性求值的，即只有在用到是才会求值。

  3、默认值可以引用解构赋值的其他变量，但该变量必须已经声明。

----------

### 2.2、对象的解构赋值

数组元素是按照次序排列的，变量的取值由它的位置决定的；而对象的属性没有次序，变量必须与属性同名才能取到正确的值。

```
let { foo, bar } = { bar: 'bbb', foo: 'aaa' };
console.log(foo, bar);    // aaa bbb

let { baz } = { foo: 'aaa', bar: 'bbb'};
console.log(baz);   // undefined

//变量名与属性名不一样的情况
let { foo: baz } = { foo: 'aaa', bar: 'bbb'};
console.log(baz);   // aaa

const obj = { first: 'Hello', last: 'world' };
let { first: f, last: l } = obj;
console.log(f, l);    // Hello world
```

可以看出，对象的解构赋值的内部机制是先找到同名的属性，然后再赋值给对应的变量。真正被赋值的是后者，而不是前者。

嵌套结构对象的解构赋值

```
const obj1 = {
  p: [
    'Hello',
    {y: 'World'}
  ]
};
let {p: [x, {y}]} = obj1;
console.log(x, y);    // Hello World

let {p, p: [a, {y: b}]} = obj1;
console.log(p, a, b); // [ 'Hello', { y: 'World' } ] 'Hello' 'World'

// 这个例子稍微复杂一点，但也不难理解
const node = {
  loc: {
    start: {
      line: 1,
      column: 3
    }
  }
}
const {loc, loc: {start}, loc: {start: {line}}} = node;
console.log(loc, start, line);    // { start: { line: 1, column: 3 } } { line: 1, column: 3 } 1
```

嵌套赋值的例子：

```
let obj2 = {};
let arr = [];

({ foo: obj2.prop, bar: arr[0] } = { foo: 123, bar: true });
console.log(obj2, arr);   // { prop: 123 } [ true ]
```

对象解构也支持默认值：

```
var {g = 3} = {};
console.log(g) // 3

var {h, i = 5} = {h: 1};
console.log(h) // 1
console.log(i) // 5

var {j: k = 3} = {};
console.log(k) // 3

var {l: m = 3} = {l: 5};
console.log(m) // 5

var { message: msg = 'Something went wrong' } = {};
console.log(msg) // "Something went wrong"
```

***Notes***

默认值生效的条件是，对象属性值严格等于undefined

```
const {n = 3} = {n: undefined};
console.log(n);   // 3
const {o = 2} = {o: null};
console.log(o)    // null
```

--------------------

### 2.3、字符串的解构赋值

```
const {log} = console;
const [a, b, c, d, e] = 'Hello';
log(a, b, c, d, e);   // H e l l o

const { length: len } = 'Hello';
log(len);   // 5
```

----------------------------------------

### 2.4、数值和布尔值的解构赋值

解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。

```
let {toString: s} = 123;
console.log(s === Number.prototype.toString);   // true

let {toString: b} = true;
console.log(b === Boolean.prototype.toString);  // true
```

***Notes***

解构赋值的规则是，只要等号右边的值不是对象或数组，就先将其转换为对象。（undefined和null无法转换为对象，直接报错）。

```
let {prop: a} = undefined;
console.log(a);   // TypeError: Cannot destructure property `prop` of 'undefined' or 'null'.

let {prop: b} = null;
console.log(b);   // TypeError: Cannot destructure property `prop` of 'undefined' or 'null'.
```

--------------------------------------

### 2.5、函数参数的解构赋值

```
const {log: print} = console;

function add([x, y]) {
  return x + y;
}

print(add([1, 2]));   // 3

print([[1,2], [3, 4]].map(([x, y]) => x + y));    // [ 3, 7 ]

//默认参数的例子
function move({x = 0, y = 0} = {}) {
  return x + y;
};

print(move({x: 1, y: 3}));    // 4
print(move({x: 1}))     // 1
print(move({}))     // 0
print(move())     // 0

[1, undefined, 3].map((x = 'yes') => x);  // [ 1, 'yes', 3 ]
```

-----------------------------

### 2.6、圆括号的问题

只要有可能导致解构的歧义，就不得使用圆括号。

- **不得使用圆括号的情况**

  1、变量声明语句

  2、函数参数

  3、赋值语句的模式

  案例参考如下：

  ```
  // 1、变量声明语句，全部报错
  let [(a)] = [1];
  
  let {x: (c)} = {};
  let ({x: c}) = {};
  let {(x: c)} = {};
  let {(x): c} = {};
  
  let { o: ({ p: p }) } = { o: { p: 2 } };
  
  // 2、函数参数（报错）
  function f([(z)]) {return z;}
  function f([z,(x)]) { return x; }
  
  3、赋值语句模式
  ({ p: a }) = { p: 42 };
  ([a]) = [5];
  [({ p: a }), { x: c }] = [{}, {}];
  ```

- ***可以使用圆括号的情况***

  只有一种：赋值语句的非模式部分可以使用圆括号

  ```
  [(b)] = [3]; // 正确
  ({ p: (d) } = {}); // 正确
  [(parseInt.prop)] = [3]; // 正确
  ```

  ### 2.7、用途

  - **交换变量的值**

    ```
    let x = 1;
    let y = 2;
    
    [x, y] = [y, x];
    console.log(x, y);  // 2 1
    ```

  - **从函数返回多个值**

    ```
    function example() {
      return [1, 2, 3];
    }
    let [a, b, c] = example();
    console.log(a, b, c);   // 1 2 3
    
    // 返回一个对象
    function example1() {
      return {
        foo: 1,
        bar: 2
      };
    }
    let { foo, bar } = example1();
    console.log(foo, bar);  // 1 2
    ```

  - **函数参数的定义**

    ```
    // 参数是一组有次序的值
    function f([x, y, z]) { ... }
    f([1, 2, 3]);
    
    // 参数是一组无次序的值
    function f({x, y, z}) { ... }
    f({z: 3, y: 2, x: 1});
    ```

  - **提取JSON数据**

    ```
    let jsonData = {
      id: 42,
      status: "OK",
      data: [867, 5309]
    };
    
    let { id, status, data: number } = jsonData;
    
    console.log(id, status, number);
    // 42, "OK", [867, 5309]
    ```

  - **函数参数的默认值**

  - **遍历Map结构**

    ```
    const map = new Map();
    map.set('first', 'hello');
    map.set('second', 'world');
    
    for (let [key, value] of map) {
      console.log(key + " is " + value);
    }
    // first is hello
    // second is world
    ```

  - **输入模块的指定方法**

    ```
    import React, {Component} from 'react';
    ```

-----------------------------------

## 3、字符串的扩展

### 3.1、字符串的遍历接口

ES6 为字符串添加了遍历器接口，使得字符串可以被for...of循环遍历

```
const s = 'foo';
for(let a of s) {
  console.log(a);
}
```

--------------------------

### 3.2、includes(), startsWith(), endsWith()

用来确定一个字符串是否包含在另一个字符串中：

- indexOf
- includes()  返回布尔值，表示是否找到了参数字符串
- startsWith()  返回布尔值，表示参数字符串是否在源字符串头部
- endsWith()  返回布尔值，表示参数字符串是否在源字符串的尾部

```
const str = 'Hello Worls';

console.log(str.startsWith('Hello'));   // true
console.log(str.startsWith('hello'));   // false
console.log(str.endsWith('ls'));    // true
console.log(str.includes('o W'));   // true

console.log(str.startsWith('Hello', 1));   // false
console.log(str.endsWith('He', 2));    // true	(针对前n个字符)
console.log(str.includes('o W', 5));   // false
```

***Notes***

第二个参数表示开始搜索的位置

-------------------

### 3.3、repeat()

repeat()方法返回一个新字符串，表示将原字符串重复n次

```
const {log: print} = console;

print('*'.repeat(2));   // **
print('hello'.repeat(2));   // hellohello
print('na'.repeat(0));  // ''
print('na'.repeat(2.9));  // nana
print('na'.repeat(-0.9));  // ''(0到-1之间取0)
print('na'.repeat(NaN));  // '' 等同于0
print('na'.repeat('na')); // ''
print('na'.repeat('3')); // nanana
print('na'.repeat(Infinity));   // RangeError: Invalid count value
print('na'.repeat(-1));   // RangeError: Invalid count value
```

-------------------------------

### 3.4 、padStart()、padEnd()

ES2017引入了字符串补齐全长度的功能：如果某个字符串不够指定长度，会在头部或尾部补全。

- padStart() 用于头部补全
- padEnd()哟关于尾部补全

```
print('x'.padStart(5, 'ab'));   // ababx
print('x'.padStart(4, 'ab'));   // abax
print('xxxxx'.padStart(4, 'ab'));   // xxxxx
print('xx'.padStart(4, 'abc'));   // abxx
print('x'.padEnd(4, 'ab'));   // xaba
print('x'.padEnd(5, 'ab'));   // xabab
print('xx'.padEnd(4, 'abc'));   // xxab
```

***Notes***

padStart常见用途：1、为数值补全指定位数；2、提示字符串格式

```
print('1'.padStart(10, 0));   // 0000000001
print('12'.padStart(10, 0));  // 0000000012
print('123456'.padStart(10, 0));  // 0000123456

print('12'.padStart(10, 'YYYY-MM-DD'));   // YYYY-MM-12
print('09-12'.padStart(10, 'YYYY-MM-DD'));  // YYYY-09-12
```

--------------------------------

### 3.5、模板字符串

​	模板字符串（template string）是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量。

```
// 普通字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
 not legal.`

console.log(`string text line 1
string text line 2`);

// 字符串中嵌入变量
let name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`

let greeting = `\`Yo\` World!`;

// trim方法会消除字符串中的空格
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`.trim());
```

---------------------

## 4、正则表达式

-------------------------------

### 4.1、RegExp构造函数

ES5中，RegExp构造函数的参数有两种情况:

- 参数是字符串，这时第二个参数表示正则表达式的修饰符(flag);

  ```
  var regex = new RegExp('xyz', 'i');
  // 等同于
  var regex = /xyz/i;
  var str = 'xyz';
  
  console.log(regex.test(str));	// true
  ```

- 参数是一个正则表达式，这时会返回一个原有正则表达式的拷贝

  ```
  var regx1 = new RegExp(/xyz/i);
  // 等同于
  var regx1 = /xyz/i;
  console.log(regx1.test(str));   // true
  ```

  **Notes**

  ES5不允许此时使用第二个参数添加修饰符，否则会报错。

  ```
  var regex = new RegExp(/xyz/, 'i');
  // Uncaught TypeError: Cannot supply flags when constructing one RegExp from another
  ```

  ES6:如果RegExp构造函数第一个参数是一个正则表达式对象，那么可以使用第二个参数指定修饰符。而且，返回的正则表达式会忽略原有的正则表达式的修饰符，只使用新指定的修饰符。

  ```
  var regex1 = new RegExp(/xyz/ig, 'i');
  console.log(regex1.flags);    // i
  ```

-------------------

### 4.2、字符串的正则方法

字符串对象共有4个方法可以使用正则表达式：matchs()、replace()、search()、split()。ES6使这4个方法在语言内部全部调用RegExp的实例方法，从而做奥所有与正则相关的方法都定义在RegExp对象上。

- `String.prototype.match` 调用 `RegExp.prototype[Symbol.match]`
- `String.prototype.replace` 调用 `RegExp.prototype[Symbol.replace]`
- `String.prototype.search` 调用 `RegExp.prototype[Symbol.search]`
- `String.prototype.split` 调用 `RegExp.prototype[Symbol.split]`

----------------------------------

### 4.3、y修饰符

ES6为正则表达式添加了y修饰符，叫做“粘连”（sticky）修饰符。作用与g修饰符类似，也是全局匹配，后一次匹配都从上一个匹配成功的下一个位置开始。不同之处在于，g修饰符只要剩余未之中存在匹配就行，而y修饰符会确保匹配必须从剩余的第一个位置开始，这也就是“粘连”的含义。

```
var s = 'aaa_aa_a';
var r1 = /a+/g;
var r2 = /a+/y;
var r3 = /a+_/y;

console.log(r1.exec(s));    // [ 'aaa', index: 0, input: 'aaa_aa_a', groups: undefined ]
console.log(r1.exec(s));    // [ 'aa', index: 4, input: 'aaa_aa_a', groups: undefined ]
console.log(r1.exec(s));    // [ 'a', index: 7, input: 'aaa_aa_a', groups: undefined ]

console.log(r2.exec(s));    // [ 'aaa', index: 0, input: 'aaa_aa_a', groups: undefined ]
console.log(r2.exec(s));    // null
	
console.log(r3.exec(s));    // [ 'aaa_', index: 0, input: 'aaa_aa_a', groups: undefined ]
console.log(r3.exec(s));    // [ 'aa_', index: 4, input: 'aaa_aa_a', groups: undefined ]
```

使用lastIndex属性来说明y修饰符

```
const REGEX = /a/g;
const strs = 'xaya';

REGEX.lastIndex = 2;    // 指定从2号位置（y）开始匹配
const match = REGEX.exec(strs);   // 匹配成功

console.log(match.index);   // 3(在3号位置匹配成功)
console.log(REGEX.lastIndex);   // 4 下一次匹配从4号位开始
console.log(REGEX.exec(strs));  // null

const REGEX1 = /a/y;

REGEX1.lastIndex = 2;   // 指定从2号位置(y)开始匹配
console.log(REGEX1.exec(strs));   // null 不是粘连，匹配失败

REGEX1.lastIndex = 3;
const match2 = REGEX1.exec(strs);

console.log(match2.index);   // 3 (在3号位置匹配成功)
console.log(REGEX1.lastIndex);   // 4 下一次匹配从4号位开始
```

**Notes**

y修饰符的设计本意就是让头部匹配的标志(^)在全局匹配中都有效

在split方法中使用y修饰符，原字符串必须以分隔符开头。这也就意味着，只要匹配成功，数组的第一个成员肯定是空字符串。后续的分隔符只有紧跟前面的分隔符才会被识别。

```
console.log('x##'.split(/#/y));   // [ 'x', '', '' ]
console.log('##x'.split(/#/y));   // [ '', '', 'x' ]
console.log('#x#'.split(/#/y));   // [ '', 'x', '' ]
console.log('##'.split(/#/y));   // [ '', '', '' ]
console.log('aaxa'.replace(/a/gy, '-'));  // --xa
// 单独一个y修饰符对match方法只能返回第一个匹配
console.log('a1a2a3'.match(/a\d/y));    // [ 'a1', index: 0, input: 'a1a2a3', groups: undefined ]
console.log('a1a2a3'.match(/a\d/gy));    // [ 'a1', 'a2', 'a3' ]
```

-----------------

### 4.4、sticky属性

```
var r = /hello\d/y;
console.log(r.sticky);    // true(表示是否设置了y修饰符)
```

-------------------

### 4.5、flags属性

```
console.log(/abc/ig.flags);     // gi(ES6的flags属性，返回正则表达式的修饰符)
console.log(/abc/ig.source);    // abc(ES5的source属性，返回正则表达式的修饰符)
```

------------------

### 4.6、s修饰符

ES2018引入/s修饰符，使得.可以匹配任意单个字符。这被称为dotAll模式，即点代表一切字符。

```
const re = /foo.bar/s;
// const re = new RegExp('foo.bar', 's');

console.log(re.test('foo\nbar'));   // true
console.log(re.dotAll);   // true
console.log(re.flags);    // s
```

------------------------

### 4.7、具名组匹配

- ***简介***

正则表达式使用圆括号进行组匹配。

```
const RE_DATE = /(\d{4})-(\d{2})-(\d{2})/;
const matchDate = RE_DATE.exec('2019-01-01');

console.log(matchDate);     
// [ '2019-01-01', '2019', '01', '01', index: 0, input: '2019-01-01', groups: undefined ]
console.log(matchDate[1]);
console.log(matchDate[2]);
console.log(matchDate[3]);
```

组匹配的问题：每一组匹配的含义不容易看出来，而且只能用数字序号引用。

具名组匹配：允许为每一个组匹配指定一个名字。

```
const RE_DATE = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const matchDate = RE_DATE.exec('2019-01-01');
const matchObj = matchDate.groups || {}

console.log(matchDate[1])    // 2019
console.log(matchObj.year);  // 2019
console.log(matchObj.month); // 01
console.log(matchObj.day);   // 01
```



- ***解构赋值和替换***

  ```
  let {groups: {one, two}} = /^(?<one>.*):(?<two>.*)$/u.exec('foo:bar');
  
  console.log(one);   // foo
  console.log(two);   // bar
  
  // 字符串替换时，使用$<组名>来引用具组名
  let re1 = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/u;
  console.log('2015-01-02'.replace(re1, '$<day>/$<month>/$<year>'));  // 02/01/2015
  
  const re_date = '2015-01-02'.replace(re1, (
    matched, // 整个匹配结果 2015-01-02
    capture1, // 第一个组匹配 2015
    capture2, // 第二个组匹配 01
    capture3, // 第三个组匹配 02
    position, // 匹配开始的位置 0
    S, // 原字符串 2015-01-02
    groups // 具名组构成的一个对象 {year, month, day}
  ) => {
  	let {day, month, year} = groups;
  	return `${day}/${month}/${year}`;
  });
  
  console.log(re_date);   // 02/01/2015
  ```

- **引用**

  如果正则表达式内部引用某个“具名组匹配”，可以使用``\k<组名>``、``\1`` 的写法

  ```
  const RE_TWICE = /^(?<word>[a-z]+)!\k<word>$/;
  
  console.log(RE_TWICE.test('abc!abc'));  // true
  console.log(RE_TWICE.test('abc!abe'));  // false
  
  const RE_TWICE = /^(?<word>[a-z]+)!\1$/;
  
  console.log(RE_TWICE.test('abc!abc'));  // true
  console.log(RE_TWICE.test('abc!abe'));  // false
  
  const RE_TWICE = /^(?<word>[a-z]+)!\k<word>!\1$/;
  
  console.log(RE_TWICE.test('abc!abc!abc'));  // true
  console.log(RE_TWICE.test('abc!abc!abe'));  // false
  ```

------------------------------------

## 5、数值的扩展

------------------

### 5.1、二进制和八进制表示法

ES6提供了二进制和八进制数值的新的写法，分别用前缀``0b``或``0B``和``0o``或``0O``表示。

```
console.log(0b110011);    // 51
console.log(0o767);     // 503

// 将二进制和八进制字符串数值转换为十进制数值
console.log(Number('0b111'));   // 7
console.log(Number('0o10'));    // 8
```

-------------------------

### 5.2、Number.isFinite()、Number.isNaN()

- Number.isFinite()用来检查一个数值是否为有限的(finite)。
- Number.isNaN()用来检查一个值是否为NaN。

```
console.log(Number.isFinite(15)); // true
console.log(Number.isFinite(0.8)); // true
console.log(Number.isFinite(NaN)); // false
console.log(Number.isFinite(Infinity)); // false
console.log(Number.isFinite(-Infinity)); // false
console.log(Number.isFinite('foo')); // false
console.log(Number.isFinite('15')); // false
console.log(Number.isFinite(true)); // false

console.log(Number.isNaN(NaN)); // true
console.log(Number.isNaN(15)); // false
console.log(Number.isNaN('15')); // false
console.log(Number.isNaN(true)); // false
console.log(Number.isNaN(9/NaN)); // true
console.log(Number.isNaN('true' / 0)); // true
console.log(Number.isNaN('true' / 'true')); // true
```

----------------------------

### 5.3、Number.parseInt()、Number.parseFloat()

ES6 将全局方法`parseInt()`和`parseFloat()`，移植到`Number`对象上面，行为完全保持不变。

```
// ES5的写法
parseInt('12.34') // 12
parseFloat('123.45#') // 123.45

// ES6的写法
Number.parseInt('12.34') // 12
Number.parseFloat('123.45#') // 123.45

Number.parseInt === parseInt // true
Number.parseFloat === parseFloat // true
```

***Notes***

这样做的目的是，逐步减少全局方法，使得语言逐步模块化。

-----------------------

### 5.4、Number.isInteger()

Number.isInteger()用来判断一个值是否为整数。（注意：在JavaScript内部，整数和浮点数是同样的存储方式，因此3和3.0被视为同一个值）。

```
console.log(Number.isInteger(3));   // true
console.log(Number.isInteger(3.0)); // true
console.log(Number.isInteger('3')); // false
console.log(Number.isInteger(25.1));  // false
console.log(Number.isInteger());    // false
console.log(Number.isInteger(null));  // false
console.log(Number.isInteger(true));  // false
// 小数的精度达到了小数点后16位，转换成二进制超过了53个二进制位，导致最后的2丢失
console.log(Number.isInteger(3.0000000000000002)); // true

// 如果一个数值的绝对值小于Number.MIN_VALUE（5E-324），即小于 JavaScript 能够分辨的最小值，会被自动转为 0
console.log(Number.isInteger(5E-324)); // false
console.log(Number.isInteger(5E-325)); // true
```

**Notes**

鉴于以上两种情况，如果对数据精度要求较高，不建议使用``Number.isInteger()``来判断一个数是否是整数。

---------------

### 5.5、Number.EPSILON

ES6在Number对象上面新增一个极小的常量``Number.EPSILON``，目的在于为浮点数计算设置一个误差范围，我们知道浮点数计算是不精确的。但是如果这个误差能够小于``Number.EPSILON``，我们就可以认为得到了正确结果。因此，``Number.EPSILON``的实质是一个可接受的误差范围。

```
console.log(Number.EPSILON);    // 2.220446049250313e-16
console.log(0.1 + 0.2);   // 0.30000000000000004
console.log(0.1 + 0.2 - 0.3);   // 5.551115123125783e-17
console.log(5.551115123125783e-17 < Number.EPSILON);    // true

function withErrorMargin (left, right) {
  return Math.abs(left - right) < Number.EPSILON;
}

console.log(withErrorMargin(0.1 + 0.2, 0.3));   // true
console.log(withErrorMargin(0.2 + 0.2, 0.3));   // false
```

---------------------

### 5.6、安全整数和``Number.isSafeInteger()``

JavaScript能够准确表示的整数范围在``-2^52``到``2^53``之间（不含两个端点），超出这个范围就无法精确表示。

```
console.log(Math.pow(2, 53));   // 9007199254740992
console.log(9007199254740992);  // 9007199254740992
console.log(9007199254740993);  // 9007199254740992
console.log(Math.pow(2, 53) === Math.pow(2, 53) + 1);   // 9007199254740992
```

ES6引入了``Number.MIN_SAFE_INTEGER``和``Number.MAX_SAFE_INTEGER``两个常量用来表示这个范围的上下极限。

```
console.log(Number.MAX_SAFE_INTEGER === Math.pow(2, 53) - 1);   // true
console.log(Number.MAX_SAFE_INTEGER === -Number.MIN_SAFE_INTEGER);  // true
console.log(Number.MAX_SAFE_INTEGER === 9007199254740991);  // true
```

``Number.isSafeInteger()``用来判断一个整数是否落在这个范围之内。

```
console.log(Number.isSafeInteger('a')); // false
console.log(Number.isSafeInteger(null)); // false
console.log(Number.isSafeInteger(NaN)); // false
console.log(Number.isSafeInteger(Infinity)); // false
console.log(Number.isSafeInteger(-Infinity)); // false

console.log(Number.isSafeInteger(3)); // true
console.log(Number.isSafeInteger(1.2)); // false
console.log(Number.isSafeInteger(9007199254740990)); // true
console.log(Number.isSafeInteger(9007199254740992)); // false

console.log(Number.isSafeInteger(Number.MIN_SAFE_INTEGER - 1)); // false
console.log(Number.isSafeInteger(Number.MIN_SAFE_INTEGER)); // true
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER)); // true
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1)); // false
```

**Notes**

实际使用这个函数时，需要注意验证运算结果是否安全落在安全整数的范围内，另外不只验证运算结果，还要同时验证参与运算的每个值。

----------------------------

### 5.7、Math对象的扩展

- ***``Math.trunc()``***

  ``Math.trunc``方法用于去除一个数的小数部分，（内部使用Number方法将其先转换为数值）。

  ```
  Math.trunc(4.1); // 4
  Math.trunc(4.9); // 4
  Math.trunc(-4.1); // -4
  Math.trunc(-4.9); // -4
  Math.trunc(-0.1234); // -0
  
  Math.trunc('123.456'); // 123
  Math.trunc(true); //1
  Math.trunc(false); // 0
  Math.trunc(null); // 0
  
  Math.trunc(NaN);      // NaN
  Math.trunc('foo');    // NaN
  Math.trunc();         // NaN
  Math.trunc(undefined) // NaN
  ```

- ***``Math.sign()``***

  ``Math.sign``方法用来判断一个数到底是正数、负数，还是零（对于非数值，会将其先转换为数值）。

  1、参数为整数，返回``+1``

  2、参数为负数，返回``-1``

  3、参数为``0``，返回``0``

  4、参数为``-0``，返回``-0``

  5、其他值，返回``NaN``

- **``Math.cbrt()``**

  ``Math.cbrt``方法用于计算一个数的立方根（对于非数值，``Math.cbrt``方法内部先使用``Number``方法转换为数值）。

  ```
  Math.cbrt(-1) // -1
  Math.cbrt(0)  // 0
  Math.cbrt(1)  // 1
  Math.cbrt(2)  // 1.2599210498948734
  
  Math.cbrt('8') // 2
  Math.cbrt('hello') // NaN
  ```

--------------

### 5.8、指数运算符

ES2016新增了一个指数运算符(**)。

```
console.log(2 ** 3);  // 8
console.log(2 ** 2);  // 4
let a = 2;
console.log(a **= 5); // 32
```

--------------------------

## 6、函数的扩展

-------------------------

### 6.1、函数参数的默认值

- ***基本用法***

  ```
  function log(x, y = 'World') {
    console.log(x, y);
  }
  
  log('Hello') // Hello World
  log('Hello', 'China') // Hello China
  log('Hello', '') // Hello
  
  function Point(x = 0, y = 0) {
    this.x = x;
    this.y = y;
  }
  
  const p = new Point();
  p // { x: 0, y: 0 }
  ```

  **优点**

  1、简洁

  2、阅读代码的人可以立刻意识到哪些参数是可以省略的，不用查看函数体或文档

  3、有利于将来代码优化，即使未来版本彻底去掉这个参数，也不会导致以前的代码无法运行

  **限制**

  1、参数变量是默认声明的，所以不能用let或const再次声明。

  2、使用参数默认值时，函数不能有同名参数。

  ```
  function foo(x = 5) {
    let x = 2;    // SyntaxError: Identifier 'x' has already been declared
    const x = 2;
  }
  
  function foo(x, x = 5, y = 1) {  
  	// SyntaxError: Duplicate parameter name not allowed in this context
  	// ...
  }
  ```

  ***Notes***

  参数默认值不是传值的，而是每次都重新计算默认值表达式的值（惰性求值）。

  ```
  let x = 99;
  function foo(p = x + 1) {
    console.log(p);
  }
  
  foo() // 100
  
  x = 100;
  foo() // 101
  ```

  上面代码中，参数p的默认值是``x + 1``。这时，每次调用函数foo都会重新计算``x + 1``,而不是默认``p``等于100。

- ***与解构赋值默认值结合使用***

```
function foo({x, y = 5}) {
  console.log(x, y);
}

foo({}) // undefined 5
foo({x: 1}) // 1 5
foo({x: 1, y: 2}) // 1 2
foo() // TypeError: Cannot read property 'x' of undefined
```

- ***参数默认值的位置***

  通常情况下，定义了默认值的参数应该是函数的尾参数。非尾部的参数设置默认值，实际上这个参数是无法省略的。

```
// 例一
function f(x = 1, y) {
  return [x, y];
}

f() // [1, undefined]
f(2) // [2, undefined])
f(, 1) // 报错
f(undefined, 1) // [1, 1]

// 例二
function f(x, y = 5, z) {
  return [x, y, z];
}

f() // [undefined, 5, undefined]
f(1) // [1, 5, undefined]
f(1, ,2) // 报错
f(1, undefined, 2) // [1, 5, 2]

function foo(x = 5, y = 6) {
  console.log(x, y);
}

foo(undefined, null) // 5 null
```

- ***函数的``length``属性***

  指定了默认值以后，函数的length属性将返回没有指定默认值的参数个数。

```
(function (a) {}).length // 1
(function (a = 5) {}).length // 0
(function (a, b, c = 5) {}).length // 2

(function(...args) {}).length // 0

// 设置的默认值的参数不是尾参数，那么length属性也不再计入后面的参数
(function (a = 0, b, c) {}).length // 0
(function (a, b = 1, c) {}).length // 1
```

- ***作用域***

  一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域。等到初始化结束，这个作用域就会消失。这种语法行为在不设置参数默认值时是不会出现的。

```
var x = 1;

function f(x, y = x) {
  console.log(y);
}

f(2) // 2

let x = 1;

function f(y = x) {
  let x = 2;
  console.log(y);
}

f() // 1

let foo = 'outer';

function bar(func = () => foo) {
  let foo = 'inner';
  console.log(func());
}

bar(); // outer
```

- ***应用***

  利用参数默认值，可以指定某一个参数不得省略，如果省略就抛出一个错误。

  ```
  function throwIfMissing() {
    throw new Error('Missing parameter');
  }
  
  function foo(mustBeProvided = throwIfMissing()) {
    return mustBeProvided;
  }
  
  foo() // Error: Missing parameter
  ```

------------------

### 6.2、``rest``参数

ES6引入了``rest``参数（形式为“...变量名”）,这样就不需要``arguments``对象了。``rest``参数搭配的变量是一个数组，该变量将多余的参数放入其中。

```
function add(...values) {
  let sum = 0;

  for (var val of values) {
    sum += val;
  }

  return sum;
}

add(2, 5, 3) // 10
```

***``Notes``***

1、``rest``参数之后不能再有其他参数，否则会报错。

2、函数的``length``属性不包括``rest``参数。

----------------------------------

### 6.3、严格模式

ES6规定：只要函数参数使用了默认值、解构赋值或者扩展运算符，那么函数内部就不能显示设定为严格模式。（原因：函数内部的严格模式同样适用于函数体和函数参数）

两种方法规避：

1、设定全局性的严格模式。

2、把函数包在一个无参数的的立即执行函数中。

```
'use strict';

function doSomething(a, b = a) {
  // code
}

const doSomething = (function () {
  'use strict';
  return function(value = 42) {
    return value;
  };
}());
```

--------------------------

### 6.4、``name``属性

函数的``name``属性，返回该函数的函数名。

```
function foo() {}
foo.name // "foo"

var f = function () {};

// ES5
f.name // ""
// ES6
f.name // "f"

const bar = function baz() {};

// ES5
bar.name // "baz"
// ES6
bar.name // "baz"

function foo() {};
foo.bind({}).name // "bound foo"

(function(){}).bind({}).name // "bound "
```

------------------------

### 6.5、箭头函数

- **基本用法**

  ```
  var f = v => v;
  var f = () => 5;
  var sum = (num1, num2) => num1 + num2;
  var sum = (num1, num2) => { return num1 + num2; }
  let getTempItem = id => ({ id: id, name: "Temp" });
  
  let fn = () => void doesNotReturn();
  const full = ({ first, last }) => first + ' ' + last;
  
  const isEven = n => n % 2 === 0;
  const square = n => n * n;
  
  [1,2,3].map(x => x * x);
  var result = values.sort((a, b) => a - b);
  
  const numbers = (...nums) => nums;
  
  numbers(1, 2, 3, 4, 5)
  // [1,2,3,4,5]
  
  const headAndTail = (head, ...tail) => [head, tail];
  
  headAndTail(1, 2, 3, 4, 5)
  // [1,[2,3,4,5]]
  ```

- ***注意点***

  - 函数体内的``this``对象，就是定义时所在的对象，而不是运行时所在的对象。

  - 不可以当作构造函数，也就是说，不可以使用``new``命令，否则会抛错。
  - 不可以使用``arguments``对象，该对象在函数体内不存在，可以使用``rest``代替。
  - 不可以使用``yeild``命令，因此箭头函数不能作用于``Genertor``函数。

  ```
  function foo() {
    setTimeout(() => {
      console.log('id:', this.id);
    }, 100);
  }
  
  var id = 21;
  
  foo.call({ id: 42 }); // id: 42
  ```

- ***嵌套的箭头函数***

  箭头函数内部还可以再使用箭头函数。

  ```
  const plus1 = a => a + 1;
  const mult2 = a => a * 2;
  
  mult2(plus1(5))
  // 12
  ```

---------------------

### 6.6、绑定``this``

箭头函数可以绑定``this``对象，大大减少了现实绑定``this``对象的写法(``call``、``apply``、``bind``)。但是，箭头函数并非适用于所有的场合，ES7提出了“函数绑定”运算符，用来取代``call``、``apply``、``bind``调用。``Babel``转码器已经支持。

函数绑定运算符(::)，双冒号左边是一个对象，右边是一个函数。该运算符会自动将左边的对下岗作为上下文环境（``this``）绑定到右边的函数上。

```
foo::bar;
// 等同于
bar.bind(foo);

foo::bar(...arguments);
// 等同于
bar.apply(foo, arguments);
```

----------------------------

### 6.7、尾调用优化

- ***尾调用***：某个函数的最后一步是调用另一个函数。

```
function f(x){
  return g(x);
}

// 以下三种情况都不属于尾调用
// 情况一
function f(x){
  let y = g(x);
  return y;
}

// 情况二
function f(x){
  return g(x) + 1;
}

// 情况三（等同于 g(x); return undefined）
function f(x){
  g(x);
}

// 尾调用不一定出现在函数尾部，只要是最后一步操作即可
function f(x) {
  if (x > 0) {
    return m(x)
  }
  return n(x);
}
```

----

- ***尾调用优化***

  ***调用帧***：函数调用会在内存中形成一个“调用记录”，又称“调用帧”，保存调用位置和内部变量等信息。

  ***调用栈*** : 如果在函数A的内部调用函数B，那么在A的调用帧上方还会形成一个B的调用帧。等到B运行结束，将结果返回到A，B的调用帧才会消失。如果在函数B的内部还有一个函数C，那么就还有一个C的调用帧，以此类推。所有的调用帧就形成一个“调用栈”。

  尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用帧，因为调用位置、内部变量等信息都不会再用到了，直接用内层函数的调用帧取代外层函数即可。

```
function f() {
  let m = 1;
  let n = 2;
  return g(m + n);
}
f();

// 等同于
function f() {
  return g(3);
}
f();

// 等同于
g(3);
```

**尾调用优化** ： 只保留内层函数的调用帧。

***意义*** : 如果所有函数都是尾调用，那么完全可以做到每次执行时调用帧只有一项，这将大大节省内存。

**注意** ：

只有不再用到外层函数的内部变量，内层函数的调用帧才会取代外层函数的调用帧，否则无法进行“尾调用优化”。

```
// 不会进行尾调用优化，因为内层函数inner用到了外层函数的one 变量
function addOne(a){
  var one = 1;
  function inner(b){
    return b + one;
  }
  return inner(a);
}
```

----

- **尾递归**

函数调用自身称为递归。如果尾调用自身就称为尾递归。

递归非常耗内存，因为需要同时保存成千上万个调用帧，很容易发生“栈溢出”错误。但对于尾递归来说，由于只存在一个调用帧，所以永远不会发生“栈溢出”错误。

```
function factorial(n) {
  if (n === 1) return 1;
  return n * factorial(n - 1);
};

console.log(factorial(5));  // 120(O(n))

function factorial1(n, total) {
  if (n === 1) return total;
  return factorial1(n - 1, n * total);
}

console.log(factorial1(5, 1));  // 120(O(1))
```

```
function Fibonacci(n) {
  if ( n <= 1) return 1;
  return Fibonacci(n -1) + Fibonacci(n - 2);
}

console.log(Fibonacci(10));   // 89

function Fibonacci1(n, ac1 = 1, ac2 = 1) {
  if (n <= 1) return ac2;
  return Fibonacci1(n - 1, ac2, ac1 + ac2);
}

console.log(Fibonacci1(10));  // 89
console.log(Fibonacci1(100)); // 573147844013817200000
console.log(Fibonacci1(1000));  // 7.0330367711422765e+208
```

-----

- **递归函数的改写**

  递归函数改成尾递归的实现方法：把所有用到的函数的内部变量改写成函数的参数。

  缺点：不太直观

  方法：

  1、再尾递归函数之外，再提供一个正常形式的函数。

  2、使用参数默认值。

```
function tailFactorial(n, total) {
  if (n === 1) return total;
  return tailFactorial(n - 1, n * total);
}
// 提供一个正常形式的函数
function factorial(n) {
  return tailFactorial(n, 1);
}

factorial(5) // 120

// 采用默认值
function factorial(n, total = 1) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5) // 120
```

***总结***

递归本质上是一种循环操作。纯粹的函数式编程语言没有循环操作命令，所有的循环都用递归实现，这就是为什么尾递归对这些语言极其重要。对于其他支持“尾调用优化”的语言（比如Lua、ES6），只需要知道循环可以用递归代替，而一旦使用递归，就最好使用尾递归。

----

- ***严格模式***

ES6的尾递归优化只在严格模式下开启，正常模式下是无效的（在正常模式下函数内部有两个变量，可以跟踪函数的调用栈）。

 - ``func.arguments``返回调用时函数的参数。
 - ``func.caller``返回调用当前函数的那个函数。

严格模式下禁用这两个变量，所以尾调用模式仅在严格模式下生效。

----

### 6.8、函数参数的尾逗号

ES2017中有一个提案：允许函数最后一个参数有尾逗号。

```
function clownsEverywhere(
  param1,
  param2,
) { /* ... */ }

clownsEverywhere(
  'foo',
  'bar',
);
```



----

## 7、数组的扩展

----------

### 7.1、扩展运算符

- ***含义***

  扩展运算符是三个点(...)，它将一个数组转为用逗号分隔的参数序列。

  ```
  console.log(...[1, 2, 3]) // 1 2 3
  
  console.log(1, ...[2, 3, 4], 5) // 1 2 3 4 5
  
  [...document.querySelectorAll('div')] // [<div>, <div>, <div>]
  
  function f(v, w, x, y, z) { }
  const args = [0, 1];
  f(-1, ...args, 2, ...[3]);
  
  (...[1, 2]) // Uncaught SyntaxError: Unexpected number
  
  console.log((...[1, 2])) // Uncaught SyntaxError: Unexpected number
  
  console.log(...[1, 2]) // 1 2
  ```

  ***Notes***

  扩展运算符如果放在括号中，JavaScript引擎就会认为这是函数调用。

- ***替代函数的``apply``方法***

  由于扩展运算符可以展开数组，所以不再需要使用``apply``方法将数组转为函数的参数。

  ```
  // ES5 的写法
  function f(x, y, z) {
    // ...
  }
  var args = [0, 1, 2];
  f.apply(null, args);
  
  // ES6的写法
  function f(x, y, z) {
    // ...
  }
  let args = [0, 1, 2];
  f(...args);
  
  // ES5 的写法
  Math.max.apply(null, [14, 3, 77])
  
  // ES6 的写法
  Math.max(...[14, 3, 77])
  
  // 等同于
  Math.max(14, 3, 77);
  
  // ES5的 写法
  var arr1 = [0, 1, 2];
  var arr2 = [3, 4, 5];
  Array.prototype.push.apply(arr1, arr2);
  
  // ES6 的写法
  let arr1 = [0, 1, 2];
  let arr2 = [3, 4, 5];
  arr1.push(...arr2);
  
  // ES5
  new (Date.bind.apply(Date, [null, 2015, 1, 1]))
  // ES6
  new Date(...[2015, 1, 1]);
  ```

- ***扩展运算符的应用***

  1、复制数组

  ```
  const a1 = [1, 2];
  // 写法一
  const a2 = [...a1];
  // 写法二
  const [...a2] = a1;
  ```

  2、合并数组

  ```
  const arr1 = ['a', 'b'];
  const arr2 = ['c'];
  const arr3 = ['d', 'e'];
  
  // ES5 的合并数组
  arr1.concat(arr2, arr3);
  // [ 'a', 'b', 'c', 'd', 'e' ]
  
  // ES6 的合并数组
  [...arr1, ...arr2, ...arr3]
  // [ 'a', 'b', 'c', 'd', 'e' ]
  ```

  3、与解构赋值结合

  ```
  const [first, ...rest] = [1, 2, 3, 4, 5];
  first // 1
  rest  // [2, 3, 4, 5]
  
  const [first, ...rest] = [];
  first // undefined
  rest  // []
  
  const [first, ...rest] = ["foo"];
  first  // "foo"
  rest   // []
  
  // 扩展运算符用于数组赋值，只能放到参数最后一位
  const [...butLast, last] = [1, 2, 3, 4, 5]; // 报错
  
  const [first, ...middle, last] = [1, 2, 3, 4, 5]; // 报错
  ```

  4、函数的返回值

  ```
  // 从数据库取出一行数据，通过扩展运算符，直接将其传入构造函数Date
  var dateFields = readDateFields(database);
  var d = new Date(...dateFields);
  ```

  5、字符串

  ​	将字符串转换为真正的数组。

  ```
  console.log([...'hello']);  // [ 'h', 'e', 'l', 'l', 'o' ]
  ```



  6、实现了``Iterator``接口的对象

  ​	任何定义了遍历器(``Iterator``)接口的对象，都可以用扩展运算符转换为真正的数组。

  ```
  let nodeList = document.querySelectorAll('div');
  let array = [...nodeList];
  ```



  7、``Map``和``Set``结构、``Generator``函数

  ​	扩展运算符内部调用的是数据结构的``Iterator``接口，因此只要具有``Iterator``接口的对象，都可以使用扩展运算符。

  ```
  let map = new Map([
    [1, 'one'],
    [2, 'two'],
    [3, 'three'],
  ]);
  
  let arr = [...map.keys()];
  console.log(arr);   // [ 1, 2, 3 ]
  
  // Generator函数运行后会返回一个遍历器对象，因此可以使用扩展运算符。
  const go = function*() {
    yield 1;
    yield 2;
    yield 3;
  }
  
  console.log([...go()]);   // [ 1, 2, 3 ]
  ```

---------------------

### 7.2、``Array.from()``

``Array.from()``方法用于将两类对象转换为真正的数组：类似数组的对象和可以遍历的对象。

```
let arrayLike = {
  '0': 'a',
  '1': 'b',
  '2': 'c',
  length: 3
};

console.log(Array.from(arrayLike));   // [ 'a', 'b', 'c' ]

// 实际应用中，常见类似的数组的对象是DOM操作返回的NodeList集合，以及函数内部的arguments对象。
// NodeList对象
let ps = document.querySelectorAll('p');
Array.from(ps).forEach(p => {
  console.log(p);
});

// arguments对象
function foo() {
  var args = Array.from(arguments);
  // ...
}

// 可遍历对象（部署Iterator接口）
console.log(Array.from('Hello'));   // [ 'H', 'e', 'l', 'l', 'o' ]

let namesSet = new Set(['a', 'b']);
console.log(Array.from(namesSet));  // [ 'a', 'b' ]

// 真正的数组
console.log(Array.from([1, 2, 3]));   // [ 1, 2, 3 ]
```

类似数组的对象：本质特征只有一点，即必须有``length``属性。因此任何有``length``竖向的对象，都可以通过``Array.from()``方法转换为数组。

```
console.log(Array.from({length: 3})); // [ undefined, undefined, undefined ]
```

***Notes***

``Array.from``接受第二个参数，类似于数组的``map``方法

```
console.log(Array.from([1, 2, 3], x => x * x)); // [ 1, 4, 9 ]
// 等同于
console.log(Array.from([1, 2, 3]).map(x => x * x)); // [ 1, 4, 9 ]

Array.from([1, , 2, , 3], (n) => n || 0);	// [1, 0, 2, 0, 3]
```

--------------

### 7.3、``Array.of()``

``Array.of``方法用于将一组值转换为数组。这个方法的主要母的是弥补数组构造函数``Array()``的不足。``Array.of``总是返回参数值组成的数组。如果没有参数，就返回一个空数组。

```
Array.of(3, 11, 8) // [3,11,8]
Array.of(3) // [3]
Array.of(3).length // 1

Array.of() // []
Array.of(undefined) // [undefined]
Array.of(1) // [1]
Array.of(1, 2) // [1, 2]
```

---------------

### 7.4、数组实例的``copyWithin()``

数组实例的``copyWithin``方法会在当前数组内部将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组。

- ``Array.prototype.copyWithin(target, start = 0, end = this.length)``

  ``target``： 从该位置开始替换数组

  ``start``(可选)：从该位置开始读取数据，默认为0，如果为负值，表示倒数。

  ``end``（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示倒数。

```
[1, 2, 3, 4, 5].copyWithin(0, 3) // [4, 5, 3, 4, 5]
console.log([1, 2, 3, 4, 5].copyWithin(0, 3, 4));   // [ 4, 2, 3, 4, 5 ]
console.log([1, 2, 3, 4, 5].copyWithin(0, -2, -1));   // [ 4, 2, 3, 4, 5 ]
```

------------------

### 7.5、数组实例的``find()``和``findIndex()``方法

``find()``：找出第一个符合条件的数组成员，参数为一个回调函数，所有数组成员依次执行该回调，知道找到第一个返回值为``true``的成员，然后返回该成员，若无则返回``undefined``;

``findIndex()``：返回第一个符合条件的数组成员的位置，若无，则返回``-1``。

```
console.log([1, 4, -5, 10].find((n) => n < 0));   // -5
console.log([1, 5, 10, 15].find(value => value > 9)) // 10
console.log([1, 5, 10, 15].findIndex(value => value > 9));  // 2
```

--------------

### 7.6、数组实例的``fill()``

``fill()``：使用给定值填充一个数组，（第二个参数和第三个参数表示填充的起始和结束位置）。

```
console.log(['a', 'b', 'c'].fill(7));   // [ 7, 7, 7 ]
console.log(new Array(3).fill(7));    // [ 7, 7, 7 ]
console.log(['a', 'b', 'c'].fill(7, 1, 2));    // [ 'a', 7, 'c' ]

let arr = new Array(3).fill({name: 'zhangsan'});
arr[0].name = 'lisi';
console.log(arr);   // [ { name: 'lisi' }, { name: 'lisi' }, { name: 'lisi' } ]
```

**``Notes``**

如果填充类型为对象，那么被赋值的是同一个内存地址的对象，而不是深拷贝对象。

-----------------------

### 7.7、数组实例的``entries()``、``keys()``、``values()``

ES6提供了三个新的方法 -------- ``entries()``、``keys()``、``values()``,用于遍历数组。他们都返回一个遍历器对象，可用``for...of``循环遍历。

``entries()``：对键值对的遍历。

``keys()``：对键名的遍历。

``values()``：对键值的遍历。

```
for(let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0 1

for(let item of ['a', 'b'].values()) {
  console.log(item);
}
// a b

for(let [index, item] of ['a', 'b'].entries()) {
  console.log(index, item);
}
// 0 'a'
// 1 'b'

let letter = ['a', 'b', 'c'];
let entrie = letter.entries();

console.log(entrie.next().value);   // [ 0, 'a' ]
console.log(entrie.next().value);   // [ 1, 'b' ]
console.log(entrie.next().value);   // [ 2, 'c' ]
```

-------------------------------

### 7.8、数组实例的``includes()``

``Array.prototype.includes``方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的``includes``方法类似。

第二个参数表示搜索的起始位置，默认为``0``。（如果为负，则表示倒数的位置，如果大于数组长度，则会重置为0开始）。

```
console.log(['1', '2', '3'].includes('1'));   // true
console.log(['1', '2', '3'].includes(1));   // false
console.log(['1', '2', '3'].includes('4'));   // false
console.log(['1', '2', NaN].includes(NaN));   // true

console.log([1, 2, 3, 4, 5].includes(1, 1));  // false
console.log([1, 2, 1, 3, 4, 5].includes(5, -1));  // false
console.log([1, 2, 1, 3, 4, 5].includes(1, -7));  // true

console.log([NaN].includes(NaN));   // true
```

``indexOf``方法有两个缺点：

1、不够语义化，其含义是找到参数值的第一个出现的位置，所以要比较是否不等于``-1``，表达起来不够直观。

2、内部使用严格相等运算符(===)进行判断，会导致``NaN``的误判。

-------------------

### 7.9、数组的空位

数组的空位指数组的某一个位置没有任何值（空位不是``undefined``，一个位置的值等于``undefined``依然是有值的。空位是没有任何值的）。

```
console.log(new Array(3));    // [ <3 empty items> ]
console.log(0 in [undefined, undefined, undefined]);    // true
console.log(0 in [,,,]);    // false
```

ES5对空位的处理：

1、``forEach()``、``filter()``、``every()``、``some()``都会跳过空位。

2、``map()``会跳过空位，但会保留这个值。

3、``join()``和``toString()``会将空位视为``undefined``，而``undefined``和``null``会被处理成空字符串。

````
[, 1].forEach(x => console.log(x));   // 1
[1, , 3].filter(x => console.log(x)); // 1 3
[,, 'a'].every(x => console.log(x === 'a'));  // true
console.log([,, 'a'].map(x => x = 1));  // [ <2 empty items>, 1 ]
````

ES6明确将空位转为``undefined``

```
console.log(Array.from([1, , 3]));    // [ 1, undefined, 3 ]
console.log([...[1, , 3]]);   // [ 1, undefined, 3 ]
console.log([, 'a', 'b',,].copyWithin(2, 0));   // [ <1 empty item>, 'a', <1 empty item>, 'a' ]
console.log(new Array(3).fill('a'));    // [ 'a', 'a', 'a' ]

for(let i of [,, 3]) {
  console.log(i);   // undefined undefined 3
}
```

**``Notes``**

由于空位的处理规则非常不统一，所以建议避免出现空位。

-----

## 8、对象的扩展

### 8.1、属性的简洁表示法

ES6允许直接写入变量和函数作为对象的属性和方法。(ES6允许在对象中只写属性名，不写属性值。这时，属性值等于属性名所代表的变量)

```
const foo = 'bar';
const baz = {foo};
baz // {foo: "bar"}

// 等同于
const baz = {foo: foo};

function f(x, y) {
  return {x, y};
}

// 等同于
function f(x, y) {
  return {x: x, y: y};
}

f(1, 2) // Object {x: 1, y: 2}

const o = {
  method() {
    return "Hello!";
  }
};

// 等同于
const o = {
  method: function() {
    return "Hello!";
  }
};

let birth = '2001/09/01';

let obj = {
  name: 'zhangsan',
  birth,
  sayHi() {
    console.log('Hello World');
  }
}
```

用于函数的返回值。

```
function getPoint() {
  const x = 0;
  const y = 1;
  return {x, y};
}

console.log(getPoint());    // { x: 0, y: 1 }
```

属性的``getter()``和``setter()``。

```
const cart = {
  _wheels: 4,
  get wheels() {
    return this._wheels;
  },
  set wheels(values) {
    if(values < this._wheels) {
      throw new Error('数值太小了!');
    }
    this._wheels = values;
  }
}
```

**``Notes``**

简洁写法中属性名总是字符串。

```
const obj = {
  class () {}
};

// 等同于

var obj = {
  'class': function() {}
};

// 如果某个方法的值是一个 Generator 函数，前面需要加上星号。
const obj = {
  * m() {
    yield 'hello world';
  }
};
```

------------------------------

### 8.2、属性名表达式

JavaScript语言定义对象的属性有两种方式。

1、直接使用标识符作为属性名。

2、用表达式作为属性名，这时要将表达式放在括号内。

```
// 方法一
obj.foo = true;

// 方法二
obj['a' + 'bc'] = 123;
```

ES6允许字面量定义对象时使用方法二（表达式作为的对象的属性），即把表达式放在方括号内。

```
let propKey = 'foo';
let propObj = {
  [propKey]: true,
  ['a' + 'bc']: 123
}
console.log(propObj);   // { foo: true, abc: 123 }

let lastName = 'lisi';
const aPerson = {
  'first name': 'Hello',
  [lastName]: 'lisi'
}

console.log(aPerson);   // { 'first name': 'Hello', lisi: 'lisi' }

let funcObj = {
  ['say' + 'Hi']() {
    console.log("Hello World");
  }
}

funcObj.sayHi();    // Hello World
```

**Notes**

属性名表达式如果是一个对象，默认情况下会自动将对象转换为字符串``[object Object]``。

```
// 报错
const foo = 'bar';
const bar = 'abc';
const baz = { [foo] };

// 正确
const foo = 'bar';
const baz = { [foo]: 'abc'};

const keyA = {a: 1};
const keyB = {b: 2};

const myObject = {
  [keyA]: 'valueA',
  [keyB]: 'valueB'
};

myObject // Object {[object Object]: "valueB"}
```

------------

### 8.3、方法的``name``属性

函数的``name``属性返回函数名。对象方法也是函数，因此也有``name``属性。

```
const person = {
  sayName() {
    console.log('Hello');
  }
}

console.log(person.sayName.name);   // sayName
```

--------------------

### 8.4、``Object.is()``

用来比较两个值是否严格相等，与严格相等运算符(===)行为基本一致

```
console.log(Object.is('foo', 'foo'));   // true
console.log(Object.is({}, {}));     // false
console.log(Object.is(+0, -0));     // false
console.log(Object.is(NaN, NaN));   // true
```

--------------------------

### 8.5 ``Object.assign()``

``Object.assign()``方法用来将源对象的所有可枚举属性，复制到目标对象上。

```
var target = {a: 1};
var source = {b: 1};
var source2 = {c: 1};

// 基本用法
Object.assign(target, source, source2);
console.log(target);  // { a: 1, b: 1, c: 1 }

// 只有一个参数的情况
target = {a: 1};
console.log(Object.assign(target));   // { a: 1 }

// 对于不是对象的属性先转换成对象
console.log(typeof Object.assign(2));   // object
console.log(Object.assign(undefined));   // Cannot convert undefined or null to object
console.log(Object.assign(null));   // Cannot convert undefined or null to object

// 源对象不是对象的情况
console.log(Object.assign({a: 1}, undefined));  // { a: 1 }
console.log(Object.assign({a: 1}, null));   // { a: 1 }
console.log(Object.assign({a: 1}, 2));  // { a: 1 }
console.log(Object.assign({a: 1}, 'str'));   // { '0': 's', '1': 't', '2': 'r', a: 1 }

// 原因：只有字符串的包装对象，会产生可枚举的属性
const v1 = 'abc';
const v2 = true;
const v3 = 10;
console.log(Object.assign({}, v1, v2, v3));   // { '0': 'a', '1': 'b', '2': 'c' }

var target = {a: 1, b: 2};
var source = {b: 1};
var source2 = {c: 1};

// 重复属性
Object.assign(target, source, source2);
console.log(target);  // { a: 1, b: 1, c: 1 }

// 不可枚举的属性和继承的属性不会被复制
const source3 = {};
Object.assign(target, Object.defineProperty(source3, 'invisible', {
  enumerable: false,
  value: 'hello'
}));

console.log(target);    // { a: 1, b: 1, c: 1 }
console.log(source3);   // {}

// 属性名为Symbol值的属性
target = {a: 'b'};
Object.assign(target, {[Symbol('c')]: 'd'})
console.log(target);    // { a: 'b', [Symbol(c)]: 'd' }
```

**注意点**

``Object.assign``方法属于浅拷贝，而不是深拷贝。也就是说，如果源对象的某个属性是对象，那么目标对象拷贝得到的是这个对象的引用。

```
// 对于嵌套的对象，直接覆盖
target = {a: {b: 'c', d: 'e'}};
source = {a: {f: 'Hello'}};

Object.assign(target, source);
console.log(target);    // { a: { f: 'Hello' } }
```

**用途**

- 为对象添加属性

  ```
  class Point {
    constructor(x, y) {
      Object.assign(this, {x, y});
    };
  };
  
  const p = new Point(1, 2);
  console.log(p);   // Point { x: 1, y: 2 }
  ```

- 为对象添加方法

  ```
  Object.assign(Point.prototype, {
    getAge() {
      return 12;
    }
  });
  
  // 等同于
  // Point.prototype.getAge = function() {return 12};
  
  console.log(p.getAge());    // 12
  ```

- 克隆对象

  ```
  function clone(origin) {
    return Object.assign({}, origin);
  }
  
  console.log(clone(p));    // { x: 1, y: 2, age: 12 }
  ```

- 合并多个对象

  ```
  const merge = (target, ...source) => Object.assign(target, ...source);
  
  const merge1 = (...source) => Object.assign({}, ...source);
  ```

- 为属性指定默认值

  ```
  const DEFAULT = {
    logLevel: 0,
    outputFormat: 'html'
  }
  
  const processContent = (options) => Object.assign({}, DEFAULT, options);
  
  console.log(processContent());    // { logLevel: 0, outputFormat: 'html' }
  ```

---------------

### 8.6、属性的可枚举性

对象的每一个属性都具有一个描述对象，用于控制该属性的行为。``Object.getOwnPropertyDescriptor``方法可以获得该属性的描述对象。

```
var obj = {foo: 123};
console.log(Object.getOwnPropertyDescriptor(obj, 'foo'));

//{ value: 123,
// writable: true,
// enumerable: true,
// configurable: true }
```

对象的``enumerable``属性称为“可枚举性”，如果该属性为``false``，就表示某些操作会忽略当前属性。

以下操作会忽略``enumerable``为``false``的属性。

- ``for...in()``：只遍历对象自身的和继承的可枚举属性。
- ``Object.keys()``：返回对象自身的所有可枚举属性的键值。
- ``JSON.stringify()``：只串行化对象自身的可枚举属性。
- ``Object.assign()``：会忽略``enumrable``为``false``的属性，只复制对象自身的可枚举的属性。

**``Notes``**

最初引入``enumrable``的目的就是为了让某些属性规避掉``for...in``操作。例如对象原型的``toString``和``length``属性。

例如，对象的``toString``方法和数组的``length``属性就是通过这种方法不会被``for...in``遍历到。

```
const test1 = Object.getOwnPropertyDescriptor(Object.prototype, 'toString').enumerable;
console.log(test1);   // false

const test2 = Object.getOwnPropertyDescriptor([], 'length').enumerable;
console.log(test2);   // false

// 另外，所有Class的原型的方法都是不可枚举的。
const test2 = Object.getOwnPropertyDescriptor([], 'length').enumerable;
console.log(test2);   // false
```

**总结**

操作中引入继承的属性会让问题复杂化，大多数时候，我们只关心对象自身的属性。所以，尽量不要使用``for...in``循环，而用``Object.keys()``代替。

-----------------------

### 8.7、属性的遍历

``ES6``一共有5种方法可以遍历对象的属性。

- ``for...in``：遍历自身的和继承的可枚举属性(不含``Symbol``属性)。
- ``Object.keys(obj)``：返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含``Symbol``属性）。
- ``Object.getOwnPropertyNames(obj)``：返回一个数组，包含对象自身的所有属性（不含``Symbol``,含不可枚举属性）。
- ``Object.getOwnPropertySymbols(obj)``：返回一个数组，包含对象自身所有的``Symbol``属性。
- ``Reflect.ownKeys(obj)``：返回一个数组，包含对象自身的所有属性（包括``Symbol``属性和不可枚举属性）。

**遍历次序规则**

- 首先遍历所有属性名为数值的属性，按照数字排序。
- 遍历所有属性名为字符串的属性，按照生成时间排序。
- 遍历所有属性名为``Symbol``值的属性，按照生成时间排序。

```
const arr = Reflect.ownKeys({[Symbol()]: 0, b: 0, 10: 0, 2: 0, a: 0});
console.log(arr);   // [ '2', '10', 'b', 'a', Symbol() ]
```

--------

### 8.8、``__proto__属性``

``__proto__``属性用来读取或设置当前对象的``prototype``对象。

```
var obj = {
    method() {
        console.log('hello');
    }
}

obj.__proto__ = somOtherObj;
```

**Notes**

该属性没有写入ES6正文，只是写入了附录，本质上是一个内部属性，而不是一个正式的对外的API。**建议不要使用这个属性**。而是使用``Object.setPrototypeOf()``、``Object.getPrototypeOf()``或``Object.create()``来代替。

- ``Object.setPrototypeOf()``

  此方法的作用与``__proto__``相同，用来设置一个对象的``prototype``对象，返回参数对象本身。

```
// 用法
Object.setPrototypeOf(object, prototype);

var o = Object.setPrototypeOf({}, null);

let proto = {};
let obj = { x: 10 };
Object.setPrototypeOf(obj, proto);

proto.z = 20;
proto.y = 40;

console.log(obj.x); // 10
console.log(obj.y); // 40
console.log(obj.z); // 20

// 对于第一个参数不是对象的，先转换为对象，无法转换的则报错
console.log(Object.setPrototypeOf(1, {}) === 1);    // true
console.log(Object.setPrototypeOf('foo', {}) === 'foo');    // true
console.log(Object.setPrototypeOf(true, {}) === true);  // true
console.log(Object.setPrototypeOf(null, {}));   // TypeError: Object.setPrototypeOf called on null or undefined
console.log(Object.setPrototypeOf(undefined, {}));  // TypeError: Object.setPrototypeOf called on null or undefined
```

- ``Object.getPrototype()``

  该方法用于读取一个对象的``prototype``对象。

```
function Rectangle() {}

var rec = new Rectangle();

console.log(Object.getPrototypeOf(rec) === Rectangle.prototype);    // true
Object.setPrototypeOf(rec, Object.prototype);
console.log(Object.getPrototypeOf(rec) === Rectangle.prototype);    // false

// 如果参数不是对象，则会自动转换为对象
console.log(Object.getPrototypeOf(1));  // [Number: 0]
console.log(Object.getPrototypeOf('foo'));  // [String: '']
console.log(Object.getPrototypeOf(true));   // [Boolean: false]
console.log(Object.getPrototypeOf(1) === Number.prototype); // true
console.log(Object.getPrototypeOf('foo') === String.prototype); // true
console.log(Object.getPrototypeOf(true) === Boolean.prototype); // true

// 如果参数无法转换为对象，则直接报错
Object.getPrototypeOf(null);    // TypeError: Cannot convert undefined or null to object
Object.getPrototypeOf(undefined); // TypeError: Cannot convert undefined or null to object
```

-------------

### 8.9、对象的遍历

- ``Object.keys()``:返回一个数组，包括参数对象自身的所有可遍历属性的键名（不含继承的，ES5）。

  **ES7提案**：``Object.values()``、``Object.entries()``作为遍历一个对象的补充手段，供``for...of``循环使用。

  ```
  let {keys, values, entries} = Object;
  let obj1 = {x: 1, y: 2, z: 3};
  
  for(let key of keys(obj1)) {
      console.log(key);   // x y z
  }
  
  for(let value of values(obj1)) {
      console.log(value);     // 1 2 3
  }
  
  for(let [key, value] of entries(obj1)) {
      console.log([key, value]);      //  ['x', 1 ] [ 'y', 2 ] [ 'z', 3 ]
  }
  ```

- ``Object.values``方法返回一个数组，包括参数对象（不含继承的）的所有可遍历属性的键值。

  ```
  let obj = {foo: 43, bar: '123'};
  
  console.log(Object.values(obj));    // [ 43, '123' ]
  
  obj = { 100: 'a', 2: 'b', 7: 'c' };
  console.log(Object.values(obj));    // [ 'b', 'c', 'a' ]
  
  // 只返回对象可遍历的属性
  obj = Object.create({}, {
      p: {
          value: 42,
          enumerable: true
      }
  });
  
  console.log(Object.values(obj));    // [ 42 ]
  
  // 如果不是对象则转换为对象（数值和布尔值的包装类型都不会为实例添加非继承的属性）
  console.log(Object.values('foo'));  // [ 'f', 'o', 'o' ]
  console.log(Object.values(true));   // []
  console.log(Object.values(42));     // []
  ```

- ``Object.entries``方法返回一个数组，包括对象自身的（不含继承的）所有可遍历的属性的键值对数组。

  ```
  let obj = {foo: 43, bar: '123'};
  console.log(Object.entries(obj));   // [ [ 'foo', 43 ], [ 'bar', '123' ] ]
  
  obj = {one: 1, two: 2};
  
  for(let [key, value] of Object.entries(obj)) {
      console.log(key, value);    // one 1    two 2
  }
  
  // 将对象转换为真正的``Map``结构
  const map = new Map(Object.entries(obj));
  console.log(map);   // Map { 'one' => 1, 'two' => 2 }
  
  // 自己实现``Object.entries``方法
  function* entries(obj) {
      for(let key of Object.keys(obj)) {
          yield [key, obj[key]];
      }
  }
  
  function entries(obj) {
      let arr = [];
  
      for(let key of Object.keys(obj)) {
          arr.push([key, obj[key]]);
      }
  
      return arr;
  }
  
  ```

  --------

### 8.10、``super``关键字

ES6新增``super``关键字，指向当前对象的原型对象。

```
const proto = {
    foo: 'Hello'
};

const obj = {
    foo: 'world',
    find() {
        return super.foo;
    }
};

Object.setPrototypeOf(obj, proto);
console.log(obj.find());    // Hello
```

**Notes**

``super``关键字表示原型对象时，只能用在对象的方法中，用在其他地方会报错。

```
// 报错
const obj = {
  foo: super.foo
}

// 报错
const obj = {
  foo: () => super.foo
}

// 报错
const obj = {
  foo: function () {
    return super.foo
  }
}
```

-------------------

### 8.11、对象的扩展运算符

- 解构赋值

  对象的解构赋值用于从一个对象取值，相当于将所有可遍历的，但尚未被读取的属性，分配到指定的对象上。所有的键和他们的值，都会被拷贝到新对象上。

```
const { x, y, ...z} = {x: 0, y: 1, a: '1', b: '2'};

console.log(x);     // 0
console.log(y);     // 1
console.log(z);     // { a: '1', b: '2' }
```

**Notes**

1、对象右边要求是一个等号，若为``undefined``或``null``就会报错。

2、解构赋值必须是最后一个参数，否则会报错。

3、解构赋值的拷贝是浅拷贝，如果一个键的值是复合类型的值（数组、函数、对象），那么解构赋值拷贝的是这个对象的引用。而不是这个值的副本。

4、扩展运算符的解构赋值，不能复制继承自原型对象的属性。

5、变量声明语句之中，如果使用解构赋值，扩展运算符后面必须是一个变量名，而不能是一个解构赋值表达式。

```
let { x, y, ...z } = null; // 运行时错误
let { x, y, ...z } = undefined; // 运行时错误

let { ...x, y, z } = someObject; // 句法错误
let { x, ...y, ...z } = someObject; // 句法错误

obj = {a: {b: 1}};
let {...c} = obj;

obj.a.b = 2;
console.log(c.a.b); // 2

let o1 = { a: 1};
let o2 = { b: 2};
o2.__proto__ = o1;
let {...o3} = o2;

console.log(o3);    // { b: 2 }
console.log(o3.a); // undefined

const o = Object.create({ x: 1, y: 2});
o.z = 3;

let { x, ...newObj } = o;
let { y, z} = newObj;

console.log(x, y, z);   // 1 undefined 3
```





































