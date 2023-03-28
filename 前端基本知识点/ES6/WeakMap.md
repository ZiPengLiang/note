# WeakMap

**`WeakMap`** 对象是一组键/值对的集合，其中的键是弱引用的。其键必须是对象，而值可以是任意的。

```javascript
//基本语法
new WeakMap([iterable])
```

### 参数

**iterable**：Iterable 是一个数组（二元数组）或者其他可迭代的且其元素是键值对的对象。每个键值对会被加到新的 WeakMap 里。null 会被当做 undefined。



### 特点

#### 1. WeakMap 只接受对象作为键名

```javascript
const map = new WeakMap();
map.set(1, 2);
// TypeError: Invalid value used as weak map key
map.set(null, 2);
// TypeError: Invalid value used as weak map key
```

WeakMap 的 key 只能是 `Object` 类型。 原始数据类型 是不能作为 key 的。

#### 2. WeakMap 的键名所引用的对象是弱引用

**弱引用**：在计算机程序设计中，弱引用与强引用相对，是指不能确保其引用的对象不会被垃圾回收器回收的引用。 一个对象若只被弱引用所引用，则被认为是不可访问（或弱可访问）的，并因此可能在任何时刻被回收。

在 JavaScript 中，一般我们创建一个对象，都是建立一个强引用：

```javascript
var obj = new Object()
```

只有该对象没有被引用也就是将obj设置为null后，回收机制才有可能将obj回收

而如果我们能创建一个弱引用的对象：

```javascript
// 假设可以这样创建一个
var obj = new WeakObject();
```

实际上我们不需要做什么，当回收机制执行后，obj所引用的对象自动会被回收

实际上建立了强引用关系对象是比较难以回收，举个例子

```javascript
const key = new Array(5 * 1024 * 1024);
const arr = [
  [key, 1]
];

console.log(arr) // [Array(2)]
key = null
console.log(arr) //[Array(2)]

//实际上key原本指向的数组并未被销毁
```

使用这种方式，我们其实建立了 arr 对 key 所引用的对象(我们假设这个真正的对象叫 Obj)的强引用。

当你设置 `key = null` 时，只是去掉了 key 对 Obj 的强引用，并没有去除 arr 对 Obj 的强引用，所以 Obj 还是不会被回收掉。

那么怎么回收强引用的对象呢？这就需要用到了delete(key) 然后再将key = null

```javascript
let map = new Map();
let key = new Array(5 * 1024 * 1024);
map.set(key, 1);
map.delete(key);
key = null;
```



### 属性

**WeakMap.prototype.constructor**

​		返回创建WeakMap实例的原型函数。 `WeakMap`函数是默认的。



### 方法

`WeakMap.prototype.delete(key)`

​		移除key的关联对象。执行后 `WeakMap.prototype.has(key)`返回`false`。

`WeakMap.prototype.get(key)`

​		返回`key关联对象`, 或者 `undefined`(没有key关联对象时)。

`WeakMap.prototype.has(key)`

​		根据是否有key关联对象返回一个Boolean值。

`WeakMap.prototype.set(key, value)`

​		在WeakMap中设置一组key关联对象，返回这个 `WeakMap`对象。



```javascript
const wm1 = new WeakMap(),
      wm2 = new WeakMap(),
      wm3 = new WeakMap();
const o1 = {},
      o2 = function(){},
      o3 = window;

wm1.set(o1, 37);
wm1.set(o2, "azerty");
wm2.set(o1, o2); // value可以是任意值,包括一个对象或一个函数
wm2.set(o3, undefined);
wm2.set(wm1, wm2); // 键和值可以是任意对象,甚至另外一个WeakMap对象

wm1.get(o2); // "azerty"
wm2.get(o2); // undefined,wm2中没有o2这个键
wm2.get(o3); // undefined,值就是undefined

wm1.has(o2); // true
wm2.has(o2); // false
wm2.has(o3); // true (即使值是undefined)

wm3.set(o1, 37);
wm3.get(o1); // 37

wm1.has(o1);   // true
wm1.delete(o1);
wm1.has(o1);   // false
```

