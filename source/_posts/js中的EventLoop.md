---
title: js中的EventLoop
date: 2020-01-09 00:24:21
tags: JavaScript
---

#### 一、由promise和setTimeout引出的问题

面试中最尴尬的情况大概就是当对方抛出问题时，你却不知从何说起。当问起promise和setTimeout时接下来很大程度上都会问到执行顺序，引出js的EventLoop的话题。

#### 二、什么是EventLoop

​	JavaScript语言的一大特点就是单线程，这已经成了这门语言的核心特征，将来也不会改变。为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质。

​	js中任务可以分成两种，一种是同步任务（synchronous），另一种是异步任务（asynchronous）。

```javascript
* 所有的同步任务会在主线程中依次执行，而异步任务会被挂起；
* 当被挂起的异步任务有了运行结果，回调函数就会被添加在任务队列中，该任务队列等待被主线程下次执行；
* 当主线程内当然的同步任务被执行完之后，就会获取任务队列中任务，依次同步执行；
* 依次往复，每当主线程中的任务被执行完后，都会去重新获取任务队列中的任务，这就是js的EventLoop
```

当然上面说的是最简单的模型，当任务队列中还存在异步任务时，会发生什么呢？

```javascript
* 当主线程执行一个新的任务队列时，回依次执行里面的同步任务，
* 若碰到异步任务，会将它挂起，待异步任务有了运行结果，回调函数会被添加到新的任务队列
* 新的任务对象，等待被主线程下次获取，从而执行。
```

到这里就可以回答一个初级的面试问题了，当使用setTimeout时延时指定的时间后，函数会不会立即执行？

答案是不会立即执行，因为setTimeout延迟指定时间后，只是将函数放入任务队列中，主线程首先得先执行完当前的所有同步任务后，才能去获取新的任务队列中的任务，然后依次执行任务队列中的任务。

当然，事情远不如此简单。异步跟异步也是有区分的。

#### 三、宏任务（macro-task）和微任务（micro-task）

宏任务一般包括：setTimeout，setInterval

微任务一般包括：Promise（then方法的执行）、process.nextTick

区别在哪呢？

```javascript
* 假设所有的异步任务都有了运行结果，它们的回调被放在了两个任务队列中：宏任务队列和微任务队列
* 当主线程执行完毕后，首先执行微任务队列中的任务，然后执行红任务队列中的任务
* 如果在微任务队列中，有新的微任务，则该微任务队列执行完毕后，会立即执行新的微任务队列，然后再执行宏任务队列
* 当执行宏任务队列时，同样若存在新的微任务，也会在本宏任务队列执行完后立即执行微任务队列；若存在新的宏任务时，新的宏任务会被挂起然后放在新的宏任务队列中，等待主线程的下次轮询。
```

上面说的比较拗口，可以换一种说法：

```javascript
* 所有宏任务有了运行结果，回调会被存放了任务队列中；
* 所有的微任务有了运行结果，回调会被理解添加到当前主线程的执行序列中，
* 这样当主线程执行到最后，都会继续执行所有微任务的回调，然后获取新的任务队列进行轮询
```

更进阶的面试题就是，说出下面的代码的输出：

```javascript
setTimeout(function(){
    console.log('1');
},0);
new Promise(function(resolve){
    console.log(2);
    process.nextTick(function() {
        console.log('3');
    })
    resolve();
}).then(function(){
    console.log(4);
});
console.log(5);
```

正确答案是：2-5-3-4-1

网上有更多的练习题，可以自己去看看。本文讲到这里

#### 四、其它

宏任务还有[setImmediate](http://nodejs.org/docs/latest/api/timers.html#timers_setimmediate_callback_arg)，node的事件轮询也有一些差别，这里不再详述，可以参考下面的文档。

参考：

[<http://www.ruanyifeng.com/blog/2014/10/event-loop.html>](<http://www.ruanyifeng.com/blog/2014/10/event-loop.html>)

[<https://blog.csdn.net/qq_41805715/article/details/84849280>](<https://blog.csdn.net/qq_41805715/article/details/84849280>)
[https://blog.csdn.net/u014298440/article/details/88430838](https://blog.csdn.net/u014298440/article/details/88430838)

