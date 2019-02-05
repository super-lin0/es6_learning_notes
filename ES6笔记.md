---
[MENU]
typora-copy-images-to: imgs
---

# ES6标准入门

[TOC]

## 1、let 和const

### 1.1、let命令

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

## 2、变量的解构赋值

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

### 2.3、字符串的解构赋值

```
const {log} = console;
const [a, b, c, d, e] = 'Hello';
log(a, b, c, d, e);   // H e l l o

const { length: len } = 'Hello';
log(len);   // 5
```

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

## 3、字符串的扩展

### 3.1、字符串的遍历接口

ES6 为字符串添加了遍历器接口，使得字符串可以被for...of循环遍历

```
const s = 'foo';
for(let a of s) {
  console.log(a);
}
```

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

## 4、正则表达式

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

### 4.2、字符串的正则方法

字符串对象共有4个方法可以使用正则表达式：matchs()、replace()、search()、split()。ES6使这4个方法在语言内部全部调用RegExp的实例方法，从而做奥所有与正则相关的方法都定义在RegExp对象上。

- `String.prototype.match` 调用 `RegExp.prototype[Symbol.match]`
- `String.prototype.replace` 调用 `RegExp.prototype[Symbol.replace]`
- `String.prototype.search` 调用 `RegExp.prototype[Symbol.search]`
- `String.prototype.split` 调用 `RegExp.prototype[Symbol.split]`

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

### 4.4、sticky属性

```
var r = /hello\d/y;
console.log(r.sticky);    // true(表示是否设置了y修饰符)
```

### 4.5、flags属性

```
console.log(/abc/ig.flags);     // gi(ES6的flags属性，返回正则表达式的修饰符)
console.log(/abc/ig.source);    // abc(ES5的source属性，返回正则表达式的修饰符)
```

### 4.6、s修饰符

ES2018引入/s修饰符，使得.可以匹配任意单个字符。这被称为dotAll模式，即点代表一切字符。

```
const re = /foo.bar/s;
// const re = new RegExp('foo.bar', 's');

console.log(re.test('foo\nbar'));   // true
console.log(re.dotAll);   // true
console.log(re.flags);    // s
```

### 4.6、具名组匹配

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

## 5、数值的扩展

### 5.1、二进制和八进制表示法

ES6提供了二进制和八进制数值的新的写法，分别用前缀``0b``或``0B``和``0o``或``0O``表示。

```
console.log(0b110011);    // 51
console.log(0o767);     // 503

// 将二进制和八进制字符串数值转换为十进制数值
console.log(Number('0b111'));   // 7
console.log(Number('0o10'));    // 8
```

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

### 5.8、指数运算符

ES2016新增了一个指数运算符(**)。

```
console.log(2 ** 3);  // 8
console.log(2 ** 2);  // 4
let a = 2;
console.log(a **= 5); // 32
```

## 6、函数的扩展

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

### 6.5、箭头函数






































