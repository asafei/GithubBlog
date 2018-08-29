---
title: react中的css
date: 2018-08-29 23:02:46
tags: React
---
### 3、react中的css

#### 3.1 react中的css过度动画

下面我们实现一个过渡动画效果，页面一个文字，一个按钮，点击按钮则文字逐渐消失，再点击，文字逐渐显示。

> 1、新建一个react组件APP
>
> ```javascript
> import React { Component , Fragment} from 'react';
> import './App.css';//引入css文件
> 
> class App extends Component{
>     constructor(props){
>         super(props);
>         this.state={
>             show:true
>         }
>         this.handleToggole=this.handleToggole.bind(this);
>     }
>     render(){
>         return ({
>             <Fragment>
>             	<div className={this.state.show?'show':'hide'}>hello</div>
>             	<button onClick={this.handleToggole}>tonggle<button>
>             </Fragment>
>         });
>     }
>     handleToggole(){
>         this.setState({
>    			//若为true则变成false，若为false则变成true
>             show:this.state.show?false:true;
>         });
>     }
> }
> ```
>
> 2、新建App.css，定义show和hide的样式
>
> ```javascript
> .show {
>     opacity:1;
>     transition:all 1s ease-in;//css3的动画样式，在1s内实现过渡
> }
> 
> .hide {
>     opacity:0
>     transition:all 1s ease-in;
> }
> ```



#### 3.2 react中的css动画效果

什么是css的动画效果呢？其实是指通过 @keyframes定义的一些css动画

可以在App.css文件中这个定义并使用动画

```javascript
.show {
    opacity:1;
    transition:all 1s ease-in;//css3的动画样式，在1s内实现过渡
}

//使用动画
.hide {
    animation:hide-item 2s ease-in;
}
//定义动画
@keyframes hide-item{
    0% {//当进度为0%时 的状态
        opacity:1;
        color:red;
    }
    50% {//当进度为50%时 的状态
        opacity:0.5;
        color:green;
    }
    100% {//当进度为100%时 的状态
        opacity:0;
        color:blue;
    }
    
}
```

这是我们的文字具有动画效果了，可是此时会发现问题，当文字动画执行完之后，文字并没有消失，

这是因为，我们动画的最后一帧并没有保存，该怎么做呢，很简单，

只需要在animation最后添加 forwards即可

```javascript
animation:hide-item 2s ease-in forwards;
```

这样就实现了想要效果，同理可以对show样式做同样的处理



到此刻就可以实现纯css3的动画，如果想实现js辅助动画，这种方法就解决不了了。那该如何做呢？可以使用第三方框架

#### 3.3 使用react-transition-group实现动画

可以使用react-transition-group实现js辅助动画

##### 3.3.1 CSSTransition的css方式实现动画

