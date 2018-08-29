---
title: react生命周期函数
date: 2018-08-29 23:01:16
tags: React
---
## react生命周期函数

### 1、什么是生命周期函数

生命周期函数是指组件在某一刻会自动执行的函数

* 1、初始化initialization

  > 在组件创建时，被执行的函数；进行props和state的设置，可以放在constructor函数中

* 2、挂载mounting

  > 当组件被挂载的时候执行，该周期包括如下
  >
  > * componentWillMount：组件被挂载之前执行，通常组件只会挂载一次
  > * render：组件被渲染，可执行多次
  > * componentDidMount：组件被挂载之后执行，通常组件只会挂载一次

* 3、更新update

  > 包括props和state更新
  >
  > * 3.1 props
  >
  >   > * componentWillReceiveProps
  >   >
  >   > 一个组件从父组件接受参数，只要父组件的render函数被重新执行了，子组件的这个生命周期函数就会被执行。
  >   >
  >   > 如果这个组件第一次存在于父组件中，不会执行；
  >   >
  >   > 如果这个之前已经存在于父组件中，才会执行
  >   >
  >   > * shouldComponentUpdate:返回true则继续执行，返回false则下一步不执行
  >   >
  >   > * componentWillUpdate
  >   >
  >   > * render
  >   >
  >   > * componeneDidUpdate
  >
  > * 3.2 state
  >
  >   > * shouldComponentUpdate:返回true则继续执行，返回false则下一步不执行
  >   >
  >   > * componentWillUpdate
  >   >
  >   > * render
  >   >
  >   > * componeneDidUpdate
  >

* 4、卸载unmounting 

  > 当一个组件即将被从页面剔除的时候，执行
  >
  > componentWillUnmount:

注意：每个组件都存在生命周期函数



### 2、生命周期函数使用场景

生命周期函数中，除了render之外，都可以不存在。

这是由于react中所有的组件都是继承自React.Component，在Component中内置了所有的其余生命周期函数，唯独没有内置render函数的实现。

#### 2.1 shouldComponentUpdate性能提升

在todolistdemo中，当我们在input中输入东西时，都是都会同时更新input组件和所有的子组件item。之前我们讲过组件重新渲染的条件时state或props变化时，回重新渲染；但是现在item的数据并没有变化，为啥为跟着父组件重新渲染呢？这是因为父组件的render函数执行时，子组件的render都会被执行，这样就会造成很多无谓的渲染，如何避免这种情况呢？

这时shouldComponentUpdate就派上了用场，在内部直接返回false，就可以实现，该组件只在挂载时执行一次，而不会随着数据的更新再次渲染

```javascript
shouldComponentUpdate(){
    return false;//此时该组件不会随着数据的更新而发生重新渲染
}
```

这样强制返回，简单暴力，不是很合适，可以根据实际情况来判断，当我们用到的数据真的发生改变时就返回true，否则就返回false。可以根据shouldComponentUpdate函数的两个参数来判断

```javascript
//在渲染时我们只用到了state中的content的内容
shouldComponentUpdate(nextProps,nextState){
    if(nextProps.content!==this.props.content){
       return true;
    }else{
       return false;
    }
}
```

这样就提升了我们组件的性能，减少了无谓的组件渲染。

目前为止，我们讲到提升react的性能的方式有以下几种了

> 1、将自定义函数与this的绑定，放在构造函数中
>
> 2、标签的key值不能使用index
>
> 3、使用shouldComponentUpdate，避免无效的组件渲染

#### 2.2 react发生ajax请求

如果我们需要在组件中发送ajax请求，这个请求该放在什么地方呢？比如我们要请求我们要展示多少条，如果放在render中，那么随着render函数的多次调用，ajax请求也会被执行多次。如何让这个ajax请求只执行一次呢？

这时componentDidMount就派上了用场，因为这个函数只会在组件被挂载之后执行一次，之后就不会再被执行了。componentWillMount也有此作用，但是如果放在componentWillMount中可能会与后期的高级技术发生冲突（不再详述），所以不放这里。

react并没有封装ajax请求，此时我们可以使用第三方的框架，axios

> 1、首先通过控制台，进入项目目录todolist，
>
> 2、输入 yarn add axios，此时就能将axios载入到项目之中了
>
> 3、在TodoList组件中，引入axios 
>
> `import axios from 'axios'`
>
> 4、在componentDidMount中发送请求
>
> ```javascript
> componentDidMount(){
>     axios.get('/spi/todolist')
>         .then(()=>{alert('success')})//成功
>         .catch(()=>{alert('error')});//失败
> }
> ```



#### 2.3 使用charles实现本地数据的mock

当前开发模式都是前后端分离，为了方便测试需要进行数据的模拟，这就是mock。而charles就可以实现这种模拟。如何模拟呢？参考下面步骤

> 1、首先下载charles
>
> 2、在桌面创建模拟数据todolist.json，在里面填写要模拟的数据
>
> ```javascript
> ["react","vue","webgl"]
> ```
>
> 3、打开cahrles工具：tools -> Map Loacal
>
> 进行url（localhost:3000/api/todolist）的配置即可，并且map到桌面上指定的todolist.json文件
>
> 4、然后进入刷新我们的页面，之前的请求就可以得到数据了
>
> 5、在componentDidMount中对ajax请求的结果进行处理，setState即可
>
> ```javascript
> componentDidMount(){
>     axios.get('/spi/todolist')
>         .then((res)=>{
>             this.setState(()=>{
>                 return  {
>                     list:res.data
>                 }
>             });
>     	})//成功
>         .catch(()=>{alert('error')});//失败
> }
> 
> //或者这么简写
> componentDidMount(){
> 		    axios.get('/api/todolist')
> 		        .then((res)=>{
>                 		//小括号即表示return
> 		        		this.setState(()=>({list:[...res.data]}));
> 		        })//成功
> 		        .catch(()=>{alert('error')});//失败
> 		}
> ```
>
> 如此就实现了ajax请求，并且和组件建立关系

charles可以抓到浏览器对外的一些请求，作为代理服务器

----
本笔记参考慕课网视频[https://coding.imooc.com/class/229.html](https://coding.imooc.com/class/229.html)
