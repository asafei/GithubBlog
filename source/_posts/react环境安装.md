---
title: react环境安装
date: 2018-08-29 22:55:47
tags: React
---

## 一、React 开发环境的准备

### 1、引入.js文件来使用React

### 2、使用脚手架工具来编码

* GRUNT
* GULP
* Webpack
* Create-react-app（官方推荐的脚手架工具）

但是使用脚手架写的内容不能被浏览器直接识别，需要脚手架工具做代码的编译，才能被浏览器运行。



## 二、安装create-react-app

如果你的node版本为6.0及以上，可以使用以下命令：

```javascript
npm install -g create-react-app	//安装脚手架工具create-react-app
create-react-app my-app		//使用脚手架生成工程my-app
cd my-app		//进入my-app工程
npm run start		//入启动my-app工程
```

如果你的npm版本为5.2.0及以上，可以使用以下命令

```javascript
npx create-react-app my-app
cd my-app		//进入my-app工程
npm start		//入启动my-app工程
```



## 三、工程目录简介

* package.json  //任何一个脚手架工具中都会有一哥package.json文件。它代表node的一个包文件，包括项目介绍和一些依赖的包.

  只所以能通过npm 润 start来启动工程服务，是因为package.json文件中有

    "script":{
        "start":"react-scripts start"
        //...
    }

* node_modules //第三方的模块

* public

  ​	favicon.ico //网页标签栏的icon

  ​	index.html //首页模版

  ​	mainfest.json 

* src //源码文件

  ​	registerserviceWorker //PWA progressive web application，在https服务器访问过，即使断网仍旧可以再访问的功能

* 。。。

## 四、React 组件

组件就是页面上的一部分、

### 1、定义react组件

```javascript
class App extends React.Component{
    //render返回什么，就展示什么
    render(){}
}
```

注意 

```javascript
import { Component } from 'react'

等价于

import React form 'react';
const Component= React.Component;
```



### 2、ReactDOM

这是一个第三方的模块，它有一个方法叫render，该方法可以帮助把一个组件挂载到dom节点上

```javascript
//将APP组件渲染到root标签中 
ReactDOM.render(<APP />, document.getElementById('root'));
//<APP />是jsx语法，

//包括组件中render()方法内返回的`<div>`也是jsx语法，都得通过引入React才能使用
import React from 'react';
```

### 3、jsx语法

自定定义的组件，可以直接通过jsx标签来使用，比如上面的`<APP />`

注意：组件的命名必须是大写字母开头，

一般大写字符开头都是jsx组件，小写字母开头的标签都是h5的标签

----
本笔记参考慕课网视频[https://coding.imooc.com/class/229.html](https://coding.imooc.com/class/229.html)

  

  

  


