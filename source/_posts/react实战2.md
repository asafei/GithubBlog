---
title: react实战2
date: 2018-10-18 22:21:56
tags: React
---
### 一、修改搜索框

#### 1.1、修改搜索框文字颜色和间距

当在搜索框填写的内容过长时，间距就不够了，并且输入的文字颜色有点深。我们首先调整一下文字颜色，在`common/header/style.js`中修改

```javascript
export const NavSearch=styled.input.attrs({
	placeholder:'搜索'
})`
	width:160px;
	height:38px;
		padding:0 30px 0 20px;
	box-sizing:border-box;
	border:none;
	outline:none;
	margin-top:9px;
	margin-left:20px;
	border-radius:19px;
	background:#eee;
	font-size:14px;
		color:#666;
	&::placeholder{
		color:#999;
	}
`;
```

#### 1.2、修改搜索框鼠标动画

下面我们来添加一下鼠标的移入移出效果，搜过框的宽度锁着鼠标的变化会自动的变化。当聚焦的时候，搜索框就会变长。

react不建议我们直接操作dom，所以我们可以通过定义数据来实现，首先我们在`common/header/index.js`中添加constructor函数，并在里面定义state；并且在NavSearch标签中添加className={this.state.focused},

```javascript
constructor(props){
    super(props);
    this.state={
        focused:false,
    };
}
...
<SearchWrapper>
	<NavSearch className={this.state.focused ? 'focused':''}></NavSearch>
	<i className='iconfont'>&#xe6dd;</i>
</SearchWrapper>
```

然后我们在`common/header/style.js`中添加focused的样式

```javascript
export const NavSearch=styled.input.attrs({
	placeholder:'搜索'
})`
	width:160px;
	height:38px;
	padding:0 35px 0 20px;
	box-sizing:border-box;
	border:none;
	outline:none;
	margin-top:9px;
	margin-left:20px;
	border-radius:19px;
	background:#eee;
	font-size:14px;
	color:#666;
	&::placeholder{
		color:#999;
	}
        &.focused {
            width:200px;
        }
`;
```

并且注意当搜索框变长时，右边搜索按钮背景色也发生了变化，此时需要修改`common/header/index.js`中的i标签中添加新的className，并且在`common/header/style.js`中SearchWrapper标签的样式中设置iconfont搜索按钮的背景颜色

```javascript
<SearchWrapper>
<NavSearch className={this.state.focused ? 'focused':''}></NavSearch>
<i className='iconfont'>&#xe6dd;</i>
</SearchWrapper>

//改为
<SearchWrapper>
<NavSearch className={this.state.focused ? 'focused':''}></NavSearch>
<i className={this.state.focused ? 'focused iconfont':'iconfont'}>&#xe6dd;</i>
</SearchWrapper>
```

```javascript
export const SearchWrapper=styled.div`
	float:left;
	position:relative;
	.iconfont {
		position:absolute;
		right:5px;
		bottom:5px;
		width:30px;
		line-height:30px;
		border-radius:15px;
		text-align:center;
        &.focused{
			background:#777;
			color: #fff;
        }
	}
`;
```

#### 1.3、实现搜索框事件绑定

接下来实现，通过事件来实现长短切换，首先focused默认时false，然后在NavSearch标签中添加onFocus属性，在constructor实现对this的绑定，然后就可以添加handleInputFocus函数了

```javascript
constructor(props){
	super(props);
	this.state={
		focused:false,
	};
	this.handleInputFocus=this.handleInputFocus.bind(this);
}
...
<NavSearch 
	className={this.state.focused ? 'focused':''}
	onFocus={this.handleInputFocus}>
</NavSearch>
...
handleInputFocus(){
    this.setState({
    	focused:true
    });
}
```

这时就实现了当搜索框聚焦时，就会变长，并且搜索按钮背景色变暗。可以失焦时，却没有变短，这很好办，只需添加onBlur属性即可

```java
constructor(props){
	super(props);
	this.state={
		focused:false,
	};
	this.handleInputFocus=this.handleInputFocus.bind(this);
    this.handleInputBlur=this.handleInputBlur.bind(this);
}
...
<NavSearch 
	className={this.state.focused ? 'focused':''}
	onFocus={this.handleInputFocus}
	onBlur={this.handleInputBlur}>
</NavSearch>
...
handleInputFocus(){
    this.setState({
    	focused:true
    });
}
handleInputBlur(){
    this.setState({
    	focused:false
    });
}
```

此时就实现了聚焦和失焦的效果，但是此时还没有动画的效果，下面添加动画。

#### 1.4、添加动画变长变短

首先在控制台，进入jianshu项目，安装react-transition-group的插件，

```javascript
npm install react-transition-group --save
```

然后在`common/header/index.js`中引入CSSTransition，然后用CSSTransition将NavSearch包裹起来

```javascript
import { CSSTransition } from 'react-transition-group';
。。。
<CSSTransition
	in={this.state.focused} //入场动画
	timeout={200} //动画时长
	classNames="slide"
	>
    <NavSearch 
        className={this.state.focused ? 'focused':''}
        onFocus={this.handleInputFocus}
        onBlur={this.handleInputBlur}>
    </NavSearch>
</CSSTransition>
```

此时就会在外层挂载几个样式，然后找到SearchWrapper标签的样式，在`common/header/style.js`中SearchWrapper标签的样式中设置slide-enter和slide-enter-active设置展开效果，通过slide-exit和slide-exit-active设置缩回效果

