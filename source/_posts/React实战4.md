---
title: React实战4
date: 2018-11-05 22:38:00
tags: React
---
### 详情页面的布局

1、首先在detail下面创建布局组件style.js

2、然后在detail/index.js中引入上面的布局组件

3、在style.js中编写详情页的标题组件、内容组件，在内容组件中放置img和p等标签；

4、给详情页面添加redux

> * 在detail文件夹下创建store文件夹，并在里面创建actionCreators.js、constants.js、index.js、reducer.js
>
> * 在reducer.js中添加defaultState	,需要将带html标签的内容全部包进去
>
> * 在src/store/reducer.js中，引入详情页的reducer
>
> * 在detail的index.js中引入reducer，并展示内容，注意当需要展示带有html标签内容时需要使用`dangerouslySetInnerHTML`
>
>   ```<DetailContent dangerouslySetInnerHTML={{__html:this.props.content}}>
>   //在DetailContent组件中展示html格式的内容
>   <DetailContent dangerouslySetInnerHTML={{__html:this.props.content}}>
>   </DetailContent>
>   ```
>
> * 用活数据代替死数据，componentDidMount触发请求行为，在actionCreators中引入axios发送请求
>
> * 通过路由，点击不通的item，实现不同详情页的跳转
>
>   > 两种方式：
>   >
>   > * 动态路由
>   >
>   >   在home的List页面中，将`<Link key={index} to='/detail'>`改为`<Link key={index} to={'/detail/'+item.get('id')}>`
>   >
>   >   进入APP.js，为了匹配所有的detail，将`<Route path='/detail' exact component={Detail}></Route>`改为`<Route path='/detail/:id' exact component={Detail}></Route>`
>   >
>   >   在detail的index.js页面获取页面ID `this.props.match.params.id`,传递给getDetail方法





### React中的异步组件

之前我们的代码，在控制台可以看到无论我们查看哪个页面，访问的js都没有改变。说明所有的js代码全部在一个文件里面，这样会影响速度，我们希望当我们家在首页时只加载首页代码，当我们访问详情页时只加载详情页的代码。

这就需要使用异步组件，就会非常简单。可以使用react-loadable组件

安装

```javascript
yarn add react-loadable
//或
npm install react-loadable
```

我们想要家在详情页时，再家在详情页的代码，应该怎么做呢？在detail文件夹下创建loadable.js的文件

```javascript
import React from 'react';
import Loadable from 'react-loadable';

const LoadableComponent = Loadable({
  loader: () => import('./'),
  loading(){
  	return <div>正在加载</div>
  },
});

export default ()=> <LoadableComponent/>
```

然后来到App.js中，将饮用Detail的引用改为loadable

```javascript
import Detail from './pages/detail';

//改为
import Detail from './pages/detail/loadable.js';
```

这时，会报错TypeError: Cannot read property 'params' of undefined

这是由于原先我们是直接使用Detail组件，现在变成了使用loadable.js了，而router是跟Detail对应的，现在对不上了，怎么办呢？

在detail的index.js文件中从react-router-dom引入withRouter，然后在使用connect中加入withRouter

```javascript
import {withRouter} from 'react-router-dom';
。。。
export default connect(mapStateToProps,mapDispatchToProps)(withRouter(Detail));
```

它就是让detail有能力获取到router中的内容。

这时就实现了对详情页的异步加载



### 项目上线

将项目打包

```javascript
npm run build
```

就会生成项目代码，放在后端即可
