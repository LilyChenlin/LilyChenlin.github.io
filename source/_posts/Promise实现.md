---
title: Promise实现
date: 2022-03-15 22:26:58
update: 2022-03-15 22:26:58
tags:
---
# 1. 初始结构

## 原生

```javascript
let promise1 = new Promise((resolve, reject) => {
  
})
```

- 传入func，func的参数分别是resolve, reject
<!--more-->
## 实现

```javascript
class myPromise {
  constructor(func) {
    func(this.resolve, this.reject)
  }
  resolve(){}
  reject(){}
}
```

# 2. 状态

1. 共有三种状态，分别是
   - Pending
   - Fulfilled
   - Rejected

## 实现

```javascript
class MyPromise {
    static PENDING = 'pending';
    static FULFILLED = 'fulfilled';
    static REJECTED = 'rejected';
    constructor(func) {
        this.status = MyPromise.PENDING;
        func(this.resolve, this.reject);
    }
    resolve() {
        if (this.status === MyPromise.PENDING) {
            this.status = MyPromise.FULFILLED;
        }
    }
    reject() {
        if (this.status === MyPromise.PENDING) {
            this.status = MyPromise.REJECTED;
        }
    }
}

```

> static关键字

1. static定义静态方法/静态属性

2. 类相当于实例的原型，实例能够继承原型上的方法和属性；

3. 加上static关键字，方法或者属性就不会被实例继承，需要通过类来调用。

### 执行

```javascript
let promise1 = new MyPromise((resolve, reject) => {
    resolve();
})
```

![1.png](1.png)

现象：报错Cannot read property 'status' of undefined

原因: 
1. 首先，我们在创建新实例的时候确实绑定了this指向

2. 但是，我们是在新实例被创建后再在外部环境下执行resolve方法的，我们可以在resolve打印一下this指向

![2.png](2.png)

可以看到，此时this指向undefined

解决办法：绑定this值，可以通过bind, 箭头函数。

```javascript
class MyPromise {
		...
    constructor(func) {
				....
        func(this.resolve.bind(this), this.reject.bind(this));
    }
		...
}
```

很好，此时this指向对了。

# 3. 传参数

## 原生

1. 原生的promise可以给resolve/reject回调传入参数，并且在then中可以接收到。

```javascript
let promise1 = new Promise((resolve, reject) => {
    resolve('42');
})
```

## 实现

1. 在constructor构造函数中定义变量result，用来存储传进来的参数

```javascript
class MyPromise {
		...
    constructor(func) {
      ...
        this.result = null;
      ...
    }
    resolve(result) {
        if (this.status === MyPromise.PENDING) {
						...
            this.result = result;
        }
    }
    reject(result) {
        if (this.status === MyPromise.PENDING) {
						...
            this.result = result;
        }
    }
}
```

# 4. then

## 原生

```javascript
let promise1 = new Promise((resolve, reject) => {
    resolve('42');
})
promise1.then(
    result => { console.log(result) },
    result => {console.log(result)}
)
```

1. 原生的then支持传入两个回调函数
2. 但是只能执行其中一个回调函数，如果状态是fulfilled，执行第一个回调函数；否则，执行第二个回调函数

## 实现

1. 定义then函数，传入两个回调函数

```javascript
class MyPromise {
		....
    then(onFulFILLED, onREJECTED) {
        if (this.status === MyPromise.FULFILLED) {
            onFulFILLED(this.result)
        }
        if (this.status === MyPromise.REJECTED) {
            onREJECTED(this.result)
        }
    }
}
```

# 5. 执行异常

## 异常一

### 原生

```javascript
let promise1 = new Promise((resolve, reject) => {
    throw new Error('执行异常')
})
promise1.then(
    result => { console.log(result) },
    result => {console.log(result.message)}
)

// 执行异常
```

1. 在new Promise时抛出错误，throw new Error('...')；在then中的第二个回调函数中可以catch到这个错误

### 实现

#### 目前效果

同样的，我们可以在我们现有的实现代码中抛出相同异常，看看会发生什么？

