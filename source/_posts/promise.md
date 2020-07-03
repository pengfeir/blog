---
title: 聊聊promise
date: 2020-07-03 11:13:18
description: promise实现
tags: promise
categories: promise
---

### promise 的作用

promise 我们异步编程经常在使用，主要用来解决

- 多个异步串行问题，链式调用（还是基于回调）
  ```
  ajax(v=>{
    ajax(v,t=>{
      ...
    })
  })
  ```
- 多个异步并发，同时拿到结果 promise.all

### promise 的特性

- promise 有三个状态 resolve reject pending ，其中只有等待状态才可以改成成功或者失败
- 每个 promise 需提供一个 executor，这个函数立即执行，会返回一个 promise 实例
  举个例子

```
new Promise(() => {
  console.log("执行");
});
```

其中

```
() => {
  console.log("执行");
}
```

就是 executor，会立即执行，只有 then 才是异步

- executor 上有成功(resolve)和失败的方法(reject)
- new promise 返回 promise 实例，实例上存在 then 方法

#### 简易版 promise

基于上面 promise 那些特性，我们来实现一个(promise/A+规范)[https://promisesaplus.com/]

##### 1.同步 promise

```
const PENDING = "PENDING";
const RESOLVE = "RESOLVE";
const REJECT = "REJECT";
class Promise {
  constructor(executor) {
    this.value = undefined;
    this.status = PENDING;
    this.reason = undefined;
    let resolve = (value) => {
      if (this.status === PENDING) {
        this.status = RESOLVE;
        this.value = value;
      }
    };
    let reject = (reason) => {
      if (this.status === PENDING) {
        this.status = REJECT;
        this.value = reason;
      }
    };
    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }
  then(onFulfilled, onReject) {
    if (this.status === RESOLVE) {
      onFulfilled(this.value);
    }
    if (this.status === REJECT) {
      onReject(this.value);
    }
  }
}
//测试一下
new Promise((resolve, reject) => {
  // resolve('成功了');
  // reject("失败了");
  throw new error("22");
}).then(
  (v) => {
    console.log("成功", v);
  },
  (err) => {
    console.log("失败", err);
  }
);

```

#### 2.异步 promise

可以看出来，上述代码虽然实现了 promise 最基础的功能，但是不支持异步，要支持异步就用到了我们的发布订阅者模式，实现如下

```
const PENDING = "PENDING";
const RESOLVE = "RESOLVE";
const REJECT = "REJECT";
class Promise {
  constructor(executor) {
    this.value = undefined;
    this.status = PENDING;
    this.reason = undefined;
    //发布订阅者模式来监听异步数据的返回
    this.onResolvedCallback = [];
    this.onRejectedCallback = [];
    let resolve = (value) => {
      if (this.status === PENDING) {
        this.status = RESOLVE;
        this.value = value;
        this.onResolvedCallback.forEach((fn) => fn());
      }
    };
    let reject = (reason) => {
      if (this.status === PENDING) {
        this.status = REJECT;
        this.value = reason;
        this.onRejectedCallback.forEach((fn) => fn());
      }
    };
    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }
  then(onFulfilled, onReject) {
    if (this.status === RESOLVE) {
      onFulfilled(this.value);
    }
    if (this.status === REJECT) {
      onReject(this.value);
    }
    if (this.status === PENDING) {
      this.onResolvedCallback.push(() => {
        onFulfilled(this.value);
      });
      this.onRejectedCallback.push(() => {
        onRejected(this.value);
      });
    }
  }
}
//异步
new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve("成功了");
  }, 1000);
}).then(
  (v) => {
    console.log("成功", v);
  },
  (err) => {
    console.log("失败", err);
  }
);
```

至此一个简易版的 promise 就实现，但是此 promise 并不支持链式调用。

#### 完善版 promise

#### 1.链式调用

```
const RESOLVE = "RESOLVE";
const REJECT = "REJECT";
const PENDING = "PENDING";
const resolvePromise = (promise2, x, resolve, reject) => {
  if (promise2 === x) {
    //防止resolve自身造成死循环
    return reject(new TypeError("Chaining cycle detected for promise"));
  }
  let called;
  //判断x是promise还是普通值
  if ((typeof x == "object" && x !== null) || typeof x === "function") {
    //判断一个对象是promise是then方法
    try {
      let then = x.then;
      if (typeof then === "function") {
        //有then说明是promise
        then.call(
          x,
          (y) => {
            if (called) {
              return;
            }
            called = true;
            resolvePromise(promise2, y, resolve, reject); //解析y保证是普通值
          },
          (r) => {
            if (called) {
              return;
            }
            called = true;
            reject(r);
          }
        );
      } else {
        resolve(x);
      }
    } catch (err) {
      if (called) {
        return;
      }
      called = true;
      reject(then);
    }
  } else {
    //普通值
    resolve(x);
  }
};
class Promise {
  constructor(executor) {
    this.status = PENDING;
    this.value = undefined;
    this.reason = undefined;
    //发布订阅，用来解决异步回调
    this.onResolvedCallback = [];
    this.onRejectedCallback = [];
    let resolve = (value) => {
      if (this.status === PENDING) {
        this.value = value;
        this.status = RESOLVE;
        this.onResolvedCallback.forEach((fn) => fn());
      }
    };
    let reject = (reason) => {
      if (this.status === PENDING) {
        this.reason = reason;
        this.status = REJECT;
        this.onRejectedCallback.forEach((fn) => fn());
      }
    };
    try {
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }
  then(onFulfilled, onRejected) {
    //onFulfilled和onRejected是非必传的，但链式调用是支持参数传递的，因此要在onFulfilled，onRejected没传的时候把参数传递下去
    onFulfilled =
      typeof onFulfilled === "function" ? onFulfilled : (val) => val; //promise.then().then().then(val=>console.log(val))解决多个then不传onFulfilled
    onRejected =
      typeof onRejected === "function"
        ? onRejected
        : (error) => {
            throw error;
          };
    //promise.then().then().then(null,error=>console.log(error))解决多个then不传onFulfilled
    let promise2 = new Promise((resolve, reject) => {
      if (this.status === RESOLVE) {
        //加入setTimeout是为了把promise加入下一个任务，让其内部能拿到promise2
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value);
            resolvePromise(promise2, x, resolve, reject);
          } catch (err) {
            reject(err);
          }
        }, 0);
      }
      if (this.status === REJECT) {
        setTimeout(() => {
          try {
            let x = onRejected(this.reason);
            resolvePromise(promise2, x, resolve, reject);
          } catch (err) {
            reject(err);
          }
        }, 0);
      }
      if (this.status === PENDING) {
        this.onResolvedCallback.push(() => {
          setTimeout(() => {
            try {
              let x = onFulfilled(this.value);
              resolvePromise(promise2, x, resolve, reject);
            } catch (err) {
              reject(err);
            }
          }, 0);
          let x = onFulfilled(this.value);
        });
        this.onRejectedCallback.push(() => {
          setTimeout(() => {
            try {
              let x = onRejected(this.reason);
              resolvePromise(promise2, x, resolve, reject);
            } catch (err) {
              reject(err);
            }
          }, 0);
        });
      }
    });
    return promise2;
  }
  catch(errCb) {
    //没有传成功的then方法
    return this.then(null, errCb);
  }
}
```

#### Promise 上方法的实现

##### 1.Promise.resolve

返回一个成功解析的 promise

```
Promise.resolve= function(val){
  return new Promise((resolve)=>{
    resolve(val)
  })
}
Promise.resolve(10).then((v) => {
  console.log(v);//10
});

```

##### 2.Promise.reject

```
Promise.reject = function (reason) {
  return new Promise((null, reject) => {
    reject(reason);
  });
};
Promise.reject(10).then((v) => {
  console.log(v);//10
});

```

##### 3.Promise.defer

延迟方法,用来吧异步方法编程 promise 对象

```
Promise.defer = function () {
  let dfd = {};
  dfd.promise = new Promise((resolve, reject) => {
    dfd.resolve = resolve;
    dfd.reject = reject;
  });
  return dfd;
};
function getData() {
   let dfd = Promise.defer();
   ajax(function (data,err) {
     if (err) {
      dfd.reject(err);
    }
    dfd.resolve(data);
   });
   return dfd.promise;
}
getData.then(v=>{
  console.log(v)
})
```

##### 4.Promise.all

并发，可以传普通值，方法返回一个 Promise 实例，此实例在 iterable 参数内所有的 promise 都“完成（resolved）”或参数中不包含 promise 时回调完成（resolve）；如果参数中 promise 有一个失败（rejected），此实例回调失败（reject），失败的原因是第一个失败 promise 的结果。

```
function isPromise(promise) {
  return typeof promise.then === "function";
}
Promise.all = function (promises) {
  return new Promise((resolve, reject) => {
    let ids = 0;//计算接口成功次数
    let result = [];
    const processData = (data, i) => {
      result[i] = data;
      if (++ids === promises.length) {
        console.log("result", result);
        resolve(result);
      }
    };
    for (let i = 0; i < promises.length; i++) {
      let curPromise = promises[i];
      if (isPromise(curPromise)) {
        curPromise.then(
          (v) => {
            processData(v, i);
          },
          (err) => reject(err)
        );
      } else {
        processData(curPromise, i);
      }
    }
  });
};

let b = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(100);
  }, 1000);
});
let a = 10;
Promise.all([b, a]).then((v) => {
  console.log("v", v);//100，10
});
```

##### 5.finally

finally() 方法返回一个 Promise。在 promise 结束时，无论结果是 fulfilled 或者是 rejected，都会执行指定的回调函数。这为在 Promise 是否成功完成后都需要执行的代码提供了一种方式。
这避免了同样的语句需要在 then()和 catch()中各写一次的情况。

```
//finally参数是回调
Promise.prototype.finally = function (cb) {
  return this.then(
    (data) => {
      return Promise.resolve(cb()).then((data) => data);
    },
    (err) => {
      return Promise.resolve(cb()).then((err) => err);
    }
  );
};

Promise.resolve(100)
  .then((v) => {
    console.log("c", v);
  })
  .finally(() => {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        console.log("a");
        resolve(1);
      }, 3000);
    });
  });//100，a
Promise.resolve(200)
  .finally(() => {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        console.log("b");
        resolve(2);
      }, 3000);
    });
  })
  .then((v) => {
    console.log("c", v);
  });// b,200
```
