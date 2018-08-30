# throttle和debounce

https://www.cnblogs.com/wilber2013/p/5893426.html

### 概念

- debounce：把触发非常频繁的事件合并成一次执行
- throttle：设置一个阀值，在阀值内，把触发的事件合并成一次执行；当到达阀值，必定执行一次事件

### 应用场景

debounce

- 对于键盘事件，当用户输入比较频繁的时候，可以通过debounce合并键盘事件处理
- 对于ajax请求的情况，例如当页面下拉超过一定返回就通过ajax请求新的页面内容，这时候可以通过debounce合并ajax请求事件

throttle

- 对于键盘事件，当用户输入非常频繁，但是我们又必须要在一定时间内（阀值）内执行处理函数的时候，就可以使用throttle
  - 例如，一些网页游戏的键盘事件
- 对于鼠标移动和窗口滚动，鼠标的移动和窗口的滚动会带来大量的事件，但是在一段时间内又必须看到页面的效果
  - 例如对于可以拖动的div，如果使用debounce，那么div会在拖动停止后一下子跳到目标位置；这时就需要使用throttle

### 实现

```js
var debounce = function(action, delay) {
    var timer = null;
    return function() {
    	var self = this,args = arguments;
        timer&&clearTimeout(timer);
        timer = setTimeout(function(){
            action.apply(self, args);
        }, delay);
    }
}
```

```js
var throttle = function(action, delay) {
    var startTime = 0;
    return function() {
        var currentTime = new Date();
        if(currentTime - startTime>=delay) {
            action.apply(this, arguments);
            startTime = currentTime;
        }
    }
}
```

