# TypeScript --类

### 类

```typescript
class Greeter {
    //属性
    greeting: string;
    //构造函数
    constructor(message: string) {
        this.greeting = message;
    }
    //方法
    greet() {
        return "Hello, " + this.greeting;
    }
}
//实例化
let greeter = new Greeter("world");
```

### 继承

基于类的程序设计中最基本的模式是允许使用继承来扩展现有的类

```typescript
/*父类
	定义了一个name的属性
	拥有一个构造函数 --并给name赋值
	方法move
*/
class Animal{
    name:string;
    constructor(theName:string){this.name = theName};
    move(distanceInMeters:number =0){
        console.log(`${this.name} moved ${distanceInMeters}m`)
    }
}

//子类
class Snake extends Animal{
    //super关键字用于访问和调用一个对象的父对象上的函数。
    constructor(name:string){super(name)}
    //重定义move方法
    move(distanceInMeters = 5){
        console.log("Slithering...");
        super.move(distanceInMeters);
    }
}
class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 45) {
        console.log("Galloping...");
        super.move(distanceInMeters);
    }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the Palomino");
//采用默认值
sam.move();
//修改distanceInMeters
tom.move(34);

//结果

Slithering...
Sammy the Python moved 5m.
Galloping...
Tommy the Palomino moved 34m
```

### 公共，私有与受保护的修饰符

公有`public` -- 默认

在上面的例子里，我们可以自由的访问程序里定义的成员。 如果你对其它语言中的类比较了解，就会注意到我们在之前的代码里并没有使用 `public`来做修饰；例如，C#要求必须明确地使用`public`指定成员是可见的。 在TypeScript里，成员都默认为 `public`。

你也可以明确的将一个成员标记成`public`。 我们可以用下面的方式来重写上面的 `Animal`类：

```typescript
class Animal {
    public name: string;
    public constructor(theName: string) { this.name = theName; }
    public move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```



私有`private`

当成员被标记成`private`时，它就不能在声明它的类的外部访问(包括子类)。比如：

```typescript
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

new Animal("Cat").name; // Error: 'name' is private;
```

TypeScript使用的是结构性类型系统， 当我们比较两种不同的类型时，并不在乎它们从何处而来，如果所有成员的类型都是兼容的，我们就认为它们的类型是兼容的

当我们比较带有`private`或`protected`成员的类型的时候，情况就不同了。 如果其中一个类型里包含一个 `private`成员，那么只有当另外一个类型中也存在这样一个`private`成员， 并且它们都是来自同一处声明时，我们才认为这两个类型是兼容的。 对于 `protected`成员也使用这个规则。

```typescript
class Animal {
	//私有变量 name
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
    constructor() { super("Rhino"); }
}

class Employee {
    //私有变量 name
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

let animal = new Animal("Goat");
let rhino = new Rhino();
let employee = new Employee("Bob");
//由于Rhino为animal的子类，name的来源为Animal，同源
animal = rhino; // true
//不同源
animal = employee; // Error: Animal and Employee are not compatible
```



保护`protected`

`protected`修饰符与`private`修饰符的行为很相似，但有一点不同，`protected`成员在派生类中仍然可以访问。例如：

```typescript
class Person {
    protected name: string;
    constructor(name: string) { this.name = name; }
}

class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name)
        this.department = department;
    }

    public getElevatorPitch() {
        //由于子类能有访问父类的保护属性，所以可以访问到name
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
console.log(howard.getElevatorPitch());
//在Person外部无法访问name属性
console.log(howard.name); // error
```

注意，我们不能在`Person`类外使用`name`，但是我们仍然可以通过`Employee`类的实例方法访问，因为`Employee`是由`Person`派生而来的。

构造函数也可以被标记成`protected`。 这意味着这个类不能在包含它的类外被实例化，但是能被继承。比如

```typescript
class Person {
    protected name: string;
    protected constructor(theName: string) { this.name = theName; }
}

// Employee can extend Person
class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name);
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
let john = new Person("John"); // Error: The 'Person' constructor is protected
```

