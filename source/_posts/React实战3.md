---
title: React实战3
date: 2018-11-05 22:36:42
tags: React
---
## 一、路由

#### 1.1 路由和路由规则

React中路由表示根据url的不同显示不同的内容

在安装react中安装路由

```javascript
npm install react-router-dom
```

进入APP.js，看到大的组件中只显示header，我们希望访问首页时不仅显示header，而且还显示首页。访问详情页时把详情页给显示出来。

首先从react-router-dom中引入BrowserHistory、Router。注意Proveder中只能有一个元素，所以它里面的内容需要用一个div包起来，对BrowserRouter也是只能包含一个要素。

```javascript
import React, { Component } from 'react';
import { Provider } from 'react-redux';
import { BrowserRouter, Route } from 'react-router-dom';
import Header from './common/header/index.js';
import store from './store/index.js';

class App extends Component {
  render() {
    return (
    	<Provider store={store}>
    		<div>
	      		<Header />
	      		<BrowserRouter>
	      			<div>
		      			<Route path='/' render={()=><div>home</div>}></Route>
		      			<Route path='/detail' render={()=><div>detail</div>}></Route>
	      			</div>
	      		</BrowserRouter>
      		</div>
      	</Provider>

    );
  }
}

export default App;

```

这时当我们访问http://localhost:3000和http://localhost:3000/detail时回出现home和homedetail。这时因为当访问http://localhost:3000时/也能匹配上/detail。如果我们访问http://localhost:3000/detail只想看到detail不想看到home怎么办？很简单，只需要加一个exact就可以了。

表示只有路径和指定的路径完完全全相等的时候才显示后面的内容

```javascript
<Route path='/' exact render={()=><div>home</div>}></Route>
<Route path='/detail' exact render={()=><div>detail</div>}></Route>
```

#### 1.2 拆分首页

首先在src目录下创建一个pages的文件夹，表示有多少个页面，然后在pages里面创建两个文件夹分别叫做home和detail。

在home文件夹下创建一个index.js的文件，这时react组件

```javascript
import React, { Component } from 'react';

class Home extends Component{
	render(){
		return (
			<div>Home</div>
		);
	}
export default Home;
```

同理在detail文件夹下也创建一个index.js的文件。



然后在App.js文件中引入这两个文件，并在Route标签中引入这两个组件。在标签中引入组件使用compinent

```javascript
import Home from './pages/home';
import Detail from './pages/detail';
...
<Route path='/' exact component={Home}></Route>
<Route path='/detail' exact component={Detail}></Route>
```

然后在home目录下创建一个style.js的文件，

```javascript
import styled from 'styled-components';

export const HomeWrapper=styled.div`
	width:960px;
	margin:0, auto;
	height:300px;
	background:red;
`;
```

在`home/index.js`中引入HomeWrapper组件即可

```javascript
...
import { HomeWrapper } from './style.js';
class Home extends Component{
	render(){
		return (
			<HomeWrapper></HomeWrapper>
		);
	}
}
```

在style.js中创建HomeLeft和HomeRight，并在home/index.js中引用

```javascript
import styled from 'styled-components';

export const HomeWrapper=styled.div`
	overflow:hidden;
	width:960px;
	margin:0, auto;
`;

export const HomeLeft=styled.div`
	float:left;
	margin-left:15px;
	padding-top:30px;
	width:625px;
`;

export const HomeRight=styled.div`
	width:240px;
	float:right;
`;
```

```javascript
import React, { Component } from 'react';
import { HomeWrapper , HomeLeft, HomeRight} from './style.js';
class Home extends Component{
	render(){
		return (
			<HomeWrapper>
				<HomeLeft>left</HomeLeft>
				<HomeRight>right</HomeRight>
			</HomeWrapper>
		);
	}

}
export default Home;
```

如何在left中引入banner图呢？可以在HomeLeft中引入img标签并且在`home/style.js`文件中添加样式，这样图片就正常显示了

```javascript
<HomeLeft>
	<img className='banner-img' src="//upload.jianshu.io/admin_banners/web_images/4516/cd9298634ca88ca71fc12752acf47917967a5d31.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/1250/h/540" />
</HomeLeft>
```

```javascript
export const HomeLeft=styled.div`
	float:left;
	margin-left:15px;
	padding-top:30px;
	width:625px;
	.banner-img {
		width:625px;
		height:270px;
	}
