---
title: redux中间件
date: 2018-09-12 23:44:33
tags: React
---
## Redux中间件

### 一、解决action类型太多的问题

在之前的例子中，我们在TodoList文件中创建了多种action，并且在reducer文件中对多种action做了区分处理。如果action的数量变得多了以后，就很容易出现错误并且不好调试。如何解决这个问题呢？

我们可以在store下创建一个actionType.js的文件。在里面将不同的action定义成常量，这样就可以在其他文件中直接使用了，又不容易出现错误

actionType.js文件内容可以如下：

```javascript
export const CHANGE_INPUT_VALUE="change_input_value";
export const ADD_TODO_ITEM="add_todo_item";
export const DELETE_TODO_ITEM="delete_todo_item";
```

然后在TodoList中引入这三个常量即可：

```javascript
import {CHANGE_INPUT_VALUE,ADD_TODO_ITEM,DELETE_TODO_ITEM} from './actionType.js';
...
	handleInputChange(event){
		const action={
			type:CHANGE_INPUT_VALUE,
			value:event.target.value,
		};
		store.dispatch(action);
		}
	  
	  //当感知到数据发生变化时，就回去新的state数据
	  handleStoreChange(event){
	  	this.setState(store.getState());
	  }
	  
	  handleBtnClick(event){
	  	const action={
			type:ADD_TODO_ITEM,
		};
		store.dispatch(action);
	  }
	  
	  handleItemDelete(index){
	  	const action={
			type:DELETE_TODO_ITEM,
		    index,
		};
		store.dispatch(action);
	  }
```

同理也需要对reducer中引入这些常量。

### 二、创建action创建器

在之前的例子中，我们都是直接创建action对象，虽然方便，但是这种方式却不便于管理。我们需要一个action创建器来统一创建相应的action。

在store下面创建一个actionCreators.js的文件

输入

```javascript
import {CHANGE_INPUT_VALUE} from './actionType.js';

export const getInputChangeAction=(value)=>({
    type=CHANGE_INPUT_VALUE,
    value,
})
```

然后我们在TodoList文件中修改该action的创建方式

```javascript
import { getInputChangeAction } from './store/actionCreators.js';
...
handleInputChange(event){
		const action=getInputChangeAction(event.target.value);
		store.dispatch(action);
		}
```

同理对其他的action做同样的处理

### 三、Redux使用三原则

* **store唯一**：整个工程只有一个store
* **只有store能够改变自己的内容**：很多同学认为是reducer改变的store中的内容，其实不是的，reducer只是拿到了之前的数据，然后创建一个新的数据，将新的数据返回给了store；其实还是store将数据进行更新
* **Reducer必须是存函数**：存函数给定固定的输入就一定会有固定的输出，而且不会有任何副作用（除了返回东西不会做任何改变）。

store最常用的四个方法：

* createStore
* store.dispatch
* store.getState
* store.subscribe

### 四、拆分UI组件和容器组件

在之前的例子中我们在组件TodoList中既处理UI部分也处理逻辑部分，这样会造成很多混乱，可以考虑将组件拆分成UI组件和容器组件。

#### 4.1编写UI组件

在UI组件中只展示UI，容器部分只处理逻辑

我们可以新建一个TodoListUI.js;

将UI编写UI部分；

此时该组件中的设计的逻辑部分只能通过父组件传递过来,使用this.props.XXX的方式；

最后将该组件输出export

```javascript
import React, {Component} from 'react';
import { Input , Button , List} from 'antd';

class TodoListUI extends Component{
	render(){
		return (
	      <div style={{marginTop:'10px' , marginLeft:'10px'}}>
	        <div>
	        		<Input 
	        			value={this.props.inputValue} placeholder="todo info" 
	        			style={{width:'300px',marginRight:'10px'}} 
	        			onChange={this.props.handleInputChange}/>
	        		<Button type="primary" onClick={this.props.handleBtnClick}>提交</Button>
	        	</div>
	        <List
	        	  style={{width:'300px',marginTop:'10px'}}
	          bordered
	          dataSource={this.props.list}
	          renderItem={(item,index) => (<List.Item onClick={()=>{this.props.handleItemDelete(index)}}>{item}</List.Item>)}
	        />
	      </div>
	    );
	}
}
export default TodoListUI;

```



#### 4.2 修改逻辑组件部分

在TodoList.js文件中，引入TodoListUI，直接添加到render方法中。

然后将TodoListUI需要用到的一些逻辑通过属性传递过去

