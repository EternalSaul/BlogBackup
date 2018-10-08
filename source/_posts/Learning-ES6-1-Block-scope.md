---
title: 'Learning ES6(1):块作用域'
date: 2017-12-06 14:46:19
toc: true
tags:
  - javascript
  - ES6
  - 学习笔记
---

  最近主要是在学习3个东西，一个就是单元测试，第二个就是jvm，还有一个就是es6了，目前的话jvm已经学完了，就差把笔记搬运从笔记本上到博客上，单元测试和es6的话都是正在进行，es6的是阮一峰的[ECMAScript 6 入门](https://github.com/ruanyf/es6tutorial)，大神已经把书开源道github上了，推荐大家去看那本书，博客这里主要用于我个人用于以后回忆某些知识用的。

  至于es6是什么可以自行使用搜索引擎，或看上面那本书的第1章，这里我就不介绍了。

---

  在ES6之前，js中是没有**块级作用域**这个东西的，只有**函数**和**全局**作用域
  ES6中的第一个特性，即以let来声明一个变量，这个变量只在当前代码块中有效，比如对于如下代码：

```javascript
!function () {
    {
        let k = 3;//k作用域只在当前代码块中
    }
    alert(k); //报错，外层代码不能去引用内层代码块中的k，如果上面是var则输出3
}();

!function () {
    let k = 3;
    {
        alert(k);//正常输出3
    }
}();

!function () {
    let k = 4;
    {
        let k = 5;
        alert(k);//输出k=5
    }
    alert(k);//输出k=4
}();
```

<!--more-->

##  let变量特性

### 变量提升和暂时性死区

  对于var声明的变量会有变量提升问题，即是在作用域开头变量已经存在但是没有赋值，**var定义的变量只能是函数或者全局作用域有效的，没有块这个概念**。比如对于以下代码：

```javascript
!function () {
    alert(f);//输出undefined，var声明的变量标识在函数作用域有效，有变量提升
    var f = 1;
}();

//实际相当于执行代码：
!function () {
  	var f = undefined;
    alert(f);
    f = 1;
}();
```

但是对于let来说，这样声明就报错，**let声明的变量没有变量提升**，作用域是块级。

```javascript
!function () {
  //TDZ开始
    alert(k);//报错:Uncaught ReferenceError: k is not defined
    let k = 1;//TDZ结束
}();
```

现在我们来引入一个概念：暂时性死区(temporl dead zone,TDZ),这个概念是对于let和const变量而言的，es6引入它的目的在于，让js程序员养成使用前先定义的好习惯。

```
* 在阮一峰的《ECMAScript 6 入门》中提到
* ES6 明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。
* 凡是在声明之前就使用这些变量（任何操作），就会报错。
*
* 在块中，let定义一个变量n之前的区域，都称之为“展暂时性死区”
```

总之，暂时性死区的本质就是，**只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。**

### 不可重复声明

在同一个作用域中，var声明的变量可以重复声明，会取后声明的值

```javascript
!function () {
    //正常运行
    var f = 1;
    var f = 2;
}();
```

而在同一个作用域中，let声明的变量不可以重复声明，会报错

```javascript
!function () {

    //报错
    let k = 4;
    let k = 5;
    
    //如果同一个变量已经被var声明，那么如果它再用let声明同样报错
    //let f=3;
}();

!function () {

    var f = 4;
    
    //如果同一个变量已经被var声明，那么如果它再用let声明同样报错
    //相反一样
    let f=3;//这一行报错
}();
```

### 用let做for循环参数

ES5中由于闭包关系，我们不得不用一段不太漂亮的代码来实现for循环中i的不同取值

```
!function () {
    var a = {};
    for (let i = 0; i < 10; i++) {
        a[i] = function (arg) {//闭包函数，以变量arg来保存当前i的值，从而传递给内部的返回函数
            return function () {
                alert(arg)
            }
        }(i);
    }
    a[6]()
}();
```

在ES6中我们可以这样做:

```javascript
!function () {
    var a = {};
    for (let i = 0; i < 10; i++) {
        a[i] = function () {
            alert(i);
        }
    }
    a[6]()
}();

/*
 这里很多人可能会理解错误，因为for语句和{}代码块其实是两个作用域
 而let只能在当前作用域有效，那么{}代码块里的数据是怎么来的呢？
 其实这里每次循环都创建了一个新的i，这个新的i的值在旧的i的基础上计算
 */
 
//类似于执行一个这样的，过程
var a = {};
{ let k = 0;  
    for (;k < 10;) {
      let i = k; 
      a[i] = function () {
        alert(i);
      };
      k++;
    }
}
a[6](); // 6

//在babel中转成es代码是这样的：
var a = {};

var _loop = function _loop(i) {
    a[i] = function () {
        alert(i);
    };
};

for (var i = 0; i < 10; i++) {
    _loop(i);
}
```

## const变量

  const声明常量，一旦声明就不能被改变，所以const声明时必须立刻赋值
如果下代码不赋值就抛出：Uncaught SyntaxError: Missing initializer in const declaration

```javascript
const k;
alert(k);
```

**const同样只在块作用域内有效，不可重复声明，且存在暂时性死区**

  const只能保证它所指向的地址不可变，但是这个地址的数据结构怎么变它管不了，比如：

```javascript
const man={};
man.name="Tom";
man.name="Jack";
alert(man.name);//输出jack
```

  如果要保证对象不可变，还是必须用Object.freeze去冻结一个对象，比如

```javascript
const man=Object.freeze({name:"Tom"});
man.name="Jack";//不报错，但是不起作用，如果是严格模式就报错了
alert(man.name);
```

这种冻结方式对对象的对象引用属性没效果，但是你可以以递归去写一个冻结函数。

## 块级作用域和函数

在es5浏览器中，下面的代码是可以正常运行的，并输出I am inside，因为function声明会提升到函数作用域顶端

```javascript
!function(){
    function f() { console.log('I am outside!'); }
    !function () {
        if (false) {
            function f() { console.log('I am inside!'); }
        }
        f();
    }();
}();
```

  在es5浏览器中，上面的代码是可以正常运行的，并输出I am inside，因为function声明会提升到函数作用域顶端，实际上执行下面这段：

```javascript
!function(){
	function f() { console.log('I am outside!'); }
    (function () {
        function f() { console.log('I am inside!'); }
        if (false) {
        }
        f();
    }());
}();
```
  在es6浏览器中，在块级作用域中声明函数，类似var声明变量，下面这段代码会抛出错误:Uncaught TypeError: f is not a function
实际上es6浏览器执行的代码，是类似于下面这段：

```javascript
!function(){
    function f() { console.log('I am outside!'); }
    (function () {
        var f = undefined;
        if (false) {
            f=function() { console.log('I am inside!'); }
        }
        f();
    }());
}();
```

  最佳实践是：避免在块级作用域内声明函数，在有必要的情况下，也应该写成函数表达式，而不是函数声明语句。

## 全局作用域的let和const

在es5中，我们知道浏览器中的顶层对象指的是window和self对象(Node中是global)

浏览器中对于var和function命令声明的全局变量，其实就是window的属性

es6为了改变这种含糊的表达方式，但是又不得不得向以前的代码妥协，于是它做出了以下区分：
首先它对于var和function命令声明的全局变量意义不变
对于let,const,class声明的全局变量，不再是顶层对象的属性

```javascript
function abc(){
    alert("abc");
}
window.abc();

var k="kkk";
alert(window.k);

let i="iii";
alert(window.i);//undefinded
```