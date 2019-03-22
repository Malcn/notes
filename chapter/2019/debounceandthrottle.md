# 函数防抖(debounce)与函数节流(throttle)

## debounce

定义：如果一个函数持续地触发，那么只在它结束后过一段时间执行一次。

基本实现：

```bash
function debounce(func, delay = 500) {
    var timeout;
    return function (e) {
        clearTimeout(timeout);
        var context = this,
            args = arguments;
        timeout = setTimeout(function () {
            func.apply(context, args);
        }, delay)
    };
};
var func = debounce(function (v) {
    console.log('--', v)
});

var i = 0;
function change() {
    func(i);
    setTimeout(() => {
        i < 10 && change();
    }, i*100);
    i++;
}
change();
// 5,6,7,8,9
```

## throttle

定义：如果一个函数持续的，频繁地触发，那么让它在一定的时间间隔后再触发

```bash
function throttle(fn, wait) {
  var timeout;
  var start = Date.now();
  var wait = wait || 100;
  return function() {
    var context = this,
      args = arguments,
      curr = new Date() - 0;
    clearTimeout(timeout);
    if (curr - start >= wait) {
      console.log("now", curr, curr - start); //注意这里相减的结果，都差不多是100左右
      fn.apply(context, args); //只执行一部分方法，这些方法是在某个时间段内执行一次
      start = curr;
    } else {
      //让方法在脱离事件后也能执行一次
      timeout = setTimeout(function() {
        fn.apply(context, args);
      }, wait);
    }
  };
}
var func = throttle(function(v) {
  console.log("---", v);
});
var i = 0;
while (i < 10000) {
  func(i);
  i++;
}
```