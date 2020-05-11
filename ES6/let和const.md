# ES6 -- let 和const

### 块级作用域

**变量提升**：JavaScript 中，函数及变量的声明都将被提升到函数的最顶部。

因此会出现这种情况

```javascript
console.log(value) //undefined
var value = 1
```

为什么不是not defined呢？

实际在javascript代码是这样执行的

```javascript
var value;
console.log(value);
value = 1
```

为了加强对变量生命周期的控制，ECMAScript 6 引入了块级作用域。

块级作用域存在于：

- 函数内部
- 块中(字符 { 和 } 之间的区域)

这也是常见问题for循环中i值得问题

```javascript
for (var i = 0; i < 10; i++) {
    ...
}
console.log(i); // 10
    
```

## let 和 const

块级声明用于声明在指定块的作用域之外无法访问的变量

let 和 const 都是块级声明的一种。

特点：

1. 不会被提升

   ```javascript
   console.log(value)//value is not defined 
   let value = 1
   ```

2. 重复声明报错

   ```javascript
   var value =1;
   let value = 2; //Uncaught SyntaxError: Identifier 'value' has already been declared
   ```

3. 不绑定全局作用域

   当在全局作用域中使用 var 声明的时候，会创建一个新的全局变量作为全局对象的属性。

   ```javascript
   var value = 1;
   console.log(window.value); // 1
   ```

   然而 let 和 const 不会：

   ```javascript
   let value = 1;
   console.log(window.value); // undefined
   ```



let 和const 的区别

const：声明常量，其值一旦别设置了就不能被修改，否则会报错

值得一提的是：const声明不允许的是修改绑定，但允许修改值

于就是当const声明对象的时候是可以被修改的

```javascript
const data = {
    value:1
}
data.value = 2;
data.num = 3

//报错
data = {}
```

那如何实现常量对象呢

通过Object自带的方法freeze()将对象变成不可拓展修改的常量对象

```javascript
const  foo = Object.freeze({
    value:1
})
//不起作用，甚至在严格模式中会报错
foo.value = 123;
```

拓展

Object.freeze(obj) -- 冻结一个对象，使其不可拓展、删除、修改，并且不能修改该对象已有属性的可枚举性、可配置性、可写性。并返回该对象

Object.isFrozen(obj)   --返回对象是否被冻结

Object.preventExtensions()--让一个对象变的不可扩展，也就是永远不能再添加新的属性

Object.isExtensible(obj)--返回obj是否可扩展。

Object.seal() --让一个对象变的不可扩展且不可删除（封闭），也就是永远不能再添加新的属性、删除已由属性

Object.isSealed(obj) --返回obj是否被封闭。



## 临时死区

临时死区（Temporal Dead Zone），简称TDZ

在javascript引擎扫描变量声明的时候，大致会将所扫描到的变量进行2中操作，要么将其提升到作用于顶部（var 声明），要么将变量放到TDZ中（let 和const）。当TDZ中的变量被访问的时候，会报错，当变量被声明后才被移除TDZ。

举个例子

```javascript
//全局属性
var value = 'global';

(function(){
	console.log(value);
	let value = 'local'
})()

{
	console.log(value)
	const value = 'local'
}

//最终结果都输出value is not defined，至于原因就是TDZ
```

