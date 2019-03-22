# 原型与原型链

JavaScript 中，万物皆对象，对象又分为：普通对象和函数对象，也就是Object 和 Function

## 基本概念

* 对象：属性和方法的集合，即变量和函数的封装。每个对象都有一个`__proto__`属性，指向这个对象的构造函数的原型对象
* 构造器函数：用于创建对象的函数，通过new关键字生成对象(构造函数的首字母必须大写，用来区分于普通函数)
* 原型对象：每个函数都有一个`prototype`属性，它是一个指向原型对象的指针
* 每个对象都有 `__proto__` 属性，但只有函数对象才有 `prototype` 属性

## 普通对象与函数对象

举个栗子：

```bash

function f1(){};
var f2 = function();
var f3 = new Function('args','alert(args)');

// 函数对象 f1，f2,f3

var c1 = {};
var c2 = new Object();
var c3 = new function(){};

// 普通对象 c1，c2，c3
```

凡是通过new Function()创建的对象都是函数对象，其他的都是普通对象。Function Object 也都是通过 New Function()创建的所以它们也是函数对象

> 注意：尽管可以使用 Function 构造函数创建函数，但最好不要使用它，因为用它定义函数比用传统方式要慢得多。不过，所有函数都应看作 Function 类的实例。

## 构造函数

构造函数始终都应该以一个大写字母开头；构造函数本身也是函数，只不过可以用来创建对象；要创建构造函数的新实例，必须使用new操作符

举个栗子：

```bash
function Person(name){
    this.name = name;
    this.sayHello=function(){
        console.log(`hello ${this.name}`)
    }
}
var girl = new Person("lili");
console.log(girl.constructor===Person); //true
// constructor属性存在于原型对象中，他指向了构造函数
girl.sayHello(); //hello lili
```

构造函数调用所经历的步骤：

* 创建一个新的对象
* 将构造函数的作用域赋给新对象（因此this指向了这个新对象）
* 执行构造函数中的代码（为这个新对象添加属性）
* 返回新对象

> 构造函数存在的问题：每个方法都要在每个实例上重新创建一遍。（每定义一个函数就是实例化了一个对象），不同实例上的同名函数是不相等的。其解决方法便是使用原型模型

## 原型对象

> 每一个函数都有prototype属性，它指向一个对象，即原型对象。

```bash
function Person(name){
    this.name=name
}
var p1=new Person('lisi')
var p2=new Person('lili')

Person.prototype.constructor === Person
```

 constructor指向构造函数，每个原型对象 ( `Person.prototype` ) 都有一个constructor属性，指向prototype属性所在的函数 ( Person )

## 原型链

JS 在创建对象（不论是普通对象还是函数对象）的时候，都有一个叫做 `__proto__` 的内置属性，用于指向创建它的函数对象的原型对象 `prototype`

```bash
function Person(name) {
    this.name = name
}
Person.prototype.say = () => {
    console.log('Hi !');
}
var p1 = new Person('lisi');
p1.say();

p1.__proto__ === Person.prototype
```

### 属性搜索原则

对象在访问属性的时候, 首先在当前对象中查找，找到就停止查找直接使用该属性，如果没有找到则顺着`__proto__`属性向上查找，直到查找到原型链的顶端 `Object.prototype.__proto__`

这种由 `__proto__` 属性连接而成的链条就是 原型链

### 所有函数对象的 `__proto__` 都指向Function.prototype，它是一个空函数（Empty function）

```bash
Number.__proto__ === Function.prototype  // true
Boolean.__proto__ === Function.prototype // true
String.__proto__ === Function.prototype  // true
Array.__proto__ === Function.prototype   // true

// 所有的构造器都来自于Function.prototype，甚至包括根构造器Object及Function自身

Object.__proto__ === Function.prototype  // true
Function.__proto__ === Function.prototype // true

// 唯一一个typeof XXX.prototype为 function 的 prototype
console.log(typeof Function.prototype) // function
console.log(typeof Object.prototype)   // object
console.log(typeof Number.prototype)   // object
console.log(typeof Boolean.prototype)  // object
console.log(typeof String.prototype)   // object
console.log(typeof Array.prototype)    // object

console.log(Function.prototype) // 空函数（Empty function）

```

### 总结

```bash
Object.__proto__ === Function.prototype // true
Function.__proto__ === Function.prototype // true
```

 Object 与 Function 都是函数对象，都是通过new Function()创建的，所以:  
 `Object.__proto__ 指向 Function.prototype`，  
 `Function.__proto__ 指向 Function.prototype`，  

 结论：所有函数对象的 `__proto__` 都指向 `Function.prototype`

 ```bash
Function.prototype.__proto__ === Object.prototype  //true

Object.prototype.__proto__  //null
 ```

`Function.prototype` 这个函数对象比较特殊，它的 `__proto__` 指向了 `Object.prototype`，而 `Object.prototype` 的 `__proto__` 指向了 `null`

