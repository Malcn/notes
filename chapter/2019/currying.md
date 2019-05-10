# 函数柯里化

柯里化, 即 Currying 的音译。 Currying 是编译原理层面实现多参函数的一个技术。

Currying 为实现多参函数提供了一个递归降解的实现思路——把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数。通俗来说只传递给函数一部分参数来调用它，让它返回一个函数去处理剩下的参数。

> 柯里化又称部分求值，其含义是给函数分步传递参数，每次传递参数后部分应用参数，并返回一个更具体的函数接受剩下的参数，这中间可嵌套多层这样的接受部分参数函数，直至返回最后结果。
因此柯里化的过程是逐步传参，逐步缩小函数的适用范围，逐步求解的过程。

**举个栗子：**

我们对`add`函数进行柯里化

```bash
function add(x, y) {
  return x + y;
}

function curryAdd(x){
    return function(y){
        return x+y;
    }
}

curryAdd(3)(5);

```

## Currying 使用场景

### 参数复用

上文中对add方法来讲固定第一个参数为3之后，该方法就变成了一个将接收的变量值加3的一个方法，复用了参数X

### 提前确认

兼容现代浏览器以及IE浏览器的事件添加方法是一个常见的场景

```bash
var addEvent = function(el, type, fn, capture) {
    if (window.addEventListener) {
        el.addEventListener(type, function(e) {
            fn.call(el, e);
        }, capture);
    } else if (window.attachEvent) {
        el.attachEvent("on" + type, function(e) {
            fn.call(el, e);
        });
    }
};
```

很明显上面的代码每次添加事件的时候都会走一遍if else 判断，怎样只要一次判定就可以了呢，没错就是柯里化。下面我们来改写上面的方法

```bash
var addEvent = (function () {
    if (window.addEventListener) {
        return function (el, sType, fn, capture) {
            el.addEventListener(sType, function (e) {
                fn.call(el, e);
            }, (capture));
        };
    } else if (window.attachEvent) {
        return function (el, sType, fn, capture) {
            el.attachEvent("on" + sType, function (e) {
                fn.call(el, e);
            });
        };
    }
})();
```

### 延迟执行

我们js中经常使用的bind，实现的机制就是Currying。

```bash
Function.prototype.bind = function(context) {
  var _this = this;
  var args = [].slice.call(arguments, 1);

  return function() {
    return _this.apply(context, args);
  };
};
```

## 通用的方法

```bash
const curry = (fn, ...arg) => {
  let all = arg || [],
    length = fn.length;
  return (...rest) => {
    let _args = all.slice(0);
    _args.push(...rest);
    if (_args.length < length) {
      return curry.call(this, fn, ..._args);
    } else {
      return fn.apply(this, _args);
    }
  };
};
let test = curry(function(a, b, c) {
  console.log(a + b + c);
});

test(1, 2)(3);
test(1)(2)(3);
```

### 总结

* 为函数式编程而生，函数式编程及其思想，是值得关注、学习和应用的事物,推荐阅读[mostly-adequate-guide](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/)

* Currying 的思想极大地助于提升函数的复用性。

## 反柯里化

反柯里化的作用在与扩大函数的适用性，使本来作为特定对象所拥有的功能的函数可以被任意对象所用

反柯里化的形式化描述:

`obj.func(arg1, arg2)=>func(obj, arg1, arg2)`

### 通用反柯里化函数

```bash
var uncurrying= function (fn) {
    return function () {
        var args=[].slice.call(arguments,1);
        return fn.apply(arguments[0],args);
    }
};
```

使用时 调用 uncurrying 并传入一个现有函数 fn， 反柯里化函数会返回一个新函数，该新函数接受的第一个实参将绑定为 fn 中 this的上下文，其他参数将传递给 fn 作为参数。

所以，对反柯里化更通俗的解释可以是： **函数的借用**，是函数能够接受处理其他对象，通过借用泛化、扩大了函数的使用范围。

举个栗子：

```bash
var obj = {};
var pushUncurrying = uncurrying(Array.prototype.push);
obj.push = function (obj) {
    pushUncurrying(this,obj);
};
obj.push('first');
console.log(obj); // {0: "first", push: ƒ, length: 1}

```

我们知道对象是没有 push 方法的但是我们可以借用 Array 来处理 push,由原生的数组方法（js引擎）来维护 伪数组对象的 length 属性和数组成员。

同理我们还可以接着添加：

```bash
var indexof=uncurrying(Array.prototype.indexOf);
obj.indexOf = function (obj) {
    return indexof(this,obj);
};
obj.push("second");
console.log(obj.indexOf('first'));   // 0
console.log(obj.indexOf('second'));  // 1
```

#### uncurrying还可以这么写：

```bash
var uncurrying= function (fn) {
    return function () {
        var context=[].shift.call(arguments);
        return fn.apply(context,arguments);
    }
};

var uncurrying= function (fn) {
    return function () {
        return Function.prototype.call.apply(fn,arguments);
    }
};

var uncurrying= function (fn) {
    return function(context,...args){
        return fn.call(context,...args);
    }
};

// 终极无敌高逼格版本，对于个人而言还是喜欢上一个版本，简单清晰明了
var uncurrying=Function.prototype.bind.bind(Function.prototype.call);
```

### 函数借用的前提是函数语言上需要支持鸭子类型

****当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。****