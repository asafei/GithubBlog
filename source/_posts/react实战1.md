---
title: react实战1
date: 2018-10-18 22:19:18
tags: React
---
## React实战-简书

### 一、安装React

创建名为`jiashu`的react-app，并且清理工程目录，仅保留index.js、index.css和App.js文件。步骤略。

### 二、css样式管理

#### 2.1 react中css使用

在index.css文件中添加样式

```javascript
.dell {
	background:red;
}
```

然后在App.js文件中，给div添加className，这时页面的div就变成了红色。

注意：并没有给App.js文件引入index.css，只有文件index.js中引用了index.css文件，但是我们在App.js中依然可以使用其中的样式文件。这是因为css文件一旦在一个文件被引入，那么就会全局生效，即全局任何一个文件都能使用这里面的样式。

但是这样容易产生混乱，不建议这么做，这时推荐使用一个第三方的css管理，名为`styled-compenents`

#### 2.2 react中使用styled-compenents管理css

首先安装该包

```javascript
npm install --save styled-components
//或
yarn add styled-components
```

然后我们给`index.css`文件改名为`style.js`，然后在index.js文件中修改对应的引用

```javascript
~~ import './index.css'; ~~
import './style.js';
```

然后在style.js中如何创建全局样式呢，如下

```javascript
import { injectGlobal } from 'styled-components';
injectGlobal`
	body {
      margin: 0;
      padding: 0;
      font-family: sans-serif;
    }
    .dell {
        background:red;
    }
`
```

这时再看页面，又能正常显示了。

那么如何数据跟组件一对一的样式呢，后面会有介绍。

由于浏览器的差异，如何做到让不同浏览器的样式统一呢？这时需要用到一个reset.css的文件

#### 2.3 完成浏览器样式的统一