```javascript
import React, { Component } from 'react';
import 'antd/dist/antd.css';
import store from './store/index';
import { getInputChangeAction , getAddItemAction , getDeleteItemAction } from './store/actionCreator.js';
import TodoListUI from './TodoListUI';

class TodoList extends Component {
	constructor(props){
		super(props);
		this.state=store.getState();
		this.handleInputChange.bind(this);
		this.handleStoreChange=this.handleStoreChange.bind(this);
		this.handleBtnClick=this.handleBtnClick.bind(this);
		this.handleItemDelete=this.handleItemDelete.bind(this);
		
		store.subscribe(this.handleStoreChange);
	}
	
	  render() {
	    return <TodoListUI 
	    				inputValue={this.state.inputValue}
	    				list={this.state.list}
	    				handleInputChange={this.handleInputChange}
	    				handleStoreChange={this.handleStoreChange}
	    				handleBtnClick={this.handleBtnClick}
	    				handleItemDelete={this.handleItemDelete}
	    			/>;
	  }
	  
	  handleInputChange(event){
		const action=getInputChangeAction(event.target.value);
		store.dispatch(action);
		}
	  
	  //当感知到数据发生变化时，就回去新的state数据
	  handleStoreChange(event){
	  	this.setState(store.getState());
	  }
	  
	  handleBtnClick(event){
	  	const action=getAddItemAction();
		store.dispatch(action);
	  }
	  
	  handleItemDelete(index){
	  	const action=getDeleteItemAction(index);
		store.dispatch(action);
	  }
}

export default TodoList;
```

此时UI组件和逻辑组件就分开了，功能同样实现



### 五、无状态组件

一个组件如果没有构造函数，就可以把它写成无状态组件，比如我们上面的TodoListUI，可以改成下面的样式

这时使用props就不用使用this了

```javascript
import React, {Component} from 'react';
import { Input , Button , List} from 'antd';

const TodoListUI=(props)=>{
    return (
	      <div style={{marginTop:'10px' , marginLeft:'10px'}}>
	        <div>
	        		<Input 
	        			value={props.inputValue} placeholder="todo info" 
	        			style={{width:'300px',marginRight:'10px'}} 
	        			onChange={props.handleInputChange}/>
	        		<Button type="primary" onClick={props.handleBtnClick}>提交</Button>
	        	</div>
	        <List
	        	  style={{width:'300px',marginTop:'10px'}}
	          bordered
	          dataSource={props.list}
	          renderItem={(item,index) => (<List.Item onClick={()=>{props.handleItemDelete(index)}}>{item}</List.Item>)}
	        />
	      </div>
	    );
}；
export default TodoListUI;
```

这时候，就完成了，无状态组件性能比较高。因为它只是一个函数，而不需要一些生命周期函数.



### 六、Redux发送ajax请求

想初始化我们的列表，首先在actionType文件中添加一个action类型

```javascript
export const INIT_LIST="init_list";
```

然后在actionCreator中创建该action

```javascript
import {CHANGE_INPUT_VALUE,ADD_TODO_ITEM,DELETE_TODO_ITEM,INIT_LIST} from './actionType.js';

export const getInputChangeAction=(value)=>({
    type:CHANGE_INPUT_VALUE,
    value,
});
export const getAddItemAction=()=>({
    type:ADD_TODO_ITEM,
});
export const getDeleteItemAction=(index)=>({
    type:DELETE_TODO_ITEM,
    index,
});
export const initListAction=(data)=>({
    type:INIT_LIST,
    data,
});

```

在TodoList中引入该action，并且引入axios，添加componentDidMount生命周期函数,在该函数中创建action，然后将action传递给store

```javascript
import { getInputChangeAction , getAddItemAction , getDeleteItemAction,initListAction } from './store/actionCreator.js';
import axios from 'axios'; 
...
componentDidMount(){
	  	axios.get('/list.json').then((res)=>{
	  		const data=res.data;
	  		const action=initListAction(data);
	  		store.dispatch(action);
	  	});
	  }
```

最后写reducer，在reducer中完成对数据的处理

```javascript
import {CHANGE_INPUT_VALUE,ADD_TODO_ITEM,DELETE_TODO_ITEM,INIT_LIST} from './actionType.js';
。。。
if(action.type===INIT_LIST){
		const newState=JSON.parse(JSON.stringify(state));
        newState.list=action.data;
		return newState;
	}
```

### 七、Redux-thunk中间件发送ajax请求

