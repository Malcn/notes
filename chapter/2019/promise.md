# 异步：现在与将来

> 异步与并行常常被混为一谈。记住，异步是关于现在和将来的时间间隙，而并行是关于能够同时发生的事情。

javascript 是基于事件循环(eventLoop)来处理异步的，那么什么是事件循环？先看一段伪代码

```bash
var eventLoop=[];
var event;
while(true){
    // 一次tick
    if(eventLoop.length > 0){
        // 拿到队列中的下一个事件
        event = eventLoop.shift();
        // 现在，执行下一个事件
        try{
            event();
        }catch(err) {
            reportError(err);
        }
    }
}
```

> 循环的每一轮称为一个tick，每一次tick会从队列中取出一个事件并执行,这些事件就是回调函数,对于setTimeout(..)并没有立即把回调函数放入时间循环队列中，而是等定时器到时后把回调函数放到eventLoop队列的队尾,如果队列中已存在一些事件，回调函数就会等待，所以setTimeout(..)定时器的精度可能不高。

## 任务

在ES6中,有一个新的概念建立在事件循环队列之上,叫做任务队列(job queue)，它是挂在事件循环队列的每一个tick之后的一个队列。就好比在银行排队办理业务，轮到你办理业务的时候你突然想起来还有一个事要办，这个时候你会告知柜台还有件事要办而不是再次去排队，这个事就是一个任务，是插入到一次tick的后面而不是插入到事件队列的队尾。

>一个任务可能引起更多任务被添加到同一个队列末尾所以，理论上说，任务循环（job loop）可能无限循环（一个任务总是添加另一个任务，以此类推），进而导致无法转移到下一个事件循环 tick。

关于事件循环与任务，可以参考另一片文章[事件循环与任务队列](https://malcn.gitbook.io/notes/js/eventloop)

## 承诺 (Promise)

类型检查（typecheck）一般用术语鸭子类型（duck typing）来表示——“如果它看起来像只鸭子，叫起来像只鸭子，那它一定就是只鸭子”。于是识别Promise类似下面这种方式

```bash
if (
 p !== null &&
 (
 typeof p === "object" ||
 typeof p === "function"
 ) &&
 typeof p.then === "function"
) {
 // 假定这是一个Promise
}
else {
 // 不是Promise
}
```

使用promise来规避回调金字塔,ajax请求方法返回一个Promise对象

``` bash
// 假定工具ajax( {url}, {callback} )存在
// Promise-aware ajax
function request(url) {
 return new Promise(function(resolve,reject){
    // ajax(..)回调应该是我们这个promise的resolve(..)函数
    ajax( url, resolve );
 });
}

request("url").then(res=>{
  //ajax返回的数据
  console.log(res);
});
```

### Promise并行与竟态

并行执行异步任务

```bash
var p1 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 500, 'P1');
});
var p2 = new Promise(function (resolve, reject) {
    setTimeout(resolve, 600, 'P2');
});
// 同时执行p1和p2，并在它们都完成后执行then:
Promise.all([p1, p2]).then(function (results) {
    console.log(results); // ['P1', 'P2']
});
```

设置超时，Promise也提供了解决方案，其使用了一种称为竞态的高级抽象机制：

```bash
// 用于超时一个Promise的工具
function timeoutPromise(delay) {
 return new Promise( function(resolve,reject){
 setTimeout( function(){
 reject( "Timeout!" );
 }, delay );
 } );
}
// 设置foo()超时
Promise.race([
 foo(), // 试着开始foo()
 timeoutPromise( 3000 ) // 给它3秒钟
]).then(
 function(){
 // foo(..)及时完成！
 },
 function(err){
 // foo()被拒绝，或者只是没能按时完成
 // 查看err来了解是哪种情况
 }
);
```

### 是可信任的Promise吗？

如果向 Promise.resolve(..) 传递一个非 Promise、非 thenable 的立即值，就会得到一个用
这个值填充的 promise。下面这种情况下，promise p1 和 promise p2 的行为是完全一样的：

```bash
var p1 = new Promise( function(resolve,reject){
 resolve( 42 );
} );
var p2 = Promise.resolve( 42 );
```

而如果向 Promise.resolve(..) 传递一个真正的 Promise，就只会返回同一个 promise：

```bash
var p1 = Promise.resolve( 42 );
var p2 = Promise.resolve( p1 );
p1 === p2; // true
```

如果向 Promise.resolve(..) 传递了一个非 Promise 的 thenable 值，前者就会
试图展开这个值，而且展开过程会持续到提取出一个具体的非类 Promise 的最终值。

```bash
var p = {
 then: function(cb) {
 cb( 42 );
 }
};

Promise.resolve( p )
.then(
 function fulfilled(val){
 console.log( val ); // 42
 },
 function rejected(err){
 // 永远不会到达这里
 }
);
```

Promise.resolve(..) 可以接受任何 thenable，将其解封为它的非 thenable 值。从 Promise.
resolve(..) 得到的是一个真正的 Promise，是一个可以信任的值。如果你传入的已经是真
正的 Promise，那么你得到的就是它本身，所以通过 Promise.resolve(..) 过滤来获得可信
任性完全没有坏处。

从完成（或拒绝）处理函数返回 thenable 或者 Promise 的时候也会发生同样的展开。考虑：

```bash
var p = Promise.resolve( 21 );
p.then( function(v){
 console.log( v ); // 21
 // 创建一个promise并将其返回
 return new Promise( function(resolve,reject){
 // 用值42填充
 resolve( v * 2 );
 } );
} )
.then( function(v){
 console.log( v ); // 42
} );
```

虽然我们把 42 封装到了返回的 promise 中，但它仍然会被展开并最终成为链接的 promise
的决议，因此第二个 then(..) 得到的仍然是 42。如果我们向封装的 promise 引入异步，一
切都仍然会同样工作：

```bash
var p = Promise.resolve( 21 );
p.then( function(v){
 console.log( v ); // 21
 // 创建一个promise并返回
 return new Promise( function(resolve,reject){
 // 引入异步！
 setTimeout( function(){
 // 用值42填充
 resolve( v * 2 );
 }, 100 );
 } );
} )
.then( function(v){
 // 在前一步中的100ms延迟之后运行
 console.log( v ); // 42
} );
```

使链式流程控制可行的 Promise 固有特性:

* 调用 Promise 的 then(..) 会自动创建一个新的 Promise 从调用返回
* 在完成或拒绝处理函数内部，如果返回一个值或抛出一个异常，新返回的（可链接的）Promise 就相应地决议。
* 如果完成或拒绝处理函数返回一个 Promise，它将会被展开，这样一来，不管它的决议值是什么，都会成为当前 then(..) 返回的链接 Promise 的决议值。

### 术语：决议、完成以及拒绝

对于术语决议（resolve）、完成（fulfill）和拒绝（reject）

现在我们来解释为什么单词 resolve（比如在 Promise.resolve(..) 中）如果用于表达结果
可能是完成也可能是拒绝的话，既没有歧义，而且也确实更精确：

```bash
var rejectedTh = {
 then: function(resolved,rejected) {
 rejected( "Oops" );
 }
};
var rejectedPr = Promise.resolve( rejectedTh );
```

Promise.resolve(..) 会将传入的真正 Promise 直接返回，对传入的 thenable 则会展开。如果这个 thenable 展开得到一个拒绝状态，那么从 Promise.resolve(..) 返回的 Promise 实际上就是这同一个拒绝状态。所以对这个 API 方法来说Promise.resolve(..) 是一个精确的好名字，因为它实际上的结果可能是完成或拒绝。