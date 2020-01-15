## async/await的基本用法



首先，我们使用`async`关键字，您可以将它放在函数声明之前，将其转换为async function；

**async function**用来定义一个返回AsyncFunction 对象的异步函数。异步函数是指通过事件循环异步执行的函数，它会通过一个隐式的Promise 返回其结果。如果你在代码中使用了异步函数，就会发现它的语法和结构会更像是标准的同步函数。

```javascript
function resolveAfter2Seconds() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve('resolved');
    }, 2000);
  });
}
async function asyncCall() {
  console.log('calling');
  var result = await resolveAfter2Seconds();
  console.log(result);
  // expected output: 'resolved'
}

asyncCall();
//calling
//resolved
```



#### 语法

```javascript
async function name([param[, param[, ... param]]]) { statements }
```

参数：

​	name：函数名

​	param：要传递给函数的参数

​	statements：函数体语言

返回值：

​	返回的Promise对象会运行执行(resolve)异步函数的返回结果，或者运行拒绝(reject)——如果异步函数抛出异常的话。



#### 描述

异步函数可以包含`await`指令，该指令会暂停异步函数的执行，并等待Promise执行，然后继续执行异步函数，并返回结果。

记住，await 关键字只在异步函数内有效。如果你在异步函数外使用它，会抛出语法错误。

注意，当异步函数暂停时，它调用的函数会继续执行(收到异步函数返回的隐式Promise)

一个详细的示例

```javascript
var resolveAfter2Seconds = function() {
  console.log("starting slow promise");
  return new Promise(resolve => {
    setTimeout(function() {
      resolve("slow");
      console.log("slow promise is done");
    }, 2000);
  });
};

var resolveAfter1Second = function() {
  console.log("starting fast promise");
  return new Promise(resolve => {
    setTimeout(function() {
      resolve("fast");
      console.log("fast promise is done");
    }, 1000);
  });
};

var sequentialStart = async function() {
  console.log('==SEQUENTIAL START==');

  // 1. Execution gets here almost instantly
    //先创建一个slow的定时器--用了2秒
  const slow = await resolveAfter2Seconds();
  console.log(slow); // 2. this runs 2 seconds after 1.
	//当slow完成后再创建一个fast的定时器 --用了1s
  const fast = await resolveAfter1Second();
  console.log(fast); // 3. this runs 3 seconds after 1.
  //一共花了3秒  
}

var concurrentStart = async function() {
  console.log('==CONCURRENT START with await==');
    //开始定义slow的定时器
  const slow = resolveAfter2Seconds(); // starts timer immediately
    //开始定义fast的定时器
  const fast = resolveAfter1Second(); // starts timer immediately
//由于是同时开始定义定时器的，所以2个定时器是同时开始计时
    
  // 1. Execution gets here almost instantly
    //等待slow定时器返回结果
  console.log(await slow); // 2. this runs 2 seconds after 1.
    //当slow返回结果后，fast由于已经完成了1s了，所以能立即返回数据
  console.log(await fast); // 3. this runs 2 seconds after 1., immediately after 2., since fast is already resolved
    //该程序只执行了2秒
}

var concurrentPromise = function() {
  console.log('==CONCURRENT START with Promise.all==');
  return Promise.all([resolveAfter2Seconds(), resolveAfter1Second()]).then((messages) => {
      //等待slow的结果返回
    console.log(messages[0]); // slow
    console.log(messages[1]); // fast
  });
    //改程序执行了2秒
}

var parallel = async function() {
  console.log('==PARALLEL with await Promise.all==');
  //同上
  // Start 2 "jobs" in parallel and wait for both of them to complete
  await Promise.all([
      (async()=>console.log(await resolveAfter2Seconds()))(),
      (async()=>console.log(await resolveAfter1Second()))()
  ]);
}

// This function does not handle errors. See warning below!
var parallelPromise = function() {
  console.log('==PARALLEL with Promise.then==');
  resolveAfter2Seconds().then((message)=>console.log(message));
  resolveAfter1Second().then((message)=>console.log(message));
}

sequentialStart(); // after 2 seconds, logs "slow", then after 1 more second, "fast"
//一共花了3秒，
//结果
//==SEQUENTIAL START==
//starting slow promise
//slow
//starting fast promise
//fast promise is done
//fast

// wait above to finish
setTimeout(concurrentStart, 4000); // after 2 seconds, logs "slow" and then "fast"
//一共花了6秒，
//结果
//==CONCURRENT START with await==
//starting slow promise
//starting fast promise
//fast promise is done
//slow promise is done
//slow
//fast

// wait again
setTimeout(concurrentPromise, 7000); // same as concurrentStart
//一共花了9秒
//==CONCURRENT START with Promise.all==
//starting slow promise
//starting fast promise
//fast promise is done
//slow promise is done
//slow
//fast

// wait again
setTimeout(parallel, 10000); // truly parallel: after 1 second, logs "fast", then after 1 more second, "slow"
//一共花了12秒
// wait again
setTimeout(parallelPromise, 13000); // same as parallel
```

使用async函数重写 promise 链

```javascript
function getProcessedData(url) {
  return downloadData(url) // 返回一个 promise 对象
    .catch(e => {
      return downloadFallbackData(url)  // 返回一个 promise 对象
    })
    .then(v => {
      return processDataInWorker(v); // 返回一个 promise 对象
    });
}
```

重写成单个asynv函数后

```javascript
async function getProcessedData(url) {
  let v;
  try {
    v = await downloadData(url); 
  } catch (e) {
    v = await downloadFallbackData(url);
  }
  return processDataInWorker(v);
}
```



### async/await的缺陷

Async / await对于了解非常有用，但还有一些缺点需要考虑。

Async / await使您的代码看起来是同步的，并且在某种程度上它使它的行为更加同步。 `await`关键字阻止执行所有代码，直到promise完成，就像执行同步操作一样。它允许其他任务在此期间继续运行，但您自己的代码被阻止。

这意味着您的代码可能会因为大量等待的promises相继发生而变慢。每个`await`将等待前一个完成，而实际上你想要的是promises同时开始处理（就像我们没有使用async / await时那样）。

有一种模式可以缓解这个问题 ––通过关闭所有promise进程将`Promise`对象存储在变量中，然后等待触发它们。让我们看一些证明这个概念的例子。

```javascript
function timeoutPromise(interval) {
  return new Promise((resolve, reject) => {
    setTimeout(function(){
      resolve("done");
    }, interval);
  });
};
//比较耗时的方法 按累计来算，估计要花费9s来执行
async function timeTest() {
  await timeoutPromise(3000);
  await timeoutPromise(3000);
  await timeoutPromise(3000);
}

async function timeTest() {
  const timeoutPromise1 = timeoutPromise(3000);
  const timeoutPromise2 = timeoutPromise(3000);
  const timeoutPromise3 = timeoutPromise(3000);

  await timeoutPromise1;
  await timeoutPromise2;
  await timeoutPromise3;
}
```

在这里，我们将三个Promise对象存储在变量中，这样可以同时启动它们关联的进程

接下来，我们等待他们的结果 - 因为promise都在基本上同时开始处理，promise将同时完成;当您运行第二个示例时，您将看到警报框报告总运行时间超过3秒！

您必须仔细测试您的代码，并在性能开始受损时牢记这一点





原文来源：[async和await:让异步编程更简单]([https://developer.mozilla.org/zh-CN/docs/learn/JavaScript/%E5%BC%82%E6%AD%A5/Async_await](https://developer.mozilla.org/zh-CN/docs/learn/JavaScript/异步/Async_await))

