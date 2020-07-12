---
title: hooks
date: 2020-07-12 15:15:29
description:
tags: hooks
categories: hooks
---

#### ReactHooks 原理

##### useState

- useSate 的简单用法是这样的

```
import React,{useState} from "react";
import ReactDOM from "react-dom";
function Counter(a) {
  let [state, setState] = useState(0);
  let [number, setNumber] = useState(10);
  return (
    <>
      <p>{state}</p>
      <p>{number}</p>
      <button onClick={() => setState(state + 1)}>+</button>
      <button onClick={() => setNumber(number + 1)}>+</button>
    </>
  );
}
function render() {
  ReactDOM.render(<Counter />, document.getElementById("root"));
}
render();

```

可以看到

1. useState 可以传多个
2. useState 能独立执行
3. useState 抛出了初始状态和方法并且可以重新定义

- 我们基于这些特性实现一下

```
import React from "react";
import ReactDOM from "react-dom";
let lastStates = [];//可以支持多个useState
let index = 0;
function useState(initialState) {
  const currentIndex = index;//每个useState的下标
  lastStates[index] = lastStates[index] || initialState;
  function setState(newState) {
    lastStates[currentIndex] = newState;
    render();
  }
  return [lastStates[index++], setState];
}
function Counter(a) {
  let [state, setState] = useState(0);
  let [number, setNumber] = useState(10);
  return (
    <>
      <p>{state}</p>
      <p>{number}</p>
      <button onClick={() => setState(state + 1)}>+</button>
      <button onClick={() => setNumber(number + 1)}>+</button>
    </>
  );
}
function render() {
  ReactDOM.render(<Counter />, document.getElementById("root"));
  index = 0;//每次渲染后下标得重制为初始状态
}
render();
```

- React 中是通过类似单链表的形式来代替数组 lastStates 的
- 每个组建都会生成一个 lastStates
- 第一次渲染时候,是根据 useState 顺序，逐个声明 state 并且将其放入全局 lastStates 中，如果有判断可能导致下标发生变化

##### useMemo

1. useMemo 可以用来优化组件的更新.
2. useMemo 一般用于密集型计算大的一些缓存.
   举个例子

```
import React, { useState, memo, useMemo } from "react";
import ReactDOM from "react-dom";
function Counter() {
  let [name, setName] = useState("张三");
  let [age, setAge] = useState(18);
  return (
    <>
      <button onClick={() => setName(name + "的父亲")}>改变名字</button>
      <button onClick={() => setAge(age + 18)}>改变性别</button>
      <ChildrenName name={name}></ChildrenName>
      <ChildrenAge age={age}></ChildrenAge>
    </>
  );
}
function ChildrenName({ name }) {
  console.log("子组件ChildrenName");
  return (
    <>
      <div>{name}</div>
    </>
  );
}
function ChildrenAge({ age }) {
  console.log("子组件ChildrenAge");
  return (
    <>
      <div>{age}</div>
    </>
  );
}
function render() {
  ReactDOM.render(<Counter />, document.getElementById("root"));
}
render();
```

上面组件当中，Counter 有两个自组件 ChildrenName，ChildrenAge。点击按钮改变名字或者性别都能导致两个组件的更新。

```
import React, { useState, memo } from "react";
import ReactDOM from "react-dom";
function Counter() {
  let [name, setName] = useState("张三");
  let [age, setAge] = useState(18);
  return (
    <>
      <button onClick={() => setName(name + "的父亲")}>改变名字</button>
      <button onClick={() => setAge(age + 18)}>改变性别</button>
      <ChildrenName name={name}></ChildrenName>
      <ChildrenAge age={age}></ChildrenAge>
      //<ChildrenAge age={{age}}></ChildrenAge>
    </>
  );
}
function ChildrenName({ name }) {
  console.log("子组件ChildrenName");
  return (
    <>
      <div>{name}</div>
    </>
  );
}
ChildrenAge = memo(ChildrenAge);
function ChildrenAge({ age }) {
  console.log("子组件ChildrenAge");
  return (
    <>
      <div>{age}</div>
      //<div>{age.age}</div>
    </>
  );
}
function render() {
  ReactDOM.render(<Counter />, document.getElementById("root"));
}
render();

```

