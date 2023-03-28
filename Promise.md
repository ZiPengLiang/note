# 初步了解Promise

> ​	Promise是一个对象，用于表示一个异步请求操作的最终结果（不管是成功和失败），并将其结果值返回出去，在当前的javascript程序中，promise用法众多，是一种十分流行的异步编程方式



### 语法

```javascript
 new Promise(function(resolve,reject){
	if(...){
       resolve()
    }else{
        reject()
    }
})
```

​		用法：

在创建Promise时能通过传入一个带有resolve和reject参数的callback函数，并在Promise执行时立即执行，resolve和reject两个函数作为参数传到callback函数中，并起到修改promise状态的作用。

​	resolve():将promise的状态改为fulfilled(完成状态);

​	reject():将promise的状态改为rejected(失败状态)



顺带一提

​	**Promise** 对象是一个代理对象（代理一个值），被代理的值在Promise对象创建时可能是未知的。它允许你为异步操作的成功和失败分别绑定相应的处理方法（handlers）。 这让异步方法可以像同步方法那样返回值，但并不是立即返回最终执行结果，而是一个能代表未来出现的结果的promise对象

​	Promise有3个状态

​		1.`pengding`：初始状态，既不是成功，也不是失败

​		2.`fulfilled`：代表操作成功

​		3.`rejected`：代表操作失败

​	当promise返回任意一种结果的时候，Promise对象的then方法绑定的处理方法就会被执行。

​	重点：`Promise.prototype.then` 和`Promise.prototype.catch` 返回的都是Promise对象，因此他们可以被链式调用

![](Promise/promises.png)



### 方法

#### 	1、**Promise.all(iterable)**

​		**Promise.all(iterable)**方法返回一个Promise实例，该实例在iterable参数内所有的promise都完成（resolved）或参数中不包含 `promise` 时回调完成（resolve）；如果参数中  `promise` 有一个失败（rejected），此实例回调失败（reject），失败原因的是第一个失败 `promise` 的结果。

```javascript
//一个成功的回掉
var p1 = Promise.resolve(3);
//一个String类型
var p2 = 4;
//异步完成的操作
var p3 = new Promise((resolve,reject)=>{
    setTimeout(resolve,100,'foo')
})
//一个异步失败的操作
var p4 =  Promise.reject(5)
//所有参数都返回完成状态
Promise.all([p1,p2,p3]).then(res=>{
    输出结果
    console.log(res)
})
//[3,4,'foo']

//p4返回一个错误，则立即停止操作并将错误返回
Promise.all([p1,p2,p3,p4]).then(res=>{
    console.log(res)
}).catch(err=>{
    //通过catch捕获错误，并将其返回输出出来
    console.log(err)
})
//5


```



#### 		2、**Promise.race(iterable)**

​	当iterable参数里的任意一个子promise被成功或失败后，父promise马上也会用子promise的成功返回值或失败详情作为参数调用父promise绑定的相应句柄，并返回该promise对象。

```javascript
let p1 = new Promise((resolve,reject)=>{
	setTimeout(()=>{
        resolve('one')
    },1000)
})
let p2 = new Promise((resolve,reject)=>{
	setTimeout(()=>{
        resolve('two')
    },5000)
})
let p3 = new Promise((resolve,reject)=>{
	setTimeout(()=>{
        reject('three')
    },2000)
})
let p4 = new Promise((resolve,reject)=>{
	setTimeout(()=>{
        reject('four')
    },1000)
})
let p5 = new Promise((resolve,reject)=>{
	setTimeout(()=>{
        reject('five')
    },500)
})
let p6 = new Promise((resolve,reject)=>{
	setTimeout(()=>{
        reject('six')
    },200)
})

Promise.race([p1,p2]).then(res=>{
	console.log(res)
})

Promise.race([p1,p3]).then(res=>{
	console.log(res)
}).catch(err=>{
	console.log(err)
})
Promise.race([p1,p4]).then(res=>{
	console.log(res)
}).catch(err=>{
	console.log(err)
})
Promise.race([p4,p5]).then(res=>{
	console.log(res)
}).catch(err=>{
	console.log(err)
})
Promise.race([p5,p6]).then(res=>{
	console.log(res)
}).catch(err=>{
	console.log(err)
})
//one
//one
//one
//four
//five (bug)
```

