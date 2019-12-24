---
title: webpack的使用
date: 2019-12-24 22:22:38
tags: webpack
---
### 一、工程初始化

#### 1.1 入门

* 创建目录catirl

* 进入目录，执行初始化

* ```javascript
  cd catirl
  npm init -y
  npm install webpack webpack-cli --save-dev
  ```

* 创建源文件夹，并添加文件index.js

* ```javascript
  catirl
    |- package.json
  + |- index.html
  + |- /src
  +   |- index.js
  ```

  
  

**注意：**此时我们在`package.json`中保留`"main": "index.js"`即可，没有必要改为`"private": true`，保留的好处是，我们可以随时调试源代码，但是这样的不支持源代码中带有`import`和`export`语法 。



#### 1.2 支持import语法

为了支持`import`和`export`语法 ,webpack可以做转，在`package.json`文件，修改如下：

```javascript
+   "private": true,
-   "main": "index.js",
```

然后创建一个dist的目录，将`index.html`移动进去，并执行命令

```javascript
npx webpack
```

此时会将我们的脚本作为[入口起点](https://www.webpackjs.com/concepts/entry-points)，然后 [输出](https://www.webpackjs.com/concepts/output) 为 `main.js`，将`main.js`引入`index.html`就可以继续访问页面了。

通过npm进行打包，在`package.json`中补充

```javascript
"scripts": {
    "build": "webpack"
  },
```

就可以使用npm命令进行构建了

```javascript
npm run build
```



#### 1.3 进一步定制webpack

以上使用的都是默认的webpack的配置，如何定制呢，比如修改源文件入口文件、输出文件路径、输出文件名称，这时候就得配置`webpack.config.js`文件了。

新建`webpack.config.js`文件：

```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',    //入口文件
  output: {
    filename: 'bundle.js',    //输出文件名称
    path: path.resolve(__dirname, 'dist')//输出文件路径
  }
};
```

还有很多更复杂的配置，以后再讲

#### 1.4 可调试源码

此时碰到一个问题，打包出来的 `main.js`或 `bundle.js`是压缩过的代码，很难调试，这时候怎么办呢？[参考](<https://www.jb51.net/article/150357.htm>)

此时就需要配置`webpack.config.js`了，在里面添加

```javascript
module.exports = {
 devtool: 'source-map',
 entry:"",
 output: {
 }
 ....
}
```

再次执行`npm run build`就可以看到，生成的文件多了一个`bundle.js.map`文件,

有了它，就可以调试源码的页面了。

但是这样还不是很方便，在编写代码过程中，我们往往需要实时热更新加载。这该怎么办呢？

#### 1.5 热更新加载

[参考](https://blog.csdn.net/a2013126370/article/details/88249664)

首先需要一个本地web服务器，可以使用`webpack-dev-server`，安装该模块，并启动

```javascript
npm install webpack-dev-server
npx webpack-dev-server
```

这样就启动了，打开浏览器输入http://localhost:8080/就可以访问当前目录了

当然，我们可以定制启动的路径，端口号等。

在`webpack.config.js`中添加：

```javascript
module.exports = {
  devServer: {//开发服务器的配置
    //端口号配置，默认为8080
    port: 3000,
    //进度条
    progress: true,
    //指定打开浏览器显示的目录，默认为根目录（项目目录）
    contentBase: './dist'
  },
  ...
}
```

这时候就可以通过输入http://localhost:3030/ 直接访问`dist`根目录了，

此时我们在源码中直接修改代码，就可以在页面中实时更新了。

**疑问**

这里我有个疑问，如果之前我没有构建`bundle.js`或`bundle.js.map`文件，该server还能继续使用吗，我在`dist`文件夹下删除其余的文件，启动server，照样可以看到内容，奇怪。

然后我将html文件中对`bundle.js`的引用给注销掉，再次启动server就不行了，说明server自动构建了临时`bundle.js`了。这个构建规则遵从之前的打包配置规则。验证了一下，确实如此。

在`webpack.config.js`中将`bundle.js`名字改为`main.js`，

```javascript
output: {
    // filename: 'bundle.js',
    filename: 'main.js', 
    ...
  }
```

再次启动server，页面就不正常显示了。但是在浏览器中输入http://localhost:3030/main.js，是可以看到代码内容的，说明web服务器中生成了`main.js`文件。



**结论：**说明通过server启动访问的页面，优先访问web服务器内部生成的js文件。



如何让这个html文件中对js文件的引用也自动化起来呢，不至于这么摸不着头脑。

#### 1.6 通过模版html自动配置js文件

使用`html-webpack-plugin`插件

首先安装该插件

```javascript
npm install html-webpack-plugin
```

然后在`webpack.config.js`文件中配置

```javascript
//引入
let HtmlWebpackPlugin = require('html-webpack-plugin');
...
module.exports = {
...
 plugins: [ //数组：放着所有的webpack插件
 	// 配置
    new HtmlWebpackPlugin({
      template: './dist/template.html',
      filename: './dist/index.html'
    })
  ]
}
```

其中template是模版路径，`template.html`是模版内容，内容为

```html
<!doctype html>
<html>
  <head>
    <title>Catirl</title>
  </head>
  <body>
  </body>
</html>
```

而`filename`属性的意思就是所生成HTML文件名，内容只是比`template`所指的HTML文件多引入你之前在`output-filename`中输出的js文件。

**注意：** 通过server生成的html和js文件都没有保存在本地。

至此就完成了热加载的配置工作。