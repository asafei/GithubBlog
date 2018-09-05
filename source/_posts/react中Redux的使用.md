---
title: react中Redux的使用
date: 2018-09-06 01:56:25
tags: React
---
## Redux

react只是一个轻量级的视图框架，想用它做一个大型应用是不可能的，所以需要一个数据层的框架跟它结合起来使用，这时就可以使用redux了。

### 一、Redux简介

Redux是一个数据层框架。再react中如果想在组件间传递数据，需要繁琐的数据传递。使用redux，将组件的数据放在一个公共的区域，这样每当这个区域的数据发生的变化时，所有相关的组件都会随着更新。

Redux=Reducer+Flux



### 二、Redux的工作流程

```javascript
借书人               借书语句         管理员	 图书记事本
					dispatch   （previousState
                    action）       ，action）
										（newState）
ReactComponents---ActionCreators-->Store<-->Reducers
        ^                             |
    	|------------------------------
```



### 三、使用antD创建todoList布局

#### 3.1 初始化一个工程

初始化一个名为todolist的工程，清洁里面只剩一个index.js的文件；

然后创建一个TodoList的组件

```javascript
import React, { Component } from 'react';

class TodoList extends Component {
  render() {
    return (
      <div>
        hello world!
      </div>
    );
  }
}

export default TodoList;
```

#### 3.2 安装AntD

