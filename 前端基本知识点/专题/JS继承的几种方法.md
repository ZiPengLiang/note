# JS继承的方法

> javascript是一种面向对象的弱类型语言，其拥有三大特性--封装，继承以及多态，那么该如何在JS中实现继承呢？



## JS继承的实现方法

1. 原型链继承
2. 构造继承
3. 实例继承
4. 拷贝继承
5. 组合继承
6. 寄生组合继承



首先，创建一个父类

```javascript
function Person(name,age){
    //属性
	this.name = name;
    this.age = age
}
//原型方法
Person.prototype.show = function(){
    alert(`Hi，I'm ${this.name},I am ${this.age} years old`);
}
```



### 1.原型链继承

​	核心：将父类的实例作为子类的原型

```javascript
function Teacher(teach){
	this.teach = teach
}
Teacher.prototype = new Person('老王',48)
let teacher = new Teacher('数学');

console.log(teacher.name)  //老王
console.log(teacher.teach) //数学
```

​	原理：Teacher将Person的实例写在了自己的原型 --- Teacher.prototype上，当调用Teacher的属性时，浏览器会先在Teacher()中搜索，若没有则向Teacher的原型上（Person的实例）上寻找，若无则往Person中的原型中查找。

​	特点：

​		1、非常纯粹的继承关系，实例是子类的实例，也是父类的实例

​		2、父类新增原型方法/原型属性，子类都能访问到

​		3、简单，易于实现

​	 缺点：

​		1、无法实现多继承

​		2、来自原型对象中的所有属性被实例共享



### 2.构造继承

​	核心：使用父类的构造函数来增强子类的实例

```javascript
function Teacher(teach){
    Person.call(this,'老王','48');
    this.teach = teach
}
let teach  = new Teacher('数学')；
console.log(teach.name);  //老王
console.log(teach.teach);	//数学

```

​	原理：通过call将Person的属性指向Teacher，相当于将Person中的属性复制一遍到了Teacher中

​	特点：

​		1、创建子类实例可以向父类中传递数据

​		2、可以实现多继承

​	缺点：

​		1、这实际上是子类的实例

​		2、只能继承父类中的方法和属性，不能继承父类原型中的属性和方法

​		3、无法实现函数复用



### 3.实例继承

​	核心：为父类的实例添加属性，并将其作为子类的实例返回出去

```javascript
function Teach(teach){
    var instance = new Person('老王','48');
    instance.teach = teach
    return instance
}

let teach = new Teach('数学')；
console.log(teach.name) //老王
console.log(teach.teach) //数学
console.log(teach instanceof Person); // true
console.log(teach instanceof Teach); // false
```

​	原理：简单来说就是在子类的构造函数中创建一个Person的实例，并往该实例中添加属性，在子类中返回该实例实现继承

​	特点：

​		1、不限制调用方式---new Teacher()  或 Teacher() 都能创建实例

​	缺点：

​		1、实例是父类的实例

​		2、不支持多继承

​	

### 4.拷贝继承

```javascript
function Teacher(name,age,teach){
    var person = new Person(name,age);
    for(var v in person){
        Teacher.prototype[v] = person[v]
    }
    Teacher.prototype.teach = teach
}

let teach = new Teache('老王','48','数学')；

```

​	原理：通过便利将Person实例的属性一一拷贝到Teacher的原型中

​	特点：

​		1、支持多继承

​	缺点：

​		1、效率低，占内存

​		2、不能获取无法遍历到的方法



### 5.组合继承

核心：通过调用父类的实例来实现属性继承，并通过将父类的实例作为子类的原型，实现函数复用

```javascript
function Teacher(name,age,teach){
    Person.call(this,name,age)
    this.teach = teach
}
Teacher.prototype = new Person()

var teach = new Teacher('老王','48','数学');
console.log(teach.name);
console.log(teach instanceof Person); // true
console.log(teach instanceof Teacher); // true
```

​	特点：

​		1、能继承实例中的属性和方法，也能继承实例中的属性和方法

​		2、不存在引用属性共享问题

​		3、可以传参

​		4、函数可复用

​	缺点：

​		1、调用了两次父类构造函数，生成了两份实例



### 6.寄生组合继承

​	核心：通过寄生方式，砍掉父类的实例属性，这样，在调用两次父类的构造的时候，就不会初始化两次实例方法/属性，避免的组合继承的缺点

```javascript
function Teacher(name,age,teach){
    Person(this,name,age)
    this.teach = teach
}
(function(){
    var Super = function(){}
    Super.prototype = Person.prototype
    Teacher.prototype = new Super()
})()
Teacher.prototype.constructor = Teacher; // 需要修复下构造函数
var teach = new Teacher('老王','48','数学');
console.log(teach.name);
console.log(teach instanceof Person); // true
console.log(teach instanceof Teacher); //true
```

特点：完美

缺点：比较复杂