---
title: React高阶组件笔记
date: 2020-04-02 23:25:34
tags: React
---
### 一、高阶组件的概念

* 高阶组件就是接收一个组件作为参数并返回一个数组
* 高阶组件是一个函数，并不是组件

### 二、为什么需要高阶组件

* 多个组件都需要某个相同的功能，使用高阶组件减少重复实现

### 三、高阶组件示例

* react-redux中的connnect

```javascript
export default connect(mapStateToProps,mapDispatchToProps)(Header);
```

### 四、编写高阶组件

* 实现一个普通组件
* 将普通组件使用函数包裹

```javascript
import React ,{Component} from 'react';
//接收一个入参，参数就是一个组件
function d(WrappedComponent){
    return class D extends Component{
        render(){
            return (
            	<div>
                	<WrappedComponent />
                </div>
            );
        }
    }
}
export default d;
```

### 五、使用高阶组件

* higherOrderComponent(WrappedComponent);

* @higherOrderComponent. 装饰器模式

  > 1、运行命令：react-scripts reject  中途选yes
  >
  > 2、安装依赖：npm install -D babel-preset-stage-2
  >
  > ​			npm install -D babel-preset-react-native-stage-0
  >
  > 3、添加.babelrc的文件
  >
  > ​	{
  >
  > ​		"presets":["react-native-stage-0/decorator-support"]
  >
  > ​	}
  >
  > 此时我们的被装饰者可以这么写
  >
  > ```javascript
  > import React ,{Component} from 'react';
  > import d from './D';
  > 
  > @d
  > class B extends Component{
  >     render(){
  >         return (
  >         	<div>
  >             	//...
  >             </div>
  >         );
  >     }
  > }
  > ```
  >
  >

### 六、高阶组件的应用

* 代理方式的高阶组件

  > 返回的新组件类直接继承自React.Component类，新组件扮演的角色传入参数组件的一个代理，在新组件的render函数中，将被包裹组件渲染出来，除了高阶组件自己要做的工作，其余功能全都转手给了被包裹的组件

* 继承方式的高阶组件

  > 采用继承关联作为参数的组件和返回组件，加入传入的组件参数是WrappedComponent，那么返回的组件就直接继承自WrappedComponent

* 高阶组件显示名

  > 在使用高阶组件时，有时候不好调试，并不知道最终渲染的是哪一个子组件，这时候就需要使用displayName了，可组件内部可以定义该静态变量
  >
  > ```javascript
  > static displayName=`newComponent${getDisplayName(WrappedComponent)}`;//getDisplayName()是在外部自定义的
  > ```
  >
  >

#### 6.1 代理方式的高阶组件

* 操纵prop.   	可以增加、删除属性
* 抽取状态 
* 访问ref
* 包装组件

### 七、实战

#### 7.1 初始化工程

```javascript
create-react-app tabbar
//若未安装create-react-app，可以使用
npx create-react-app tabbar
```

在`src`文件夹下新建`components`的文件夹，用来放置组件类；

在`components`的文件夹下，新建`tabbar`的文件夹；

然后在`tabbar`文件夹下新建`index.js`的文件，作为开发的主文件，同时在`tabbar`文件夹下新建一个样式文件`index.css`；

并且在App.js中引入Tarbar组件

```javascript
//index.js
import React ,{Component} from 'react';
class Tarbar extends Component{
    render(){
        return (
        	<div className='tarbar'>
            	123
            </div>
        );
    }
}

//index.css
.tarbar{
    height:50px;
}

//App.js
import React ,{Component} from 'react';
import Tabbar from './component/tarbar';
class Tarbar extends Component{
    render(){
        return (
        	<div className='App'>
            	<Tarbar />
            </div>
        );
    }
}
```

#### 7.2使用iconfont

进入阿里巴巴矢量图标库，下载四个我们需要的图标（home、car、category、user）。添加项目-下载到本地-解压，然后将里面iconfont开头的文件拷出

```java
iconfont.css
iconfont.eot
iconfont.js
iconfont.svg
iconfont.ttf
iconfont.woff
```



在`src`文件夹下新建`static`的文件夹，用来放置所有的静态文件，将上面的iconfont的文件放进来。

然后在App.js文件中引入iconfont

```javascript
//App.js
import React ,{Component} from 'react';
import Tabbar from './component/tarbar';
import './static/iconfont.css';//引入iconfont

class Tarbar extends Component{
    render(){
        return (
        	<div className='App'>
            	<Tarbar />
            	<div className='iconfont iconfont-home'></div>
            </div>
        );
    }
}
```