### readonly修饰符

你可以使用`readonly`关键字将属性设置为只读的。 只读属性必须在声明时或构造函数里被初始化。

```typescript
class Octopus {
    //只读属性
    readonly name: string;
    readonly numberOfLegs: number = 8;
    constructor (theName: string) {
        this.name = theName;
    }
}
//能读，但是不能赋值
let dad = new Octopus("Man with the 8 strong legs");
dad.name = "Man with the 3-piece suit"; // error! name is readonly.
```

#### 参数属性

在上面的例子中，我们不得不定义一个受保护的成员`name`和一个构造函数参数`theName`在`Person`类里，并且立刻给`name`和`theName`赋值。 这种情况经常会遇到。 *参数属性*可以方便地让我们在一个地方定义并初始化一个成员。 下面的例子是对之前 `Animal`类的修改版，使用了参数属性：

```typescript
class Animal {
    //private name: string --- 实现了赋值和定义
    constructor(private name: string) { }
    move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

注意看我们是如何舍弃了`theName`，仅在构造函数里使用`private name: string`参数来创建和初始化`name`成员。 我们把声明和赋值合并至一处。

参数属性通过给构造函数参数添加一个访问限定符来声明。 使用 `private`限定一个参数属性会声明并初始化一个私有成员；对于`public`和`protected`来说也是一样。

### 存取器

TypeScript支持通过getters/setters来截取对象成员的访问。 它能帮助你有效的控制对对象成员的访问。下面来看如何把一个简单的类改写成使用`get`和`set`。 首先，我们从一个没有使用存取器的例子开始。

```typescript
//这是一个没有存取器的类
class Employee {
    fullName: string;
}
//创建一个实例
let employee = new Employee();
//可以随意设置fullName，但有时候会造成麻烦
employee.fullName = "Bob Smith";
if (employee.fullName) {
    console.log(employee.fullName);
}
```

下面这个版本里，我们先检查用户密码是否正确，然后再允许其修改员工信息。 我们把对 `fullName`的直接访问改成了可以检查密码的`set`方法。 我们也加了一个 `get`方法，让上面的例子仍然可以工作。

```typescript
let passcode = "secret passcode";

class Employee {
    //将_fullName设置为私有属性，防止外部调用和访问
    private _fullName: string;
	//获取_fullName
    get fullName(): string {
        return this._fullName;
    }
	//设置_fullName,内有判断
    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            console.log("Error: Unauthorized update of employee!");
        }
    }
}

let employee = new Employee();
//调用设置_fullName的get方法
employee.fullName = "Bob Smith";
//调用set的方法
if (employee.fullName) {
    alert(employee.fullName);
}
```

对于存取器有下面几点需要注意的：

首先，存取器要求你将编译器设置为输出ECMAScript 5或更高。 不支持降级到ECMAScript 3。 		其次，只带有 `get`不带有`set`的存取器自动被推断为`readonly`。 这在从代码生成 `.d.ts`文件时是有帮助的，因为利用这个属性的用户会看到不允许够改变它的值。

### 静态属性

到目前为止，我们只讨论了类的实例成员，那些仅当类被实例化的时候才会被初始化的属性。 我们也可以创建类的静态成员，这些属性存在于类本身上面而不是类的实例上。

 在这个例子里，我们使用 `static`定义`origin`，因为它是所有网格都会用到的属性。 每个实例想要访问这个属性的时候，都要在 `origin`前面加上类名。 如同在实例属性上使用 `this.`前缀来访问属性一样，这里我们使用`Grid.`来访问静态属性。

```typescript
class Grid {
    //静态属性，定义了坐标
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

### 抽象类

抽象类做为其它派生类的基类使用。 它们一般不会直接被实例化。 不同于接口，抽象类可以包含成员的实现细节。`abstract`关键字是用于定义抽象类和在抽象类内部定义抽象方法。

```typescript
//通过abstract来定义抽象类
abstract class Animal {
    //抽象方法，没有具体的方法内容，只能在派生类中实现
    abstract makeSound(): void;
    move(): void {
        console.log('roaming the earch...');
    }
}
```

抽象类中的抽象方法不包含具体实现并且必须在派生类中实现。

