---
title: 写一个Promise
date: 2019-03-05 19:51:41
tags: JavaScript
---

## 序言

JS和Node.js通过事件驱动、事件循环实现了异步编程，提高了资源利用率，提高了性能，但也因此带来了异步编程的难点。为了解决异步编程带来的难点，社区出了好几个解决方案，事件发布/订阅模式、Promise/Deferred模式、流程控制库，最终Promise成为了ES6中一个解决异步编程难点的API，再配合ES2017中Async/Await真正解决异步编程的难点。

<!-- more -->

## 介绍Promise

### Promise的含义

所谓Promise，就是一个对象，用来传递异步操作的消息。它代表了某个未来才会知道结果的事件（通常是一个异步操作），并且这个事件提供统一的API，可供进一步处理。

Promise有两个特点:

1. 对象的状态不受外界影响。Promise对象代表一个异步操作，有3种状态：Pending(进行中)，Resolved(已完成，又称Fuilfilled)和Rejected(已失败)。只有异步操作的结果可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
2. 一旦状态改变就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变只有两种可能，从Pending变为Resolved和从Pending变为Rejected。只要其中之一发生，状态就凝固了，不会再变，会一直保持这个结果。

### Promise基本用法

ES6规定，Promise对象是一个构造函数，用来生成Promise实例。

```js
const promise = new Promise(function () {
	// ... some code
	
	if (/* 异步操作成功 */) {
		resolve(value);
	} else {
		reject(error);
	}
});
promise.then(function (value) {
	//success
}, function (value) {
	//failure
});
```

## 实现Promise

### 基于事件发布/订阅实现

```js
const EventEmitter = require('events');
class Promise extends EventEmitter {
	constructor (handler) {
		super();
		this.state = 'pendding';
		this.resolve = this.resolve.bind(this);
		this.reject = this.reject.bind(this)
		if (typeof handler == 'function') {
			handler(this.resolve, this.reject);
		}
	}

	resolve (obj) {
		if (this.state == 'pendding') {
			this.state = 'resolved';
			this.emit('resolved', obj);
		}
	}

	reject (err) {
		if (this.state == 'pendding') {
			this.state = 'rejected';
			this.emit('rejected', err);
		}
	}

	then (resolvedHandler, rejectedHandler) {
		if (typeof resolvedHandler == 'function') {
			this.once('resolved', resolvedHandler);
		}

		if (typeof rejectedHandler == 'function') {
			this.once('rejected', rejectedHandler);
		}
		return this;
	}

	catch (rejectedHandler) {
		if (typeof rejectedHandler == 'function') {
			this.once('rejected', rejectedHandler);
		}
	}
}
```

### 实现链式调用