经完善后，APP.js的文件为

```javascript
//App.js
import React ,{Component} from 'react';
import Tabbar from './component/tarbar';
import './static/iconfont.css';//引入iconfont

const tabbar={
    {img:'icon-home',text:'首页'}，
    {img:'icon-fenlei',text:'分类'}，
    {img:'icon-gouwuche',text:'购物车'}，
    {img:'icon-user',text:'我的'}，
}

class Tarbar extends Component{
    render(){
        return (
        	<div className='tabbar'>
            	<div className='tabbar-content'>
                    {
                        tabbar.map((v,i)=>{
                            <div key={i} className='tabbar-item'>
                                <div className={'iconfont '+v.img}></div>
                                <div>{v.text}</div>
                            </div>
                        });
                    }
    			</div>
            </div>
        );
    }
}
```

接下来index.css文件中添加样式文件

```css
...
.iconfont{
    font-size:28px !important;//!important强制生效
}
.tarbar{
    height:50px;
    position:absolute;
    bottom:0;
    width:100%;
    border:1px solid #ccc;
    padding:5px 0;
}
.tabbar-content{
    display:flex;
}
.tabbar-item{
    flex:1;//每个元素的区间是均分的
}
```

#### 7.4 添加事件

需要给tabbar-item添加点击切换事件，图标响应不同颜色。思路：通过点击事件-改变state-实时更新样式

```javascript
//Tabbar
...
class Tarbar extends Component{
    constructor(props){
        super(props);
        this.state={
            index:''
        }
    }
    itemChange=(i)=>{
        this.setState({index:i});
    }
    
    render(){
        return (
        	<div className='tabbar'>
            	<div className='tabbar-content'>
                    {
                        tabbar.map((v,i)=>{
                            <div key={i} className={'tabbar-item'+ this.state.index===i?'active:'' } onClick={()=>this.itemChange(i)}>
                                <div className={'iconfont '+v.img}></div>
                                <div>{v.text}</div>
                            </div>
                        });
                    }
    			</div>
            </div>
        );
    }
}

//在index.css中给active添加样式
    .active{
        color:red;
    }
```

#### 7.5 添加路由

安装包

```javascript
cnpm install -S react-router-dom
```

在src 目录下新建文件`router.js`

```javascript
import React from 'react';
import {BrowserRouter,Route,Switch} from 'react-router-dom ';
//下面四个页面在src目录的pages的页面
import Home from "./pages/home";
import Category from "./pages/category";
import Car from "./pages/car";
import User from "./pages/user";

export default ()=>(
	<BrowserRouter>
    	<Switch>
    		<Route path='/home' component={Home}></Route>
			<Route path='/category' component={Category}></Route>
			<Route path='/car' component={Car}></Route>
			<Route path='/user' component={User}></Route>
    	</Switch>
    </BrowserRouter>
);
```

然后在App.js中将router.js导入即可

```javascript
//App.js
import React ,{Component} from 'react';
import Tabbar from './component/tarbar';
import RouterMap from './router';
class Tarbar extends Component{
    render(){
        return (
        	<div className='App'>
            	<RouterMap />
            </div>
        );
    }
}
```

这样当我们在浏览器输入`localhost:3030/home`时就显示的Home页面 



#### 7.6 Tabbar路由的切换

接下来我们需要在Home、Car、Category、User中引入Tabbar

```javascript
import React ,{Component} from 'react';
import Tabbar from './component/tarbar';

class Home extends Component{
    render(){
        return (
        	<div>
            	<Tabbar />
            </div>
        );
    }
}
```

为了方便，将Home等组件内容用一张图片代替

```javascript
import React ,{Component} from 'react';
import Tabbar from './component/tarbar';

class Home extends Component{
    render(){
        return (
        	<div>
            	<img className='bg' src={require('../static/imgs/home.png')} />
            	<Tabbar />
            </div>
        );
    }
}
export default Home;

//在APP.css中添加样式
    .bg{
        width:100%;
        height:100%;
    }
```

此时我们点击下面四个按钮，只是按钮 变红，实际上我们想让我们的路由改变，展示不同的页面；

我们需要修改Tabbar各item的点击事件，可以先将数据写进tabbar数据中

