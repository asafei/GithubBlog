---
title: react组件细节
date: 2018-08-29 23:04:35
tags: React
---
## 一 、React细节基础

### 1、Fragment，去掉多余的外出div

在组件的类中，render方法中所有的标签都得包含在一个div中，这样在生产的html页面中就会始终多一个div，要去掉这个最外层多余的div怎么办呢？

```
<div>	
	<div>
		<input value={this.state.inputValue} onChange={this.handlerInputChange}/>
		<button className='red-btn' onClick={this.handlerBtnClick}>add</button>
	</div>
	<ul>
		<li>学英语</li>
		<li>学汉语</li>
	</ul>
</div>
```

在react16的版本中，提供了一个叫做`Fragment`的占位符，这样就不会出现多余的外层div了

```javascript
//如果没有这句话，只能使用<React.Fragment>
import React, { Component , Fragment } from 'react';
。。。

。。。
<Fragment>	
	<div>
		<input value={this.state.inputValue} onChange={this.handlerInputChange}/>
		<button className='red-btn' onClick={this.handlerBtnClick}>add</button>
	</div>
	<ul>
		<li>学英语</li>
		<li>学汉语</li>
	</ul>
</Fragment>
```



### 2、React中响应式设计思想

​	react不直接操作dom来响应页面，而是操作数据，让react自动来响应布局。

​	react如何创建数据呢？

2.1首先需要在组件的构造函数中声明状态，然后把数据定义在这个状态中

```javascript
constructor(props){
	super(props);
    //this.state就是组件的状态
	this.state={
		list:[],
		inputValue:""
	};
}
```

2.2然后让标签和状态里中数据做一个绑定，该怎么做呢？

```javascript
//比如input中的值是有value属性决定的，就可以如下和数据绑定
<input value={this.state.inputValue} />
```

这样input标签就与状态中的inputValue数据绑定在了一起



### 3、React中事件绑定

​	原生的标签事件绑定都是使用的小写字母，在react 中必须使用驼峰式的规则。

```javascript
<input value={this.state.inputValue} onchange=...>
//必须改为
<input value={this.state.inputValue} onChange=.../>

//在jsx的语法里使用表达式，必须将表达式包括在打括号中，其中handlerInputChange函数在组件内定义，与render方法平行
<input value={this.state.inputValue} onChange={this.handlerInputChange}/>
```

注意此时在handlerInputChange函数中直接使用this.state会报错，显示this指向为undefined，这是由于此处this指向的是input标签，而不是该组件类，如果让它执行该组件类呢，需要绑定this

```javascript
//这是在handlerInputChange函数中直接使用this了
<input value={this.state.inputValue} onChange={this.handlerInputChange.bind(this)}/>
```

为了书写的简洁，可以对所有的函数在constructor中进行this的绑定，这样在标签中就不用书写bind语法了。

```javascript
constructor(props){
		super(props);
		this.state={
			list:[],
			inputValue:""
		};
		this.handlerInputChange=this.handlerInputChange.bind(this);
		this.handlerBtnClick=this.handlerBtnClick.bind(this);
		this.handlerDelete=this.handlerDelete.bind(this);
		
	}
//注意改变state中的数据，不能直接使用this.state进行改变，必须使用this.setState方法操作 
handlerInputChange(event){
		this.setState({
			inputValue:event.target.value
		});
		
	}
...
...
render() {
	    return (
		    	<Fragment>
		      <div>
		      	<input value={this.state.inputValue} onChange={this.handlerInputChange}/>
		      	<button className='red-btn' onClick={this.handlerBtnClick}>add</button>
		      </div>
		      <ul>{this.getTodoItems()}</ul>
		   </Fragment>
	    );
	  }
```



每一个标签最好有一个key值，特别是当多个相同的标签同时使用时。

在react中，不允许直接改变state的数据，必须通过setState来改变

### 4、jsx

#### 4.1 jsx中写注释

在jsc语法中写注释需要将注释放在大括号中