`;
```

#### 1.3 划分首页组件

可以把首页内容划分成几个组件

在home文件夹下创建一个名为components的文件夹，在里面创建Recommend.js、List.js、Writer.js、Topic.js的文件。

###### 1.3.1 Topic的编程

由于我们首页的几个组件过于简单，如果给每一个小组件都设计一个style就显得过于设计。所以统一给它们上一层目录的style

进入到Topic.js文件，使用TopicWrapper标签，并且在`home/style.js`文件中创建该样式

```javascript
import React, { Component } from 'react';
import { TopicWrapper , TopicItem} from '../style';

class Topic extends Component{
	render(){
		return (
			<TopicWrapper>
				<TopicItem>
					<img className='topic-pic' src="//upload.jianshu.io/collections/images/283250/%E6%BC%AB%E7%94%BB%E4%B8%93%E9%A2%98.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/64/h/64" />
					手绘手绘
				</TopicItem>
			</TopicWrapper>
		);
	}
}

export default Topic;
```

```javascript
export const TopicWrapper=styled.div`
	overflow:hidden;
	padding: 20px 0 10px 0;
	margin-left:-18px;
`;

export const TopicItem=styled.div`
	float:left;
	height:32px;
	line-height:32px;
	padding-right:10px;
	margin-bottom:18px;
	margin-left:18px;
	background:#f7f7f7;
	font-size:14px;
	color:#000;
	border:1px solid #dcdcdc;
	border-radius:4px;
	.topic-pic {
		display:block;
		margin-right:10px;
		float:left;
		width:32px;
		height:32px;
	}
`;
```



###### 1.3.2 创建reducer

在`src/store/reducer`中大的reducer将若干个小的reducer组合一起。所以home页面应该由自己的reducer管理自己的数据。

在home目录下创建一个store的文件夹，并且在里面创建reducer.js文件，参照header下的reducer

```javascript
import { fromJS } from 'immutable'

const defaultState=fromJS({
	
});

export default (state=defaultState,action)=>{
	switch(action.type){

		default:
			return state;
	}
}
```

我们的topicItem肯定是一个数组来管理的，所以reducer中就需要定义一个数组

```javascript
import { fromJS } from 'immutable'

const defaultState=fromJS({
	topicList:[
		{id:1,title:'手绘',imgUrl:'//upload.jianshu.io/collections/images/283250/%E6%BC%AB%E7%94%BB%E4%B8%93%E9%A2%98.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/64/h/64'},
		{id:2,title:'读书',imgUrl:'//upload.jianshu.io/collections/images/4/sy_20091020135145113016.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/64/h/64'},
		{id:3,title:'自然科普',imgUrl:'//upload.jianshu.io/collections/images/76/12.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/64/h/64'},
		{id:4,title:'故事',imgUrl:'//upload.jianshu.io/collections/images/95/1.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/64/h/64'},
		{id:5,title:'简书电影',imgUrl:'//upload.jianshu.io/collections/images/21/20120316041115481.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/64/h/64'},
		{id:6,title:'摄影',imgUrl:'//upload.jianshu.io/collections/images/83/1.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/64/h/64'},
		{id:7,title:'旅行 在路上',imgUrl:'//upload.jianshu.io/collections/images/13/%E5%95%8A.png?imageMogr2/auto-orient/strip|imageView2/1/w/64/h/64'},
		
	]
});

export default (state=defaultState,action)=>{
	switch(action.type){

		default:
			return state;
	}
}
```

并且在`src/store/reducer`中引入该ruducer,同理我们不想让外部直接饮用该reducer，需要在pages/home/store下创建index.js，来引入和导出该reducer

```javascript
import {combineReducers} from 'redux-immutable';
import { reducer as headerReducer } from '../common/header/store';
import { reducer as homeReducer } from '../pages/home/store';

export default combineReducers({
	header:headerReducer,
	home:homeReducer
})
```

```javascript
//pages/home/store下创建index.js
import reducer from './reducer';

export { reducer };
```

###### 1.3.3 连接Topic和reducer

这时候就需要让Topic和store做连接了，怎么连接?在Topic.js中引入react-redux的connect即可

