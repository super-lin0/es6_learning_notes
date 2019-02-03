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

  ![变量提升](F:\projects\script-user\es6_learning_notes\imgs\变量提升.jpg)

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

   本质:

   ​	只要进入当前作用域，所要使用的变量就已经存在，但是不可获取，只有等到声明变量的那一行代码出现，

   才可以获取和使用该变量。

- **不允许重复声明**

  let不允许在相同的作用域内重复声明同一个变量。

### 1.2、块级作用域

- 为什么需要

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






