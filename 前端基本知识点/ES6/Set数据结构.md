# Set 数据结构

### 基本介绍

ES6新的数据结构Set

类似数组，但其成员值是唯一的，没有重复的值



### 初始化

Set本来是一个构造函数，用来生成Set数据结构

```javascript
let set = new Set([1,1,2,3,4,4]);
//简易除重
console.log(set) // Set(4) {1, 2, 3, 4}
```



### 属性和方法

#### 操作方法

1. add(value)：添加某个值，返回 Set 结构本身。
2. delete(value)：删除某个值，返回一个布尔值，表示删除是否成功。
3. has(value)：返回一个布尔值，表示该值是否为 Set 的成员。
4. clear()：清除所有成员，无返回值。

```javascript
let set = new Set();
console.log(set.add(1).add(2)); // Set [ 1, 2 ]

console.log(set.delete(2)); // true
console.log(set.has(2)); // false

console.log(set.clear()); // undefined
console.log(set.has(1)); // false
```



#### 遍历方法

1. keys()：返回键名的遍历器
2. values()：返回键值的遍历器
3. entries()：返回键值对的遍历器
4. forEach()：使用回调函数遍历每个成员，无返回值

```javascript
let set = new Set(['a','b','c']);
console.log(set.keys())// SetIterator {"a", "b", "c"}
console.log([...set.keys()]); // ["a", "b", "c"]

console.log(set.values()); // SetIterator {"a", "b", "c"}
console.log([...set.values()]); // ["a", "b", "c"]

console.log(set.entries()); // SetIterator {"a", "b", "c"}
console.log([...set.entries()]); // [["a", "a"], ["b", "b"], ["c", "c"]]
```



#### 属性

1. Set.prototype.constructor：构造函数，默认就是 Set 函数。
2. Set.prototype.size：返回 Set 实例的成员总数。