```javascript
import React, { Component } from 'react';
import { connect } from 'react-redux';
import { TopicWrapper , TopicItem} from '../style';

class Topic extends Component{
	render(){
		return (
			<TopicWrapper>
				{
					this.props.list.map((item)=>{
						return (
							<TopicItem key={item.get('id')}>
								<img 
									className='topic-pic'
									src={item.get('imgUrl')} />
									{item.get('title')}
							</TopicItem>
						);
					})
				}

			</TopicWrapper>
		);
	}
}

const mapStateToProps=(state)=>{
	return {
		list:state.get('home').get('topicList')
	}
}

export default connect(mapStateToProps,null)(Topic);
```

###### 1.3.4 文章列表

和Topic没什么区别，只是再走一遍流程，不再重复

```javascript
import React, { Component } from 'react';
import { connect } from 'react-redux';
import { ListItem , ListInfo} from '../style';

class List extends Component{
	render(){
		return (
			<div>
				{
					this.props.list.map((item)=>{
						return (
							<ListItem key={item.get('id')}>
								<img className='list-pic' src={item.get('imgUrl')} />
								<ListInfo>
									<h3 className='title'>{item.get('title')}</h3>
									<p className='desc'>{item.get('desc')}</p>
								</ListInfo>
							</ListItem>
						)
					})
				}
			</div>
		);
	}
}

const mapStateToProps=(state)=>{
	return {
		list:state.get('home').get('articleList')
	}
}

export default connect(mapStateToProps,null)(List);
```

```javascript
const defaultState=fromJS({
	...
	articleList:[
		{id:1,title:'全城百姓举牌欢迎，不料被将军看到牌后一字，悄悄对手下说：屠城',desc:'文/历史大农 简短一句话，剖析一段史。让“历史大农”用简洁的语言，为您讲述一段历史辛密。 军事家、战略家——“明太祖”朱元璋 明太祖朱元璋，作为...',imgUrl:'//upload-images.jianshu.io/upload_images/11919269-0a22c403002a1371?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240'},
		{id:2,title:'每天最重要的2小时：让自己成为高效的人',desc:'工作、加班、学习、运动、陪伴家人，当然还有娱乐，我们每天都被时间的脚步追赶着，每一刻都是忙碌的，而忙碌永远没有劲头。 当下，时间俨然已经成为最宝...',imgUrl:'//upload-images.jianshu.io/upload_images/10746568-26f4733a3ad87c62.png?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240'},
		{id:3,title:'是可忍，孰不可忍！｜140字微小说',desc:'多少个夜晚 你真心相伴 不离不弃 多少个夜晚 你亲昵纠缠 不眠不休 你已经在我心底留下了深深的印迹 我想你 魂牵梦萦 如今 我终于将你捧在了手心...',imgUrl:'//upload-images.jianshu.io/upload_images/12309992-4e21d79f92914f76.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240'},
		{id:4,title:'还在纠结秋冬穿什么？8套秋冬搭配轻松搞定',desc:'第一套： 杏色的针织荷叶边的垂感中长裙搭配同色系的折线宽松毛衣，视觉上看着很舒服，比较优雅的颜色搭配，适合皮肤中性偏白的女生，很显气质，同时竖条...',imgUrl:'//upload-images.jianshu.io/upload_images/12861608-90ae71664fbb9d99?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240'},

	]
});
```

可是这时候控制台会报一些警告，这时由于使用img标签都要填一个alt属性，只需要给img添加一个alt标签即可。

```javascript
alt=''
```

###### 1.3.5 组件向样式传递属性

有时候我们希望在组件中定义一个属性，然后在style中使用，怎么办呢？

我们可以在组件RecommendItem中定义一个imgUrl的属性，然后在style中通过props来传递

```javascript
//Recommend.js
<RecommendItem imgUrl="http://cdn2.jianshu.io/assets/web/banner-s-3-7123fd94750759acf7eca05b871e9d17.png">
</RecommendItem>
```