```javascript
export const SearchWrapper=styled.div`
	float:left;
	position:relative;
    .slide-enter {
		width:160px;
		transition:all .2s ease-out;
    }
    .slide-enter-active {
		width:240px;
    }
	.slide-exit {
		width:240px;
		transition:all .2s ease-out;
    }
    .slide-exit-active {
		width:160px;
    }
	.iconfont {
		position:absolute;
		right:5px;
		bottom:5px;
		width:30px;
		line-height:30px;
		border-radius:15px;
		text-align:center;
		&.focused{
			background:#777;
			color: #fff;
        }
	}
`;
```

### 二、使用React-Redux管理数据

首先我们看CSSTransition标签，它会给内部的NavSearch标签自动添加样式，这些样式在它外层样式（SearchWrapper）中定义，包含slide-enter和slide-enter-active、slide-exit和slide-exit-active。而实际中，应该将这些样式写在NavSearch标签上。

首先我们将SearchWrapper样式中的slide-enter和slide-enter-active、slide-exit和slide-exit-active样式剪切到NavSearch样式中，注意在前面添加`&`表示同级样式

```javascript
export const NavSearch=styled.input.attrs({
	placeholder:'搜索'
})`
	width:160px;
	height:38px;
	padding:0 35px 0 20px;
	box-sizing:border-box;
	border:none;
	outline:none;
	margin-top:9px;
	margin-left:20px;
	border-radius:19px;
	background:#eee;
	font-size:14px;
	color:#666;
	&::placeholder{
		color:#999;
	}
	&.focused {
        width:240px;
    }
	&.slide-enter {
		width:160px;
		transition:all .2s ease-out;
    }
    &.slide-enter-active {
		width:240px;
    }
	&.slide-exit {
		width:240px;
		transition:all .2s ease-out;
    }
    &.slide-exit-active {
		width:160px;
    }
`;
```

这里用到了state来管理数据，如果是小型项目这么写没问题，但是大型项目时需要使用redux来管理数据。为了便于维护，所有的数据尽量放在redux中

#### 2.1 安装使用redux

首先在jianshu目录先安装redus和react-redux

```javascript
//安装redux数据框架
npm install --save redux
//方便在react中使用redux
npm install --save react-redux
```

然后在src目录下创建一个store的文件夹，在store文件夹下创建一个index.js的文件；并且创建一个reducer(`src/store/reducer.js`)给它

```javascript
import { createStore } from 'redux';
import reducer from './reducer';

const store=createStore(reducer);

export default store;
```

```javascript
const defaultState={};

export default (state=defaultState,action)=>{
	return state;
}
```

这时就创建好了store，就需要往store中存数据和取数据 。

#### 2.2 使用store存取数据

打开`src/App.js`,引入store，并且从react-redux中引入Provider，然后在reder

函数句将Header标签用Provider包裹起来，设置store属性，表示Provider把store中的数据都提供给了它内部的组件

```javascript
import React, { Component } from 'react';
import { Provider } from 'react-redux';
import Header from './common/header/index.js';
import store from './store';

class App extends Component {
  render() {
    return (
    	<Provider store={store}>
      		<Header />
      	</Provider>

    );
  }
}

export default App;
```

然后在`src/common/header/index.js`文件中，做一下数据的连接，首先从react-redux引入connect，并且利用connect进行输出

```javascript
import React , { Component } from 'react';
import { connect } from 'react-redux';
。。。
const mapStateToProps=(state)=>{
	return {}
}

const mapDispatchToProps=(dispatch)=>{
	return {}
}

export default connect(mapStateToProps,mapDispatchToProps)(Header);
```

然后就可以清理`src/common/header/index.js`文件中的数据了，删除里面的数据定义

~~this.state={
	focused:false,
};~~

在`src/store/index.js`文件中定义默认的state，包含focus

```javascript
const defaultState={
	focused:false
};

export default (state=defaultState,action)=>{
	return state;
}
```

这样就把focused的数据放在了redux的仓库里。接着就应该将仓库里的focused映射到props中去，可以在mapStateToProps中操作

```javascript
const mapStateToProps=(state)=>{
    return {
        focused:state.focused
    }
}
```

此时就可以将`src/common/header/index.js`中涉及focus的数据，改为this.props.focused了。

这时也可以把之前的handle函数全部删除了，因为不直接操作state了。修改onFocus和onBlur，并在mapDispatchToProps中创建相应的action，然后做分发

```javascript
<NavSearch 
	className={this.props.focused ? 'focused':''}
	onFocus={this.props.handleInputFocus}
	onBlur={this.props.handleInputBlur}></NavSearch>
。。。
const mapDispatchToProps=(dispatch)=>{
	return {
		handleInputFocus(){
			const action={
				type:'search_focus'
			};
			dispatch(action);
		},
		handleInputBlur(){
			const action={
				type:'search_blur'
			};
			dispatch(action);
		}
	}
}
```

然后在`src/store/reducer.js`中对action类型进行判断，

```javascript
const defaultState={
	focused:false
};

export default (state=defaultState,action)=>{
	if(action.type==='search_focus'){
		return {
			focused:true
		}
	}
	if(action.type==='search_blur'){
		return {
			focused:false
		}
	}
	return state;
}
```

此时，我们的页面又有聚焦和失焦功能了。 

此时我们也就没必要使用构造函数了，header组件就变成了无状态组件，可以将它变成无状态组件