---

## JS继承

### 寄生组合继承

思想：通过寄生方式，砍掉父类的实例属性，避免了组合继承的缺点（调用两次父类构造函数）

> 组合继承是继承的一种实现方式，下面会有介绍

```bash
function Foo(name) {
    this.name = name;
}
Foo.prototype.sayName = function () {
    console.log(this.name);
};

function Bar(name, age) {
    Foo.call(this, name);
    this.age = age;
}
// 我们创建了一个新的 Bar.prototype 对象并关联到 Foo.prototype
Bar.prototype = Object.create(Foo.prototype);
// 注意！现在没有 Bar.prototype.constructor 了
// 如果你需要这个属性的话可能需要手动修复一下它
// Bar.prototype.constructor = Bar
Bar.prototype.sayAge = function () {
    console.log(this.age);
};
var a = new Bar("lili", 18);
a.sayName(); // lili
a.sayAge(); // 18
```

`Object.create(Foo.prototype)`里面的实现机制类似下面这样

```bash
function object(o) {
    function F() {};
    F.prototype = o;
    return new F();
}
```

// ES6 之前  
`Bar.ptototype = Object.create( Foo.prototype );`  

// ES6新增了一个方法，Object.setPrototypeOf，可以直接创建关联，而且不用手动添加constructor属性  
`Object.setPrototypeOf( Bar.prototype, Foo.prototype );`
`console.log(Bar.prototype.constructor === Bar) // true`

下面介绍一些其他方式实现继承，同时也是 寄生组合继承 的演进过程

### 原型链继承

思想：子类的原型指向父类的实例

```bash
function Foo() {
    this.name = 'lisi';
    this.arr = [1, 2];
    this.sayName = function () {
        console.log(this.name);
    }
}

function Bar() {

}
Bar.prototype = new Foo();

var s1 = new Bar();
var s2 = new Bar();
console.log(s1.arr); // [1, 2]
s1.arr.push(3);
console.log(s1.arr); // [1, 2, 3]
console.log(s2.arr); // [1, 2, 3]

s1.sayName(); // lisi
s1.name = 'zhangsan';
s1.sayName(); //zhangsan
```

缺点：

* 无法向父类的构造函数传参
* 来自原型对象的引用属性是所有实例共享的

### 借用构造函数继承

思想：借用父类的构造函数来增强子类实例，等于是把父类的实例属性复制一份给子类实例装上

```bash
function Foo(name) {
    this.name = name;
    this.arr = [1, 2];
    this.sayName = function () {
        console.log(this.name);
    }
}

function Bar(name) {
    Foo.call(this);
}

var s1 = new Bar();
var s2 = new Bar();
console.log(s1.arr); // [1, 2]
s1.arr.push(3);
console.log(s1.arr); // [1, 2, 3]
console.log(s2.arr); // [1, 2]
```

缺点：

* 只能继承父类的实例属性和方法，不能继承原型属性/方法
* 无法实现函数的复用，每个子类都有父类实例函数的副本

### 组合继承

思想：通过调用父类构造，继承父类的属性并保留传参的优点，然后通过将父类实例作为子类原型，实现函数复用

```bash
function Foo(name) {
    this.name = name;
    this.arr = [1, 2];
}

Foo.prototype.sayName = function () {
    console.log(this.name);
}

function Bar(name) {
    Foo.call(this, name); // 第二次调用
}

Bar.prototype = new Foo(); // 第一次调用
Bar.prototype.constructor = Bar;

var s1 = new Bar('zhangsan');
var s2 = new Bar('lisi');
console.log(s1.arr); // [1, 2]
s1.arr.push(3);
console.log(s1.arr); // [1, 2, 3]
console.log(s2.arr); // [1, 2]

s1.sayName(); // zhangsan
s2.sayName(); // lisi
```

缺点：

* 调用了两次父类的构造函数

### 原型式继承

思想：借助原型，可以基于已有的对象创建新对象

```bash
function object(o) {
    function F() {};
    F.prototype = o;
    return new F();
}

var obj = {
    name: 'lili'
};

var p = object(obj)
console.log(p.name)
//这样 p 对象就可以拿到 obj 的属性了
```

ES5新增了方法规范化了原型式继承。即： `Object.create()`

### 寄生式继承

思想：在继承对象的同时通过添加属性与方法增强这个新对象，原型式继承的加强版

```bash
function createAnother(o) {
    var obj = Object.create(o); // 通过调用函数创建一个新对象
    obj.sayHi = function () { // 给这个对象添加方法
        console.log("hi");
    }
    return obj; //返回这个新对象
}

function Foo() {}
Foo.prototype.sayName = function () {
    console.log(this.name);
}
var now = createAnother(Foo.prototype);
now.name = 'lili'
now.sayName(); // lili
now.sayHi(); // hi
```