 抽象方法的语法与接口方法相似。 两者都是定义方法签名但不包含方法体。

 然而，抽象方法必须包含 `abstract`关键字并且可以包含访问修饰符。

```typescript
//抽象类和接口作用相识，相当于给类方法做了签名
abstract class Department {

    constructor(public name: string) {
    }

    printName(): void {
        console.log('Department name: ' + this.name);
    }

    abstract printMeeting(): void; // 必须在派生类中实现
}

class AccountingDepartment extends Department {

    constructor() {
        super('Accounting and Auditing'); // constructors in derived classes must call super()
    }

    printMeeting(): void {
        console.log('The Accounting Department meets each Monday at 10am.');
    }
	//由于抽象类中没有该方法会报错
    generateReports(): void {
        console.log('Generating accounting reports...');
    }
}

let department: Department; // ok to create a reference to an abstract type
department = new Department(); // error: cannot create an instance of an abstract class
department = new AccountingDepartment(); // ok to create and assign a non-abstract subclass
department.printName();
department.printMeeting();
department.generateReports(); // error: method doesn't exist on declared abstract type
```

### 高级技巧

#### 构造函数

当你在TypeScript里声明了一个类的时候，实际上同时声明了很多东西。 首先就是类的 *实例*的类型。

```typescript
class Greeter {
    //属性
    greeting: string;
    //构造方法--在实例时候会被调用
    constructor(message: string) {
        this.greeting = message;
    }
    //方法
    greet() {
        return "Hello, " + this.greeting;
    }
}
//声明了greeter的实例类型是Greeter
let greeter: Greeter;
greeter = new Greeter("world");
console.log(greeter.greet());
```

这里，我们写了`let greeter: Greeter`，意思是`Greeter`类的实例的类型是`Greeter`。 这对于用过其它面向对象语言的程序员来讲已经是老习惯了。

我们也创建了一个叫做*构造函数*的值。这个函数会在我们使用 `new`创建类实例的时候被调用。下面我们来看看，上面的代码被编译成JavaScript后是什么样子的：

```javascript
//一个自调用方法
let Greeter = (function () {
    //实际上是一个原型对象
    function Greeter(message) {
        this.greeting = message;
    }
    //将方法放在原型上
    Greeter.prototype.greet = function () {
        return "Hello, " + this.greeting;
    };
    //返回这方法
    return Greeter;
})();

let greeter;
greeter = new Greeter("world");
console.log(greeter.greet());
```

上面的代码里，`let Greeter`将被赋值为构造函数。我们当调用 `new`并执行了这个函数后，便会得到一个类的实例。这个构造函数也包含了类的所有静态属性。换个角度说，我们可以认为类具有 *实例部分*与*静态部分*这两个部分。

让我们稍微改写一下这个例子，看看它们之前的区别：

```typescript
class Greeter {
    //静态属性
    static standardGreeting = "Hello, there";
    //公有属性
    greeting: string;
    greet() {
        if (this.greeting) {
            return "Hello, " + this.greeting;
        }
        else {
            return Greeter.standardGreeting;
        }
    }
}

let greeter1: Greeter;
greeter1 = new Greeter();
console.log(greeter1.greet());
//创建了一个greeterMaker变量，保存Greeter的构造函数
//通过typeof Greeter意思是取招待员类的类型，而不是实例的类型 -- 构造函数的类型
let greeterMaker: typeof Greeter = Greeter;
greeterMaker.standardGreeting = "Hey there!";

let greeter2: Greeter = new greeterMaker();
console.log(greeter2.greet());
```

### 把类当做接口使用

类定义会创建两个东西：类的实例类型和一个构造函数。因为类可以创建出类型，所以你能够在允许使用接口的地方使用类。

```typescript
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3};
```