```javascript
import React from 'react';
import { connect } from 'react-redux';
import { HeaderWrapper , Logo , Nav , NavItem , NavSearch, Addition,Button,SearchWrapper} from './style.js';
import { CSSTransition } from 'react-transition-group';

const Header= (props)=>{
	return (
			<HeaderWrapper>
				<Logo href='/'/>
				<Nav>
				    <NavItem className='left active'>首页</NavItem>
				    <NavItem className='left'>下载App</NavItem>
				    <NavItem className='right'>登陆</NavItem>
				    <NavItem className='right'>
				    	<i className='iconfont'>&#xe636;</i>
				    </NavItem>
				    <SearchWrapper>
				    	<CSSTransition
							in={ props.focused }
							timeout={200}
							classNames="slide"
							>
						    <NavSearch 
						    	className={props.focused ? 'focused':''}
						    	onFocus={props.handleInputFocus}
						    	onBlur={props.handleInputBlur}></NavSearch>
					    </CSSTransition>
					    <i className={props.focused ? 'focused iconfont':'iconfont'}>&#xe6dd;</i>
				    </SearchWrapper>
				</Nav>
				<Addition>
					<Button className='writting'>
						<i className='iconfont'>&#xe60e;</i>
						写文章
					</Button>
					<Button className='reg'>注册</Button>
				</Addition>
			</HeaderWrapper>
	);
}

const mapStateToProps=(state)=>{
	return {
		focused:state.focused
	}
}

const mapDispatchToProps=(dispatch)=>{
	return {
		handleInputFocus(){
			const action={
				type:'search_focus'
			};
			dispatch(action);
		},
		handleInputBlur(){
			const action={
				type:'search_blur'
			};
			dispatch(action);
		}

	}
}
export default connect(mapStateToProps,mapDispatchToProps)(Header);
```

这时我们还不能使用redux开发者工具，怎么办呢，使用**redux-devtools-extension**,这时需哟啊对store对什么处理呢？从redux中引入compose，然后使用composeEnhancers增强功能即可

```javascript
import { createStore, compose } from 'redux';
import reducer from './reducer';


const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;
const store=createStore(reducer,composeEnhancers());

export default store;
```

####2.3 reducer分类 

reducer可以看作管理员手册，但是当手册管理内容增多时，怎么办呢？可以对手册作个分类，这样图书馆管理员再次查找手册时，就可以先找分类，然后再找内容。redux就可以将一个reducer拆分成很多小的reducer。

因为focused就是header的数据，我们可以在`src/common/header/`下创建文件夹store，并在里面创建reducer.js的文件，然后将`src/store/reducer`中的数据剪切到`src/common/header/store/reducer.js`之中。

这时`src/store/reducer`已经为空，项目也会报错，相当于把`src/store/reducer`中的笔记本拆分出去了，这里如何将拆分出去的笔记本整合起来呢？Redux提供了combineReducers使用，它可以将很多小的reducer合并成大的reducer，可以在`src/store/reducer`中如下

```javascript
import {combineReducers} from 'redux';
import headerReducer from '../common/header/store/reducer.js';

export default combineReducers({
	header:headerReducer
})
```

此时页面也恢复正常了，通过**redux-devtools-extension**工具可以看到focused位于了header下，而不再是顶层。但是这时搜索框的动画却没了，怎么回事呢

因为我们在`src/common/header/index.js`中的mapStateToProps是直接使用state 下的focused，这里需要将它们改造成state.header.focused

```javascript
const mapStateToProps=(state)=>{
	return {
		focused:state.header.focused
	}
}
```

这时我们的页面就恢复正常了

#### 2.3 总结1

我们可以对每一部分，创建自己的reducer，然后在总的reducer中引入各个部分的reducer，并利用redex的combineReducers来创建一个总的reducer，并输出

```javascript
import {combineReducers} from 'redux';
import headerReducer from '../common/header/store/reducer.js';

const reducer=combineReducers({
	header:headerReducer
})
export default reducer;
```

这里还有一个可优化的地方，我们看到在引入headerReducer时路径非常长，我们可以在`src/common/header/store`下新建一个index.js的文件，然后在里面引入自己的reducer，并且输出；然后我们在总的reducer文件中就可以直接引用该`src/common/header/store`就可以，它会自动寻找该路径下的index.js文件；为了避免命名冲突可以使用as关键字。

`src/common/header/store/index.js`

```javascript
import reducer from './reducer.js';

export { reducer };
```

`src/store/index.js`

```javascript
import {combineReducers} from 'redux';
~~import headerReducer from '../common/header/store/reducer.js';~~
import { reducer as headerReducer } from '../common/header/store';

export default combineReducers({
	header:headerReducer
})
```



#### 2.4 使用actionCreator创建action

这里还要注意，创建action时尽量不要使用字符串，我们使用actionCreator来通过常量创建action。

我们在`src/common/header/store/`下创建一个名为`actionCreators.js`的文件。

```javascript
export const searchFocus=()=>({
    type:'search_focus'
});

export const searchBlur=()=>({
    type:'search_blur'
});
```

然后在`src/common/header/index.js`中引入该creator，然后修改mapDispatchToProps中相应ation的创建

```javascript
import * as actionCreators from './store/actionCreators';
。。。
const mapDispatchToProps=(dispatch)=>{
	return {
		handleInputFocus(){
			const action=actionCreators.searchFocus();
			dispatch(action);
		},
		handleInputBlur(){
			const action=actionCreators.searchBlur();
			dispatch(action);
		}

	}
}
```

更进一步，将`src/common/header/store/actionCreator.js`中的type用常量给替换掉。在

`src/common/header/store/`下创建一个constants.js的文件；并且在`src/common/header/store/actionCreator.js`和在`src/common/header/store/reducer.js`文件中 中引用该常量；

```javascript
export const SEARCH_FOCUS='header/SEARCH_FOCUS';
export const SEARCH_BLUR='header/SEARCH_BLUR';
```

```javascript
import * as constants from './constants.js';

export const searchFocus=()=>({
    type:constants.SEARCH_FOCUS
});

export const searchBlur=()=>({
    type:constants.SEARCH_BLUR
});
```