```javascript
render() {
	    return (
		    <Fragment>
            	{/*下面是div*/}
            	{//下面是div}
		      <div>
		      	<input value={this.state.inputValue} onChange={this.handlerInputChange}/>
		      	<button className='red-btn' onClick={this.handlerBtnClick}>add</button>
		      </div>
		      <ul>{this.getTodoItems()}</ul>
		   </Fragment>
	    );
	  }
```

#### 4.2 jsx中如果不希望对字符串转义

例如输入“<h1>hello </h1>”，不希望在页面输出“<h1>hello </h1>”，而是直接显示“hello”，此时可以使用dangerouslySetInnerHTML

```
getTodoItems(){
		return (
			this.state.list.map((item,index) => {
		      		return (
		      			<Item 
		      				delete={this.handlerDelete} 
		      				key={index} 
		      				{//表示解析item中的html标签，不被转义}
		      				dangerouslySetInnerHTML={{__html:item}}
		      				index={index}
		      			/>
		      		)
		      })
		);
	}
```

#### 4.3 光标的聚焦

在html中，可以使用label标签做聚焦，例如想通过点击某一文字让光标聚焦在input中，本来使用for属性，就可以，但是在jsx语法中需要使用htmlFor属性来代替

```javascript
render() {
	    return (
		    <Fragment>
		      <div>
            	{//给htmlFor传递某标签的id即可}
            	<label htmlFor="insertArea" >哈哈哈</label>
		      	<input id="insertArea" value={this.state.inputValue} onChange={this.handlerInputChange}/>
		      	<button className='red-btn' onClick={this.handlerBtnClick}>add</button>
		      </div>
		      <ul>{this.getTodoItems()}</ul>
		   </Fragment>
	    );
	  }
```

#### 4.4 组件数据的传递

父组件可以通过属性想子组件传递信息

```javascript
//TodoList组件
。。。
getTodoItems(){
		return (
			this.state.list.map((item,index) => {
		      		return (
		      			<Item 
		      				delete={this.handlerDelete} 
		      				key={index} 
		      				content={item} 
		      				index={index}
		      			/>
		      		)
		      })
		);
	}

//item组件
。。。
render(){
		const {content}=this.props;
		return (
			<div onClick={this.handlerDelete}>{content}</div>
		)
	}
```

子组件想要调用父组件中的方法，可以在父组件中通过属性将父组件中的方法传递给子组件，供子组件调用

代码同上，注意需要绑定父组件的this

#### 4.5 setState参数

在使用setState改变状态数据时，可以设置一个参数为prevState表示上一个状态的数据

```javascript
handlerDelete(index){
    this.setState((prevState)=>({
        const list=[...prevState.list];
        list.splice(index,1);
        return {list};
    }));
}
		
```

**注意：**父组件可以向子组件传值，但是子组件不能改变这个值，这就是react中单向数据流的概念。如果非要在子组件中修改这个值，可以通过父组件传给子组件的方法来操作。

#### 4.6、子组件的传值

react中子组件间的传值会非常复杂，因此建议使用其它框架或方法完成子组件间的值传递。这也是react只是视图层框架的原因。

### 5深入React

#### 5.1 chrome的react调试

安装扩展React Developer Tools

#### 5.2 PropTypes和DefaultProps

虽然子组件可以接受父组件传来的属性，但是子组件如果不能定义该属性的类型的话，就容易发生错误。此时需要对属性做校验，例如对Item组件添加组件交验：

* 首先引入PropTypes
* 然后在代码中添加属性类型校验

```javascript
import React, { Component }from "react";
import PropsTypes from 'prop-types';//引入包

class Item extends Component{
	...
}
//定义子组件接受属性的类型校验
Item.propTypes={
    content:PropTypes.string,//限定父组件传来的属性content必须是字符串
    delete:PropTypes.func,//限定父组件传来的属性delete必须是函数
    index:propTypes.number,//限定父组件传来的属性index必须是数字
}
export default Item;

```

这样的话，如果父组件传来的属性不符合要求的话，就会给出明显的错误提示。

