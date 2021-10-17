# Promise

## promise方法

![img](https://cdn.nlark.com/yuque/0/2020/png/735885/1609225111638-4e78fd87-aa46-4581-ab8c-e7ef354139fd.png)

## promise对象

注意点：

参数resolver必须是Function

Promise实例（this）贯穿全文



**普通new Promise()执行过程**

```javascript
// ! 创建promise对象
function Promise(resolver) {
  // !创建promise id 
  // ? 每执行一次promise就会创建一个promise id
  this[PROMISE_ID] = nextId();
  // !设置 初始状态和结果
  this._result = this._state = undefined;
  // ! 初始化_subscribers属性，说明Promise使用发布订阅模式，间接证明Promise是有异步操作
  this._subscribers = [];
  // ! resolver不为空函数
  if (noop !== resolver) {
    // ! resolver必须为函数Function，否则报错
    typeof resolver !== 'function' && needsResolver();
    // ! 判断this是否来源于Promise构造函数（new Promise），是则调initializePromise函数，否则报错
    this instanceof Promise ? initializePromise(this, resolver) : needsNew();
  }
}
// 报错函数
function needsResolver() {
   throw new TypeError('You must pass a resolver function as the first argument to the promise constructor');
 }

// 报错函数
function needsNew() {
   throw new TypeError("Failed to construct 'Promise': Please use the 'new' operator, this object constructor cannot be called as a function.");
 }

// ! 执行new Promise传入的参数 resolve 或 reject 方法
  // ! 入参是当前promise实例和传入的参数（函数方法或值）
  function initializePromise(promise, resolver) {
    // ? 将resolvePromise函数，rejectPromise函数当初参数传入
    // ?，将执行结果当参数传入
    try {
      resolver(function resolvePromise(value) {
        // ? resolve 执行方法
        resolve$1(promise, value);
      }, function rejectPromise(reason) {
        // ? reject 执行方法
        reject(promise, reason);
      });
    } catch (e) {
      // ? 异常处理
      reject(promise, e);
    }
  }

  // ! 执行完成处理方法
 function resolve$1(promise, value) {
    // ? 当前返回值为promise实例，报错
    if (promise === value) {
      reject(promise, selfFulfillment());
    } else if (objectOrFunction(value)) { // ? 入参为对象或函数
      var then$$1 = void 0;
      try {
        // ? 入参还存在then方法
        then$$1 = value.then;
      } catch (error) {
        // ? 执行错误。抛出异常
        reject(promise, error);
        return;
      }
      // ? 对参数进行进一步处理
      handleMaybeThenable(promise, value, then$$1);
    } else {
      // ? 改变promise状态
      fulfill(promise, value);
    }
  }

	 // ! 入参做进一步处理
  // ! 入参为当前promise实例，执行promise传入的参数，当前参数then方法
  function handleMaybeThenable(promise, maybeThenable, then$$1) {
    // ? 判断当前入参是否与promise实例对象一致，且存在then方法，且存在resolve方法。
    // ? 则根据当前promise状态，执行对应promise方法
    if (maybeThenable.constructor === promise.constructor && then$$1 === then && maybeThenable.constructor.resolve === resolve$$1) {
      handleOwnThenable(promise, maybeThenable);
    } else {
      // ? 不存在then方法，执行结束方法
      if (then$$1 === undefined) {
        fulfill(promise, maybeThenable);
      } else if (isFunction(then$$1)) {
        // ? 存在他很函数，执行trycatch方法
        handleForeignThenable(promise, maybeThenable, then$$1);
      } else {
        // ? 其他情况
        fulfill(promise, maybeThenable);
      }
    }
  }

  // ! 根据当前promise状态执行操作
  function handleOwnThenable(promise, thenable) {
    if (thenable._state === FULFILLED) {
      fulfill(promise, thenable._result);
    } else if (thenable._state === REJECTED) {
      reject(promise, thenable._result);
    } else {
      subscribe(thenable, undefined, function (value) {
        return resolve$1(promise, value);
      }, function (reason) {
        return reject(promise, reason);
      });
    }
  }

// ! 修改promise状态为FULFILLED
  function fulfill(promise, value) {
    if (promise._state !== PENDING) {
      return;
    }

    promise._result = value;
    promise._state = FULFILLED;

    if (promise._subscribers.length !== 0) {
      asap(publish, promise);
    }
  }

// ! 修改promise状态为REJECTED
  function reject(promise, reason) {
    if (promise._state !== PENDING) {
      return;
    }
    promise._state = REJECTED;
    promise._result = reason;

    asap(publishRejection, promise);
  }
```

**普通promise流程图**

![img](https://cdn.nlark.com/yuque/0/2020/png/735885/1609210066181-27e3d023-8ba0-4de0-bc4b-9953cba04d39.png)

**总结：**

**1.初始化了_subscribers属性，说明Promise使用发布/订阅模式，间接证明promise****有异步操作；**

**2.创建对象的时候执行**initializePromise**方法，说明构造函数接收的参数（类型为Function）是同步执行的**

**3.Promise三种状态 Pending， Fulfilled，Rejected，且状态只会改变一次，且不是可逆的**



Promsion then操作方法

```javascript
  // ! Promise then方法
  // ! Promise then方法返回的是一个新的promise对象
  function then(onFulfillment, onRejection) {
    // ? this赋值给parent
    var parent = this;
    // ? 创建新的对象
    var child = new this.constructor(noop);
    // ? 当前promise id为空则创建一个promise id
    if (child[PROMISE_ID] === undefined) {
      makePromise(child);
    }
    // ? 当前promise状态
    var _state = parent._state;

    // ? 如果当前有状态值，则去当前状态的方法，根据状态执行不同promise方法
    if (_state) {
      var callback = arguments[_state - 1];
      asap(function () {
        return invokeCallback(_state, child, callback, parent._result);
      });
    } else {
      // ? 执行队列任务
      subscribe(parent, child, onFulfillment, onRejection);
    }
    // ? 返回创建的对象
    return child;
  }
```

**Promsion catch操作方法**

```javascript
// ! 执行then方法，传入onRejection方法
    Promise.prototype.catch = function _catch(onRejection) {
      return this.then(null, onRejection);
    };
```

**Promsion finally操作方法**

```javascript
// ! finally方法当前promise实例
    Promise.prototype.finally = function _finally(callback) {
      var promise = this;
      var constructor = promise.constructor;
      // ? 如果传入的值是函数，根据是执行完成或错误调用方法
      if (isFunction(callback)) {
        return promise.then(function (value) {
          return constructor.resolve(callback()).then(function () {
            return value;
          });
        }, function (reason) {
          return constructor.resolve(callback()).then(function () {
            throw reason;
          });
        });
      }
      // ? 入参不是函数，根据参数返回执行resolve或reject
      return promise.then(callback, callback);
    };
```

**总结：catch，finally根据入参调用promise.then的方法**



**Promsion all操作方法**

入参promise实例的数组

```javascript
function all(entries) {
   return new Enumerator(this, entries).promise;
} 
var Enumerator = function () {
    // ! promise.all 执行方法
    // ! promise.all 入参当前promise实例（this），promise集合（array）
    function Enumerator(Constructor, input) {
      // ? 赋值this实例
      this._instanceConstructor = Constructor;
      // ? 重新创建新的实例对象
      this.promise = new Constructor(noop);
      // ? 如果没有promise id则创建id
      if (!this.promise[PROMISE_ID]) {
        makePromise(this.promise);
      }
      // ? 入参是数组则执行流程，否则错误reject处理
      if (isArray(input)) {
        // ? 记录promise数组长度
        this.length = input.length;
        // ? 记录还有多少promise未执行，默认都没有执行
        this._remaining = input.length;
        // ? 记录每个promise对象最终的返回值
        this._result = new Array(this.length);

        if (this.length === 0) {
          // ? 空数组，改变promise状态为fulfilled，结束
          fulfill(this.promise, this._result);
        } else {
          this.length = this.length || 0;
          // ? 顺序执行promise
          this._enumerate(input);
          if (this._remaining === 0) { //? 每个promise状态都变为fulfilled，结束
            fulfill(this.promise, this._result);
          }
        }
      } else {
        reject(this.promise, validationError());
      }
    }
    // ! 循环迭代每个promise对象，通过_eachEntry执行每个promise
    Enumerator.prototype._enumerate = function _enumerate(input) {
      for (var i = 0; this._state === PENDING && i < input.length; i++) {
        this._eachEntry(input[i], i);
      }
    };
    // ! 执行promise ,入参为每一个promise和当前下标
    Enumerator.prototype._eachEntry = function _eachEntry(entry, i) {
      // ? 当前promise.all实例（this）,父promise
      var c = this._instanceConstructor;
      // ? 获取当前实例的静态方法resolve
      var resolve = c.resolve;

      // ? 判断静态resolve和源码resolve$$1是否相同
      if (resolve === resolve$$1) { // ? 相同
        var _then = void 0;
        var error = void 0;
        var didError = false;
        try {
          _then = entry.then;
        } catch (e) {
          didError = true;
          error = e;
        }
        // ? 判断当前静态then是否与源码then一致，且状态不为PENDING
        if (_then === then && entry._state !== PENDING) {
          // ? 处理状态为fulfilled或rejected操作
          this._settledAt(entry._state, i, entry._result);
        } else if (typeof _then !== 'function') {
          // ? 当前promise的then不是函数则剩余未执行promise少一，直接返回结果（当前promise的值）
          this._remaining--;
          this._result[i] = entry;
        } else if (c === Promise) { // ? 父promise为promise实例
          var promise = new c(noop);
          if (didError) {   //  ? 执行错误原因
            reject(promise, error);
          } else { // ? 做进一步promise处理
            handleMaybeThenable(promise, entry, _then);
          }
          // ! 存入订阅队列执行promise
          this._willSettleAt(promise, i);
        } else {
          // ! 存入订阅队列执行promise
          this._willSettleAt(new c(function (resolve) {
            return resolve(entry);
          }), i);
        }
      } else { // ? 不同
        this._willSettleAt(resolve(entry), i);
      }
    };
     // ? 处理状态为fulfilled或rejected操作
    Enumerator.prototype._settledAt = function _settledAt(state, i, value) {
      var promise = this.promise;


      if (promise._state === PENDING) {
        this._remaining--;

        if (state === REJECTED) {
          reject(promise, value);
        } else {
          this._result[i] = value;
        }
      }

      if (this._remaining === 0) {
        fulfill(promise, this._result);
      }
    };
    // ! 存入订阅队列执行promise
    Enumerator.prototype._willSettleAt = function _willSettleAt(promise, i) {
      var enumerator = this;

      subscribe(promise, undefined, function (value) {
        return enumerator._settledAt(FULFILLED, i, value);
      }, function (reason) {
        return enumerator._settledAt(REJECTED, i, reason);
      });
    };

    return Enumerator;
  }();
```

**总结：**

**1.all方法的参数是一个promise的数组**

**2.promise.all()返回的是一个promise对象（父promise）。故可以链式的调用then方法**

**3.then(value)参数返回的是一组有序的值，内部会顺序的记录每一个promise对象的状态和值**

**4.所有promise对象改变后，触发****父promise状态更新**



**Promsion race操作方法**

```javascript
function race(entries) {
    /*jshint validthis:true */
    var Constructor = this;
    // ? 入参不是数组则直接报错结束
    if (!isArray(entries)) {
      return new Constructor(function (_, reject) {
        return reject(new TypeError('You must pass an array to race.'));
      });
    } else {
      // ? 循环执行promise。每执行一个promise就改变父promise（race返回的promise对象）
      return new Constructor(function (resolve, reject) {
        var length = entries.length;
        for (var i = 0; i < length; i++) {
          Constructor.resolve(entries[i]).then(resolve, reject);
        }
      });
    }
  }
```

**总结：**

**1. promise.race入参是一个数组，如果不是数组则状态变为rejected，报错直接返回**

**2.循环遍历执行传入的promise，每执行就改变父promise（****promise.race返回的promise对象****）**



**Promsion resolve操作方法**

```javascript
function resolve$$1(object) {
    /*jshint validthis:true */
    var Constructor = this;
    // ? 如果入参是object且属性与promise的属性一致，直接返回入参
    if (object && typeof object === 'object' && object.constructor === Constructor) {
      return object;
    }
    // ? 创建新的实例对象，执行resolve$1，更新promise状态，返回内部promise对象
    var promise = new Constructor(noop);
    resolve$1(promise, object);
    return promise;
  }
```

**Promsion reject操作方法**

```javascript
 // ! 创建对象，内部调用reject方法，改变状态为rejected状态。返回内部创建的promise对象
  function reject$1(reason) {
    /*jshint validthis:true */
    var Constructor = this;
    var promise = new Constructor(noop);
    reject(promise, reason);
    return promise;
  }
```

**Promsion asap操作方法**

```javascript
var asap = function asap(callback, arg) {
    queue[len] = callback;
    queue[len + 1] = arg;
    len += 2;
    if (len === 2) {

        //如果len为2，则意味着我们需要安排异步刷新。
        //如果在刷新队列之前将其他回调排队在队列中，则它们
        //将由我们计划的刷新处理
      // If len is 2, that means that we need to schedule an async flush.
      // If additional callbacks are queued before the queue is flushed, they
      // will be processed by this flush that we are scheduling.
      if (customSchedulerFn) {
        customSchedulerFn(flush);
      } else {
        scheduleFlush();
      }
    }
  };
```

**总结：**

- **函数接收两个参数**

- - **callback 回调函数**
  - **arg 参数**

- **参数存入队列**
- **判断 len 长度**

- - **这里判断**`**len===2**`**的原因在于初始化**`**queue**`**数组长度为 1000 但值为**`**undefined**`**,将参数存入数组并判断长度只为确定数组内有值(可立即执行的函数).**

- **customSchedulerFn**

- - **对开发者提供了**`**setScheduler**`**方法对**`**customSchedulerFn**`**赋值实现定制化调用队列,(假如在此次准备开始调用队列之前还有其它事情要处理完成之后才可以调用,则可以实现** `**customSchedulerFn**` **函数)这里不做过多说明.**

- **scheduleFlush**

- - **这个函数针对不同的执行环境(Node、浏览器)等做了处理,由于我们在浏览器端执行,所以直接看**`**useMutationObserver**`**函数.**