引入 memo 后可以看到再改变 name 不会触发 ChildrenAge 组件更新，但是这时候如果我们传入的 age 是对象的话还是会触发更新。
这时候引入 useMemo

```
import React, { useState, memo } from "react";
import ReactDOM from "react-dom";
let lastMemo;
let lastMemoDependencies;
function useMemo(callback, dependencies) {
  if (lastMemoDependencies) {
    let isChanged;
    isChanged = !dependencies.every((item, index) => {
      return lastMemoDependencies[index] === item;
    });
    if (isChanged) {
      lastMemo = callback();
      lastMemoDependencies = dependencies;
    }
  } else {
    lastMemo = callback();
    lastMemoDependencies = dependencies;
  }
  return lastMemo;
}
function Counter() {
  let [name, setName] = useState("张三");
  let [age, setAge] = useState(18);
  let memoAge = useMemo(
    () => ({
      age,
    }),
    [age]
  );
  return (
    <>
      <button onClick={() => setName(name + "的父亲")}>改变名字</button>
      <button onClick={() => setAge(age + 18)}>改变性别</button>
      <ChildrenName name={name}></ChildrenName>
      <ChildrenAge age={{memoAge}}></ChildrenAge>
    </>
  );
}
function ChildrenName({ name }) {
  console.log("子组件ChildrenName");
  return (
    <>
      <div>{name}</div>
    </>
  );
}
ChildrenAge = memo(ChildrenAge);
function ChildrenAge({ age }) {
  console.log("子组件ChildrenAge");
  return (
    <>
      <div>{age.age}</div>
    </>
  );
}
function render() {
  ReactDOM.render(<Counter />, document.getElementById("root"));
}
render();
```

将对象包裹起来可以再传进去可以看到再改变名字不会触发 ChildrenAge 组件更新。
我们实现一个 useMemo，接收两个参数 callback,dependencies,并且只对数据有效，callback 是内部执行了

```
let lastMemo;
let lastMemoDependencies;
function useMemo(callback, dependencies) {
  if (lastMemoDependencies) {
    let isChanged;
    isChanged = !dependencies.every((item, index) => {
      return lastMemoDependencies[index] === item;
    });
    if (isChanged) {
      lastMemo = callback();
      lastMemoDependencies = dependencies;
    }
  } else {
    lastMemo = callback();
    lastMemoDependencies = dependencies;
  }
  return lastMemo;
}
```

简单的 useMemo 就实现了，如果有多个 useMemo，类似 useState 将 lastMemo，lastMemoDependencies 换成数组，加入下标即可。

##### useCallback

useCallback 和 useMemo 极其相似，只不过一个用来缓存数据，一个用来缓存函数

```
let lastCallback;
let lastCallbackDependencies;
function useCallback(callback,dependencies){
  if(lastCallbackDependencies){
    //看看新的依赖数组是不是每一项都跟老的依赖数组中的每一项都相同
    let changed = !dependencies.every((item,index)=>{
      return item == lastCallbackDependencies[index];
    });
    if(changed){
      lastCallback = callback;
      lastCallbackDependencies = dependencies;
    }
  }else{//没有渲染过
    lastCallback=callback;
    lastCallbackDependencies=dependencies;
  }
  return lastCallback;
}
```

举个例子

```
function List({bar, baz}) {
  React.useEffect(() => {
    axios('url',data)
  }, [ids, obj])
  return <div>...</div>
}

function NewsList() {
  const obj = React.useCallback(() => {}, [])
  const ids = React.useMemo(() => [2,3,4], [])
  return <List id={id} name={name} />
}
```

比如上述例子，当 useEffect 的依赖是非原始数据类型的时候，useCallback，useMemo 就派上用场了

##### useReducer

useReducer 实现

```
let lastState;
function useReducer(reducer,initialState){
  lastState= lastState||initialState;
  function dispatch(action){
    lastState= reducer(lastState,action);
    render();
  }
  return [lastState,dispatch];
}
```

应用