不同浏览器内核中对body等标签默认样式的设置是不同的，为了代码再所有浏览器上表现是一致的，这就可以使用[reset.css](https://meyerweb.com/eric/tools/css/reset/)，将里面的内容，拷贝到我们的全局样式里面

```javascript
import { injectGlobal } from 'styled-components';
injectGlobal`
	html, body, div, span, applet, object, iframe,
    h1, h2, h3, h4, h5, h6, p, blockquote, pre,
    a, abbr, acronym, address, big, cite, code,
    del, dfn, em, img, ins, kbd, q, s, samp,
    small, strike, strong, sub, sup, tt, var,
    b, u, i, center,
    dl, dt, dd, ol, ul, li,
    fieldset, form, label, legend,
    table, caption, tbody, tfoot, thead, tr, th, td,
    article, aside, canvas, details, embed, 
    figure, figcaption, footer, header, hgroup, 
    menu, nav, output, ruby, section, summary,
    time, mark, audio, video {
        margin: 0;
        padding: 0;
        border: 0;
        font-size: 100%;
        font: inherit;
        vertical-align: baseline;
    }
    /* HTML5 display-role reset for older browsers */
    article, aside, details, figcaption, figure, 
    footer, header, hgroup, menu, nav, section {
        display: block;
    }
    body {
        line-height: 1;
    }
    ol, ul {
        list-style: none;
    }
    blockquote, q {
        quotes: none;
    }
    blockquote:before, blockquote:after,
    q:before, q:after {
        content: '';
        content: none;
    }
    table {
        border-collapse: collapse;
        border-spacing: 0;
    }
`
```

### 三、头部区块的编写

一般头部区块，是很多页面都要公用的。

#### 3.1 创建header组件

首先在src目录下创建一个文件夹common，在common文件夹中再创建一个header的文件夹，然后再header文件夹中创建index.js文件，这个就是header的组件。接下来我们来定义这个组件

```javascript
import React , { Component } from 'react';
class Header extends Component{
    render(){
        return {
            <div>
            	header
            </div>
        }
    }
}
export default Header;
```

然后再`src/App.js`文件中引入Header组件，并使用

```javascript
import React,{Component} from 'react';
import Header from './common/header/index.js';
class App extends Component{
    render(){
        return {
            <Header />
        }
    }
}
export default App;
```

#### 3.2创建header样式

在`src/common/header/`文件夹下，创建一个`style.js`的样式文件，在里面编写Header自己用的样式

```javascript
import styled from 'styled-components';
export const HeaderWrapper=styled.div`
	height:56px;
	background:red;
`;
```

这样就可以在Header组件(`src\common\header\index.js`)中，引入该样式并使用了

```javascript
import React , { Component } from 'react';
import {HeaderWrapper} from './style.js';
class Header extends Component{
    render(){
        return {
            <HeaderWrapper>
            	header
            </HeaderWrapper>
        }
    }
}
export default Header;
```

这时header的简易样式就出来了。

查看简书官网可以看到header有58px高，并且有个1px的边框，可以这么修改

```javascript
height:58px;
border-bottom:1px soid #f0f0f0;
```

#### 3.3 展示logo

首先去简书官网，将左上角的logo下载到我们项目的`src/state/`目录下，命名为`logo.png`。

接下来我们在`src/common/header/style.js`中定义一个Logo的组件，由于点击该组件可以进行条状，所以该组件是一个a标签

注意在属性url的不能这么写`background:url('../../statics/logo.png')`因为webpack打包时会当成字符串，而因该使用引用的方式引用该图片，然后使用`${}`形式进行使用;

图片有点大，还要注意给图片设置大小`background-size`

```javascript
import logoPic from '../../common/statics/logo.png';
...
export const Logo=styled.a`
	position: absolute;
	top: 0;
	left: 0;
	display:block;
	width: 100px;
	height: 58px;
	background:url(${logoPic});
	background-size:contain;
`;
```

然后就可以在`/src/common/header/index.js`文件中使用它了，a标签是一个链接，如何让点击时回到首页呢？可以直接给Logo设置`href="/"`属性

```javascript
import React , { Component } from 'react';
import { HeaderWrapper , Logo} from './style.js';
class Header extends Component{
    render(){
        return (
			<HeaderWrapper>
				<Logo href='/' />
			</HeaderWrapper>
        )   
    }
}
export default Header;
```

当然也可以在style.js中设置href属性，可以这么写style

```javascript
export const Logo=styled.a.attrs({
    href:'/'
})`
	position: absolute;
	top: 0;
	left: 0;
	display:block;
	width: 100px;
	height: 58px;
	background:url(${logoPic});
	background-size:contain;
`;
```

然后我们编写header的中间组件

#### 3.4 编写header中间部分Nav

可以现在`/src/common/header/index.js`文件中添加一个新组件Nav，再编写

```javascript
import React , { Component } from 'react';
import { HeaderWrapper , Logo, Nav} from './style.js';
class Header extends Component{
    render(){
        return (
			<HeaderWrapper>
				<Logo href='/' />
            	<Nav />
			</HeaderWrapper>
        )   
    }
}
export default Header;
```

然后在`src/common/header/style.js`中定义一个Nav的组件

```javascript
export const Nav=styled.div`
	width:960px;
	height:100%;
	margin:0px;
	background:green;
`;
```

#### 3.5 填充Nav

这样就能看到我们的中间了，接下来我们往Nav中添加东西，添加4个NavItem，为了给它们添加浮动，给它们添加样式

```javascript
import { HeaderWrapper , Logo , Nav ,NavItem} from './style.js';
...
<Nav>
    <NavItem className='left'>首页</NavItem>
    <NavItem className='left'>下载App</NavItem>
    <NavItem className='right'>登陆</NavItem>
    <NavItem className='right'>Aa</NavItem>
</Nav>
```

接下来我们就能在`src/common/header/style.js`中定义一个NavItem的组件了，其中`&.left`表示NavItem组件有left属性的话，就执行向左浮动；

同时它们也有一些共有的属性如line-height、padding、font-size等；

```javascript
export const NavItem=styled.div`
	line-height:58px;
	padding:0 15px;
	font-size:17px;
    &.left{
        float:left;
    }
	&.right{
        float:right;
    }
`;
```

同时也可以给文字指定颜色，比如让首页为红色，其它设为黑色，右侧文字黑色浅一些：

```javascript
<NavItem className='left active'>首页</NavItem>
```

```javascript
export const NavItem=styled.div`
	line-height:58px;
	padding:0 15px;
	font-size:17px;
	color:#333;
    &.left{
        float:left;
    }
	&.right{
        float:right;
		color:#969696;
    }
    &.active{
		color:#ea6f5a;
    }
`;
```



#### 3.6 编写NavSearch组件

再Nav中添加NavSearch组件

```javascript
import { HeaderWrapper , Logo , Nav ,NavItem, NavSearch} from './style.js';
...
<Nav>
    <NavItem className='left'>首页</NavItem>
    <NavItem className='left'>下载App</NavItem>
    <NavItem className='right'>登陆</NavItem>
    <NavItem className='right'>Aa</NavItem>
	<NavSearch></NavSearch>
</Nav>
```

然后在`src/common/header/style.js`中定义一个NavSearch的组件

```javascript
export const NavSearch=styled.input.attrs({
	placeholder:'搜索'
})`
	width:160px;
	height:38px;
	padding:0 20px;
	box-sizing:border-box;
	border:none;
	outline:none;
	margin-top:9px;
	margin-left:20px;
	border-radius:19px;
	background:#eee;
	font-size:14px;
	&::placeholder{
		color:#999;
	}
`;
```

其中`box-sizing:border-box;`	表示input框不拉伸；`&::placeholder`表示框内文字颜色；

#### 3.7 写注册组件

首先引入Addition，仔里面添加两个Button然后编写style，通过给Button设置className来自定义属性，相应的再style中可以使用`&.XXX`来编写对应的样式

```javascript
import { HeaderWrapper , Logo , Nav ,NavItem, NavSearch,Addition,Button} from './style.js';
...
<Nav>
    ...
</Nav>
<Addition>
	<Button className='writting'>写文章</Button>
	<Button className='reg'>注册</Button>
</Addition>
```

```javascript
export const Addition=styled.div`
	position:absolute;
	right:0;
	top:0;
	height:56px;
`;
export const Button=styled.div`
	float:right;
	margin-top:9px;
	margin-right:20px;
	padding:0 20px;
	line-height:38px;
	border-radius:19px;
	border:1px solid #ec6149;
	font-size:14px;
	&.reg{
		color:#ec6149;
	}
	&.writting{
		color:#fff;
		background:#ec6149;
	}
`;
```

#### 3.8 使用iconfont

首先进入[iconfont.cn](iconfont.cn)下载几个iconfont图标，放在`src/statics/iconfont/`文件夹下，包括下面五个文件：

* iconfont.css
* iconfont.eot
* iconfont.svg
* iconfont.ttf
* iconfont.woff

然后打开iconfont.css文件，在里面将iconfont开头内容，改为添加相对引用的，并且可以删除下面没用的内容

```css
//before
@font-face {font-family: "iconfont";
  src: url('iconfont.eot?t=1536939256958'); /* IE9*/
  src: url('iconfont.eot?t=1536939256958#iefix') format('embedded-opentype'), /* IE6-IE8 */
  url('iconfont.ttf?t=1536939256958') format('truetype'), /* chrome, firefox, opera, Safari, Android, iOS 4.2+*/
  url('iconfont.svg?t=1536939256958#iconfont') format('svg'); /* iOS 4.1- */
}
//after
@font-face {font-family: "iconfont";
  src: url('./iconfont.eot?t=1536939256958'); /* IE9*/
  src: url('./iconfont.eot?t=1536939256958#iefix') format('embedded-opentype'), /* IE6-IE8 */
  url('./iconfont.ttf?t=1536939256958') format('truetype'), /* chrome, firefox, opera, Safari, Android, iOS 4.2+*/
  url('./iconfont.svg?t=1536939256958#iconfont') format('svg'); /* iOS 4.1- */
}
```

iconfont我们倾向于将它们全局使用

首先我们将iconfont.css文件改名为iconfont.js，并且引入

```javascript
import { injectGlobal } from 'styled-components';
```

然后就可以将里面的内容引入全局语法，如下

```javascript
import { injectGlobal } from 'styled-components';

injectGlobal`
  @font-face {font-family: "iconfont";
    src: url('./iconfont.eot?t=1536939256958'); /* IE9*/
    src: url('./iconfont.eot?t=1536939256958#iefix') format('embedded-opentype'), /* IE6-IE8 */
    url('data:application/x-font-woff;charset=utf-8;base64,d09GRgABAAAAAAW0AAsAAAAACEAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAABHU1VCAAABCAAAADMAAABCsP6z7U9TLzIAAAE8AAAARAAAAFY8hEjPY21hcAAAAYAAAABeAAABnLRjHYpnbHlmAAAB4AAAAdIAAAIA2xiyuGhlYWQAAAO0AAAALwAAADYSoy/taGhlYQAAA+QAAAAcAAAAJAfeA4VobXR4AAAEAAAAAA4AAAAQEAAAAGxvY2EAAAQQAAAACgAAAAoBSgCEbWF4cAAABBwAAAAeAAAAIAESAFluYW1lAAAEPAAAAUUAAAJtPlT+fXBvc3QAAAWEAAAAMAAAAEHS0b6peJxjYGRgYOBikGPQYWB0cfMJYeBgYGGAAJAMY05meiJQDMoDyrGAaQ4gZoOIAgCKIwNPAHicY2BkYWCcwMDKwMHUyXSGgYGhH0IzvmYwYuRgYGBiYGVmwAoC0lxTGBye8T27y9zwv4EhhrmBoQEozAiSAwDusgzOeJztkEEOgDAIBAdbTWOML/Fo+iBPfrpXvlGBevARLhkCmw0HgBlIxmFkkBvBdZkr4SfW8DPV9mI1ge5atfX+nUISiRJXxZOy8GuLfr5b8q8NYq4D/7e2AdMDWkcVrgAAeJw1jk1PE1EUhs+ZO/ejfsw47W3HkrY6HWamk0g7zrRTg7VIQg1SExsSiAsVo1GwwkpXJGyElf4G2PAPuurKVBP30BUr/oihtYWQnMWb532S94AKMB6TAQHgoMMsVAAs27O4jVGKuJ7NOKFxFObRrtnMLrq16gJW7SI3NUzLTBTGT1AZfFoenbU+ov5+6QtlCuVdPAuaXx+htViud98+na9slPL3Z5xgOCQw8nHhjmunRr9p7tvfShz469qttvOK5rLpXOgUAECZ/NQnf8gypMECcJpYK6OnIS+gGcZ1MxPGVc+ZoCbWJ0hDZWh75PTw6ERVT44aO5VzY65oXPw76BHSO9jvqWrvorLTuKoPT4lz79ywHyS/q739a+Vyc0z2CIG78GKyKZnnMq5hAetNNAsYNaehjOgWdbzMU56ZKtfW9GLXK+MVMzmTmbCOcZU839rcO2YJdkMm1B/bfis/31CEcjs9Gggpdul6J/gw50c8tbrpddwEvSkT/pofdbIWTeorrYdb1dfbss91KXBDrLC8V/YES6UYZVbJ0JS8yqWh4DMhdks/V4O2KYXIHHe9RZPJJKUzS370OWi1dYMVsy+jN78e94XUOb7jEuA/i1FlEwAAeJxjYGRgYADiF3ZLN8bz23xl4GZhAIHrB7/+QND/d7AwMHsAuRwMTCBRAHx7DUcAeJxjYGRgYG7438AQw8IAAkCSkQEVsAAARwoCbXicY2FgYGBBwgAAsAARAAAAAAAAAEoAhAEAAAB4nGNgZGBgYGHwBWIQYAJiLiBkYPgP5jMAABCXAWwAAHicZY9NTsMwEIVf+gekEqqoYIfkBWIBKP0Rq25YVGr3XXTfpk6bKokjx63UA3AejsAJOALcgDvwSCebNpbH37x5Y08A3OAHHo7fLfeRPVwyO3INF7gXrlN/EG6QX4SbaONVuEX9TdjHM6bCbXRheYPXuGL2hHdhDx18CNdwjU/hOvUv4Qb5W7iJO/wKt9Dx6sI+5l5XuI1HL/bHVi+cXqnlQcWhySKTOb+CmV7vkoWt0uqca1vEJlODoF9JU51pW91T7NdD5yIVWZOqCas6SYzKrdnq0AUb5/JRrxeJHoQm5Vhj/rbGAo5xBYUlDowxQhhkiMro6DtVZvSvsUPCXntWPc3ndFsU1P9zhQEC9M9cU7qy0nk6T4E9XxtSdXQrbsuelDSRXs1JErJCXta2VELqATZlV44RelzRiT8oZ0j/AAlabsgAAAB4nGNgYoAALgbsgIWRiZGZkYWRlYHJMZErLTEvPSUxKzMvnbUiM7Uqk4EBAFR3Bwg=') format('woff'),
    url('./iconfont.ttf?t=1536939256958') format('truetype'), /* chrome, firefox, opera, Safari, Android, iOS 4.2+*/
    url('./iconfont.svg?t=1536939256958#iconfont') format('svg'); /* iOS 4.1- */
  }

  .iconfont {
    font-family:"iconfont" !important;
    font-size:16px;
    font-style:normal;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
`
```

 现在这个样式文件还没有全局生效，要生效该怎么做呢？

我们在`src/index.js`文件中引入该样式

```javascript
import './statics/iconfont/iconfont.js';
```

这时我们看页面控制台没有报错，就说明引用成功，接下来我们就可以使用它了。

进入`src/common/header/index.js`文件，用iconfont标签插入到相应的组件即可，可以通过i标签引入。

```javascript
render(){
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
				    <NavSearch></NavSearch>
				    <i className='iconfont'>&#xe6dd;</i>
				</Nav>
				<Addition>
					<Button className='writting'>
						<i className='iconfont'>&#xe60e;</i>
						写文章
					</Button>
					<Button className='reg'>注册</Button>
				</Addition>
			</HeaderWrapper>
        )   
    }
```

此时我们的页面就展示出来了iconfont元素了。

接下来对搜索地方进行优化，创建一个SeachWrapper样式，将NavSearch和搜索标签包起来，并且在`src/common/header/style.js`中定义该样式，如何给里面的iconfont添加样式呢？只需要使用`.iconfont{}`即可

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
	}
`;
```

这里给iconfont搜索按钮添加了圆角框，是为了以后动画服务。