```javascript
import * as constants from './constants.js';
const defaultState={
	focused:false
};

export default (state=defaultState,action)=>{
	if(action.type===constants.SEARCH_FOCUS){
		return {
			focused:true
		}
	}
	if(action.type===constants.SEARCH_BLUR){
		return {
			focused:false
		}
	}
	return state;
}
```

注意，这里我们在`src/common/header/index.js`中引用了`src/common/header/store/actionCreator.js`文件中的内容，而我们希望store下只有一个出口文件，而不是一个一个引入store下具体的文件，我们在`src/common/header/store/index.js`中作统一处理,这样这个index文件就引入了store文件夹下的所有内容，在外部只需引入这一个index文件即可

```javascript
import reducer from './reducer.js';
import * as actionCreators from './actionCreators.js';
import * as constants from './constants.js';

export { reducer , actionCreators , constants};
```

这时也需要将`src/common/header/index.js`中引入的creator换一下

```javas
~~import * as actionCreators from './store/actionCreators';~~
improt {actionCreators} from './store/index.js';
```

#### 2.5 总结2

这时，store文件夹下的内容就非常清晰了，将所有输出都通过index.js作统一处理，外部只需要引用index.js文件就可以。

#### 2.6 使用immutable来管理store中的数据

reducer只能获取state中的数据，而不能改变原始state中的数据，而时创建一个新的state返回。这样很容易出错，immutable就是解决这个问题。

immutable就是不可变更的，如果将state设成immutable，那么这个state就是不可改变的，这样reducer就不会出现问题 ，

首先安装immutable.js，

```javascript
npm install immutable
//或
yarn add immutable
```

接下来就需要将state变成immutable对象了，来到`src/common/header/store/reducer.js`,引入immutable中的fromJS，他可以帮助我们把一个js对象转化为一个immutable对象，然后就可以将defaultState转化为immutable对象了 

```javascript
import { fromJS } from 'immutable'
...
const defaultState=fromJS({
    focused:false
});

```

然后打开`src/common/header/index.js`文件，修改里面mapStateToProps方法中的数据

```javascript
const mapStateToProps=(state)=>{
    return {
        ~~focused:state.header.focused~~
        focused:state.header.get('focused ')
    }
}
```

但是此时，点击搜索框还是会报错，这是由于这时在reducer中返回的是普通对象，需要使用immutable的set方法来做更改，immutable对象set方法会结合之前immutable对象的值和设置的值返回一个全新的对象。

```javascript
export default (state=defaultState,action)=>{
	if(action.type===constants.SEARCH_FOCUS){
		return state.set('focused',true);
	}
	if(action.type===constants.SEARCH_BLUR){
		return state.set('focused',false);
	}
	return state;
}
```

这样就可以避免不小心更改state的错误了。

#### 2.7 使用redux-immutable统一数据格式

我们注意看一下 focused:state.header.get('focused ')这个代码，state是一个js对象，而state.header是一个immutable对象，所以要调用focused的时候需要xian 调用.再调用.get()。这种混在的方式不是很好。

我们可以考虑将state变成immutable对象，也就是对`src/store/reduce.js`，这时我们需要依赖一个第三方的模块，redux-immutable，

```javascript
npm install redux-immutable
```

之前我们redux从redux中来，赖在我们修改它从redux-immutable中来

```javascript
~~import {combineReducers} from 'redux';~~
import {combineReducers} from 'redux-immutable';
```

但是此时页面会报错，我们需要改变一下`src/common/header/index.js`，里面的state方法就不能用.了，需要用get()方法了，这样整个对数据的操作就统一了

```javascript
const mapStateToProps=(state)=>{
    return {
        focused:state.get('header').get('focused ')
    }
}
```

immutable中还有很多写法，比如可以将`focused:state.get('header').get('focused ')`用getIn方法改写`focused:state.getIn(['header','focused'])`，数组表示取header下面的focused项。

### 三、热门搜索布局

当我们鼠标聚焦在搜索框时，会弹出热门搜索框，，接下来我们做搜索部分的热门搜索的布局。

#### 3.1 添加SearchInfo

首先我们需要在`src/common/header/index.js`中添加一个SearchInfo组件，并且在style中添加这个组件

```javascript
import { HeaderWrapper , Logo , Nav , NavItem , NavSearch, Addition,Button,SearchWrapper,SearchInfo} from './style.js';
...
				<SearchWrapper>
				    	<CSSTransition
							in={ props.focused }
							timeout={200}
							classNames="slide"
							>
						    <NavSearch 
						    	className={props.focused ? 'focused':''}
						    	onFocus={props.handleInputFocus}
						    	onBlur={props.handleInputBlur}></NavSearch>
					    </CSSTransition>
					    <i className={props.focused ? 'focused iconfont':'iconfont'}>&#xe6dd;</i>
				    	<SearchInfo></SearchInfo>
				    </SearchWrapper>
```

然后打开`src/common/header/style.js`中编写SearchInfo

```javascript
export const SearchInfo=styled.div`
	position:absolute;
	left:0;
	top:50px;
	width:240px;
	height:100px;
	padding:0 20px;
	background:green;
`;
```

然后设置阴影，从简书官网获取，添加阴影，就可以去掉background了

```javascript
export const SearchInfo=styled.div`
	position:absolute;
	left:0;
	top:50px;
	width:240px;
	height:100px;
	padding:0 20px;
	box-shadow: 0 0 8px rgba(0,0,0,.2);
`;
```

#### 3.2 添加SearchInfoTitle

然后在SearchInfo里面添加新标签SearchInfoTitle了