在运行Promise.race()时往往会出现一些问题，就是由于2个promise运行时间过短，间隔过小，导致会出现像第五个结果的情况。

由于javascript是单线程运作的，系统是从上往下执行代码的，当系统遇到异步操作的时候，会将异步操作的代码放到一个类似于时间队列的地方，并按异步操作代码所执行的时间由小到大进行排列，当主线程的栈中的任务已经执行完毕后，系统会将时间队列中的任务一一推进主线程的栈中执行，这操作就关系到了javascript的事件循环机制。

至于为什么会出现像第五个结果的情况呢，因为两者执行的时间过于短，导致p5比p6先一步存进队列中，导致了上述情况的出现



#### 		3、**Promise.reject(reason)**

​	`**Promise.reject(reason)**`方法返回一个带有拒绝原因reason参数的Promise对象。



#### 		4、**Promise.resolve(value)**

​	`Promise.resolve(value)`方法返回一个以给定值解析后的`Promise` 对象。如果该值为promise，返回这个promise；如果这个值是thenable（即带有`"then" 方法`)），返回的promise会“跟随”这个thenable的对象，采用它的最终状态；否则返回的promise将以此值完成。此函数将类promise对象的多层嵌套展平。



#### 5、Promise.allSettled()

返回一个在所有给定的promise都已经`fulfilled`或`rejected`后的promise，并带有一个对象数组，每个对象表示对应的promise结果。

每个promise都得到结果后，不管是成功或则失败，都会等到全部完成过后才返回

```js
const promise1 = Promise.resolve(3);
const promise2 = new Promise((resolve, reject) => setTimeout(reject, 100, 'foo'));
const promises = [promise1, promise2];

Promise.allSettled(promises).
  then((results) => results.forEach((result) => console.log(result.status)));

// "fulfilled"
// "rejected"
```



#### 6、Promise.any()

接收一个`Promise`可迭代对象，只要其中的一个 `promise` 成功，就返回那个已经成功的 `promise` 。如果可迭代对象中没有一个 `promise` 成功（即所有的 `promises` 都失败/拒绝），就返回一个失败的 `promise `和[`AggregateError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/AggregateError)类型的实例，它是 [`Error`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error) 的一个子类，用于把单一的错误集合在一起。本质上，这个方法和`Promise.all()`是相反的。

```js
const pErr = new Promise((resolve, reject) => {
  reject("总是失败");
});

const pSlow = new Promise((resolve, reject) => {
  setTimeout(resolve, 500, "最终完成");
});

const pFast = new Promise((resolve, reject) => {
  setTimeout(resolve, 100, "很快完成");
});

Promise.any([pErr, pSlow, pFast]).then((value) => {
  console.log(value);
  // pFast fulfils first
})
// 期望输出: "很快完成"
```

### Promise.prototype

##### 	属性：

**Promise.prototype.constructor**

​	返回被创建的实例函数.  默认为 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 函数.

##### 	方法

###### 1、Promise.prototype.catch()

​	添加一个拒绝(rejection) 回调到当前 promise, 返回一个新的promise。当这个回调函数被调用，新 promise 将以它的返回值来resolve，否则如果当前promise 进入fulfilled状态，则以当前promise的完成结果作为新promise的完成结果.

###### 2、Promise.prototype.then()

​	添加解决(fulfillment)和拒绝(rejection)回调到当前 promise, 返回一个新的 promise, 将以回调的返回值来resolve

###### 3、Promise.prototype.finally()

​	添加一个事件处理回调于当前promise对象，并且在原promise对象解析完毕后，返回一个新的promise对象。回调会在当前promise运行完毕后被调用，无论当前promise的状态是完成(fulfilled)还是失败(rejected)





### 问题：

#### 1.当promise进入到then内部时，当内部的方法出现错误的时候，catch也会捕抓到

```javascript
p1.then(function(value) {
  console.log(value); // "Success!"
  throw 'oh, no!';
}).catch(function(e) {
  console.log(e); // "oh, no!"
}).then(function(){
  console.log('after a catch the chain is restored');
}, function () {
  console.log('Not fired due to the catch');
});
```





原文来源：[Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)