首先进入[官网](https://ant.design/docs/react/introduce-cn)，查看安装教程。

进入工程todolistantd，运行下面的命令，安装Ant D

```javascript
npm install antd --save

//或使用yarn安装
yarn add antd
```

#### 3.2 使用AntD

安装完成后，如何使用AntD呢

首先在组件中引入antD的样式，就可以使用了

```javascript
import 'antd/dist/antd.css';
```

##### 3.2.1 使用antD的Input

若要使用antD的组件可以直接在使用文档中查找相关组件，通过showCode直接将代码粘贴过来。如我们使用一个input，首先引入Input，然后就能使用了

```javascript
import React, { Component } from 'react';
import 'antd/dist/antd.css';
import { Input } from 'antd';

class TodoList extends Component {
  render() {
    return (
      <div>
        <div><Input placeholder="todo info"/></div>
      </div>
    );
  }
}

export default TodoList;
```

这样就可以在页面上看到一个input框了，和预计的样式一样。当然，可以给这个Input设置自己的样式，如下面的样式可以将input框宽度设为300px

```javascript
<Input placeholder="todo info" style={{width:'300px'}}/>
```

此时我们想在Input右边添加一个按钮怎么办呢？同理去antD中找Button使用

##### 3.2.2 使用antD的Button

首先引入Button的引用，然后直接使用即可

```javascript
import React, { Component } from 'react';
import 'antd/dist/antd.css';
import { Input , Button} from 'antd';//引入Button

class TodoList extends Component {
  render() {
    return (
      <div>
        <div>
        	<Input placeholder="todo info" style={{width:'300px'}}/>
        	<Button type="primary">Primary</Button>
        	</div>
      </div>
    );
  }
}

export default TodoList;
```

**添加间距**

此时觉得input框和button间没有间距，直接连接不太好,想给它们添加个10px的间距，可以这么做：

```javascript
<Input placeholder="todo info" style={{width:'300px',marginRight:'300px'}}/>
```

这时这两个标签都会紧贴着浏览器，想给它们整体添加一个间距，怎么做呢

可以在最外层的div中添加样式文件

```javascript
return (
      <div style={{marginTop:'10px',marginLeft:'10px'}}>
        <div>
        	<Input placeholder="todo info" style={{width:'300px'}}/>
        	<Button type="primary">Primary</Button>
        	</div>
      </div>
    );
  }
```

此时，我们的Input和Button展示的有点样子了，接下来

##### 3.2.3 使用List

首先引入List组件，

然后在TodoList组件外边创建全局列表数据，

在render中添加List组件

```javascript
import React, { Component } from 'react';
import 'antd/dist/antd.css';
import { Input , Button,List } from 'antd';//引入Button

const data = [
  'React',
  'Vue',
  'Android',
  'IOS',
  'WebGL',
];
class TodoList extends Component {
  render() {
    return (
      <div>
        <div>
        	<Input placeholder="todo info" style={{width:'300px'}}/>
        	<Button type="primary">Primary</Button>
        	</div>
        <h3 style={{ marginBottom: 16 }}>Default Size</h3>
        <List
          bordered
          dataSource={data}
          renderItem={item => (<List.Item>{item}</List.Item>)}
        />
      </div>
    );
  }
}

export default TodoList;
```

这是list也是左右满屏，可以做一下微调，可以给list添加样式，如下

```javascript
<List
	style={{width:'300px',marginTop:'10px'}}
    bordered
    dataSource={data}
    renderItem={item => (<List.Item>{item}</List.Item>)}
/>
```

这样，List就和Input等宽了。关于antD的其它空间，可以直接去官网查看用法。antD一般用来开发后台页面，效果很快。

### 四、Store的创建

##### 4.1 安装Redux

如果要使用Redux，首先得先安装Redux，可用下面命令安装

```javas
npm install --save redux
//或
yarn add redux
```

##### 4.1 创建Store

在工程的src目录下创建一个名为“store”的文件夹；

在store的文件夹下，创建一个文件叫做index.js；

然后在index.js中引入redux的一个名为createStore方法，利用这个方法就可以创建一个store出来；

如此我们就创建了一个公共数据存储仓库。

```javascript
import { createStore } from 'redux';

const store=createStore();

export store;

```

此时我们''图书管理员''已经创建好了；

但是他却记不住如何管理数据，此时他需要一个"图书记事本"来辅助管理图书；

所以创建''图书管理员''时，需要同时创建"图书记事本"，也就是Reducer

##### 4.2 创建Reducer

在store目录下，创建名为reducer.js的文件；

在里面返回一个函数；

```javascript
//state表示整个store中存储的数据
export default (state,action)=>{
    return state;
}
```

如果我们想要返回指定的state怎么办呢？可以写一个结构，返回如下：

```javascript
const defaultState={
    inputValue:'',
    list:[],
}
export default (state=defaultState,action)=>{
    return state;
}
```



我们还要把state传给store，怎么传呢？

在store/index.js文件中引入reducer，并且在创建store时，将reducer传给store

```javascript
import { createStore } from 'redux';
import reducer from './reducer';

const store=createStore(reducer);

export default store;//注意一定要加上default
```

这时，''图书管理员''和"图书记事本"已经绑定起来了？

我们该怎么将store中的数据跟TodoList组件绑定起来呢？刚才我们demo展示的data是写死的数据，可以将之清掉。

然后在TodoList控件中引入该store，

```javascript
import React, { Component } from 'react';
import 'antd/dist/antd.css';
import { Input , Button,List } from 'antd';
import store from './store/index.js';//引入store

class TodoList extends Component {
    constructor(props){
        super(props);
        this.state=store.getState();
        //console.log(store.getState());
    }
  render() {
    return (
      <div>
        <div>
        	<Input value={this.state.inputValue} placeholder="todo info" style={{width:'300px'}}/>
        	<Button type="primary">Primary</Button>
        	</div>
        <h3 style={{ marginBottom: 16 }}>Default Size</h3>
        <List
          bordered
          dataSource={this.state.list}
          renderItem={item => (<List.Item>{item}</List.Item>)}
        />
      </div>
    );
  }
}

export default TodoList;
```

自此已完成store和reducer的创建，那么该如何编写它们呢？

##### 4.3 store和reducer的编写

###### 1、Redux DevTools调试工具的安装

为了调试redux，首先需要先安装一个redux的插件，名为Redux DevTools

然后在创建store时，添加第二个参数，[参考](https://github.com/zalmoxisus/redux-devtools-extension)如下：

```javascript
const store = createStore(reducer,
			window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__());
```

这个时候我们就能在浏览器中使用Redux DevTools了，进行redux的调试了。

下面要走改变store中数据的流程

###### 2、改变store中数据

首先我们想，当我们网input中输入东西时，store中的inputValue发生改变；

我们可以给Input标签一个onChange事件，传入自定义函数handleInputCahnge	,记得在constructor中对该函数进行this的绑定

```javascript
constructor(){
    ...
    this.handleInputChange.bind(this);
}
...

onChange={this.handleInputChange}


...
handleInputChange(event){
    console.log(event.tarent.value);
}

```

接下来我们要创建一个action，接受input的值，然后将action传给store;

怎么将action的内容传给store呢？store提供了一个方法叫dispatch

```javascript
handleInputChange(event){
    const action={
        type:"change_input_value",
        value:event.target.value,
    };
    store.dispatch(action);
}
```

store接收到action传来的数据，该怎么处理呢？它需要查"图书记事本"reducer，它需要拿着action和当前store中数据一起转发给reducer，然后reducer告诉store来做什么。

幸运的是store给reducer传递数据已经自动化完成了，无需我们再做。我们只需要再reducer中对当前的数据和action进行数据的更新。比如可以如下：

```javascript
//store表示当前数据，也就是preState，action表示动作
export default (state=defaultState,action)=>{
    if(action.type==='change_input_value'){
        //对当前state做一个深拷贝
        const newState=JSON.parse(JSON.stringify(state));
        newState.inputValue=action.value;
        return newState;
    }
    return state;
}
```

**注意：reducer可以接收state，但是绝对不能修改state**

###### 3、完成action的传递和TodoList中Input的更新

此时reducer返回的state就给了store，然后store就可以把新数据替换掉老数据。如何让组件能实时更新呢？

需要让TodoList组件订阅store，在订阅接口中编写我们的更新函数即可；

更新函数中，只需要对store重新getState，然后setState即可；

这样store发生改变，TodoList就会发生更新了

```javascript
import React, { Component } from 'react';
import 'antd/dist/antd.css';
import { Input , Button , List} from 'antd';
import store from './store/index';

class TodoList extends Component {
	constructor(props){
		super(props);
		this.state=store.getState();
		this.handleInputChange.bind(this);
		this.handleStoreChange=this.handleStoreChange.bind(this);
		store.subscribe(this.handleStoreChange);
	}
	
	  render() {
	    return (
	      <div style={{marginTop:'10px' , marginLeft:'10px'}}>
	        <div>
	        		<Input 
	        			value={this.state.inputValue} placeholder="todo info" 
	        			style={{width:'300px',marginRight:'10px'}} 
	        			onChange={this.handleInputChange}/>
	        		<Button type="primary">提交</Button>
	        	</div>
	        <List
	        	  style={{width:'300px',marginTop:'10px'}}
	          bordered
	          dataSource={this.state.list}
	          renderItem={item => (<List.Item>{item}</List.Item>)}
	        />
	      </div>
	    );
	  }
	  
	  handleInputChange(event){
		    const action={
		        type:"change_input_value",
		        value:event.target.value,
		    };
		    store.dispatch(action);
		}
	  
	  //当感知到数据发生变化时，就回去新的state数据
	  handleStoreChange(event){
	  	this.setState(store.getState());
	  }
}

export default TodoList;
```

###### 4、完成提交按钮，实现list的更新

希望点击按钮时，把input中的数据存在store中的list数组中

首先给Button绑定一个onClick事件，（记得在constructor中绑定this）

```javascript
<Button type="primary" onClick={this.handleBtnClick}>提交</Button>
```



然后编写handleBtnClick的方法，先创建一个action，

```javascript
handleBtnClick(event){
	  	const action={
		    type:"add_todo_item",
		};
		store.dispatch(action);
}
```

注意这里action只定义了事件类型，没有传值，这是因为要改变的数据可以放在reducer中处理，此时可以在reducer中处理这个事件了

```javascript
if(action.type==='add_todo_item'){
		const newState=JSON.parse(JSON.stringify(state));
        //将input中的值添加到list
    	newState.list.push(newState.inputValue);
        //清空input
    	newState.inputValue='';
		return newState;
	}
```



### 五、完成TodoList的删除

在上面已经熟悉了redux的流程，这里就当回顾，顺便实现TodoList的删除的删除

首先我们对List组件的每一项进行事件绑定，List的每一项是通过List.Item进行绑定的，所以我们只需要对List.Item进行事件绑定即可

```javascript
<List
	        	  style={{width:'300px',marginTop:'10px'}}
	          bordered
	          dataSource={this.state.list}
	          renderItem={(item,index) => (<List.Item onClick={this.handleItemDelete.bind(this,index)}>{item}</List.Item>)}
	        />
```



然后我们写handleItemDelete这个方法,	接受一个index 的参数，在里面创建action，传递index，然后传递action

```javascript
handleItemDelete(index){
	  	const action={
		        type:"delete_todo_item",
		        index,
		};
		store.dispatch(action);
}
```



最后在reducer中判断delete_todo_item事件，根据index在list中删除相应下标的数据

```javascript
if(action.type==='delete_todo_item'){
		const newState=JSON.parse(JSON.stringify(state));
        newState.list.splice(action.index,1);
		return newState;
	}
```



最终我们的reducer.js文件内容如下

```javascript
const defaultState = {
    inputValue:'',
    list:  []
}

export default (state=defaultState,action)=>{
	if(action.type==='change_input_value'){
        //对当前state做一个深拷贝
        const newState=JSON.parse(JSON.stringify(state));
        newState.inputValue=action.value;
        return newState;
    }
	
	if(action.type==='add_todo_item'){
		const newState=JSON.parse(JSON.stringify(state));
        newState.list.push(newState.inputValue);
        newState.inputValue='';
		return newState;
	}
	if(action.type==='delete_todo_item'){
		const newState=JSON.parse(JSON.stringify(state));
        newState.list.splice(action.index,1);
		return newState;
	}
	
	return state;
}
```

