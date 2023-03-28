# proxy

使用 defineProperty 只能重定义属性的读取（get）和设置（set）行为，到了 ES6，提供了 Proxy，可以重定义更多的行为，比如 in、delete、函数调用等更多行为。

**Proxy** 对象用于定义基本操作的自定义行为（如属性查找，赋值，枚举，函数调用等）。

### 语法

```js
let p = new Proxy(target, handler);
```



### 参数

**target**：用`Proxy`包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。

**handler**：一个对象，其属性是当执行一个操作时定义代理的行为的函数。



### 方法

#### **Proxy.revocable()**

可以用来创建一个可撤销的代理对象。

##### 语法

```js
Proxy.revocable(target, handler);
```

##### 参数

**target**：将用 `Proxy` 封装的目标对象。可以是任何类型的对象，包括原生数组，函数，甚至可以是另外一个代理对象。

**handler**：一个对象，其属性是一批可选的函数，这些函数定义了对应的操作被执行时代理的行为。



##### 返回值

返回一个包含了代理对象本身和它的撤销方法的可撤销 `Proxy` 对象。其结构为： `{"proxy": proxy, "revoke": revoke}`，其中：

**proxy**：表示新生成的代理对象本身，和用一般方式 `new Proxy(target, handler)` 创建的代理对象没什么不同，只是它可以被撤销掉。

**revoke**：撤销方法，调用的时候不需要加任何参数，就可以撤销掉和它一起生成的那个代理对象。



一旦某个代理对象被撤销，它将变得几乎完全不可调用，在它身上执行任何的**可代理操作**都会抛出 `TypeError` 异常。一旦被撤销，这个代理对象便不可能被直接恢复到原来的状态，同时和它关联的**目标对象**以及**处理器对象**都有可能被垃圾回收掉。再次调用撤销方法 `revoke()` 则不会有任何效果，但也不会报错。



##### 实例

```js
var revocable = Proxy.revocable({}, {
  get(target, name) {
    return "[[" + name + "]]";
  }
});
var proxy = revocable.proxy;
proxy.foo;              // "[[foo]]"

revocable.revoke();

console.log(proxy.foo); // 抛出 TypeError
proxy.foo = 1           // 还是 TypeError
delete proxy.foo;       // 又是 TypeError
typeof proxy            // "object"，因为 typeof 不属于可代理操作
```



### handler 对象的方法

handler 对象是一个占位符对象，它包含`Proxy`的捕获器。

#### 基础示例

当对象中不存在属性名时，缺省返回数为`37`。例子中使用了 `get`。

```js
let handler = {
    get: function(target, name){
        return name in target ? target[name] : 37;
    }
};

let p = new Proxy({}, handler);

p.a = 1;
p.b = undefined;

console.log(p.a, p.b);    // 1, undefined

console.log('c' in p, p.c);    // false, 37
```



#### 无操作转发代理

使用了一个原生 JavaScript 对象，代理会将所有应用到它的操作转发到这个对象上

```js
let target = {};
let p = new Proxy(target, {});

p.a = 37;   // 操作转发到目标

console.log(target.a);    // 37. 操作已经被正确地转发
```



#### 验证

通过代理，你可以轻松地验证向一个对象的传值。这个例子使用了 `set`。

```js
let validator = {
  set: function(obj, prop, value) {
    if (prop === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('The age is not an integer');
      }
      if (value > 200) {
        throw new RangeError('The age seems invalid');
      }
    }

    // The default behavior to store the value
    obj[prop] = value;

    // 表示成功
    return true;
  }
};

let person = new Proxy({}, validator);

person.age = 100;

console.log(person.age); 
// 100

person.age = 'young'; 
// 抛出异常: Uncaught TypeError: The age is not an integer

person.age = 300; 
// 抛出异常: Uncaught RangeError: The age seems invalid
```

#### 

方法代理可以轻松地通过一个新构造函数来扩展一个已有的构造函数。