```
import React ,{ useReducer } from 'react';
import ReactDOM from 'react-dom';
function reducer(state,action){
  if(action.type === 'add'){
    return state+1;
  }else{
    return state;
  }
}
function Counter(){
  let [state,dispatch] = useReducer(reducer,0);
  return (
    <div>
      <p>{state}</p>
      <button onClick={()=>dispatch({type:'add'})}>+</button>
    </div>
  )

}
function render(){
  ReactDOM.render(
    <Counter/>,
    document.getElementById('root')
  );
}
render();
```

##### useContext,useEffect

```
import React, { useState, useContext, useRef} from "react";
import ReactDOM from "react-dom";
let AppContext = React.createContext();
function Counter() {
  const { state, setState } = useContext(AppContext);
  return (
    <div>
      <p>{state.number}</p>
      <button onClick={() => setState({ number: state.number + 1 })}>+</button>
    </div>
  );
}
function App() {
  let [state, setState] = useState({ number: 0 });
  const ref = useRef(0);
  return (
    <div>
      <div ref={ref}>
        <AppContext.Provider value={{ state, setState }}>
          <Counter />
        </AppContext.Provider>
      </div>
    </div>
  );
}
function render() {
  ReactDOM.render(<App />, document.getElementById("root"));
}
render();

```

实现

```
function useContext(Context){
  return Context._currentValue
}
let lastRef ;
function useRef(initialRef){
  lastRef=lastRef||initialRef;
  return {
    current:lastRef
  }
}
```

##### useEffect,useLayoutEffect

useEffect 和 useLayoutEffect 都是副作用，区别在于 useEffect 不会阻塞渲染会在 render 之后执行，useLayoutEffect 则会阻塞渲染在渲染之前执行

```
import React, { useRef, useLayoutEffect, useEffect } from "react";
import ReactDOM from "react-dom";
function Animation() {
  const ref = useRef();
  // useEffect(() => {
  //   console.log("useLayoutEffect");
  //   ref.current.style.transform = `translate(500px)`;
  //   ref.current.style.transition = "all 800ms";
  // });
  useLayoutEffect(() => {
    console.log("useLayoutEffect");
    ref.current.style.transform = `translate(500px)`;
    ref.current.style.transition = "all 800ms";
  });
  let style = {
    width: "100px",
    height: "100px",
    backgroundColor: "red",
  };
  console.log("Animation ");
  return (
    <div style={style} ref={ref}>
      内容
    </div>
  );
}
function render() {
  ReactDOM.render(<Animation />, document.getElementById("root"));
}
render();

```

上述例子中，可以明显的看到 useEffect 执行会有动画，useLayoutEffect 不会。那么怎么能达到这种效果呢，
先来看张图
![avatar](/images/useLayoutEffect.jpg)
其中 useEffect 在浏览器绘制后执行，而 useLayoutEffect 在绘制前执行，那么怎么做呢？每个组件可以理解成一个事件环，首先执行任务回调中同步代码，执行完毕后，再执行当前回调中的所有微任务，执行宏任务=>下一个任务。其中渲染页面在微任务之前执行，因此我们只需要把 useEffect 中的 callback 编程微任务，而 useLayoutEffect 变成宏任务即可。

```
et lastDependencies;
function useEffect(callback,dependencies){
  if(lastDependencies){
    let changed = !dependencies.every((item,index)=>{
      return item == lastDependencies[index];
    });
    if(changed){
      setTimeout(callback);
      lastDependencies = dependencies;
    }
  }else{
    setTimeout(callback);
    lastDependencies = dependencies;
  }
}
let lastLayoutDependencies;
function useLayoutEffect(callback,dependencies){
  if(lastLayoutDependencies){
    let changed = !dependencies.every((item,index)=>{
      return item == lastLayoutDependencies[index];
    });
    if(changed){
      Promise.resolve().then(callback);
      //queueMicrotask(callback);把callback放到微任务队列中
      lastLayoutDependencies = dependencies;
    }
  }else{//没有渲染过
    Promise.resolve().then(callback);
    //queueMicrotask(callback);把callback放到微任务队列中
    lastLayoutDependencies = dependencies;
  }
}
```
