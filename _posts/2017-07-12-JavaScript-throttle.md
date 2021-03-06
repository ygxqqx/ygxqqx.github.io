---  
published: true  
layout: post  
title: JavaScript函数节流与防抖
date: 2017-07-12  
category: work  
---  

在前端开发中，有时会为页面绑定resize事件，或者为一个页面元素绑定拖拽事件（其核心就是绑定mousemove），这种事件有一个特点，就是用户不必特地捣乱，他在一个正常的操作中，都有可能在一个短的时间内触发非常多次事件绑定程序。而大家知道，DOM操作时很消耗性能的，这个时候，如果你为这些事件绑定一些操作DOM节点的操作的话，那就会引发大量的计算，在用户看来，页面可能就一时间没有响应，这个页面一下子变卡了变慢了。甚至在IE下，如果你绑定的resize事件进行较多DOM操作，其高频率可能直接就使得浏览器崩溃。

怎么解决？函数节流就是一种办法。话说第一次接触函数节流(throttle)，还是在看impress源代码的时候，impress在播放的时候，如果窗口大小发生改变(resize)，它会对整体进行缩放(scale)，使得每一帧都完整显示在屏幕上。

## 函数节流的原理

函数节流的原理挺简单的，估计大家都想到了，那就是定时器。当我触发一个时间时，先setTimout让这个事件延迟一会再执行，如果在这个时间间隔内又触发了事件，那我们就clear掉原来的定时器，再setTimeout一个新的定时器延迟一会执行，就这样。

## 代码实现

明白了原理，那就可以在代码里用上了，但每次都要手动去新建清除定时器毕竟麻烦，于是需要封装。在《JavaScript高级程序设计》一书有介绍函数节流，里面封装了这样一个函数节流函数：

```
function throttle(method, context) {
    clearTimeout(methor.tId);
    method.tId = setTimeout(function() {
        method.call(context);
    }, 100);
}
```
它把定时器ID存为函数的一个属性。而调用的时候就直接写

```
window.onresize = function() {
    throttle(myFunc);
}
```

这样两次函数调用之间至少间隔100ms。
而impress用的是另一个封装函数：

```
var throttle = function(fn, delay){
    var timer = null;
    return function() {
        var context = this, args = arguments;
        clearTimeout(timer);
        timer = setTimeout(function() {
            fn.apply(context, args);
        }, delay);
    };
 };
```

它使用闭包的方法形成一个私有的作用域来存放定时器变量timer。而调用方法为

```
window.onresize = throttle(myFunc, 100);
```

两种方法各有优劣，前一个封装函数的优势在把上下文变量当做函数参数，直接可以定制执行函数的this变量；后一个函数优势在于把延迟时间当做变量（当然，前一个函数很容易做这个拓展），而且个人觉得使用闭包代码结构会更优，且易于拓展定制其他私有变量，缺点就是虽然使用apply把调用throttle时的this上下文传给执行函数，但毕竟不够灵活。

## 接下来是优化

函数节流让一个函数只有在你不断触发后停下来歇会才开始执行，中间你操作得太快它直接无视你。这样做就有点太绝了。resize一般还好，但假如你写一个搜索智能提示的程序，然后直接使用函数节流，那恭喜你，你会发现你在搜索框输入文字时是不会有智能提示的，只有到你输入完停下来等一小会儿时才有智能提示。

其实函数节流的出发点，就是让一个函数不要执行得太频繁，减少一些过快的调用来节流。当你改变浏览器大小，浏览器触发resize事件的时间间隔是多少？我不清楚，个人猜测是16ms（每秒64次），反正跟mousemove一样非常太频繁，一个很小的时间段内必定执行，这是浏览器设好的，你无法直接改。而真正的节流应该是在可接受的范围内尽量延长这个调用时间，也就是我们自己控制这个执行频率，让函数减少调用以达到减少计算、提升性能的目的。假如原来是16ms执行一次，我们如果发现resize时每50ms一次也可以接受，那肯定用50ms做时间间隔好一点。
而上面介绍的函数节流，它这个频率就不是50ms之类的，它就是无穷大，只要你能不间断resize，刷个几年它也一次都不执行处理函数。我们可以对上面的节流函数做拓展：

```
var throttleV2 = function(fn, delay, mustRunDelay) {
    var timer = null;
    var t_start;
    return function() {
        var context = this, args = arguments, t_curr = +new Date();
        clearTimeout(timer);
        if (!t_start) {
            t_start = t_curr;
        }
        if (t_curr - t_start >= mustRunDelay) {
            fn.apply(context, args);
            t_start = t_curr;
        } else {
            timer = setTimeout(function() {
                fn.apply(context, args);
            }, delay);
        }
    };
};
```

在这个拓展后的节流函数升级版，我们可以设置第三个参数，即必然触发执行的时间间隔。如果用下面的方法调用

```
window.onresize = throttleV2(myFunc, 50, 100);
```

则意味着，50ms的间隔内连续触发的调用，后一个调用会把前一个调用的等待处理掉，但每隔100ms至少执行一次。原理也很简单，打时间tag，一开始记录第一次调用的时间戳，然后每次调用函数都去拿最新的时间跟记录时间比，超出给定的时间就执行一次，更新记录时间。