首先在`src/common/header/index.js`SearchInfo组件内添加SearchInfoTitle，并且引入SearchInfoTitle，然后在style中定义该title

```javascript
import { HeaderWrapper , Logo , Nav , NavItem , NavSearch, Addition,Button,SearchWrapper,SearchInfo,SearchInfoTitle} from './style.js';
。。。
<SearchInfo>
	<SearchInfoTitle>热门搜索</SearchInfoTitle>
</SearchInfo>
```

```javascript
export const SearchInfoTitle=styled.div`
	margin-top:20px;
	margin-bottom:15px;
	line-height:20px;
	font-size:14px;
	color: #969696;
`;
```

#### 3.3 添加写换一批

我们在SearchInfoTitle中添加组件SearchInfoSwitch组件，同样引入并定义该样式

```javascript
import { HeaderWrapper , Logo , Nav , NavItem , NavSearch, Addition,Button,SearchWrapper,SearchInfo,SearchInfoTitle, SearchInfoSwitch} from './style.js';
。。。
<SearchInfo>
	<SearchInfoTitle>热门搜索
	<SearchInfoSwitch>换一批</SearchInfoSwitch>
	</SearchInfoTitle>
</SearchInfo>
```

```javascript
export const SearchInfoSwitch=styled.div`
	float: right;
    font-size: 13px;
    color: #969696;
    background-color: transparent;
    border-width: 0;
    padding: 0;
`;
```

#### 3.4 添加提示内容

然后在SearchInfoTitle下方添加一个div，并且在里面添加组件SearchInfoItem，同样引入并创建该样式，它是一个a标签

```javascript
import { HeaderWrapper , Logo , Nav , NavItem , NavSearch, Addition,Button,SearchWrapper,SearchInfo,SearchInfoTitle, SearchInfoSwitch, SearchInfoItem} from './style.js';
。。。
<SearchInfo>
	<SearchInfoTitle>热门搜索
	<SearchInfoSwitch>换一批</SearchInfoSwitch>
	</SearchInfoTitle>
	<div>
		<SearchInfoItem>教育</SearchInfoItem>
		<SearchInfoItem>教育</SearchInfoItem>
		<SearchInfoItem>教育</SearchInfoItem>
		<SearchInfoItem>教育</SearchInfoItem>
		<SearchInfoItem>教育</SearchInfoItem>
		<SearchInfoItem>教育</SearchInfoItem>
    </div>
</SearchInfo>
```

```javascript
export const SearchInfoItem=styled.a`
	display:block;
	float:left;
	line-height:20px;
	padding:0 5px;
	margin-right:10px;
	margin-bottom:15px;
	font-size:12px;
	border:1px solid #ddd;
	color:#333;
	border-radius:3px;
`;
```

#### 3.5 添加SearchInfoList

这时发现item越界了，这时由于在外层SearchInfo中把高度写死了，去掉下面的height就可以了。这时就可以了 。

这时我们将外层的div改为SearchInfoList，引入并创建该样式

```javascript
import { HeaderWrapper , Logo , Nav , NavItem , NavSearch, Addition,Button,SearchWrapper,SearchInfo,SearchInfoTitle, SearchInfoSwitch, SearchInfoItem, SearchInfoList} from './style.js';
。。。
<SearchInfo>
	<SearchInfoTitle>热门搜索
	<SearchInfoSwitch>换一批</SearchInfoSwitch>
	</SearchInfoTitle>
	<SearchInfoList>
		<SearchInfoItem>教育</SearchInfoItem>
		<SearchInfoItem>教育</SearchInfoItem>
		<SearchInfoItem>教育</SearchInfoItem>
		<SearchInfoItem>教育</SearchInfoItem>
		<SearchInfoItem>教育</SearchInfoItem>
		<SearchInfoItem>教育</SearchInfoItem>
    </SearchInfoList>
</SearchInfo>
```

```javascript
export const SearchInfoList=styled.div`
	overflow:hidden;
`;
```

#### 3.6 聚焦显示失焦隐藏

我们在`src/common/header/index.js`中声明一个方法，getListArea，接受一个参数，参数是真就返回列表，参数为假就不返回列表，然后在之前SearchInfo标签处添加该方法

```javascript
const getListArea=(show)=>{
    if(show){
        return (
            <SearchInfo>
                <SearchInfoTitle>热门搜索
                <SearchInfoSwitch>换一批</SearchInfoSwitch>
                </SearchInfoTitle>
                <SearchInfoList>
                    <SearchInfoItem>教育</SearchInfoItem>
                    <SearchInfoItem>教育</SearchInfoItem>
                    <SearchInfoItem>教育</SearchInfoItem>
                    <SearchInfoItem>教育</SearchInfoItem>
                    <SearchInfoItem>教育</SearchInfoItem>
                    <SearchInfoItem>教育</SearchInfoItem>
                </SearchInfoList>
            </SearchInfo>
        )
    }else{
        return null;
    }
}
。。。
				<SearchWrapper>
				    	<CSSTransition
							。。。
					    </CSSTransition>
					    <i className={props.focused ? 'focused iconfont':'iconfont'}>&#xe6dd;</i>
				    	{getListArea(props.focused)}
				    </SearchWrapper>
```

这样热门搜索框就做好了。

#### 3.7 再次换一批

header会越来约庞大，如果还用无状态组件的话会特别麻烦，需要将它换成一个常规的Component组件，继承自component，在render函数中直接返回就可以;

这里面也没有props这个参数了，需要将它们全部转为this.props；

同时需要将getListArea方法，添加到累内部，在调用的地方换为this.getListArea

```javascript
...
```

当我们第一次聚焦input框的时候，会发送ajax请求，获取热门搜索关键字，再次聚焦的时候不会发送请求，而是直接使用之前的关键字。

