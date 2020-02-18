---
title: ES7新特性
date: 2019-03-12 11:52:35
tags: JavaScript
---
#### 一、ES7新特性

常用的更新有两个：

* 求幂运算符
* 数组的includes方法

##### 1.1 求幂运算符（**）

基本使用方式，求5的2次幂

```javascript
5**2  //25
//等价于
Math.pow(5,2);
```

复合使用的话依然遵循从右向左的优先级

```javascript
2**4**2
//灯架
2**(4**2)
```

##### 1.2 Array.prototype.includes()方法

`includes()`查找一个值在不在数组里，若在，则返回`true`，反之返回`false`。 

```javascript
['a', 'b', 'c'].includes('a')     // true
['a', 'b', 'c'].includes('d')     // false
```

include也可以接收两个参数，要搜索的值和起始索引。当第二个参数被传入时，该方法会从索引处开始往后搜索（默认索引值为0）。若搜索值在数组中存在则返回`true`，否则返回`false`。

```javascript
['a', 'b', 'c', 'd'].includes('b')         // true
['a', 'b', 'c', 'd'].includes('b', 1)      // true
['a', 'b', 'c', 'd'].includes('b', 2)      // false
```

Includes方法与indexof的区别在于：

* includes更轻便，无需再判断返回值是否大于-1；

* includes可以判断Nan，而indexof不能判断Nan

  > ```javascript
  > let demo = [1, NaN, 2, 3]
  > 
  > demo.indexOf(NaN)        //-1
  > demo.includes(NaN)       //true
  > ```

* Include 对于+0和-0按相等处理的

  > ```javascript
  > [1, +0, 3, 4].includes(-0)    //true
  > [1, +0, 3, 4].indexOf(-0)     //1
  > ```

#### 二、ES8新特性

##### 2.1 async/await

传统的JavaScript中，对于异步的处理通常是通过回调函数处理的，但是一旦出现回调函数的嵌套，就很容易陷入回调地狱（callback hell）

```javascript
this.$http.jsonp('/login', (res) => {
  this.$http.jsonp('/getInfo', (info) => {
    // ...
  })
})
```



一种改进方式是使用promise，通过then方法将回调嵌套改为了链式的,尽管如此，当请求任务过多时一堆then也会造成语义的难以理解。

```javascript
ar promise = new Promise((resolve, reject) => {
  this.login(resolve)
})
.then(() => this.getInfo())
.catch(() => { console.log("Error") })
```

另一种异步机制时使用Generator函数，它通过* 、yield和next的来执行分段操作。Generator 函数是分段执行的，yield表达式是暂停执行的标记，而next方法可以恢复执行

```javascript
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

var hw = helloWorldGenerator();

hw.next()
// { value: 'hello', done: false }
hw.next()
// { value: 'world', done: false }
hw.next()
// { value: 'ending', done: true }
hw.next()
// { value: undefined, done: true }
```

虽然Generator将异步操作表示得很简洁，但是流程管理却不方便（即何时执行第一阶段、何时执行第二阶段）。此时，我们便希望能出现一种能自动执行Generator函数的方法。我们的主角来了：async/await。

ES8引入了async函数，使得异步操作变得更加方便。简单说来，它就是Generator函数的语法糖。

```javascript
async function asyncFunc(params) {
  const result1 = await this.login()
  const result2 = await this.getInfo()
}
```

处理单个异步结果

```javascript
async function asyncFunc() {
  const result = await otherAsyncFunc();
  console.log(result);
}
```

顺序处理多个异步结果

```javascript
async function asyncFunc() {
  const result1 = await otherAsyncFunc1();
  console.log(result1);
  const result2 = await otherAsyncFunc2();
  console.log(result2);
}

```

并行处理多个异步结果

```javascript
async function asyncFunc() {
  const [result1, result2] = await Promise.all([
    otherAsyncFunc1(),
    otherAsyncFunc2()
  ]);
  console.log(result1, result2);
}
```

处理错误

```javascript
async function asyncFunc() {
  try {
    await otherAsyncFunc();
  } catch (err) {
    console.error(err);
  }
}
```

##### 2.2 Object.entries()和Object.values()

###### 2.2.1Object.entries()

如果一个对象是具有键值对的数据结构，则每一个键值对都将会编译成一个具有两个元素的数组，这些数组最终会放到一个数组中，返回一个二维数组

```javascript
Object.entries({ one: 1, two: 2 })    //[['one', 1], ['two', 2]]
Object.entries([1, 2])                //[['0', 1], ['1', 2]]
```

**注意：**键值对中，如果键的值是Symbol，编译时将会被忽略。

```javascript
Object.entries({ [Symbol()]: 1, two: 2 })       //[['two', 2]]
```

`Object.entries()`返回的数组的顺序与for-in循环保持一致，即如果对象的key值是数字，则返回值会对key值进行排序，返回的是排序后的结果。

```javascript
Object.entries({ 3: 'a', 4: 'b', 1: 'c' })    //[['1', 'c'], ['3', 'a'], ['4', 'b']]
```



###### 2.2.2 Object.values()

它的工作原理跟`Object.entries()`很像，顾名思义，它只返回自己的键值对中属性的值。它返回的数组顺序，也跟`Object.entries()`保持一致。

```javascript
Object.values({ one: 1, two: 2 })            //[1, 2]
Object.values({ 3: 'a', 4: 'b', 1: 'c' })    //['c', 'a', 'b']
```

##### 2.3 padStart和padEnd

ES8提供了新的字符串方法-padStart和padEnd。`padStart`函数通过填充字符串的首部来保证字符串达到固定的长度，反之，`padEnd`是填充字符串的尾部来保证字符串的长度的。该方法提供了两个参数：字符串目标长度和填充字段，其中第二个参数可以不填，默认情况下使用空格填充。

```javascript
'Vue'.padStart(10)           //'       Vue'
'React'.padStart(10)         //'     React'
'JavaScript'.padStart(10)    //'JavaScript'
```

那么我们现在来看看第二个参数，我们可以指定字符串来代替空字符串。

```javascript
'Vue'.padStart(10, '_*')           //'_*_*_*_Vue'
'React'.padStart(10, 'Hello')      //'HelloReact'
'JavaScript'.padStart(10, 'Hi')    //'JavaScript'
'JavaScript'.padStart(8, 'Hi')     //'JavaScript'
```

`padEnd`函数作用同`padStart`，只不过它是从字符串尾部做填充

```javascript
'Vue'.padEnd(10, '_*')           //'Vue_*_*_*_'
'React'.padEnd(10, 'Hello')      //'ReactHello'
'JavaScript'.padEnd(10, 'Hi')    //'JavaScript'
'JavaScript'.padEnd(8, 'Hi')     //'JavaScript'
```



##### 2.4 Object.getOwnPropertyDescriptors()

略

##### 2.5 共享内存和原子（Shared memory and atomics）

内存管理碰撞课程：<https://segmentfault.com/a/1190000009878588>

图解 ArrayBuffers 和 SharedArrayBuffers：<https://segmentfault.com/a/1190000009878632>

用 Atomics 避免 SharedArrayBuffers 竞争条件：<https://segmentfault.com/a/1190000009878699>



参考：[](https://www.cnblogs.com/zhuanzhuanfe/p/7493433.html)