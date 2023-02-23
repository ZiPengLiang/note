# `EventEmitter`

`Node.js` 所有的异步 I/O 操作在完成时都会发送一个事件到事件队列。

`Node.js` 里面的许多对象都会分发事件：一个 `net.Server `对象会在每次有新连接时触发一个事件， 一个 `fs.readStream` 对象会在文件被打开的时候触发一个事件。 所有这些产生事件的对象都是 `events.EventEmitter` 的实例。

events 模块只提供了一个对象：`events.EventEmitter`。`EventEmitter `的核心就是事件触发与事件监听器功能的封装。

示例：

```js
// 引入 events 模块
var events = require('events');
// 创建 eventEmitter 对象
var eventEmitter = new events.EventEmitter();
event.on('some_event', function() { 
    console.log('some_event 事件触发'); 
}); 
setTimeout(function() { 
    event.emit('some_event'); 
}, 1000); 
```

`EventEmitter `对象如果在实例化时发生错误，会触发 error 事件。当添加新的监听器时，`newListener `事件会触发，当监听器被移除时，`removeListener `事件被触发。

`EventEmitter `的每个事件由一个事件名和若干个参数组成，事件名是一个字符串，通常表达一定的语义。对于每个事件，`EventEmitter `支持 若干个事件监听器。当事件触发时，注册到这个事件的事件监听器被依次调用，事件参数作为回调函数参数传递。

简单来说，同一个事件名可以绑定若干个是事件监听器，当事件触发时监听器依次执行。例如：

```js
var events = require('events'); 
var emitter = new events.EventEmitter(); 
emitter.on('someEvent', function(arg1, arg2) { 
    console.log('listener1', arg1, arg2); 
}); 
emitter.on('someEvent', function(arg1, arg2) { 
    console.log('listener2', arg1, arg2); 
}); 
emitter.emit('someEvent', 'arg1 参数', 'arg2 参数'); 

// 执行结果
listener1 arg1 参数 arg2 参数
listener2 arg1 参数 arg2 参数
```

### 方法

| 序号 | 方法 & 描述                                                  |
| :--- | :----------------------------------------------------------- |
| 1    | **`addListener(event, listener)`** 为指定事件添加一个监听器到监听器数组的尾部。与on事件没有区别 |
| 2    | **on(event, listener)** 为指定事件注册一个监听器，接受一个字符串 event 和一个回调函数。`server.on('connection', function (stream) {  console.log('someone connected!'); });` |
| 3    | **once(event, listener)** 为指定事件注册一个单次监听器，即 监听器最多只会触发一次，触发后立刻解除该监听器。`server.once('connection', function (stream) {  console.log('Ah, we have our first user!'); });` |
| 4    | **`removeListener(event, listener)`** 移除指定事件的某个监听器，监听器必须是该事件已经注册过的监听器。它接受两个参数，第一个是事件名称，第二个是回调函数名称。`var callback = function(stream) {  console.log('someone connected!'); }; server.on('connection', callback); // ... server.removeListener('connection', callback);` |
| 5    | **`removeAllListeners([event])`** 移除所有事件的所有监听器， 如果指定事件，则移除指定事件的所有监听器。 |
| 6    | **`setMaxListeners(n)`** 默认情况下， `EventEmitters `如果你添加的监听器超过 10 个就会输出警告信息。 `setMaxListeners `函数用于改变监听器的默认限制的数量。 |
| 7    | **`listeners(event)`** 返回指定事件的监听器数组。            |
| 8    | **`emit(event, [arg1], [arg2], [...])`** 按监听器的顺序执行执行每个监听器，如果事件有注册监听返回 true，否则返回 false。 |

示例： 

```js
var events = require('events');
var eventEmitter = new events.EventEmitter();

// 监听器 #1
var listener1 = function listener1() {
   console.log('监听器 listener1 执行。');
}

// 监听器 #2
var listener2 = function listener2() {
  console.log('监听器 listener2 执行。');
}

// 绑定 connection 事件，处理函数为 listener1 
eventEmitter.addListener('connection', listener1);

// 绑定 connection 事件，处理函数为 listener2
eventEmitter.on('connection', listener2);

var eventListeners = eventEmitter.listenerCount('connection');
console.log(eventListeners + " 个监听器监听连接事件。");

// 处理 connection 事件 
eventEmitter.emit('connection');

// 移除监绑定的 listener1 函数
eventEmitter.removeListener('connection', listener1);
console.log("listener1 不再受监听。");

// 触发连接事件
eventEmitter.emit('connection');

eventListeners = eventEmitter.listenerCount('connection');
console.log(eventListeners + " 个监听器监听连接事件。");

console.log("程序执行完毕。");
```

执行结果:

```
2 个监听器监听连接事件。
监听器 listener1 执行。
监听器 listener2 执行。
listener1 不再受监听。
监听器 listener2 执行。
1 个监听器监听连接事件。
程序执行完毕。
```