> 1、首先安装React Transition Group模块，参考[React Transition Group](https://reactcommunity.org/react-transition-group/)
>
> ```javascript
> # npm
> npm install react-transition-group --save
> ```
>
> 然后重新启动自己的项目
>
> 在可以看到有三个组成
>
> ```javascript
> Transition： //更底层的组件，当高层组件不够用时才用它
> CSSTransition
> TransitionGroup
> ```
>
> 下面首先介绍一下CSSTransition
>
> 2、在组件APP中引入CSSTransition，
>
> ```javascript
> import { CSSTransition } from 'react-transition-group';
> ```
>
> 然后在render函数中，就可以使用CSSTransition。只需要将div包裹在CSSTransition标签内就可以，，而不再需要那些自定义的样式，CSSTransition组件会自动的帮我们做一些class名字的增加和移除的工作，如下
>
> ```javascript
>  render(){
>         return ({
>             <Fragment>
>             	<CSSTransition>
>             		//<div className={this.state.show?'show':'hide'}>hello</div>
>             		<div>hello</div>
>                 </CSSTransition>
>             	<button onClick={this.handleToggole}>tonggle<button>
>             </Fragment>
>         });
>     }
> ```
>
> 3、CSSTransition有一些属性，需要了解下
>
> ```javascript
>  * in  ：CSSTransition什么时候感知到样式
>  * timeout : 动画时间 单位毫秒
>  * fade-enter：入场动画将要执行的第一瞬间，还没有执行时，就会被安排在div标签上
>    fade-enter-active：入场动画执行的第二个时刻，到入场动画执行结束之前，就会被安排在div标签上
>    fade-enter-done： 整个入场动画执行完成之后，就会被安排在div标签上
> ```
>
> 对于上面fade属性在使用时，需要设置前缀传给classNames，当然可以使用fade开头，也可以使用其它单词开头
>
> 4、入场动画
>
> 最终的render函数可以这么写
>
> ```javascript
> render(){
>         return ({
>             <Fragment>
>             	<CSSTransition
>             		in={this.state.show}
>                 	timeout={1000}
>                 	classNames='fade'>
>             		<div>hello</div>
>                 </CSSTransition>
>             	<button onClick={this.handleToggole}>tonggle<button>
>             </Fragment>
>         });
>     }
> ```
>
> css文件中这么写
>
> ```javascript
> .fade-enter {
>     opacity:0;
> }
> 
> .fade-enter-active {
>     opacity:1;
>     transition:opacity 1s ease-in;
> }
> 
> .fade-enter-done {
>     opacity:1;
> }
> ```
>
> 5、出场动画
>
> 这时候已经有入场动画了，我们可以把出场动画也给做了，这是就使用fade-exit`, `fade-exit-active`, `fade-exit-done
>
> ```javascript
> * fade-exit:出场动画将要执行的第一瞬间，还没有执行时，就会被安排在div标签上
>   fade-exit-active:出场动画执行的第二个时刻，到出场动画执行结束之前，就会被安排在div标签上
>   fade-exit-done:整个出场动画执行完成之后，就会被安排在div标签上
> ```
>
> 我们只需要在css文件中这么添加，render无需添加任何更新（因为已经有了fade前缀了）
>
> ```javascript
> .fade-exit {
>     opacity:1;
> }
> fade-exit-active {
>     opacity:0;
>     transition:opacity 1s ease-in;
> }
> fade-exit-done{
>     opacity:0;
> }
> ```
>
> 5、消除标签
>
> 此时我们的效果是 点击按钮实现文字的消失和再显。
>
> 如果我们想实现当文字消失时同时这个div标签也移除，而不是占的位置显示空白
>
> 此时我们可以使用unmountOnExit属性，就可以达到目的，最终render函数如下
>
> ```javascript
> render(){
>         return ({
>             <Fragment>
>             	<CSSTransition
>             		in={this.state.show}
>                 	timeout={1000}
>                 	classNames='fade'
>                 	unmountOnExit>
>             		<div>hello</div>
>                 </CSSTransition>
>             	<button onClick={this.handleToggole}>tonggle<button>
>             </Fragment>
>         });
>     }
> ```
>
>

以上时通过css的方式实现动画，CSSTransition还提供了通过js实现动画的方式防范如下

##### 3.3.2 CSSTransition的js方式实现动画

比如钩子onEntered就是当入场动画结束之后才会被调用，可以作为CSSTransition的属性调用，函数的参数el指的就时div的元素，这就可以实现js控制动画效果了。

```javascript
render(){
        return ({
            <Fragment>
            	<CSSTransition
            		in={this.state.show}
                	timeout={1000}
                	classNames='fade'
                	unmountOnExit
                	onEntered={(rl)=>{el.style.color="blue"}}>
            		<div>hello</div>
                </CSSTransition>
            	<button onClick={this.handleToggole}>tonggle<button>
            </Fragment>
        });
    }
```



##### 3.3.3 CSSTransition实现首次展示动画

此时还有一个问题，我们首次展示时，文字并没有动画效果，如何让页面首次展示时也有动画效果呢，很简单，直接在CSSTransition标签中添加appear属性为true即可。

```javascript
			<CSSTransition
            		in={this.state.show}
                	timeout={1000}
                	classNames='fade'
                	unmountOnExit
                	onEntered={(rl)=>{el.style.color="blue"}}
                    appear={true}>
            		<div>hello</div>
                </CSSTransition>
```

此时CSSTransition会在div首次展示时添加动画，但是这个动画并不叫fade-enter，而是叫fade-appear，fade-appear-active，所以我们还需要再css中添加它们

```javascript
.fade-enter .fade-appear{
    opacity:0;
}

.fade-enter-active .fade-appear-active{
    opacity:1;
    transition:opacity 1s ease-in;
}

.fade-enter-done {
    opacity:1;
}
```

#### 3.3.4 CSSTransition的group方式实现动画

上边我们讲的是针对一个元素的动画，下面讲一下，如何实现多个元素的动画效果。这就需要使用到TransitionGroup了，

1、首先我们引入该组件

```javascript
import { CSSTransition , TransitionGroup} from 'react-transition-group';
```



现在我门state的show不在存储，而已存储一个数组，如下

```javascript
this.state={
    list:[]
}
```

2、然后修改render中的动画 标签，注意TransitionGroup还需要配合CSSTransition使用，使用CSSTransition修饰每一个单独的div

```javascript
render(){
        return ({
            <Fragment>
            	<TransitionGroup>
                    {
                        this.state.list.map((item,index)=>{
                            return (
            					<CSSTransition
                                    in={this.state.show}
                                    timeout={1000}
                                    classNames='fade'
                                    unmountOnExit
                                    onEntered={(rl)=>{el.style.color="blue"}}
                                    appear={true}>
                                	<div key={index}>{item}</div>
								</CSSTransition>
                            )
                        })
                    }
                </TransitionGroup>
            	<button onClick={this.handleAddItem}>tonggle<button>
            </Fragment>
        });
    }
    
    handleAddItem(){
        this.setState((prevState)=>{
            return {
                list:[...prevState.list,"item"]
            }
        }); 
    }
```

这样我们就实现了多个元素的动画了。

----
本笔记参考慕课网视频[https://coding.imooc.com/class/229.html](https://coding.imooc.com/class/229.html)

