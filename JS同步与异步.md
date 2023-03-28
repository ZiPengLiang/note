# Javascript同步与异步问题

> 在javascript中大部分程序都是顺序执行的，即只有一个函数执行完后才会，但有时候某些程序运行时间过长，导致之后的程序也不能执行，这就是所谓的阻塞。为了解决这种情况，异步编程就出现了



### 同步Javascript

​	javascript在传统意义上是单线程的，只有通过特殊的工具才能实现多线程操作，因此在任何时候只能做一件事情，只有一个主线程，其他的事情都会被阻塞，直到前面的操作完成

​	举个例子

```javascript
var btn = document.getElementById('btn');
btn.addEventListener('click',function(){
	alert('click me');
    let pElem = document.createElement('p');
  	pElem.textContent = 'This is a newly-added paragraph.';
  	document.body.appendChild(pElem);
})
```

该代码的执行循序是：

​	1.获得DOM中的按钮

​	2.当点击按钮后，添加一个时间监听

​		(1).alert()被执行，消息弹出

​		(2).当alert完全结束后，创建一个`<p>`标签

​		(3).给`<p>`标签的文本赋值

​		(4).最后，将该段落方静DOM中

在上述例子中，当点击按钮后不会立即执行创建<p>标签的操作，直到alert完全执行完毕才会执行创建操作



### 异步Javascript

​	由于种种的原因，目前很多网页API特性使用异步代码，尤其是从外部获取资源的时候。

​	目前异步编程的方法有2中 ---老派callback和新派promise



#### 异步callback

​	异步callback实际上就是一个函数，它的作用是在当作参数传入到其他函数中，当其他函数运行完毕后然后再执行callback函数

```javascript
function getData(url,type,callback){
    //请求数据
  let xhr = new XMLHttpRequest();
  xhr.open('GET', url);
  xhr.responseType = type;
	//数据请求完毕
  xhr.onload = function() {
    //将请求到的参数作为实参传到callback中，然后执行callback函数
    callback(xhr.response);
  };

  xhr.send();
}
//将图片渲染到DOM上
function displayImage(blob) {
  let objectURL = URL.createObjectURL(blob);

  let image = document.createElement('img');
  image.src = objectURL;
  document.body.appendChild(image);
}

getData('coffee.jpg', 'blob', displayImage);

```



#### Promise

​	Promises 是新派的异步代码，现代的web APIs经常用到。**Promise** 对象用于表示一个异步操作的最终完成 (或失败), 及其结果值。当浏览器执行时，会将像promise这些异步操作放进队列中，这样就不会阻止后续的javascript执行，待主线程处理完成后再执行队列中的操作

```javascript
var promise1 = new Promise(function(resolve, reject) {
  setTimeout(function() {
    resolve('foo');
  }, 300);
});
//等待返回结果
promise1.then(function(value) {
  console.log(value);
  // expected output: "foo"
});
//由于异步执行，该console提前执行
console.log(promise1);
// expected output: [object Promise]

//最后结果
//[object Promise]
//"foo"
```



#### Promises 对比 callbacks

​	promises与旧式callbacks有一些相似之处。它们本质上是一个返回的对象，您可以将回调函数附加到该对象上，而不必将回调作为参数传递给另一个函数。

​	然而，`Promise`是专门为异步操作而设计的，与旧式回调相比具有许多优点:

​		1、您可以使用多个then()操作将多个异步操作链接在一起，并将其中一个操作的结果作为输入传递给下一个操作。这对于回调要难得多，回调常常以混乱的“末日金字塔”告终。 (也称为回调地狱)。

​		2、`Promise`总是严格按照它们放置在事件队列中的顺序调用

​		3、错误处理要好得多——所有的错误都由块末尾的一个.catch()块处理，而不是在“金字塔”的每一层单独处理。