所以在header的reducer中不仅要保存focus的数据，还要保存热门关键字数组list，

```javascript
const defaultState=fromJS({
	focused:false，
	list:[]
});
```

而发送ajax请求，我们统一放在redux-thunk中去处理，首先安装redux-thunk

```javascript
npm install redux-thunk
```

它应该在创建store时被使用，我们是在`src/index.js`中创建的store，在里面引入并使用

```javascript
import { createStore, compose , applyMiddleware} from 'redux';
import thunk from 'redux-thunk';
import reducer from './reducer';


const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;
const store=createStore(reducer,composeEnhancers(
		applyMiddleware(thunk)
	));

export default store;
```

然后就可以在action中做异步操作了，来到`src/common/header/index.js`文件，在mapDispatchToProps方法中派发一个action，

```javascript
const mapDispatchToProps=(dispatch)=>{
	return {
		handleInputFocus(){
			dispatch(actionCreators.getList());
			dispatch(actionCreators.searchFocus());
		},
		handleInputBlur(){
			dispatch(actionCreators.searchBlur());
		}
	}
}
```

并且在actionCreators.js文件中添加一个函数，之前时返回一个对象，现在不返回对象了而是返回一个函数，在这个函数中发出一个一步请求，这就可以使用一个第三方模块axios了，首先安装

```javascript
npm install axios
```

```javascript
export const getList=()=>{
	return (dispatch)=>{
		axios.get('api/headerList.json').then((res)=>{//成功

		}).catch(()=>{//失败
			console.log("error");
		})
	}
}
```

这时很明显不会获取到数据，因为我们没有这个后台接口，这时我们可以利用create-react来模拟数据，首先在public目录下创建一个文件夹api，在里面创建一个问价headList.json，在里面随意添加几个文字，这时在浏览器中输入localhost:3000/api/headerList.json，就可以访问你写入的内容了。通过这个特性我们可以写一些假数据

在json文件中添加：

```javascript
{
	'success':ture,
	"data":["行距杯2018征文","区块链","小程序","vue","毕业","PHP","故事","flutter","理财","美食","投稿","手帐","书法","PPT","穿搭","打碗碗花","简书","姥姥的澎湖湾","设计","创业","交友","籽盐","教育","思维导图","疯哥哥","梅西","时间管理","golang","连载","自律","职场","考研","慢世人","悦欣","一纸vr","spring","eos","足球","程序员","林露含","彩铅","金融","木风杂谈","日更","成长","外婆是方言","docker"]
}
```

这时我们就能在请求成功之后派发action了，如下：

```javas
axios.get('api/headerList.json').then((res)=>{//成功
	const data=res.data;
	const action={
        type:'change_list',
        data:data.data
	}
	dispatch(action);
}).catch(()=>{//失败
    console.log("error");
})
```

由于请求本身就是在actionCreators.js文件中的，所以可以这么改：

```javascript
const changeList=(data)=>{
    type:constants.CHANGE_LIST,
    data
}
...
axios.get('api/headerList.json').then((res)=>{//成功
	const data=res.data;
	dispatch(changeList(data));
}).catch(()=>{//失败
    console.log("error");
})
```

接下来就可以去`src/common/header/store/reducer.js`中继续写代码了

```javascript
if(action.type===constants.CHANGE_LIST){
		return state.set('list',action.data);
	}
```

注意这么写肯定是有问题的，因为我们在defaultState是一个immutable对象，里面的list也是一个immutable对象，而action.data是一个普通的js数组，可定会出错的。解决方法很简单，只需要在actionCreators.js中将data变成一个immutable对象就可以了

```javascript
import {fromJS} from 'immutable';
const changeList=(data)=>({
    type: constants.CHANGE_LIST,
    data: fromJS(data)
})
```

然后就可以在`src/common/header/index.js`中取数据了

在mapStateToProps方法中补充list属性

```javascript
const mapStateToProps=(state)=>{
	return {
		focused:state.get('header').get('focused'),
		list:state.getIn(['header','list'])
	}
}
```

然后修改getListArea，首先将参数去掉直接在方法内部调用this.props.focused，然后将SearchInfoList内部写一个循环表达式

```javascript
getListArea(){
		if(this.props.focused){
	    	return (
		        <SearchInfo>
		            <SearchInfoTitle>热门搜索
		            <SearchInfoSwitch>换一批</SearchInfoSwitch>
		            </SearchInfoTitle>
		            <SearchInfoList>
                        {
                            this.props.list.map((item)=>{
                            	return <SearchInfoItem key={item}>{item}</SearchInfoItem>
                        	})
                        }
		            </SearchInfoList>
		        </SearchInfo>
	        )
	    }else{
	        return null;
	    }
	}

...
{ this.getListArea() }
```

#### 3.8 简化代码

（1）

这时我们的`src/common/header/store/actionCreators.js`中大部分内容需要暴漏出去，但是像changeList的方法不用暴漏出去。像这种不暴漏的方法建议要么都放在顶部要么都放在底部；

（2）

在`src/common/header/index.js`中，导出都用this.props.list或this.props.focused，其实可以精简一下，在getListArea方法中添加入戏内容，这样就无需每次都使用this.props.XXX了，就可以直接使用focused和list了

```javascript
getListArea(){
    const {focused,list}=this.props;
    ...
}
```

同理在render方法中我们可以使用，来简化

```javascript
render(){
    const {focused,handlerInputFocused,handlerInputBlur}
    ...
}
```

（3）

在`src/common/header/store/reducer.js`文件中，大量的使用了if语句，可以使用switch语句来做替换

#### 3.9 实现换一批功能

