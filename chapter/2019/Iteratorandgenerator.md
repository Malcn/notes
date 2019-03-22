# 迭代器与生成器

在JavaScript中一个迭代器是一个对象，该对象有一个next()方法，每次调用都返回一个结果对象({done,value}),done表示序列中的值已经消费完毕，value表示迭代值

举个栗子

```bash
function makeRangeIterator(start = 0, end = Infinity, step = 1) {
    let nextIndex = start;
    let iterationCount = 0;

    const rangeIterator = {
       next: function() {
           let result;
           if (nextIndex <= end) {
               result = { value: nextIndex, done: false }
               nextIndex += step;
               iterationCount++;
               return result;
           }
           return { value: iterationCount, done: true }
       }
    };
    return rangeIterator;
}
let it = makeRangeIterator(1, 10, 2);

let result = it.next();
while (!result.done) {
 console.log(result.value); // 1 3 5 7 9
 result = it.next();
}

console.log("Iterated over sequence of size: ", result.value); // 5
```

## Iterables

> ES6 开始，从一个 iterable 中提取迭代器的方法是：iterable 必须支持一个函数，其名称是专门的 ES6 符号值 `Symbol.iterator`。调用这个函数时，它会返回一个迭代器。

如果对象想要可遍历需要实现`Iterator 接口`，下面是定义一个可迭代对象：

```bash
var obj = {
    items:[1,3,5,7,9],
    [Symbol.iterator](){
        var i = 0;
        return {
            next: () => {
                var done = (i >= this.items.length);
                var value = !done ? this.items[i++] : undefined;
                return {
                    done: done,
                    value: value
                };
            }
        };
    }
}

for (let value of obj) {
    console.log(value);
}
// 1,3,5,7,9
```

> [ .. ] 语法被称为计算属性名，Symbol.iterator 是 ES6 预定义的特殊Symbol 值之一

for..of 循环期望 obj 是 iterable，于是它寻找并调用它的 Symbol.iterator 函数。

## 生成器

生成器是一种返回迭代器的函数,一般一个函数一旦开始执行，就会运行到结束，期间不会有其他代码能够打断它并插入其间，生成器并不符合这种运行到结束的特性。

```bash
function* foo(x) {
    var y = x * (yield 1);
    return y;
}
var it = foo(6);
// 启动foo(..)
it.next();
var res = it.next(7);
res.value; // 42
```

首先传入6作为参数 x 然后执行 it.next(),这会启动 *foo(..)
> 在 *foo(..) 内部，开始执行语句 var y = x ..，但随后就遇到了一个 yield 表达式。它
就会在这一点上暂停 *foo(..)，并在本质上要求调用代码为 yield
表达式提供一个结果值。接下来，调用 it.next( 7 )，这一句把值 7 传回作为被暂停的
yield 表达式的结果。所以，这时赋值语句实际上就是 var y = 6 * 7。现在，return y 返回值 42 作为调用
it.next( 7 ) 的结果。

* 第一个 next(..) 总是启动一个生成器，并运行到第一个 yield 处
* 第二个 next(..) 调用完成第一个被暂停的 yield 表达式，也就是设置上一个yield表达式的结果值
* 第三个 next(..) 设置第二个yield表达式的值，以此类推..

如果把yield和return一起使用的话， 那么return的值也会作为最后的返回值， 如果return语句后面还有yield， 那么这些yield不生效：

```bash
function* foo() {
    yield 0;
    yield 1;
    return 2;
    yield 3;
};
let g = foo();
console.log(g.next(),g.next(),g.next(),g.next());
//输出：{ value: 0, done: false } { value: 1, done: false } { value: 2, done: true } { value: undefined, done: true }
```

## 生成器+Promise

> yield 出来一个 Promise，侦听这个 promise 的决议（完成或拒绝），通过这个 Promise 来控制生成器的迭代器,

支持 Promise 的 Generator Runner

```bash
function run(gen) {
    var args = [].slice.call(arguments, 1),
        it;
    // 在当前上下文中初始化生成器
    it = gen.apply(this, args);
    // 返回一个promise用于生成器完成
    return Promise.resolve().then(function handleNext(value) {
        // 对下一个yield出的值运行
        var next = it.next(value);
        return (function handleResult(next) {
            // 生成器运行完毕了吗？
            if (next.done) {
                return next.value;
            }
            // 否则继续运行
            else {
                return Promise.resolve(next.value)
                    .then(
                        // 成功就恢复异步循环，把决议的值发回生成器
                        handleNext,
                        // 如果value是被拒绝的 promise，
                        // 就把错误传回生成器进行出错处理
                        function handleErr(err) {
                            return Promise.resolve(
                                it.throw(err)
                            ).then(handleResult);
                        }
                    );
            }
        })(next);
    });
}

function requert() {
    return Promise.resolve(1);
}

function requert2() {
    return Promise.resolve(2);
}

function* main() {
    const res = yield requert();
    const res2 = yield requert2();
    console.log(res, res2);
}

run(main);
```

promise并发请求

```bash
function* main() {
    const res =  requert();
    const res2 = requert2();

    const r1=yield res;
    const r2 = yield res2;
    console.log(r1, r2);
}
// or
function* main() {
    const res =  requert();
    const res2 = requert2();
    var results = yield Promise.all( [res,res2] );

    const r1 = results[0];
    const r2 = results[1];
    console.log(r1, r2);
}
```

## ES6 之前的生成器

思考下面代码如何转换成ES5

```bash
// request(..)是一个支持Promise的Ajax工具
function* foo(url) {
    try {
        console.log("requesting:", url);
        var val = yield request(url);
        console.log(val);
    } catch (err) {
        console.log("Oops:", err);
        return false;
    }
}
var it = foo("http://some.url.1");
```

手动转换，根据生成器的定义大概轮廓：

```bash
function foo(url) {
    // ..
    // 构造并返回一个迭代器
    return {
        next: function (v) {
            // ..
        },
        throw: function (e) {
            // ..
        }
    };
}
```

生成器的几种状态：1.初始状态触发request请求 2.request请求成功后的状态 3.是 request(..) 失败的状态。

```bash
function foo(url) {
    // 管理生成器状态
    var state;
    // 生成器变量范围声明
    var val;

    function process(v) {
        switch (state) {
            case 1:
                console.log("requesting:", url);
                return request(url);
            case 2:
                val = v;
                console.log(val);
                return;
            case 3:
                var err = v;
                console.log("Oops:", err);
                return false;
        }
    }
    // 构造并返回一个生成器
    return {
        next: function (v) {
            // 初始状态
            if (!state) {
                state = 1;
                return {
                    done: false,
                    value: process()
                };
            }
            // yield成功恢复
            else if (state == 1) {
                state = 2;
                return {
                    done: true,
                    value: process(v)
                };
            }
            // 生成器已经完成
            else {
                return {
                    done: true,
                    value: undefined
                };
            }
        },
        "throw": function (e) {
            // 唯一的显式错误处理在状态1
            if (state == 1) {
                state = 3;
                return {
                    done: true,
                    value: process(e)
                };
            }
            // 否则错误就不会处理，所以只把它抛回
            else {
                throw e;
            }
        }
    };
}
var it=foo();
console.log(it);
console.log(it.next());
console.log(it.next());
```

## ES7：async 与 await

```bash
function requert() {
    return Promise.resolve(1);
}

function requert2() {
    return Promise.resolve(2);
}

async function main() {
    const res = await requert();
    const res2 = await requert2();
    console.log(res, res2);
}

main();
```