```javascript
const tabbar={
    {img:'icon-home',text:'首页',link:'/home'}，
    {img:'icon-fenlei',text:'分类',link:'/category'}，
    {img:'icon-gouwuche',text:'购物车',link:'/car'}，
    {img:'icon-user',text:'我的',link:'/user'}，
}
```

这样在跳转的时候我们就知道是哪一个路由了，为了简单也不可以不用点击事件，用link标签来做；

```javascript
import {Link} from 'react-router-dom';

			<div className='tabbar-content'>
                    {
                        tabbar.map((v,i)=>{
                            <Link to={v.link} key={i} className={'tabbar-item'+ this.state.index===i?'active:'' }>
                                <div className={'iconfont '+v.img}></div>
                                <div>{v.text}</div>
                            </Link>
                        });
                    }
    			</div>

```

这是link标签又默认样式，需要去掉默认样式，

在App.css中给a标签编写样式

```css
a:hover,a:visited,a:link,a:active{
    text-decoration:none;//去掉下划线
    color:#333
}
```



最后还需要给tabbar添加背景颜色，并且发现在点击不同选项时颜色不改变了。这是因为之前我们是根据数字来判断选项是否更改颜色，现在由于在不同页面中没有数字的概念，所以active是不生效的。可以通过获取当前的url来概念颜色

```javascript
//Tabbar
import {Link} from 'react-router-dom';
...
class Tarbar extends Component{
    constructor(props){
        super(props);
        this.state={
            index:''
        }
    }
    itemChange=(i)=>{
        this.setState({index:i});
    }
    
    render(){
        const url=window.location.href;
        return (
        	<div className='tabbar'>
            	<div className='tabbar-content'>
                    {
                        tabbar.map((v,i)=>{
                            <Link to={v.link} key={i} className={'tabbar-item'+ (url.indexOf(v.link)>-1)?'active:'' }>
                                <div className={'iconfont '+v.img}></div>
                                <div>{v.text}</div>
                            </Link>
                        });
                    }
    			</div>
            </div>
        );
    }
}

```

这时可以给选项添加正确的class，但是class并未生效，active的作用被覆盖了，需要使用!important 来强制生效

```css
//在index.css中给active添加样式
    .active{
        color:red !important;
    }
```

 此时还会发现另一个问题，我们的tabbar会遮挡Home、Car等页面，该如何处理呢，

可以给Home等组件的的背景css添加margin-bottom，值设为tabbar的高度

```css
//在APP.css中添加样式
.bg{
    width:100%;
    //height:100;//需要把这个属性删除，以实现内容的自动滚动
    margin-bottom:50px;
}
```

但是这时又发现我们的tabbar随着Home等组件上下滚动了，这不是我们想要的结果，查看tarbar的布局，position:absolute，我门想要tabbar相对于视口来布局的，将position改为fixed即可



#### 7.7 高阶组件改写Tabbar

上面我们已经把所有功能实现了，但是对于Home、Car、Category、User存在大量重复性的代码，这就可以使用高阶组件进行提取。

很简单，遵循两个步骤就可以了

* 实现一个普通组件
* 将普通组件使用函数包裹

在index.js中找到Tabbar，使用函数将其包裹起来

```:ab:
...
const Tabbar=(WrappedComponent)=> class Tabbar extends Component{
    ...
    render(){
        const url=window.location.href;
        reutrn (
        	<div className='tabbar-container'>
        		//注意必须将WrappedComponent在该组件渲染
        		<div className='tabbar-children'>
        			<WrappedComponent />
        		</div>
        		<div className='tabbar
        			<div className='tabbar-content'>
        			//...
        			</div>
        		div>
        	</div>
        );
    }
}

//将下面的内容放在上边
//class Tabbar extends Component{}

//在tabbar的css文件index.css中给tabbar-children设置样式,就可以把App.css文件中的.bg去掉了。
.tabbar-children{
    margin:bottom:50px;
}

```

在使用时，需要改变一下了，在Home.js中

```:ab:
import React ,{Component} from 'react';
import Tabbar from './component/tarbar';

class Home extends Component{
    render(){
        return (
        	<div>
            	<img className='bg' src={require('../static/imgs/home.png')} />
            	~~<Tabbar />~~
            </div>
        );
    }
}

export default Tabbar(Home);//输出的时候，用函数更改即可

```





link可以出发路由的改变，路由改变之后需要APP去监听做何改变




注意：此文为慕课网视频笔记
[https://www.imooc.com/learn/1075](https://www.imooc.com/learn/1075)