###### 3.9.1给提示框分页

我们热门索搜提示显示了所有的内容，而简书官网每次只显示十个内容。我们先在`src/common/header/store/reducer.js`文件中给defaultStatus添加page和totalPage两个属性，表示当前页码和总页数

```javascript
const defaultState=fromJS({
	focused:false,
	list:[],
	page:1,
	totalPage:1,
});
```

然后在`src/common/header/store/actionCreators.js`文件中给changeList方法，补充totalPage属性

```javascript
const changeList=(data)=>({
    type: constants.CHANGE_LIST,
    data: fromJS(data),
    totalPage:Math.ceil(data.length/10),
})
```

这样当取完列表项数据时，就可以知道总页数了；action会被派发给reducer，需要在`src/common/header/store/reducer.js`中对CHANGE_LIST方法不仅哟啊改变list，而且还要改变totalPage内容

```javascript
export default (state=defaultState,action)=>{
	switch(action.type){
		case constants.SEARCH_FOCUS:
			return state.set('focused',true);
		case constants.SEARCH_BLUR:
			return state.set('focused',false);
		case constants.CHANGE_LIST:
			return state.set('list',action.data).set("totalPage",action.totalPage);
		default:
			return state;
	}
}
```

然后在`src/common/header/index.js`拿到page页码

```javascript
const mapStateToProps=(state)=>{
	return {
		focused:state.get('header').get('focused'),
		list:state.getIn(['header','list']),
		page:state.getIn(['header','page']),
	}
}
```

在getListArea中使用page，来创建10个item，放置在相应位置。注意list由于是immutable对象不能直接使用下标，需要将它转化为js对象才能使用下标形式

```javascript
getListArea(){
		const {focused,list,page}=this.props;
		const newList=list.toJS();
		const pageList=[];
		for (let i = (page-1)*10; i <page*10; i++) {
			pageList.push(
				<SearchInfoItem key={newList[i]}>{newList[i]}</SearchInfoItem>
			);
		}
		if(focused){
	    	return (
		        <SearchInfo>
		            <SearchInfoTitle>热门搜索
		            	<SearchInfoSwitch>换一批</SearchInfoSwitch>
		            </SearchInfoTitle>
		            <SearchInfoList>
		                {
                            pageList
                        }
		            </SearchInfoList>
		        </SearchInfo>
	        )
	    }else{
	        return null;
	    }
	}
```

此时我们的热门索搜提示就展示10条内容了。

###### 3.9.1实现换一批换页

此时我们点击换一批，发现提示框直接消失了；简书官网则不是这样的，只有在鼠标移出搜索框时才隐藏。所有提示框不仅仅是依赖focused来隐藏的，我们需要额外监听一个属性，鼠标是否在提示框内mouseIn，首先给`src/common/header/store/reducer.js`添加mouseIn属性

```javascript
const defaultState=fromJS({
	focused:false,
	list:[],
	mouseIn:false,
	page:1,
	totalPage:1,
});
```

给constants添加MOUSE_IN和MOUSE_OUT常量

```javascript
export const MOUSE_IN="header/MOUSE_IN";
export const MOUSE_OUT="header/MOUSE_OUT";
```

给SearchInfo添加onMouseEnter和onMouseLeave;在mapStateToProps中引入mouseIn；在mapDispatchToProps中添加handlerMouseEnter,handlerMouseLeave操作；在actionCreators中创建mouseEnter和mouseLeave事件；在reducer中处理这两个事件

```javascript
const {focused,list,page,handlerMouseEnter,handlerMouseLeave}=this.props;
....
<SearchInfo
	onMouseEnter={handlerMouseEnter}
	onMouseLeave={handlerMouseLeave}
>
....
const mapStateToProps=(state)=>{
	return {
		focused:state.get('header').get('focused'),
		list:state.getIn(['header','list']),
		page:state.getIn(['header','page']),
		mouseIn:state.getIn(['header','mouseIn']);
	}
}
const mapDispatchToProps=(dispatch)=>{
	return {
		handleInputFocus(){
			dispatch(actionCreators.getList());
			dispatch(actionCreators.searchFocus());
		},
		handleInputBlur(){
			dispatch(actionCreators.searchBlur());
		},
		handlerMouseEnter(){
			dispatch(actionCreators.mouseEnter());
		},
		handlerMouseLeave(){
			dispatch(actionCreators.mouseLeave());
		}

	}
}

```

```javascript
...
export const mouseEnter=()=>({
    type:constants.MOUSE_IN
});

export const mouseLeave=()=>({
    type:constants.MOUSE_OUT
});
```

```javascript

export default (state=defaultState,action)=>{
	switch(action.type){
		case constants.SEARCH_FOCUS:
			return state.set('focused',true);
		case constants.SEARCH_BLUR:
			return state.set('focused',false);
		case constants.CHANGE_LIST:
			return state.set('list',action.data).set("totalPage",action.totalPage);
		case constants.MOUSE_IN:
			return state.set('mouseIn',true);
		case constants.MOUSE_OUT:
			return state.set('mouseIn',false);
		default:
			return state;
	}
}
```

然后就可以对getListArea方法，foucese的判断再添加一个mouseIn了

```javascript
if(focused || mouseIn){
    ...
}
```

接下来就可以实现点击事件了 给SearchInfoSwitch添加点击事件

```javascript
<SearchInfoSwitch onClick={()=>handlerChangePage(page,totalPage)}>换一批</SearchInfoSwitch>
```

其余略，同上

###### 3.9.3 给immutable对象修改多个属性

如果给immutable修改多个属性，连续使用set会和长，这时可以使用merge代替

