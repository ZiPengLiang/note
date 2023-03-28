# Generator -- 生成器

### 定义

使用 function* 语法和一个或多个 yield 表达式以创建一个函数即为生成器，当然它的返回值就是一个迭代器即生成器

例子

```javascript
function* gen(x){
	var y = yield x+2;
	return y
}
```

其实整个 Generator 函数就是一个封装的异步任务，或者说是异步任务的容器。异步操作需要暂停的地方，都用 yield 语句注明。

基本用法

```javascript
function* gen(x){
	var y = yield x+2;
	return y
}
var g = gen(1);
//g.next() 会返回一个对象，其中包含两个参数 value -- 表示该次运行的值，done -- 表示 Generator 函数是否执行完毕，即是否还有下一个阶段。通过这两个参数使得Generator函数可以暂停执行和恢复执行，这是它能封装异步任务的根本原因。
console.log(g.next());//返回 对象 {value : 3, done : false }

console.log(g.next());//返回 对象 {value : undefined, done : false }
```



### 方法

#### Generator.prototype.next()

​	**next()**`方法返回一个包含属性 `done` 和 `value` 的对象。该方法也可以通过接受一个参数用以向生成器传值。

```javascript
gen.next(value)
//参数
value -- 向生成器传递的值.
```

用法：



```js
function* gen() { 
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();
//没用return
g.next();        // { value: 1, done: false }
g.next(); 		 // { value: 2, done: false }
g.next();        // { value: 3, done: false }
//返回完成
g.next();        // { value: undefined, done: true }
```

#### Generator.prototype.return()

​	**return()**方法返回给定的值并结束生成器。

```javascript

gen.return(value)
//参数
value -- 需要返回的值
```

用法

```js
function* gen() { 
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next();        // { value: 1, done: false }
g.return("foo"); // { value: "foo", done: true }
g.next();        // { value: undefined, done: true }
```



#### Generator.prototype.throw()

`**throw**`**`()`** 方法用来向生成器抛出异常，并恢复生成器的执行，返回带有 `done` 及 `value` 两个属性的对象。

```js
gen.throw(exception)
//参数
exception -- 用于抛出的异常。 使用 Error 的实例对调试非常有帮助.
```

用法

```js
function* gen() {
  while(true) {
    try {
       yield 42;
    } catch(e) {
      console.log("Error caught!");
    }
  }
}

var g = gen();
g.next(); // { value: 42, done: false }
g.throw(new Error("Something went wrong")); // "Error caught!"
```





下面是Generator复杂点的用法

### 单个异步方法

```js
var fetch = require('node-fetch');

function* gen(){
    var url = 'https://api.github.com/users/github';
    var result = yield fetch(url);
    console.log(result.bio);
}
```

为了获得最终结果

```js
var g = gen();
var result = g.next();

result.value.then(function(data){
    return data.json();
}).then(function(data){
    g.next(data);
});

```

假若fetch（url）返回的是一个promise对象，所以result的值为：

```js
{ value: Promise { <pending> }, done: false }
```

最后我们为这个 Promise 对象添加一个 then 方法，先将其返回的数据格式化(`data.json()`)，再调用 g.next，将获得的数据传进去，由此可以执行异步任务的第二阶段，代码执行完毕。



### 多个异步任务

上述只是调用一个接口，当调用多个接口的时候，这将会变得更为麻烦

```js
var fetch = require('node-fetch');

function* gen() {
    var r1 = yield fetch('https://api.github.com/users/github');
    var r2 = yield fetch('https://api.github.com/users/github/followers');
    var r3 = yield fetch('https://api.github.com/users/github/repos');

    console.log([r1.bio, r2[0].login, r3[0].full_name].join('\n'));
}
```

为了获得最终数据，代码可能会变成这样子

```js
var g = gen();
var result1 = g.next();

result1.value.then(function(data){
    return data.json();
})
.then(function(data){
    return g.next(data).value;
})
.then(function(data){
    return data.json();
})
.then(function(data){
    return g.next(data).value
})
.then(function(data){
    return data.json();
})
.then(function(data){
    g.next(data)
});
```

这样代码显得非常冗余，并且可读性和复用性都比较差，因此可以用上递归的算法

```javascript
function run(gen) {
    var g = gen();

    function next(data) {
        var result = g.next(data);

        if (result.done) return;

        result.value.then(function(data) {
            return data.json();
        }).then(function(data) {
            next(data);
        });

    }

    next();
}

run(gen);
```