![3.png](3.png)

#### **解决办法**

在构造函数中添加try...catch，如果没有错误就走正常的func，如果有错误就直接走reject方法

```javascript
class MyPromise {
		...
    constructor(func) {
        this.status = MyPromise.PENDING;
        this.result = null;
        try {
            func(this.resolve.bind(this), this.reject.bind(this));
        } catch(err) {
            this.reject(err)
        }
    }
		...
}
```

**执行**

![4.png](4.png)

## 异常二

### 原生

1. 原生Promise规定then里面的两个参数如果不是函数的话就要被忽略

```javascript
let promise1 = new Promise((resolve, reject) => {
    resolve('42')
})
promise1.then(
    undefined,
    result => {console.log(result.message)}
)
```

由于第一个参数传入的undefined,根据规定，这个参数被忽略。所以控制台没有任何输出

### 实现

#### 目前效果

尝试在我们目前的实现代码中传入undefeind，是会报错的，如下图所示

![5.png](5.png)

#### **解决办法**

默认将onFulFILLED和onREJECTED处理成一个函数

```javascript
class MyPromise {
  ...
    then(onFulFILLED, onREJECTED) {
        onFulFILLED = typeof onFulFILLED === 'function' ? onFulFILLED : () => { };
        onREJECTED = typeof onREJECTED === 'function' ? onREJECTED : () => { };
		...
    }
}
```

此时，就能和原生一样了，如果传入的参数不是函数的话，就忽略。

# 6. 异步

## 关注点一

### 原生

首先，我们先来了解一下原生Promise的执行机制。先来看一段代码：

```javascript
console.log('1');
let promise1 = new Promise((resolve, reject) => {
    console.log('2');
    resolve('哈哈')
})
promise1.then(
    result => { console.log(result) },
    result => {console.log(result.message)}
)
console.log('3')
```

>  这里关系到事件执行机制，不懂的可以去了解一下这方面的知识。

执行顺序如下：

```javascript
1
2
3
哈哈
```

可以看到，我们先执行同步任务，所以输出1，2，3；然后执行异步任务，输出哈哈；

### 实现

#### 目前效果

让我们在目前的实现代码中执行该段代码，

![6.png](6.png)

和原生不一样呢，它并没有异步的feel～

#### **解决办法**

1. 引入setTimeout让我们的Promise.then变成异步来模仿原生Promise的执行效果把～

```javascript
class MyPromise {
		...
    then(onFulFILLED, onREJECTED) {
				...
        if (this.status === MyPromise.FULFILLED) {
            setTimeout(() => {
                onFulFILLED(this.result)
            });
            
        }
        if (this.status === MyPromise.REJECTED) {
            setTimeout(() => {
                onREJECTED(this.result)
            });
            
        }
    }
}
```

2. 还是执行上面的代码

![7.png](7.png)

哇唔～现在我们可以异步输出哈哈啦～

## 关注点二

### 原生

如果我们在new Promise中的resolve回调外面加入一个setTimeout呢？此时原生Promise的执行效果会是什么样的？

```javascript
console.log('1');
let promise1 = new Promise((resolve, reject) => {
    console.log('2');
    setTimeout(() => {
        console.log('4');
        resolve('哈哈')
        console.log('5')
    });
})
promise1.then(
    result => { console.log(result) },
    result => {console.log(result.message)}
)
console.log('3')
```

输出结果

```javascript
1
2
3
4
5
哈哈
```

### 实现

#### 目前效果

让我们继续在实现代码中执行以上的代码看看输出顺序吧～

![8.png](8.png)

咦～哈哈哪里去了？？

#### **分析原因**

“哈哈”没有被输出原因 -> then方法可能没有被执行 -> then方法中的状态可能没有被改变 -> 因此，让我们输出一下状态看看吧

![9.png](9.png)

1. 在构造函数中，status有发生了变化，并且从pending -> fulfilled；