## ES6中的继承

ES6中可以声明类（ class ）提供了extends关键字实现类的继承，定义的类只是语法糖，目的是让我们用更简洁明了的语法创建对象及处理相关的继承。

```bash
class Foo {
    constructor(name) {
        this.name = name
    }
    sayName() {
        console.log(this.name)
    }
}
class Bar extends Foo {
    constructor(name, age) {
        super(name); // 调用父类的constructor(x)
        this.age = age
    }
    sayHi() {
        console.log('Hi my name is ');
        super.sayName();
    }
}

var s1 = new Bar('lili', 18);
s1.sayHi();
```

### super 关键字

* 作为函数调用时（ 即super(...args)）， super代表父类的构造函数。
* 作为对象调用时（ 即super.method() ）， super代表父类。此时super即可以引用父类实例的属性和方法， 也可以引用父类的静态方法。
* 子类必须在constructor方法中调用super方法，并且在使用this之前调用

> ES5 的继承， 实质是先创造子类的实例对象this， 然后再将父类的方法添加到this上面（ base.call(this)）  
ES6中的继承机制完全不同，实质是先创造父类的实例对象this（ 所以必须先调用super方法，返回父类实例,如果不调用super方法， 子类就得不到this对象）， 然后再用子类的构造函数修改this。

### 类的原型( prototype )

```bash
Bar.__proto__ === Foo // true
Bar.prototype.__proto__ === Foo.prototype //true
```

这样的结果是因为， 类的继承是按照下面的模式实现的。

```bash
class Foo {

}
class Bar {

}

// B 的实例继承 A 的实例
Object.setPrototypeOf(Bar.prototype, Foo.prototype);
// B 继承 A 的静态属性
Object.setPrototypeOf(Bar, Foo);
```

Object.setPrototypeOf方法的实现:

```bash
Object.setPrototypeOf = function (obj, proto) {
    obj.__proto__ = proto;
    return obj;
}
```

### Extends 的继承目标

extends关键字后面可以跟多种类型的值  
`class Bar extends F`

`F` 只要是一个有`prototype`属性的函数， 就能被 `Bar` 继承。 由于函数都有`prototype`属性（ 除了Function.prototype函数）， 因此`F`可以是任意函数。

> `Function.prototype.prototype // undefined`

#### 分析：

```bash
class Foo {}
Foo.__proto__ === Function.prototype // true
Foo.prototype.__proto__ === Object.prototype // true

//ES5
var Foo = function () {

}
Foo.__proto__ === Function.prototype // true
Foo.prototype.__proto__ === Object.prototype // true

```

`Foo`就是一个普通函数，所以直接继承 `Funciton.prototype` 但是， `Foo` 调用后返回一个空对象（ 即Object实例）， 所以`Foo.prototype.__proto__`指向构造函数（ Object） 的prototype属性。

---

```bash
class Bar extends null {}
Bar.__proto__ === Function.prototype // true
Bar.prototype.__proto__ === undefined // true
```

`Bar`是一个普通函数， 所以直接继承 `Funciton.prototype` 但是 `Bar` 调用后返回的对象不继承任何方法， 所以它的 `__proto__` 指向`Function.prototype`， 即实质上执行了下面的代码:

```bash
class Bar extends null {
    constructor() {
        return Object.create(null);
    }
}
```

---

`Object.getPrototypeOf`方法可以用来从子类上获取父类。

```bash
Object.getPrototypeOf(Bar)===Foo //true
```

### 实例的 `__proto__` 属性

```bash
class Foo {
}
class Bar extends Foo {
}

var p1=new Foo();
var p2=new Bar();


console.log(p2.__proto__ === Bar.prototype); //true
console.log(Bar.prototype.__proto__===Foo.prototype); //true
console.log(p2.__proto__.__proto__===p1.__proto__); //true

```

### 原生构造函数的继承

原生构造函数是指语言内置的构造函数， 通常用来生成数据结构。 ECMAScript 的原生构造函数大致有下面这些

```bash
Boolean，Number，String，Array，Date，Function，RegExp，Error，Object
```

```bash
class MyArray extends Array {}
var arr = new MyArray();
arr[0] = 1;
console.log(arr); //  [1]
```

注意， 继承Object的子类， 有一个行为差异。

```bash
class NewObj extends Object {
    constructor() {
        super(arguments);
    }
}
var p=Object({ age: 18});
var p1 = new NewObj({
    age: 18
});
console.log(p); // Object {age: 18}
console.log(p1); // NewObj {}
```

上面代码中， `NewObj`继承了`Object`， 但是无法通过`super`方法向父类`Object`传参。 这是因为 ES6 改变了`Object`构造函数的行为， 一旦发现Object方法不是通过 `new Object()` 这种形式调用， ES6 规定`Object`构造函数会忽略参数。