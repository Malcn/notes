# bind, call, apply总结

* bind, call, apply 三者都是用来改变函数的this对象的指向的，第一个参数都是this要指向的对象，也就是想指定的上下文

* bind 是返回对应函数，便于稍后调用，apply 、call 则是立即调用 。

## bind介绍

bind() 方法会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，传入 bind() 方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。举个栗子：

```bash
function foo(c, d) {
    this.b = 100
    console.log(this.a)
    console.log(this.b)
    console.log(c)
    console.log(d)
}
// 我们将foo bind到{a: 1}
var func = foo.bind({ a: 1 }, '1st');
func('2nd'); // 1 100 1st 2nd
// 即使再次call也不能改变this。
func.call({ a: 2 }, '3rd'); // 1 100 1st 3rd


// 当 bind 返回的函数作为构造函数的时候，
// bind 时指定的 this 值会失效，但传入的参数依然生效。
// 所以使用func为构造函数时，this不会指向{a: 1}对象，this.a的值为undefined。如下

// new func('4th'); //undefined 100 1st 4th

```

基本实现：

```bash
Function.prototype.mybind = function (context) {
    if (typeof this !== "function") {
        throw new TypeError("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var aArgs = [].slice.call(arguments, 1),
        self = this,
        F = function () {},
        fBound = function () {
            return self.apply(
                this instanceof F ? this : context || window,
                aArgs.concat([].slice.call(arguments))
            );
        };

    F.prototype = this.prototype;
    fBound.prototype = new F();
    return fBound;
};
```

## call与apply

在 javascript 中，call 和 apply 都是为了改变某个函数运行时的上下文（context）而存在的，也就是说为了改变函数体内部 `this` 的指向

```bash
var func = function (arg1, arg2) {};
func.call(this, arg1, arg2);
func.apply(this, [arg1, arg2])

```

基本实现：

```bash
Function.prototype.mycall = function (context) {
    var content = context || window;
    content.fn = this;
    var args = [];
    for (var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }
    // 这里 args 会自动调用 Array.toString() 这个方法。
    var result = eval('content.fn(' + args + ')');
    delete content.fn;
    return result;
}

Function.prototype.myapply = function (context, arr) {
    var context = context || window;
    context.fn = this;

    var result;
    if (!arr) {
        result = context.fn();
    } else {
        var args = [];
        for (var i = 0, len = arr.length; i < len; i++) {
            args.push('arr[' + i + ']');
        }
        result = eval('context.fn(' + args + ')')
    }

    delete context.fn
    return result;
}

function add(c, d) {
    return this.a + this.b + c + d;
}

const obj = {
    a: 1,
    b: 2
};

console.log(add.call(obj, 3, 4)); // 10
console.log(add.mycall(obj, 3, 4)); // 10
console.log(add.myapply(obj, [3, 4])); // 10
```

## 常见应用场景

### 合并两个数组

```bash
var arr1 = ['1', '2'];
var arr2 = ['3', '4'];

// 将第二个数组融合进第一个数组
// 相当于 arr1.push('3', '4');
[].push.apply(arr1, arr2);

console.log(arr1);
// ['1', '2', '3', '4']
```

需要注意的是 js引擎对函数参数有最大限制，JS核心限制在 65535，有些引擎会抛出异常，有些不抛出异常但丢失多余参数。

### 获取数组中的最大值或最小值

```bash
var arr = [1, 120, 35, -100];
Math.max.apply(Math, arr); //120

// ES6
Math.max.call(Math, ...arr); // -100
```

### 类数组对象转数组

```bash
let arr = [].slice.call(arguments);

// ES6:
let arr = Array.from(arguments);
let arr = [...arguments];
```

类数组对象：

* 类数组对象是一个对象
* 具有指向对象元素的数字索引下标和 length 属性
* arguments 对象，还有DOM API 返回的 NodeList 对象都属于类数组对象

为什么要有类数组对象呢?

* 对象通过属性访问数据的时间复杂度是 O(1) 所以可以更快的操作复杂数据