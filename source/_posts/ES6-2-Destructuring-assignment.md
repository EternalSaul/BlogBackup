---
title: 'ES6(2):Destructuring assignment'
date: 2017-12-08 23:57:12
toc: true
tags:
  - javascript
  - ES6
  - 学习笔记
---

  本世纪以来的很多新语言都提供了各种赋值或者模式匹配判断的语法糖，es6也提供了对解构赋值的一系列支持使得我们可以在很多情况下节省很多行代码。但是对一个复杂模式的解构赋值很可能让维护代码的人员感到很疑惑，所以这里记录的解构赋值没有那么多奇技淫巧。

<!--more-->

## 数组解构

我们在es6中，可以向scala中操作元组和集合那样，用模式匹配的方式来对数组进行解构赋值
```javascript
{
    let [a, [b, c], d] = [1, [2, 3], 4];
    alert(c);//输出3
}

{
    let [a, b] = [, 2];
    alert(a);//a没有匹配项，解构不成功，输出undefined
    alert(b);//输出2
}

{
    let [a, b] = [1, 2, 3];
    alert(b);//输出2
}

{
    // 我们可以用...来表示一个多值匹配
    let [a, [...b], c] = [1, [2, 3, 4], 5];
    alert(b);//输出2,3,4
    alert(c);//输出5
}

{
    let [a, ...b] = [1, 2, 3, 4];
    alert(b);//输出2,3,4
}
```
### 提供默认值的解构

我们可以为解构需要的变量赋默认值
```javascript
{
    let [a, b = 4] = [1];
    alert(b);//输出4
}

{
    let [a, b = 4] = [1, undefined];//不严格等于undefined，所以赋默认值
    let [c,d=4]=[1,null];
    alert(b);//输出4
    alert(d);//输出null
}


{
    let [a, b = a] = [1];//你甚至可以这样
    alert(b);//输出1
}
```

### 惰性求值

es6中也有惰性求值这种东西的存在，比如如果结构中默认值是一个表达式那么，默认值其实是惰性求值的，即如果结构成功这个表达式根本不执行
```javascript
{
    function lazy() {
        alert("execute lazy");
        return "abc";
    }
    
    let [m = lazy(), k] = [1, 2];//不出现弹窗
}
```

### 数组结构与可遍历性

数组解构赋值的进行实际上是利用了右边集合的可遍历性，如果右边的集合不具有可遍历性则直接报错
相反的，如果右边的集合是一个可遍历的结构，当然就是可以解构的

```javascript
{
    let [k] = NaN;//Uncaught TypeError: NaN is not iterable
    // let [k]=1;
    // let [k]={};
    // let [k]=null;
    // let [k]=false;
    // let [k]=undefined;
  //全部报类似错误
}
```
## 对象解构

对象也可以使用解构方式来赋值，不过对象不再要求右边的值是一个可遍历的解构了
这种方式类似于scala中的函数传参
```javascript
{
    let {tom, jack} = {jack: "jack", tom: "tom"};
    //相当于 let {tom:tom, jack:jack} = {jack: "jack", tom: "tom"};
    alert(tom);//输出tom
}
```

### 对象结构中的默认值使用

当然我们可以赋默认值
```javascript
{
    let {tom="TOM",mary, jack} = {jack: "jack"};
    alert(tom);
    alert(mary);//输出undefined
}
```
这种语法糖更是提供了我们对对象属性的赋值，和将对象属性直接转化为变量
```javascript
{
   let man={clz:"Man",old:18};
   let {clz:clz,old:od}=man;//以属性名:变量名的方式来确定赋值
    alert(clz);
    alert(od);

   let woman={clz:"woman",old:od};//变量作为实际值
   alert(woman.old);//输出18
}
```

### 嵌套匹配结构

嵌套的匹配
```javascript
{
    let course={
        name:"Operation System",
        detail:{
            score:3.5,
            teacher:["Mark Watson","John Joy"]
        }
    };
    let {name,detail:{score,teacher:[t1,t2]}}=course;
    //如果你想获取整个detail的值，可以这样let {name,detail,detail:{score,teacher:[t1,t2]}}=course;
    alert(name);
    alert(score);
    alert(t2);//正确输出了John joy，可见新的语法糖给我们带来的便利是很明显的，虽然很少会用到
}
{
    let k=[];
    ({z:k[1],m:k[0]}={m:2,z:1});//加小括号的理由是{开头会被解释器认为是一个代码块的开始
    alert(k[0]);
}
```

### 提取函数和转化字符串

我们可以这样讲对象中的函数提取出来
```javascript
{
    let {sin,cos,log,tan}=Math;
    alert(tan);
}
```

对于字符串，我们可以将它解构为一个个字符
```javascript
{
   let [a,b,c]="abc";
   alert(a+"~"+b+"~"+c);//输出a~b~c
}
```
## 函数解构

对函数的数组参数进行解构
```
{
    function hello([x,y]) {
        alert(x+" "+y);
    }
    let v=["hello","world"];
    hello(v);//输出hello world
}
```

提供默认值和默认参数
```
{
    function halo({x="halo",y="world"}={}) {
        alert(x+" "+y);
    }
    halo({x:"hello"});
    halo({y:"Jack"});
    function hi({x,y}={x:"hi",y:"world"}) {
        alert(x+" "+y);
    }
    hi({x:"hello"});//提供了参数{x:"hello"}，不使用默认参数，输出hello undefined
    hi();//使用默认参数，输出hello world
}
```
## 解构的应用

用于交换
```javascript
{
    let [x,y]=[1,2];
    [x,y]=[y,x];
    alert(y);//输出1
}
```
用于解构函数的返回值
```javascript
function returnArray() {
    return [1,2,3];
}

function returnObj() {
    return {x:"hey",y:"Daisy"};
}
let [a,b,c]=returnArray();
let {x:m,y:k}=returnObj();
alert(k);//Daisy
alert(b);//2
```

快速提取json对象中的数据
```javascript
let json={name:"Jack",score:[83,91],old:15};
let {name:x,score:s,old:o}=json
alert(s[1]);//91
```

提取map中的键值
```javascript
const map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
    alert(key + " is " + value);
}
```