如果把过多的异步请求或逻辑放在一个组件中，会显得组件特别臃肿。我们希望将它们移除在其它地方进行统一管理。放在哪里呢？Redux-thunk可以将它们放在action 中去处理。Redux-thunk如何使用呢？

#### 7.1 安装redux-thunk

首先在github[官网](https://github.com/reduxjs/redux-thunk)上查看，在工程目录下输入

```javascript
npm install --save redux-thunk

//或
yarn add redux-thunk
```



那该如何使用中间件来创建store呢？

首先在store/index.js文件中，引入redux的applyMiddleware，并且引入thunk

```javascript
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
```

然后在createStore中使用applyMiddleware方法，将thunk放在里面里面就行了。

```javascript
const store = createStore(reducer,
			applyMiddleware(thunk)
			);
```

#### 7.2 redux同时使用dev-tools和thunk中间件

如果想将之前的`window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()`和thunk同时使用该怎么办呢？它们全都是Redux的中间件，可以去github的[REDUX_DEVTOOLS](https://github.com/zalmoxisus/redux-devtools-extension)	中查看使用方式

* 首先声明`window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()`，

* 然后扩展thunk
* 最后使用即可

```javascript
import { createStore , applyMiddleware , compose} from 'redux';
import reducer from './reducer';
import thunk from 'redux-thunk';

const composeEnhancers =window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ? window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({}) : compose;
const enhancer = composeEnhancers(
  applyMiddleware(thunk),
);
const store = createStore(reducer,enhancer);

export default store;
```

这样我们的代码就可以既支持devtools，又支持thunk了

#### 7.3 使用thunk编写代码

之前我们的aciton都是一个函数，函数返回的是一个对象，如下

```javascript
export const initListAction=(data)=>({
    type:INIT_LIST,
    data,
});
```

在使用thunk之后，我们action，函数返回的也可以是一个函数

首先我们将TodoList的componentDidMount函数中的异步代码删除。

然后在actionCreators.js中,放进来，注意这里不用在引入store了，因为返回的函数自动接收了store的dispatch方法，可以直接将里面创建的action派发出去了。

```javascript
export const getTodoList=()=>({
    return (dispatch)=>{
    	axios.get('/todolist.json').then((res)=>{
	  		const data=res.data;
	  		const action=initListAction(data);
	  		dispatch(action);
	  	});
	}
});
```

这里我们使用axios，所以需要在actionCreators.js文件中添加两个引用

```javascript
import axios from 'axios';
```

然后在TodoList.js组件的componentDidMount函数中创建getTodoList的action，注意需要先import

```javascript
import { getTodoList, getInputChangeAction , getAddItemAction , getDeleteItemAction,initListAction } from './store/actionCreator.js';
...

componentDidMount(){
    const action=getTodoList();
    store.dispatch(action);
}
```

注意这里之所以能将函数发送出去，就是因为使用了redux-thunk中间件

### 八、什么是Redux的中间件

已经使用redux中间件redux-thunk了，到底什么是中间件呢

redux的中间件指的是action和store之间，thunk就是对store的dispatch方法做了一个升级，之前的是只能接受一个对象，现在可以接收一个函数，当接受一个函数时先执行函数再分发

![](/Users/Afei/Documents/文章/js/img/redux-data-flow.png)

### 九、Redux-saga中间件

这里再强调一下，中间件指的是action和store之间，因为只有redux才有store和action。之前我们学习的redux-thunk将异步过程封装到了action之中，这样有助于我们做自动化测试，以及异步代码拆分管理。redux-saga也是做这种异步代码拆分的工作，可以用redux-saga完全代替redux-thunk。

#### 9.1 安装Redux-saga

首先进入[官网](https://github.com/redux-saga/redux-saga)，进行Redux-saga的安装

```javascript
npm install --save redux-saga
//或
yarn add redux-saga
```

然后在store/index.js组件中引入createSagaMiddleware，用于创建中间件

```javascript
import createSagaMiddleware from 'redux-saga'
...
// create the saga middleware
const sagaMiddleware = createSagaMiddleware()
```

之前在redux-thunk中是通过composeEnhancers将中间件传进入，redux-saga也一样

```javascript
const composeEnhancers =window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ ? window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({}) : compose;
const enhancer = composeEnhancers(
  applyMiddleware(sagaMiddleware),
);
const store = createStore(reducer,enhancer);
```

在store文件夹下创建一个sagas.js的文件，然后在store/index.js文件中引入该文件，并执行saga文件

```javascript
import todoSagas from './sagas';
...

// then run the saga
sagaMiddleware.run(todoSagas)
```

sagas.js文件中写什么呢？一定要导出一个generator函数

```javascript
function* todoSagas() {
}

export default todoSagas;
```

#### 9.2 使用redux-saga

在actionCreator.js创建一个action,是一个函数返回一个对象.

首先在actionType.js中创建GET_INIT_LIST

```javascript
export const GET_INIT_LIST="get_init_list";
```

```javascript
import {CHANGE_INPUT_VALUE,ADD_TODO_ITEM,DELETE_TODO_ITEM,GET_INIT_LIST} from './actionType.js';
export const getInitList=()=>({
    type:GET_INIT_LIST
});
```

然后在TodoList中使用

```javascript
componentDidMount(){
	const action=getInitList();
    store.dispatch(action);
}
```

此时我们不仅能在reducer中接收action了，还能在sagas.js中接收了，然后将异步过程放在这里。在这里将创建的action如何发布出去呢，可以使用redux-saga中的put

```javascript
import { takeEvery , put} from 'redux-saga/effects';
import { GET_INIT_LIST } from './actionType';
import axios from 'axios';
import { initListAction } from './store/actionCreator.js';

//generator函数
function* todoSagas() {
    //takeEvery：捕捉每一个XX类型的action,然后调用fetchUser这个方法
    yield takeEvery(GET_INIT_LIST, getInitList);
}

function* getInitList(){
    //这里无需使用promise了
    const res=yield axios.get('/todolist.json');
    const action=initListAction(res.data);
    yield put(action);
}

export default todoSagas;
```

注意：这是在getInitList函数中如果`yield axios.get('/todolist.json');`获取成功，肯定是没问题的，但是如果获取不成功呢，ajax失败情况怎么办呢，可以使用try-catch

```javascript
function* getInitList(){
    try{
        //这里无需使用promise了
        const res=yield axios.get('/todolist.json');
        const action=initListAction(res.data);
        yield put(action);
    }catch(e){
        console.log("网络请求失败");
    }
}
```

redux-thunk基本没有api，它只是让我们在action不仅仅返回对象，还能返回函数；

而redux-saga又很多常用的API，在大型项目中很适合

### 十、React-Redux的使用

React-Redux是一个第三方的模块，可以帮助我们在react中更加方便的使用redux。首先我们先将我们的工程清空，只剩一个index文件。

#### 10.1 安装React-Redux

```javascript
npm install --save react-redux
//或
yarn add react-redux
```

然后编写TodoList.js文件

```javascript
import React,{ Component } from 'react';

class TodoList extends Component{
    render(){
        return (
        	<div>
            	<div>
                    <input />
                    <button>提交</button>
            	</div>
            	<ul>
            		<li>VUE</li>
            	</ul>
            </div>
        );
    }
}

export default TodoList;
```

#### 10.2 创建store

首先创建store文件夹，并在下面创建index.js文件，在里面输入创建store（管理员）

```javascript
import { createStore } from 'redux';
import { reducer } from './reducer';

const store=createStore();

export default store;
```

并且在store目录下创建一个reducer.js（记事本），这是一个纯函数。并在里面定义一些初始值

```javascript

const defaultState={
    inputValue:'',
    list:[],
};
export default (state=defaultState,action)=>{
    return state;
}
```

这样我们redecer和store就创建好了。



#### 10.3 在TodoList.js中使用store中的数据

首先在TodoList.js中引入store;

并在构造函数中通过store设置setState;

然后在input中加入对state的引用

```javascript
import React,{ Component } from 'react';
import store from './store/index';

class TodoList extends Component{
    constructor(props){
        super(props);
        this.state=store.getState();
    }
    render(){
        return (
        	<div>
            	<div>
                    <input value={this.state.inputValue}/>
                    <button>提交</button>
            	</div>
            	<ul>
            		<li>VUE</li>
            	</ul>
            </div>
        );
    }
}

export default TodoList;
```

那我们该如何使用React-Redux呢？

#### 10.4 使用React-Redux

在index.js文件中，之前是直接渲染TodoList，现在我们引入react-redux，并且声明一个App，将TodoList传给App，最后将App传给RenderDom使用

```javascript
import React from 'react';
import ReactDom from 'react-dom';
import TodoList from './TodoList';
import { Provider } from 'react-redux';

const App=（
	<Provider>
      <TodoList />
    </Provider>
）；
RenderDom.render(App, document.getElementById("root"));
//RenderDom.render(<TodoList />, document.getElementById("root"));
```

这里将TodoList组件传给了Provider组件，Provider组件来自于react-redux。

provider组件可以做什么呢？provider可以提供链接store的能力

我们继续引入store，然后在provider组件中使用如下，这样Provider内的所有组件都有获取store的能力

```javascript
import store from './store';

const App=（
	<Provider store={store}>
      <TodoList />
    </Provider>
）；
```

此时TodoList组件有获取store的能力了，它怎么获取store呢？

此时我们在TodoList.js中就可以改变获取store的方式了，首先引入react-redux中的connnect，然后到处connect方法。connect的方法是让TodoList和store做连接

```javascript
import React,{ Component } from 'react';
import { connect } from 'react-redux';

class TodoList extends Component{
    render(){
        return (
        	<div>
            	<div>
                    <input value={this.state.inputValue}/>
                    <button>提交</button>
            	</div>
            	<ul>
            		<li>VUE</li>
            	</ul>
            </div>
        );
    }
}

export default connect(null,null)(TodoList);
```

connect是怎么做连接的呢？首先我们可以创建一个mapStateToProps，然后将它放在connect的一个参数处

```javascript
//将state中的数据映射给props
const mapStateToProps=(state)=>{
    return {
        inputValue：state.inputValue,
    };
};
export default connect(mapStateToProps,null)(TodoList);
```

这时我们input中的value就不是等于{this.state.inputValue}了，不使用state了改为props，需要改为

```javascript
<input value={this.props.inputValue}/>
```



然后我们实现onChange方法，在input标签中添加该监听，并且编写一个mapDispatchToProps，将它放在connect的第二个参数处。然后

```javascript
<input value={this.props.inputValue} onChange={this.props.changeInputValue)}/>
...

handleInputChange(event){
    
}
//store.dispatch 挂载到props上
const mapDispatchToProps=(dispatch)={
    return {
        changeInputValue(e){
    		const action={
                type:'change_input_value',
                value:e.target.value,
            };
			dispatch(action);
    	}
	}
}
export default connect(mapStateToProps,mapDispatchToProps)(TodoList);
```

然后就可以在reduce中做处理了

```javascript
const defaultState={
    inputValue:'',
    list:[],
};
export default (state=defaultState,action)=>{
    if(action.type==='change_input_value'){
    	cost newState=JSON.parse(JSON.stringify(state));
        newState.inputValue=action.value;
        return newState;
    }
    return state;
}
```

这里对react-redux的使用就讲解的差不多了，



接下来我们给button添加一个监听，然后在mapDispatchToProps中添加这个监听函数

```javascript
<button onClick={this.props.handleClick}>提交</button>
...

const mapDispatchToProps=(dispatch)={
    return {
        changeInputValue(e){
    		const action={
                type:'change_input_value',
                value:e.target.value,
            };
			dispatch(action);
    	}，
        handleClick(e){
            //...
        }
	}
}
```

然后在做action类型的处理即可，不再重复。

同理对于列表，可以这么处理

```javascript
<ul>
    {
    	this.props.list.map((item,index)=>({return <li key={index}>{item}</li>}));
	}
</ul>

...
const mapStateToProps=(state)=>{
    return {
        inputValue：state.inputValue,
        list:state.list,
    };
};
```

这是代码中出现太多的this.props.XXX，其实可以精简一下，在render函数中使用结构赋值，就可以在直接使用相应的函数了

```javascript
render(){
    const {inputValue,list,changeInputValue,handleClick} =this.props;
}
```

此时的TodoList就是一个UI组件，可以改为无状态组件，因为无状态组件性能比较高，可以直接写成函数的形式

```javascript
import React,{ Component } from 'react';
import { connect } from 'react-redux';

const TodoList=(props)=>{
    const {inputValue,list,changeInputValue,handleClick} =props;
    return (
        	<div>
            	<div>
                    <input value={inputValue} onChange={changeInputValue}/>
                    <button onClick={handleClick}>提交</button>
            	</div>
            	<ul>
            		{list.map((item,index)=>({return <li key={index}>{item}</li>}));}
            	</ul>
            </div>
        );
}
const mapStateToProps=(state)=>{
    return {
        inputValue：state.inputValue,
        list:state.list,
    };
};
const mapDispatchToProps=(dispatch)={
    return {
        changeInputValue(e){
    		const action={
                type:'change_input_value',
                value:e.target.value,
            };
			dispatch(action);
    	}，
        handleClick(e){
            //...
        }
	}
}

export default connect(mapStateToProps,mapDispatchToProps)(TodoList);
```

