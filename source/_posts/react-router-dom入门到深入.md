---
title: react-router-dom入门到深入
date: 2020-04-02 23:30:06
tags: React
---

### React-router-dom

#### 一、安装

```javascript
npm install react-router-dom -save-d
```

#### 二、使用

创建A和B两个页面

```javascript
//A.js
import React from 'react';
function A(){
    return (
    	<div>A</div>
    );
}

//B.js
import React from 'react';
function B(){
    return (
    	<div>B</div>
    );
}
```

创建一个路由文件router.js

```javascript
import React from 'react';
import {HashRouter, Route, Switch} from 'react-router-dom';
import A from './a';
import B from './b';

const TestRouter=()=>(
	<HashRouter>
        <Switch>
            <Route exact path="/" component={A}/>
            <Route exact path="/b" component={B}/>
        </Switch>
    </HashRouter>
);
export {TestRouter};
```

最后在组件的渲染页面，渲染该路由即可

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import {TestRouter} from './router/router';

ReactDOM.render(
  <Router/>,
  document.getElementById('root')
);
```

此时在页面输入`http://localhost:3000/#/`就会看到A页面，输入`http://localhost:3000/#/b`就会看到B页面。url中的#号目前暂不考虑

#### 三、路由的跳转

##### 3.1 a标签

可以通过a标签进行页面的跳转，如下修改A页面就可以通过点击A页面中的A

来实现跳转到B页面了

```javascript
//A.js
import React from 'react';
function A(){
    return (
    	<div>
        <a href='#/b' >A</a>
        </div>
    );
}
```

##### 3.2 通过Link标签跳转

react-router-dom中提供了Link标签也可以提供页面的跳转，Link的底层还是使用的a标签。

如下面修改B页面，就可以实现点击B跳到A页面

```javascript
//B.js
import React from 'react';
function B(){
    return (
    	<div>
        <Link to='#/'>B</Link>
        </div>
    );
}
```

##### 3.3 通过函数跳转

react-router-dom中提供了`hashHistory`，设置给组件的`history`属性，然后同过改变`this.props.history`参数，用来实现页面的跳转.

```javascript
//在router.js文件中给 HashRouter引入history属性
...
import {HashRouter, Route, Switch, hashHistory} from 'react-router-dom';
...
<HashRouter history={hashHistory}>
...

//A.js
import React from 'react';
function A(props){
    return (
    	<div>
        <a href='#/b' >A</a>
        <button onClick={() => props.history.push('b')}>test</button>
        </div>
    );
}
```

这样点击test按钮就可以使用向B页面的跳转了

其中改变`this.props.history`参数的方法有：

* push() ：添加新的路由
* replace()：替换路由（猜测是替换当前路由，被替换的路由则不能通过goBack返回，防止死循环）
* goBack()：返回上一个路由（返回上级页面）

#### 四、路由传参

##### 4.1 通过路由配置传参

在上面router.js文件中，我们只是配置了路由的绝对路径，有些时候我们在跳转某个路由时还要接收一些参数，该怎么处理呢？简单的可以通过在路由后追加'/:key'，然后再路由对应的组件通过`this.props.match.params.key`来接收到相应的参数值

如下，我们给B页面传递`name`参数。

```javascript
//router.js
const TestRouter=()=>(
	<HashRouter>
        <Switch>
            <Route exact path="/" component={A}/>
            <Route exact path="/b/:name" component={B}/>
        </Switch>
    </HashRouter>
);
...

//B.js
import React from 'react';
function B(props){
    return (
    	<div>
        	B
        	<p>我接收的参数是{props.match.params.name}</p>
        </div>
    );
}
```

当然如果传递多个参数就再追加即可

```javascript
<Route exact path="/b/:name/:age" component={B}/>
```

这样皆可以通过`http://localhost:3000/#/b/afei/21`来访问到afei和age了，

这种方式虽然可以实现传参，但是稍微少传一个参数就会报错。

如果是使用Link标签如何传参呢，很简单使用query关键 字即可

```javascript
//B.js
import React from 'react';
function B(){
    return (
    	<div>
        <Link to={path:'/a',query:{name:'afei',age:19}}>B</Link>
        </div>
    );
}
```





并且总觉得的不如`http://localhost:3000/#/b?name=afei&age=19`来的方便。（日后补上）

##### 4.2 通过函数隐式传参

那如果是通过函数跳转是，如何传参呢？很简单,在触发时给history传递对象，然后再对应的页面通过`this.props.history.location.state`获取即可，如下

```javascript
//A.js
<button onClick={() => props.history.push(
    {
    pathname:'/b',
    state:{
        name:'afei',
    	age:18
    	}
	}
)}>test</button>

//B.js
import React from 'react';
function B(props){
    return (
    	<div>
        	B
        	<p>我的名字是{props.history.location.state.name}</p>
			<p>我的年龄是{props.history.location.state.age}</p>

        </div>
    );
}
```



注意上边两种方式传参和接参方式不能混了

#### 五、BrowserRouter和hashRouter

有一个现象当我们的router.js使用HashRouter时，我们可以通过浏览器地址栏直接键入url来访问任意路由

