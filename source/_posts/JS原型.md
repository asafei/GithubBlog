---
title: JS原型
date: 2019-01-09 07:29:33
tags: JavaScript
---
前端面试题永远少不了js的原型和原型链。解释起来有点拗口，可以用一张图来讲解：

{% asset_img js_proto.png js原型示例图 %}

### 一、Function和Object

首先说一下，js中自带两个函数Object和Function，Object继承自己，Funtion继承自己，Object和Function互相是继承对方，也就是说Object和Function都既是函数也是对象。

```javascript
Function instanceof Object;//true
Object instanceof Function;//true
```

也就是说Object是Function的实例，Function是自己的实例

### 二、普通对象和普通函数

js中万物皆对象，函数是对象，函数创建出来的实例也是对象，我们将函数成为函数对象，将实例成为普通对象。

注意**函数对象本身又是Function的实例**

```javascript
function Sample(){};//Sample是一个函数对象
var s=new Sample();//s是一个普通对象

//function Sample(){}等同于
var Sample=new Function();//所以Sample又是Function的实例

```

### 三、隐式原型和显示原型

* 每个对象都有一个名为`__proto__`的内部属性（隐式原型），它指向所对应构造函数的原型对象
* 每个函数对象都有一个prototype的属性（显示原型）prototype与`__proto__`都指向构造函数的原型对象，原型对象下面又有一个constructor属性，指向这个函数对象

### 四、原型链

js中，每个对象都会在内部生成一个`__proto__ `属性，当我们访问一个对象属性时，如果这个对象不存在就会去`__proto__` 指向的对象里面找，一层一层找下去，这就是javascript原型链的概念。

**JS中所有的东西都是对象,所有的东西都由Object衍生而来, 即所有东西原型链的终点指向null**

### 五、继承

js中的普通对象是由构造函数（即函数对象）new出来的，这个对象产生于原型却不等于原型；这个普通对象是如何与原型产生联系的呢？就是通过这个叫`__proto__`完成的。

```javascript
function method(){}
var m = new method();
m===m.prototype;//false

m.__proto__====method.prototype;//true
```

参考：[http://baijiahao.baidu.com/s?id=1585354841830289519&wfr=spider&for=pc](http://baijiahao.baidu.com/s?id=1585354841830289519&wfr=spider&for=pc)

[https://www.jb51.net/article/123976.htm](https://www.jb51.net/article/123976.htm)

