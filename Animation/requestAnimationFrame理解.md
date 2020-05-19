## requestAnimationFrame理解

## requestAnimationFrame介绍

> 背景：以往的js动画是依赖定时器 `setTimeout` 或者 `setInterval` 实现的。但是定时器动画一直存在两个问题，第一个就是动画的循时间环间隔不好确定，设置长了动画显得不够平滑流畅，设置短了浏览器的重绘频率会达到瓶颈，推荐的最佳循环间隔是16ms（浏览器刷新频率是60Hz，1000ms/60）；第二个问题是定时器第二个时间参数只是指定了多久后将动画任务添加到浏览器的UI线程队列中，如果UI线程处于忙碌状态，那么动画不会立刻执行。

### 介绍

requestAnimationFrame是html5 提供的一个专门用于请求动画的API，顾名思义就是请求动画帧，它被封装在宿主对象中， window.requestAnimationFrame() 告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行。

### 相比setInterval优势

1、提升性能，防止掉帧。requestAnimationFrame是按帧对网页进行重绘。该方法告诉浏览器希望执行动画并请求浏览器在下一次重绘之前调用回调函数来更新动画，动画不会出现掉帧的情况，更加流畅。

2、在隐藏或不可见的元素中，requestAnimationFrame将不会进行重绘或回流，这当然就意味着更少的的cpu，gpu和内存使用量。

3、避免多余的事件触发，它相当于一种函数节流。在高频事件（`resize`，`scroll`等）中，使用`requestAnimationFrame`可以防止在一个刷新间隔内发生多次函数执行，这样保证了流畅性，也节省了函数执行的开销

### 使用

> 调用操作。与`setTimeout`相似，但是不需要设置间隔时间，使用一个回调函数作为参数，返回一个大于 0 的整数

```javascript
handle = requestAnimationFrame(callback); 
```

1. handle是一个整数，唯一地标识了元组在列表中的位置。

2. callback 是一个无返回值的、形参为一个时间值的函数（该时间值为由浏览器传入的从 1970 年 1 月 1 日到当前所经过的毫秒数）。

3. cancelAnimationFrame()可以通过它停止动画。

`递归`实现动画

以下实现了一个简单的进度条。

```javascript
function loadingBar(ele) {
  // 使用闭包保存定时器的编号
  let handle;
  return () => {
    // 每次触发将进度清空
    ele.style.width = "0";
    // 开始动画前清除上一次的动画定时器
    // 否则会开启多个定时器
    cancelAnimationFrame(handle);
    // 回调函数
    let _progress = () => {
      let eleWidth = parseInt(ele.style.width);
      if (eleWidth < 200) {
        ele.style.width = `${eleWidth + 5}px`;
        handle = requestAnimationFrame(_progress);
      } else {
        cancelAnimationFrame(handle);
      }
    };
    handle = requestAnimationFrame(_progress);
  };
}
```



### 兼容性

firefox、chrome、ie10以上， `requestAnimationFrame` 的支持很好，但不兼容 IE9及以下浏览器

### 参考链接

[你知道的requestAnimationFrame【从0到0.1】](https://juejin.im/post/5c3ca3d76fb9a049a979f429#heading-7)

[requestAnimationFrame，终结定时器动画时代！](https://juejin.im/post/5e8c08cd51882573b04739c8)

