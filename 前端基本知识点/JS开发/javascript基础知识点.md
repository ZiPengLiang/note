# JavaScript基础知识

[TOC]

## 普通函数与箭头函数的**this**问题

```javascript
function fn() {
    console.log(this); // 1. {a: 100}
    var arr = [1, 2, 3];

    (function() {
        console.log(this); // 2. Window
    })();

    // 普通 JS
    arr.map(function(item) {
        console.log(this); // 3. Window
        return item + 1;
    });
    // 箭头函数
    let brr = arr.map(item => {
        console.log("es6", this); // 4. {a: 100}
        return item + 1;
    });
}
fn.call({ a: 100 });
```

常见的基本是 3 种情况：es5 普通函数、es6 的箭头函数以及通过`bind`改变过上下文返回的新函数。

### es5 普通函数

- 函数被直接调用，上下文一定是`window`
- 函数作为对象属性被调用，例如：`obj.foo()`，上下文就是对象本身`obj`
- 通过`new`调用，`this`绑定在返回的实例上

### es6 的箭头函数

它本身没有`this`，会沿着作用域向上寻找，直到`global` / `window`。请看下面的这段代码：

```javascript
function run() {
    const inner = () => {
        return () => {
            console.log(this.a);
        };
    };

    inner()();
}

run.bind({ a: 1 })(); // Output: 1
```

### **bind 绑定上下文返回的新函数**

就是被第一个 bind 绑定的上下文，而且 bind 对“箭头函数”无效。请看下面的这段代码：

```javascript
function run() {
    console.log(this.a);
}
//t通过bind改变了指针的指向
run.bind({ a: 1 })(); // output: 1

// 多次bind，上下文由第一个bind的上下文决定
run.bind({ a: 2 }).bind({ a: 1 })(); // output: 2
```

再说说这几种方法的优先级：new > bind > 对象调用 > 直接调用

## 原始数据类型和判断方法

### 数据类型：

#### 七种原始类型：

- 布尔值 --Boolean
- 字符串 -- String
- 数值 -- Number
- 空 -- Null
- 未定义 --Underfinded
- Symbol (新定义)
- BigInt(新定义)

还有2中类型是Object和Function，但他们都不包含在原始类型上



### 常用的判断方法

在进行判断的时候有`typeof`、`instanceof`。对于数组的判断，使用`Array.isArray()`：

- typeof：

  - typeof 基本都可以正确判断数据类型
  - `typeof null`和`typeof [1, 2, 3]`均返回”object”
  - ES6 新增：`typeof Symbol()`返回”symbol”

- instanceof：

  - 专门用于实例和构造函数对应

  - ```javascript
    function Obj(value) {
        this.value = value;
    }
    let obj = new Obj("test");
    console.log(obj instanceof Obj); // output: true
    ```

  - 判断是否是数组：`[1, 2, 3] instanceof Array`

- Array.isArray()：ES6 新增，用来判断是否是’Array’。`Array.isArray({})`返回`false`。



## 原始类型转化

当我们对一个“对象”进行数学运算操作时候，会涉及到对象 => 基础数据类型的转化问题。

事实上，当一个对象执行例如加法操作的时候，如果它是原始类型，那么就不需要转换。否则，将遵循以下规则：

1. 调用实例的`valueOf()`方法，如果有返回的是基础类型，停止下面的过程；否则继续
2. 调用实例的`toString()`方法，如果有返回的是基础类型，停止下面的过程；否则继续
3. 都没返回原始类型，就会报错

```javascript
let a = {
    toString: function() {
        return "a";
    }
};

let b = {
    valueOf: function() {
        return 100;
    },
    toString: function() {
        return "b";
    }
};

let c = Object.create(null); // 创建一个空对象

console.log(a + "123"); // output: a123
console.log(b + 1); // output: 101
console.log(c + "123"); // 报错
```

除了`valueOf`和`toString`，es6 还提供了`Symbol.toPrimitive`供对象向原始类型转化，并且**它的优先级最高**！！稍微改造下上面的代码：

```javascript
let b = {
    valueOf: function() {
        return 100;
    },
    toString: function() {
        return "b";
    },
    [Symbol.toPrimitive]: function() {
        return 10000;
    }
};

console.log(b + 1); // output: 10001
```

最后，其实关于`instanceof`判断是否是某个对象的实例，es6 也提供了`Symbol.hasInstance`接口，代码如下：

```javascript
class Even {
    static [Symbol.hasInstance](num) {
        return Number(num) % 2 === 0;
    }
}

const Odd = {
    [Symbol.hasInstance](num) {
        return Number(num) % 2 !== 0;
    }
};

console.log(1 instanceof Even); // output: false
console.log(1 instanceof Odd); // output: true
```



## 深拷贝和浅拷贝

在 JS 中，函数和对象都是浅拷贝（地址引用），其他的，例如布尔值、数字等基础数据类型都是深拷贝（值引用）。

### 浅拷贝

值得提醒的是，ES6 的`Object.assign()`和 ES7 的`...`解构运算符都是“浅拷贝”

### 深拷贝

- 借助第三方库：jq 的`extend(true, result, src1, src2[ ,src3])`、lodash 的`cloneDeep(src)`
- `JSON.parse(JSON.stringify(src))`：这种方法有局限性，如果属性值是函数或者一个类的实例的时候，无法正确拷贝
- 借助 HTML5 的`MessageChannel`：这种方法有局限性，当属性值是函数的时候，会报错



## JS 事件流

### 事件冒泡和事件捕获

事件流分为：**冒泡**和**捕获**，顺序是先捕获再冒泡。

**事件冒泡**：子元素的触发事件会一直向父节点传递，一直到根结点停止。此过程中，可以在每个节点捕捉到相关事件。可以通过`stopPropagation`方法终止冒泡。

**事件捕获**：和“事件冒泡”相反，从根节点开始执行，一直向子节点传递，直到目标节点。

`addEventListener`给出了第三个参数同时支持冒泡与捕获：默认是`false`，事件冒泡；设置为`true`时，是事件捕获。

```html
<div id="app" style="width: 100vw; background: red;">
        <span id="btn">点我</span>
    </div>
    <script>
        // 事件捕获：先输出 "内层click事件触发"; 再输出 "外层click事件触发" 
        var useCapture = true;
        var btn = document.getElementById("btn");
        btn.addEventListener("click", function() {
            console.log("内层click事件触发");
        }, useCapture);
        var app = document.getElementById("app");
        app.onclick =
            function() {
                console.log("外层click事件触发");
            };
</script>
```

### DOM0 级 和 DOM2 级

**DOM2 级**：前面说的`addEventListener`，它定义了`DOM`事件流，捕获 + 冒泡。

**DOM0 级**：

- 直接在 html 标签内绑定`on`事件
- 在 JS 中绑定`on`系列事件

**注意**：现在通用`DOM2`级事件，优点如下：

1. 可以绑定 / 卸载事件
2. 支持事件流
3. 冒泡 + 捕获：相当于每个节点同一个事件，至少 2 次处理机会
4. 同一类事件，可以绑定多个函数