**注意：**如果在子组件中为某属性定义了某一类型，并且还使用了该属性，但是父组件没有传递该属性，程序是不会报错的。如果想改变这种情况，可以在类型校验中添加isRequired

```javascript
//定义子组件接受属性的类型校验
Item.propTypes={
    test:PropTypes.string.isRequired,//test为字符串，且必须传递，否则或报错警报
    content:PropTypes.string,
    delete:PropTypes.func,
    index:PropTypes.number,
}
```

此时父组件必须给子组件传递test的属性，不传递就会报错。但是想要如果父组件不传递时给test一个默认值，如何处理呢，此时就用到了**DefaultProps**

```javascript
Item.propTypes={
    test:PropTypes.string.isRequired,//test为字符串，且必须传递，否则或报错警报
}
//这是test是必传的，如果不传，则使用默认值。
Item.defaultProps={
    test:"hello world"
}
```

当然还有很多高级的应用，比如允许一个属性既可以是字符串也可以是数字,可以使用PropsTypes.oneOfType([PropTypes.number,PropTypes.string])，详情请见[https://reactjs.org/docs/typechecking-with-proptypes.html](https://reactjs.org/docs/typechecking-with-proptypes.html)

#### 5.3 Props、State和render函数

当组件的props或state发生改变的时候，render函数就会重新执行。

所以当你输入input时会触发handlerInputChange 函数；而handlerInputChange 函数里面改变了state的状态，进而引起render函数的重新执行，这时render就可以拿到state中新的数据来渲染。

当父组件的render函数被运行时，它的子组件的render都将被重新运行。

react中频繁的重绘，得益于它有虚拟dom

### 6 React中的虚拟dom

#### 6.1 虚拟dom

假设让我们来实现虚拟dom，会有哪些步骤呢？

> 1、stateshju
>
> 2、JSX模版
>
> 3、数据+模版，结合生成真实的dom，进行展示
>
> 4、state 发生改变
>
> 5、数据+模版，结合生成真实的dom，进行展示

常规思路就这样，但是这样会有什么问题呢？

缺陷：

第一次生成一个完整的dom片段，第二次也生成一个完整的dom片段，然后第二次的dom替换第一次的dom，非常消耗性能

如何改良呢？

> 1、stateshju
>
> 2、JSX模版
>
> 3、数据+模版，结合生成真实的dom，进行展示
>
> 4、state 发生改变
>
> 5、数据+模版，结合生成真实的dom，并不直接替换原始的dom
>
> 6、新的dom和原始dom做比较，找差异
>
> 7、找出input框发生变化，
>
> 8、只用新的dom中的input元素，替换掉老dom中的input元素

此时虽然有省去了dom替换的性能，但是增加了新老dom做比较的性能。此时性能的提升并不明显。

而react怎么做的呢（操作dom消耗性能很大，但是操作js对象不消耗性能）

>1、stateshju
>
>2、JSX模版
>
>3、生成虚拟dom（虚拟dom是一个js对象，用它来描述真实dom）
>
>`["div",{id:"abc"},["span",{},"hello world"]]`
>
>4、用虚拟dom的结构生成真实的dom，进行展示
>
>`<div id=“abc”><span>hello world</span></div>`
>
>5、state 发生变化
>
>6、数据+模版 生成新的虚拟dom
>
>`["div",{id:"abc"},["span",{},"byebye"]]`
>
>7、比较原始虚拟dom和新的虚拟dom的区别，找到区别是span中的内容（比较js对象不消耗性能）
>
>8、直接操作dom，改变span中的内容

#### 6.2 render中的流程

render函数中的JSX模版，会先转化成js对象（虚拟dom），再转化为真实dom。

所以render中返回的看似htnl标签，其实不是，它是jsx模版。

假如render函数如下：

```javascript
render(){
    return <div>hello</div>
    //return <div><span>hello</span></div>
}

//效果等同于
render(){
    return React.createElement("div",{},"hello");
    //return React.createElement("div",{},React.createElement("span",{},"hello"));
}
```

实际上React.createElement是更偏向于底层的一个接口。而上边的返回div的真实流程是

jsx 调用createElement，转化为虚拟dom（js对象），生成真实dom.

但是当我们的结构比较复杂时 ，调用createElement就会显得更加混乱，而jsx语法就是为了简洁，方便写代码。

虚拟dom有什么好处呢？

> 1、性能提升 dom的比较变成了js对象的比较
>
> 2、它使得跨端应用得以实现，react native（可以将虚拟dom 生成原生组件）

#### 6.3 react中虚拟dom的diff算法

react是如何比较新旧虚拟都没的呢？diff算法

要么state改变，要么props改变（其实props改变也会state），就会触发虚拟dom的比对。

什么时候数据会发生变化，都是调用setState时数据才发生变化。

setState是异步的，如果你连续快速调用三次setState，那么react会把这三次setState合并成一个setState，只需进行一次虚拟dom的比对，这样就不会造成浪费了

diff的比对，遵循**同层比对**，若此层dom有差别，则不会再比较下层dom。

下面说一下为什么在for循环中，为啥不要使用index作为key值，通过key值可以建立组件一对一的关联，如果使用index作为key时，当仅仅是删除一项时，会带动剩下所有组件key值的变化，相当于重新组装了所有的组件；如果使用另一种唯一值作为key值，即使删除某一项，剩余项的key值也不会发生改变，此时dom只需删除被删除的那一项即可。

### 7、react中ref的使用

ref是帮助在react中直接获取dom元素的，实际上**不推荐使用ref**。

如何使用ref，先看原来的input框流程

```javascript
render() {
	    return (
		    	<Fragment>
		      <div>
		      	<input value={this.state.inputValue} onChange={this.handlerInputChange}/>
		      </div>
		      <ul>{this.getTodoItems()}</ul>
		   </Fragment>
	    );
}

handlerInputChange(event){
		//console.log(event.target.value);
		this.setState({
			inputValue:event.target.value
		});
}
```

此时我们是通过event.target来获取数据的，而ref就可以不通过event来获取数据，而是直接通过input来获取数据，怎么做呢？

```javascript
render() {
	    return (
		    	<Fragment>
		      <div>
		      	<input 
            		value={this.state.inputValue} 
            		onChange={this.handlerInputChange}
            		//这里第一个括号里的input可以任意命名，大括号里的等号后边input就是这个input标签，
            		ref={(input)=>{this.input=input}}/>
		      </div>
		      <ul>{this.getTodoItems()}</ul>
		   </Fragment>
	    );
}

handlerInputChange(event){
    	//这里就可以直接使用ref了
		const value=this.input.value;
		this.setState({
			inputValue:value
		});
}
```

注意直接操作dom对于新手有时会出现意想不到的效果，比如当我们每次点击按钮时项计算一下当前ul中的li的个数，通常会这么做

```javascript
render() {
	    return (
		    <Fragment>
		      <div>
		      	<input value={this.state.inputValue} onChange={this.handlerInputChange}/>
		      	<button 
            		className='red-btn' 
            		onClick={this.handlerBtnClick}
            		>add</button>
		      </div>
		      <ul ref={(ul)=>{this.ul=ul}}>{this.getTodoItems()}</ul>
		   </Fragment>
	    );
}
      
      handlerBtnClick(){
		this.setState({
			list:[...this.state.list,this.state.inputValue],
			inputValue:""
		});
		console.log(this.ul.querySelectorAll("div").length);
	}
```

这是你会发现，每次输出的数量都会比实际的要少1，为什么呢？这是因为setState是异步函数，每次还没更新页面你的console就已经执行了，这时如何避免这种情况呢？

setState第二个参数提供了回调函数，供我们使用，这样就能正常获取正确的数值了

```javascript
handlerBtnClick(){
		this.setState({
			list:[...this.state.list,this.state.inputValue],
			inputValue:""
        },{}=>{
            console.log(this.ul.querySelectorAll("div").length);
        });
	}
```

----
本笔记参考慕课网视频[https://coding.imooc.com/class/229.html](https://coding.imooc.com/class/229.html)


