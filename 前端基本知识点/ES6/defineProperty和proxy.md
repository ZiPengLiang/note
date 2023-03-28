# defineProperty和proxy

### defineProperty

#### 作用

`**Object.defineProperty()**` 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。

#### 语法

```javascript
Object.defineProperty(obj,prop,descriptor)
```

#### 参数

**obj**：要在其上定义属性的对象。

**prop**：要定义或修改的属性的名称。

**descriptor**：将被定义或修改的属性描述符

​	

函数的第三个参数 descriptor 所表示的属性描述符有两种形式：**数据描述符和存取描述符**。

**两者均具有以下两种键值**：

**configurable**：当且仅当该属性的 configurable 为 true 时，该属性`描述符`才能够被改变，同时该属性也能从对应的对象上被删除。**默认为 false**

**enumerable**：当且仅当该属性的`enumerable`为`true`时，该属性才能够出现在对象的枚举属性中。**默认为 false**。



**数据描述符同时具有以下可选键值**：

**value**：该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。**默认为 `undefined`**。

**writable**：当且仅当该属性的`writable`为`true`时，`value`才能被赋值运算符改变。**默认为 false**。



**存取描述符同时具有以下可选键值**：

**get**：一个给属性提供 getter 的方法，如果没有 getter 则为 `undefined`。当访问该属性时，该方法会被执行，方法执行时没有参数传入，但是会传入`this`对象（由于继承关系，这里的`this`并不一定是定义该属性的对象）。---**默认为 `undefined`**。

**set**：一个给属性提供 setter 的方法，如果没有 setter 则为 `undefined`。当属性值修改时，触发执行该方法。该方法将接受唯一参数，即该属性新的参数值。---**默认为 `undefined`**



#### 注意事项

**属性描述符必须是数据描述符或者存取描述符两种形式之一，不能同时是两者** 。

这就意味着可以写成这样

```js
Object.defineProperty({}, "num", {
    value: 1,
    writable: true,
    enumerable: true,
    configurable: true
});
```

或者

```js
var value = 1;
Object.defineProperty({}, "num", {
    get : function(){
      return value;
    },
    set : function(newValue){
      value = newValue;
    },
    enumerable : true,
    configurable : true
});
```

此外，所有的属性描述符都是非必须的，但是 descriptor 这个字段是必须的，如果不进行任何配置，你可以这样：

```js
var obj = Object.defineProperty({}, "num", {});
console.log(obj.num); // undefined
```



### 存取器属性 --Setter和Getters

当程序查询存取器属性的值时，JavaScript 调用 getter方法。这个方法的返回值就是属性存取表达式的值。

当程序设置一个存取器属性的值时，JavaScript 调用 setter 方法，将赋值表达式右侧的值当做参数传入 setter。从某种意义上讲，这个方法负责“设置”属性值。可以忽略 setter 方法的返回值。

比如

```js
var obj = {}, value = null;
Object.defineProperty(obj, "num", {
    //获取数据
    get: function(){
        console.log('执行了 get 操作')
        return value;
    },
    //设置数据
    set: function(newValue) {
        console.log('执行了 set 操作')
        value = newValue;
    }
})

obj.num = 1 // 执行了 set 操作

console.log(obj.num); // 执行了 get 操作 // 1
```