```javascript
state.set('list',action.data).set("totalPage",action.totalPage);
//改为
state.merge({
    list:action.data,
    totalPage:action.totalPage,
});

```

#### 3.10 实现刷新旋转按钮

###### 3.10.1 添加iconfont

首先去[iconfont](www.iconfont.cn)将spin按钮Tina 驾到购物车，然后将jianshu项目下的iconfont重新下载到本地。将iconfont.eot、

iconfont.svg、iconfont.ttf、iconfont.woff文件拷贝到我们的`src/statics/iconfont/`下，然后打开我们的iconfont.js文件，将我们解压缩文件中的iconfont.css文件将里面的内容复制到iconfont.js中，将`@font-face`中的内容替换掉，其他的不用变。

这样我们项目中的iconfont就替换成了最新的，对老的功能不会有影响。

打开`src/common/header/index.js`，在SearchInfoSwitch标签内部添加一个i 标签，写入

```javascript
<SearchInfoSwitch onClick={()=>handlerChangePage(page,totalPage)}>
	<i className='iconfont'>&#xe851;</i>
	换一批
</SearchInfoSwitch>
```

此时看一些页面，发现改图标出现在了我们提示框的右下角，这时由于它跟上边的放大镜按钮的“靠右居下”的布局影响了，我们在`src/common/header/style.js`中可以看到SearchWrapper中所有的iconfont都是“绝对定位，居右居下”的样式修饰。我们可以把这个iconfont的样式换名为zoom，首先给放大镜的i标签中添加一个zoom属性

```javascript
<i className={focused ? 'focused iconfont zoom':'iconfont zoom'}>&#xe6dd;</i>
```

然后将SearchWrapper中所有的iconfont换成zoom

```javascript
export const SearchWrapper=styled.div`
	float:left;
	position:relative;
	.zoom {
		position:absolute;
		right:5px;
		bottom:5px;
		width:30px;
		line-height:30px;
		border-radius:15px;
		text-align:center;
		&.focused{
			background:#777;
			color: #fff;
        }
	}
`;
```

这时在看我们的刷新图标，就位于“换一批”左边了。然后我们就可以给我们的刷新图标添加一个spin的样式，在SearchInfoSwitch样式中书写改样式

```javascript
<i className='iconfont spin'>&#xe851;</i>
```

```javascript
export const SearchInfoSwitch=styled.div`
	float: right;
    font-size: 13px;
    color: #969696;
    background-color: transparent;
    border-width: 0;
    padding: 0;
    .spin {
    	font-size:12px;
    	margin-right:2px;
    }
`;
```

###### 3.10.2 点击换一换实现图标旋转

我们给spin样式一个transition动画，设置动画时间200ms，动画类型是ease-in，设置旋转角度，和旋转中心

```javascript
export const SearchInfoSwitch=styled.div`
	float: right;
    font-size: 13px;
    color: #969696;
    background-color: transparent;
    border-width: 0;
    padding: 0;
    .spin {
    	display:block;
    	float:left;
    	font-size:12px;
    	margin-right:2px;
    	transition:all .2s ease-in;
    	transform:rotate(0deg);
    	transform-origin:center,center;
    }
`;
```

所以当换一批被点击时，只需要让i 标签rotate的值发生变化，加360度就可以了。ref属性可以获取i 标签的真实节点给它放置一个函数;当我们的SearchInfoSwitch被点击时，会触动handlerChangePage方法，我们可以把this.spinIcon也传进去，然后就可以在handlerChangePage方法获取spin对应的dom了

```javascript
<SearchInfoSwitch onClick={()=>handlerChangePage(page,totalPage, this.spinIcon)}>
	<i ref={(icon)=>{this.spinIcon=icon}} className='iconfont spin'>&#xe851;</i>
	换一批
</SearchInfoSwitch>
。。。
const mapDispatchToProps=(dispatch)=>{
	return {
		...
		handlerChangePage(page,totalPage,spin){
			spin.style.transform="rotate(360deg)";
			if(page<totalPage){
				dispatch(actionCreators.changePage(page+1));
			}else{
				dispatch(actionCreators.changePage(1));
			}
		},
	}
}
```

这时点击换一批就有旋转效果了，不过当再次点击时就不再旋转了。有无rotate一直时360不再发生变化了。所以我们需要先看它之前的角度，然后在原油角度的基础上增加360

```javascript
handlerChangePage(page,totalPage,spin){
			let originAngle=spin.style.transform.replace(/[^0-9]/ig,'');
			if(originAngle){
				originAngle=parseInt(originAngle,10);
			}else{
				originAngle=0;
			}
			spin.style.transform="rotate(" + (originAngle+360) + "deg)";
			if(page<totalPage){
				dispatch(actionCreators.changePage(page+1));
			}else{
				dispatch(actionCreators.changePage(1));
			}
		},
```

### 四、规避不必要的ajax请求

之前我们的代码，会在每次给搜索框聚焦时发送ajax请求，其实改请求获取一次就足够了。我门可以给handleInputFocus方法传递一个list的参数，当list大小为0时才发送请求

```javascript
render(){
		const {focused,handleInputFocus,handleInputBlur,list}=this.props;
		return (
            ...
			onFocus={()=>handleInputFocus(list)}	    	
		)
}
。。。
const mapDispatchToProps=(dispatch)=>{
	return {
		handleInputFocus(list){
			(list.size===0) && dispatch(actionCreators.getList());
			dispatch(actionCreators.searchFocus());
		},
		。。。。
    }
}
```

现在就解决问题了。

接下来我们解决一个小问题，当鼠标放在SearchInfoSwitch时变成小手的样式，给SearchInfoSwitch的样式设置一个样式cursor的样式等于pointer就可以了。