```javascript
//style.js
//这时原始的
export const RecommendItem=styled.div`
	width:280px;
	height:50px;
	margin-bottom:6px;
	background:url(http://cdn2.jianshu.io/assets/web/banner-s-3-7123fd94750759acf7eca05b871e9d17.png);
	background-size:contain;
`;

//这时最新的
export const RecommendItem=styled.div`
	width:280px;
	height:50px;
	margin-bottom:6px;
	background:url(${(props)=>props.imgUrl});
	background-size:contain;
`;
```

###### 1.3.6 通过ajax获取数据

实际上我们hone页面的数据是从后台获取的，所以不能自reducer中把数据写死。

首先我们在`public/api/`下建立一个home.json文件，并写入home的数据

```javascript
{
	"success": true,
	"data": {
		"topicList": [{
			"id": 1,
			"title": "社会热点",
			"imgUrl": "//upload.jianshu.io/collections/images/261938/man-hands-reading-boy-large.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/64/h/64"
		}, {
			"id": 2,
			"title": "手手绘",
			"imgUrl": "//upload.jianshu.io/collections/images/21/20120316041115481.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/64/h/64"
		}],
		"articleList": [{
			"id": 1,
			"title": "胡歌12年后首谈车祸",
			"desc": "文/麦大人 01 胡歌又刷屏了。 近日他上了《朗读者》，而这一期的主题是“生命”，他用磁性的嗓音，朗读了一段《哈姆雷特》中的经典独白，相当震撼：...",
			"imgUrl": "//upload-images.jianshu.io/upload_images/2259045-2986b9be86b01f63?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240"
		}, {
			"id": 2,
			"title": "胡歌12年后首谈车祸：既然活下来了，就不能白白活着",
			"desc": "文/麦大人 01 胡歌又刷屏了。 近日他上了《朗读者》，而这一期的主题是“生命”，他用磁性的嗓音，朗读了一段《哈姆雷特》中的经典独白，相当震撼：...",
			"imgUrl": "//upload-images.jianshu.io/upload_images/2259045-2986b9be86b01f63?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240"
		}, {
			"id": 3,
			"title": "胡歌12年后首谈车祸：既然活下来了，就不能白白活着",
			"desc": "文/麦大人 01 胡歌又刷屏了。 近日他上了《朗读者》，而这一期的主题是“生命”，他用磁性的嗓音，朗读了一段《哈姆雷特》中的经典独白，相当震撼：...",
			"imgUrl": "//upload-images.jianshu.io/upload_images/2259045-2986b9be86b01f63?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240"
		}, {
			"id": 4,
			"title": "胡歌12年后首谈车祸：既然活下来了，就不能白白活着",
			"desc": "文/麦大人 01 胡歌又刷屏了。 近日他上了《朗读者》，而这一期的主题是“生命”，他用磁性的嗓音，朗读了一段《哈姆雷特》中的经典独白，相当震撼：...",
			"imgUrl": "//upload-images.jianshu.io/upload_images/2259045-2986b9be86b01f63?imageMogr2/auto-orient/strip|imageView2/1/w/300/h/240"
		}],
		"recommendList": [{
			"id": 1,
			"imgUrl": "http://cdn2.jianshu.io/assets/web/banner-s-3-7123fd94750759acf7eca05b871e9d17.png"
		}, {
			"id": 2,
			"imgUrl": "http://cdn2.jianshu.io/assets/web/banner-s-5-4ba25cf5041931a0ed2062828b4064cb.png"
		}]
	}
}
```

然后将home/store/reducer中的defaultState各个数据设置为空

```javascript
const defaultState=fromJS({
	topicList:[],
	articleList:[],
	recommendList:[]
});
```

然后在home/index.js中添加生命周期函数componentDidMound方法中发送ajax请求。表示当页面挂载完毕时发送请求；将获取的数据传给store，通过actioin来完成；这时候需要dispatch来发送，可是如何将home/index.js于store建立连接呢，使用connect

```javascript
mport axios from 'axios';
...
componentDidMount(){
		axios.get('api/home.json').then((res)=>{
			const result=res.data.data;
			const action={
				type:'change_home_data',
				topicList:result.topicList,
				articleList:result.articleList,
				recommendList:result.recommendList,
			}
			this.props.changeHomeData(action);
		})
	}
...
const mapDispatchToProps=((dispatch)=>{
	return {
		changeHomeData(action){
			dispatch(action);
		}
	}
})
export default connect(null,mapDispatchToProps)(Home);
```

这样就可以在/page/home/store/reducer.js中处理该action 了

```javascript
export default (state=defaultState,action)=>{
	switch(action.type){
		case 'change_home_data':
			return state.merge({
				topicList:fromJS(action.topicList),
				articleList:fromJS(action.articleList),
				recommendList:fromJS(action.recommendList)
			});
		default:
			return state;
	}
}
```

###### 1.3.7 异步代码的拆分

home/index.js应该是一个UI组件，它不应该有过多的逻辑处理，而componentDidMount中ui组件发ajax请求然后更新store，更新UI。这样不合适，可以将ajax请求直接放在changeHomeData,直接对返回结果进行派发，这样就将ui组件中逻辑剔除了

```javascript
componentDidMount(){
		this.props.changeHomeData();
	}
。。。
const mapDispatchToProps=((dispatch)=>{
	return {
		changeHomeData(action){
			axios.get('api/home.json').then((res)=>{
				const result=res.data.data;
				const action={
					type:'change_home_data',
					topicList:result.topicList,
					articleList:result.articleList,
					recommendList:result.recommendList,
				}
				dispatch(action);
			})
			
		}
	}
})
```

其实异步操作，这么放也不合理。我们可以使用redux-thunk，将它放在action中去管理

我们在home/store目录下创建一个actionCreators.js 的文件,并且在store/index.js中引入并输入，然后在home/index.js中修改

```javascript
//actionCreator.js
import axios from 'axios';

export const getHomeInfo=()=>{
	return (dispatch)=>{
		axios.get('api/home.json').then((res)=>{
			const result=res.data.data;
			const action={
				type:'change_home_data',
				topicList:result.topicList,
				articleList:result.articleList,
				recommendList:result.recommendList,
			}
			dispatch(action);
		})
	}
}
```

```javascript
//home/store/index.js
import reducer from './reducer';
import * as actionCreators from './actionCreators.js';

export { reducer , actionCreators};
```

```javascript
//home/index.js
const mapDispatchToProps=((dispatch)=>{
	return {
		changeHomeData(){
			const action=actionCreators.getHomeInfo();
			dispatch(action);
		}
	}
})
```

这样我们就把home/index.js中的逻辑移动到actionCreators中去了，这里的getHomeInfo还可以进一步优化,将action提出来

```javascript
const changeHomeData=(result)=>({
	type:'change_home_data',
	topicList:result.topicList,
	articleList:result.articleList,
	recommendList:result.recommendList,
})

export const getHomeInfo=()=>{
	return (dispatch)=>{
		axios.get('api/home.json').then((res)=>{
			const result=res.data.data;
			dispatch(changeHomeData(result));
		})
	}
}
```

同样可以创建一个constants.js文件，保存type常量

```javascript
export const CHANGE_HOME_DATA='home/CHANGE_HOME_DATA';
```

###### 1.3.8 创建阅读更多

当将列表滑动到最下面时，会出现阅读更多的按钮，点击该按钮会在列表中加载更多内容。它既可以放在列表之内，也可以放在列表之外。我们将它放在列表之内

我们在List组件中添加一个LoadMore的组件，并在style中创建该样式

```javascript
render(){
		return (
			<div>
				{
					this.props.list.map((item)=>{
						return (
							<ListItem key={item.get('id')}>
								<img className='list-pic' src={item.get('imgUrl')} alt=''/>
								<ListInfo>
									<h3 className='title'>{item.get('title')}</h3>
									<p className='desc'>{item.get('desc')}</p>
								</ListInfo>
							</ListItem>
						)
					})
				}
				<LoadMore>更多内容</LoadMore>
			</div>
		);
	}
```

```java
//style
export const LoadMore=styled.div`
    width: 100%;
    height: 40px;
    line-height:40px;
    margin: 30px 0;
    text-align: center;
    font-size: 15px;
    border-radius: 20px;
    color: #fff;
    background-color: #a5a5a5;
    display: block;
    cursor:pointer;
`;
```

然后给LoadMore添加点击事件,这时只需要创建mapDispatchToProps即可，创建action等不再赘述

```javascript
const mapDispatchToProps=(dispatch)=>{
	return {
		getMoreList(){
			dispatch(actionCreators.getMoreList());
		}
	}
}

export default connect(mapStateToProps,mapDispatchToProps)(List);
```

真实情况其实需需要翻页，只需要给reducer中添加一个articlePage的字段即可，每次getMoreList时，将该字段加1即可，不再详述。

#### 1.4 编写返回顶部

这代码量很小，单独写一个组件不合适。可以将它写在home的index.js中.

只需要在样式中定义该组件的样式；

然后添加点击事件，这里的点击事件和reducer关联不大，只是返回顶部

```javascript
handlerScrollTop(){
		window.scrollTo(0,0);
	}
...
<BackTop onClick={this.handlerScrollTop}>回到顶部</BackTop>
```

而官网的效果时，当页面往下拖的时候才会出现，而首屏时是不会出现的，这时需要通过变量来保存了。在reducer中添加

```javascript
const defaultState=fromJS({
	topicList:[],
	articleList:[],
	recommendList:[],
	articlePage:1,
	showScroll:false
});
```

然后打开home下index.js，在里面拿showScroll数据，注意当组件移除时需要将该监听移出

```javascript
{this.props.showScroll?
					<BackTop onClick={this.handlerScrollTop}>回到顶部</BackTop>:
				null}
...
	componentDidMount(){
		this.props.changeHomeData();

		this.bindEvents();
	}
	componentWillUnmount(){
		this.removeEvents();
	}

	bindEvents(){
		window.addEventListener("scroll",this.props.changeScrollShow);
	}
	removeEvents(){
		window.removeEventListener("scroll",this.props.changeScrollShow);
	}
```

这个时候，它不显示了。

这时就需要监听滚动方法了

```javascript
componentDidMount(){
	this.props.changeHomeData();
	this.bindEvents();
}

bindEvents(){
	window.addEventListener("scroll",this.props.changeScrollShow);
}
...
const mapDispatchToProps=((dispatch)=>{
	return {
		changeHomeData(){
			const action=actionCreators.getHomeInfo();
			dispatch(action);
		},
		changeScrollShow(e){
			if(document.documentElement.scrollTop>100){
				dispatch(actionCreators.toggleTopShow(true));
			}else{
				dispatch(actionCreators.toggleTopShow(false));
			}
		}
	}
})
```

然后就可以通过reducer来操作了.

此时reducer中switch代码好像有点多，可以将每个case中返回的内容提取出来

```javascript
const changeHomeData=(state,action)=>{
	return state.merge({
		topicList:fromJS(action.topicList),
		articleList:fromJS(action.articleList),
		recommendList:fromJS(action.recommendList)
	});
}
const addArticleList=(state,action)=>{
	return state.merge({
		'articleList':state.get('articleList').concat(action.list),
		'articlePage':action.nextPage
	})
}
export default (state=defaultState,action)=>{
	switch(action.type){
		case constants.CHANGE_HOME_DATA:
			return changeHomeData(state,action);
		case constants.ADD_ARTICLE_LIST:
			return addArticleList(state,action);
		case constants.TOGGLE_SCROLL_SHOW:
			return state.set('showScroll',action.show)
		default:
			return state;
	}
}
```

#### 1.5 路由跳转

由于页面很多时候并且不是所有的操作都要出发render。这时可以使用shouldComponentUpdate，来闭关频繁触发render，但是如果所有的子组件中都重写这个方法就太麻烦了，react内置了一个新的组件类型，叫做PureComponent，这个组件内部实现了shouldComponentUpdate，就不需要我们重复重写这个方法了。我们只要对List、Topic、Writer、Recommend组件都即成PureComponent就可以了。

**注意：**这里之所以可以使用PureComponent，是因为这个项目中我们使用了框架immutable.js，PureComponent和immutable数据才能无缝对接，如果没使用immutable可能就会出现问题。所以当使用PureComponent时建议使用immutable来管理数据



接下来来实现首页跳转到详情页

可以通过添加a标签，来实现页面的跳转，这时每跳转一次都会加载一次html，这就比较耗性能。这里我们要使用react-router，因为它可以实现单页面跳转，但也面跳转是指在页面跳转过程中只会加载一次html。可以使用react-router-dom中的Link来代替a标签

```javascript
import { Link } from 'react-router-dom';
<a key={index} href='/detail'>
	<ListItem>
	</ListItem>
</a>
--------
<Link key={index} to='/detail'>
	<ListItem>
	</ListItem>
</Link>
```

同理我们对头部组件的logo组件也要使用Link,注意使用Link的组件必须在Router的内部，所以我们需要在App.js中，将Header组件放在Router的内部

```javascript
<Link to='/'>
	<Logo>
</Link>
```

```javascript
class App extends Component {
  render() {
    return (
    	<Provider store={store}>
      		<BrowserRouter>
	      			<div>
                <Header />
		      			<Route path='/' exact component={Home}></Route>
		      			<Route path='/detail' exact component={Detail}></Route>
	      			</div>
	      		</BrowserRouter>
      	</Provider>

    );
  }
}

```