```javascript
//HashRouter
import React from 'react';
import {HashRouter, Route, Switch} from 'react-router-dom';
import A from './a';
import B from './b';

const TestRouter=()=>(
	<HashRouter>
        <Switch>
            <Route exact path="/" component={A}/>
            <Route exact path="/b" component={B}/>
        </Switch>
    </HashRouter>
);
export {TestRouter};
```

此时

* 当我们可以输入`http://localhost:3000/`时，地址栏会自动变为`http://localhost:3000/#/`进行正常访问；
* 当地址栏输入`http://localhost:3000/#/`可以正常访问A页面

* 当地址栏输入`http://localhost:3000/#/b`可以正常访问B页面



但是当我们的router.js中将HashRouter换成BrowserRouter时，就会出现意外。

```javascript
...
<BrowserRouter>
        <Switch>
            <Route exact path="/" component={A}/>
            <Route exact path="/b" component={B}/>
        </Switch>
    </BrowserRouter>
```



表现为：

* 当我们可以输入`http://localhost:3000/`时，可以正常访问A页面
* 当地址栏输入`http://localhost:3000/b`不能正常访问B页面，报错`Cannot GET/b`
* 当我们在A页面以`<Link to="b">B</Link>`的形式，可以正常访问B页面

神不神奇，为什么HashRouter可以直接访问路由，而BrowserRouter访问不到？

这是应为BrowserRouter每次改变路由时，会向服务器请求该路由，服务端没有该路径的资源，自然会报错了。当然可以开启一个服务，路由对应的路径下放置资源文件，此错误就会消失。

而使用HashRouter时，会在路径后边添加`/#/`,并且`/#/`及后边的内容是不会发送给服务器的，当访问`http://localhost:3000/#/b`时，其实服务器端收到的依然是`http://localhost:3000`，`/#/`及后边的内容

会直接在前端进行处理区分。

但是官方却还是推荐我们使用BrowserRouter，这就和它们内部的机制有关了

* BrowserRouter使用的是HTML5中的history的api，通过`pushState`、`replaceState`、`popState`来完成UI和URL的同步
* HashRouter使用的是URL的hash方式，来完成UI和URL的同步

下章细讲

#### 六、history和URL

下面的内容只做了解

##### 6.1 history

history接口允许操作浏览器的曾经在标签页或者框架里访问会话历史记录。它有一些自己的属性，这里是说一下比较重要的`History.state`和`History.length`

* `state`是一个只读属性，访问该属性会返回一个表示历史堆栈顶部的状态的值。它可以不必等待`popstate`事件而查看状态的方式
* `length`是一个只读属性，表示会话历史中元素的数目，包括当前的加载页。

下面来看history中三个比较重要的方法

* pushState()：按指定的名称和URL（如果提供该参数）将数据push进会话历史栈，这个数据被DOM 进行了不透明处理。可以理解为往历史记录里追加新的页面
* replaveState()：按指定的数据，名称和URL(如果提供该参数)，更新历史栈上最新的入口。这个数据被DOM 进行了不透明处理。可以理解更新历史记录里当前页面信息
* popState()：很奇怪，没有查到该方法
* back()：往上一页，模拟浏览器左上角的撤销功能
* forward()：往下一页，模拟浏览器左上角的下一页功能
* go()：通过当前页面的相对位置从浏览器历史记录( 会话记录 )加载页面。比如：参数为-1的时候为上一页，参数为1的时候为下一页。参数超过区间范围（length区间）或不合法时不做任何操作

总的来说可以将history看作是维护浏览器历史记录的管理器。

##### 6.2 URL

**URL** 接口是一个包含若干静态方法的对象，用来创建 URLs。它的属性主要是实现*URLUtils* 中定义的属性，包括：

* href：包含完整URL的字符串
* host：包含URL域名+“ : ”+端口号的字符串
* Hostname：包含URL域名的字符串
* port：包含URL端口号的字符串
* pathname：以`/`起头紧跟URL文件路径的字符串
* search：以`？`起头紧跟URL文件路径的字符串
* hash：以`#`起头紧跟URL文件路径的字符串
* username：包含在域名前面指定的用户名字符串
* password：包含在域名前面指定的密码字符串
* origin：只读属性，返回一个包含协议名、域名和端口号的字符串

* searchParams：返回一个用来访问当前URL GET请求参数的 [`URLSearchParams`](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams) 对象。

URL还有一些静态方法，不再赘述。这里只需要明白所谓的`#`是URL的hash属性的规范，就跟seacrh的`?`一样。





参考:

[https://www.jianshu.com/p/8954e9fb0c7e](https://www.jianshu.com/p/8954e9fb0c7e)

[https://reacttraining.com/react-router/web/api/BrowserRouter](https://reacttraining.com/react-router/web/api/BrowserRouter)

[https://www.cnblogs.com/soyxiaobi/p/11096940.html](https://www.cnblogs.com/soyxiaobi/p/11096940.html)

[https://developer.mozilla.org/zh-CN/docs/Web/API/History](<https://developer.mozilla.org/zh-CN/docs/Web/API/History>)

[https://developer.mozilla.org/zh-CN/docs/Web/API/URL](<https://developer.mozilla.org/zh-CN/docs/Web/API/URL>)