2. 在then函数中，没有调用status === 'fulfilled'的回调函数

   1. 在事件循环机制中，如果我们不考虑现有的微任务的执行机制，单纯从代码来看，promise1.then方法的执行会早于setTimeout；也就是在resolve执行之前即status还没有发生变化的时候，就执行的then方法。
   2. 我们可以通过在实现代码的then方法中输出一下，就可以看到执行顺序

   ```javascript
   1
   2
   执行了then方法,此时的status状态为 pending
   3
   pending
   4
   fulfilled
   5
   ```

   3. 所以尽管setTimeout中状态发生了变化到fulfilled，但是then方法已经先一步执行了，所以无法输出“哈哈”

#### 解决方法

1. 我们需要在status === 'PENDING'时，做一些判断
   1. 此时，我们执行then方法的时候，status为PENDING状态，所以我们可以将回调函数通过数组存储起来
   2. 当我们在setTimeout中调用resolve时，可以遍历数组，如果数组中有值，我们遍历调用数组中存储的回调函数callback

```javascript
class MyPromise {
  	...
    constructor(func) {
				...
        this.resolveCallBack = [];
        this.rejectCallBack = [];
				...
    }
    resolve(result) {
        if (this.status === MyPromise.PENDING) {
            this.status = MyPromise.FULFILLED;
            this.result = result;
            this.resolveCallBack.forEach(callback => {
                callback(result);
            })
        }
    }
    reject(result) {
        if (this.status === MyPromise.PENDING) {
            this.status = MyPromise.REJECTED;
            this.result = result;
        }
        this.rejectCallBack.forEach(callback => {
            callback(result);
        })
    }
}
```

此时，我们查看一下输出

![10.png](10.png)

现在，我们可以正常输出“哈哈”了。但是，我们的“哈哈”位置显然是不对的，正常情况下，他应该在5后面才输出的。

回顾实现代码，我们可以在resove和reject方法中使用setTimeout来保证输出机制

```javascript
class MyPromise {
  	...
    constructor(func) {
				...
        this.resolveCallBack = [];
        this.rejectCallBack = [];
				...
    }
    resolve(result) {
        setTimeout(() => {
            if (this.status === MyPromise.PENDING) {
                this.status = MyPromise.FULFILLED;
                this.result = result;
                this.resolveCallBack.forEach(callback => {
                    callback(result);
                })
            }
        });
    }
    reject(result) {
        setTimeout(() => {
            if (this.status === MyPromise.PENDING) {
                this.status = MyPromise.REJECTED;
                this.result = result;
            }
            this.rejectCallBack.forEach(callback => {
                callback(result);
            })
        });

    }
}
```

现在，输出顺序如下

![11.png](11.png)

# 7. 链式

## 原生

我们知道， 在原生Promise中，Promise.then方法会返回一个then，可以进行链式调用

```javascript
let promise1 = new Promise((resolve, reject) => {
    resolve('哈哈')
})
promise1.then(
    result => { console.log(result)},
    result => {console.log(result.message)}
).then(
    result => { console.log('链式调用');},
    result => { console.log(result.message) }
)
```

输出

```javascript
哈哈
链式调用
```

## 实现

### 目前效果

![12.png](12.png)

报错了，是因为我们的then方法并没有返回then方法，因此我们可以在then方法中return一个MyPromise，MyPromise有then方法，就可以实现链式调用了。

```javascript
class MyPromise {
  ....
    then(onFulFILLED, onREJECTED) {
        return new MyPromise((resolve, reject) => {
            onFulFILLED = typeof onFulFILLED === 'function' ? onFulFILLED : () => { };
            onREJECTED = typeof onREJECTED === 'function' ? onREJECTED : () => { };
					...
        })

    }
}
```

这样就可以进行链式调用了，但是链式调用中的then并不能正常输出，还有很多细节～ 慢慢实现

# 总结

在这里，我从初始结构 -> 状态 -> 传参 -> then函数 -> Promise中异常处理 -> 异步 -> 链接来实现了简单版的Promise，虽然还有很多细节没有实现，但是收获很大。