```js
// 三种状态
const PENDING = "pending";
const RESOLVED = "resolved";
const REJECTED = "rejected";
// promise 接收一个函数参数，该函数会立即执行
function MyPromise(fn) {
  let _this = this;
  _this.currentState = PENDING;
  _this.value = undefined;
  // 用于保存 then 中的回调，只有当 promise
  // 状态为 pending 时才会缓存，并且每个实例至多缓存一个
  _this.resolvedCallbacks = [];
  _this.rejectedCallbacks = [];

  _this.resolve = function (value) {
    if (value instanceof MyPromise) {
      // 如果 value 是个 Promise，递归执行
      return value.then(_this.resolve, _this.reject)
    }
    setTimeout(() => { // 异步执行，保证执行顺序
      if (_this.currentState === PENDING) {
        _this.currentState = RESOLVED;
        _this.value = value;
        _this.resolvedCallbacks.forEach(cb => cb());
      }
    })
  };

  _this.reject = function (reason) {
    setTimeout(() => { // 异步执行，保证执行顺序
      if (_this.currentState === PENDING) {
        _this.currentState = REJECTED;
        _this.value = reason;
        _this.rejectedCallbacks.forEach(cb => cb());
      }
    })
  }
  // 用于解决以下问题
  // new Promise(() => throw Error('error))
  try {
    fn(_this.resolve, _this.reject);
  } catch (e) {
    _this.reject(e);
  }
}

MyPromise.prototype.then = function (onResolved, onRejected) {
  var self = this;
  // 规范 2.2.7，then 必须返回一个新的 promise
  var promise2;
  // 规范 2.2.onResolved 和 onRejected 都为可选参数
  // 如果类型不是函数需要忽略，同时也实现了透传
  // Promise.resolve(4).then().then((value) => console.log(value))
  onResolved = typeof onResolved === 'function' ? onResolved : v => v;
  onRejected = typeof onRejected === 'function' ? onRejected : r => throw r;

  if (self.currentState === RESOLVED) {
    return (promise2 = new MyPromise(function (resolve, reject) {
      // 规范 2.2.4，保证 onFulfilled，onRjected 异步执行
      // 所以用了 setTimeout 包裹下
      setTimeout(function () {
        try {
          var x = onResolved(self.value);
          resolutionProcedure(promise2, x, resolve, reject);
        } catch (reason) {
          reject(reason);
        }
      });
    }));
  }

  if (self.currentState === REJECTED) {
    return (promise2 = new MyPromise(function (resolve, reject) {
      setTimeout(function () {
        // 异步执行onRejected
        try {
          var x = onRejected(self.value);
          resolutionProcedure(promise2, x, resolve, reject);
        } catch (reason) {
          reject(reason);
        }
      });
    }));
  }

  if (self.currentState === PENDING) {
    return (promise2 = new MyPromise(function (resolve, reject) {
      self.resolvedCallbacks.push(function () {
        // 考虑到可能会有报错，所以使用 try/catch 包裹
        try {
          var x = onResolved(self.value);
          resolutionProcedure(promise2, x, resolve, reject);
        } catch (r) {
          reject(r);
        }
      });

      self.rejectedCallbacks.push(function () {
        try {
          var x = onRejected(self.value);
          resolutionProcedure(promise2, x, resolve, reject);
        } catch (r) {
          reject(r);
        }
      });
    }));
  }
};
// 规范 2.3
function resolutionProcedure(promise2, x, resolve, reject) {
  // 规范 2.3.1，x 不能和 promise2 相同，避免循环引用
  if (promise2 === x) {
    return reject(new TypeError("Error"));
  }
  // 规范 2.3.2
  // 如果 x 为 Promise，状态为 pending 需要继续等待否则执行
  if (x instanceof MyPromise) {
    if (x.currentState === PENDING) {
      x.then(function (value) {
        // 再次调用该函数是为了确认 x resolve 的
        // 参数是什么类型，如果是基本类型就再次 resolve
        // 把值传给下个 then
        resolutionProcedure(promise2, value, resolve, reject);
      }, reject);
    } else {
      x.then(resolve, reject);
    }
    return;
  }
  // 规范 2.3.3.3.3
  // reject 或者 resolve 其中一个执行过得话，忽略其他的
  let called = false;
  // 规范 2.3.3，判断 x 是否为对象或者函数
  if (x !== null && (typeof x === "object" || typeof x === "function")) {
    // 规范 2.3.3.2，如果不能取出 then，就 reject
    try {
      // 规范 2.3.3.1
      let then = x.then;
      // 如果 then 是函数，调用 x.then
      if (typeof then === "function") {
        // 规范 2.3.3.3
        then.call(
          x,
          y => {
            if (called) return;
            called = true;
            // 规范 2.3.3.3.1
            resolutionProcedure(promise2, y, resolve, reject);
          },
          e => {
            if (called) return;
            called = true;
            reject(e);
          }
        );
      } else {
        // 规范 2.3.3.4
        resolve(x);
      }
    } catch (e) {
      if (called) return;
      called = true;
      reject(e);
    }
  } else {
    // 规范 2.3.4，x 为基本类型
    resolve(x);
  }
}
```