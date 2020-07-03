---
title: 聊聊js中的高阶函数
date: 2020-06-30 15:13:07
description: 高阶函数
tags: javascript
categories: javascript
---

#### 高阶函数

##### 1.概念

什么是高阶函数

- 函数当参数传递给另一个参数

```
function a(b) {}
  a(function (params) {
});
```

- 一个函数返回另一个函数

```
function a() {
  return function (params) {};
}
```

##### 2.应用场景

###### 1.判断类型

举个例子，目前我们判断数据类型有以下几种方法

- typeof
  其中 typeof 区分不了 object/array/null (object)
- Object.prototype.toString.call()
- instanceof 1.检测不了用构造函数创建的基本数据类型的值

```
var num=new Number(1);
console.log(typeof num);//object
```

2.instanceof 不仅检测它作用的对象的构造函数，还是检测该对象的原型链的构造函数，只要有一个符合，就会返回 true
比如:

```
var arr = new Array();
var obj = new Object();
console.log( arr instanceof Object);//true
console.log( arr instanceof Array);//true
```

- constructor
//原型被覆盖或者继承,constructor 会丢失
综上，我们暂时用 Object.prototype.toString.call()来判断数据类型

```
let typeUtil = {}
function isType(type){
  return function(content){
    Object.prototype.toString.call(content) === `[object ${type}]`
  }
}
["Array","String","Number","Function","Boolean","Date","Null","Undefined"].forEach((type)=>{
  typeUtil.["is"+type] = isType(type)
})
console.log(typeUtil.isArray([1,2]))
console.log(typeUtil.isBoolean(true))
```

首先声明一个高阶函数，然后批量生成类型判断

###### 2.方法劫持

比如一个应用场景，数组增加前要加入某些方法（vue 数据监听），这时候就需要劫持数组的 push 方法

```
let d = [1, 2, 3];
let oldPush = Array.prototype.push;
const mypush = function (...args) {
  console.log("push以前做一些事情");
  oldPush.call(this, ...args);
};
mypush(2, 3, 4);
```

###### 3.方法重写

```
function a(params) {}
Function.prototype.before = function (cb) {
  return (...args) => {
    //箭头函数没有 arguments 概念
    cb();
    this(...args);
  };
};
let fn = a.before(function () {
  console.log("函数执行前执行某些操作");
});
fn("参数1", "参数2");
```

###### 4.并发

应用场景，接口发送了 2 个请求，这2个请求都结束的时候，需拿到数据并告知做下一步的操作

```
let getdata = axios.get(url)
async function getdata1(){
 let data =  await getdata('url1')
 out(data)
}
async function getdata2(){
 let data =  await getdata("url2")
 out(data)
}
//请求完毕，做接下来的事情
function fn(arr){
   console.log(arr)
}
function after(cb,times){
  let arr = []
  return function(data){
    arr.push(data)
    if(--times ===0){
      cb(arr)
    }
  }
}
let out = after(fn,2)
```

思考，可以看到高阶函数频繁的使用了闭包
什么叫闭包呢？举个例子：

```
function test() {
   //c的当前作用域在这里
   let c = function () {};
   return c;
}
```

test 函数中创建了一个 c 函数，c 函数的作用域在 test 中
如果 test()(); //执行完就销毁，不叫闭包
如果 let d = test(); d()执行的时候 c 的执行作用域在 window 下不在 test 下，这就叫闭包，一个函数不在当前作用域执行就叫闭包