---
title: WebGL跨域图片
date: 2019-12-24 21:56:35
tags: webgl
---

##### 一、问题

通常我们加载图片时使用这种方式：

```javascript
const url='https://t6.tianditu.gov.cn/DataServer?T=img_w&x=2&y=1&l=3&tk=8971e4c7b3640d506c2dc111221af6a0';//天地图的一张切片
const image = new Image();
image.addEventListener('load', function() {
    //拿到image，做纹理数据
    ...
});
image.src=url;
```

但是当图片为跨域图像时，就出现问题了。常规浏览器报错

```javascript
Access to image at 'https://t6.tianditu.gov.cn/DataServer?T=img_w&x=2&y=1&l=3&tk=8971e4c7b3640d506c2dc111221af6a0' from origin 'http://localhost:3030' has been blocked by CORS policy: The 'Access-Control-Allow-Origin' header has a value 'https://map.tianditu.gov.cn' that is not equal to the supplied origin.
```

接下来将浏览器设置为可跨域模式（方法自行百度），此时控制台是不报错了，但是也没有任何图像展示出来。

？？？这他娘的不是说好的img标签允许跨域的吗？

##### 二、探索

通过添加错误的监听，可以看到image没有进入到`load`回调中，而是进入到了`error`的回调中。

```javascript
image.addEventListener('error', ()=>{
	console.log("error");
});
```

直接使用`<img src="private.jpg"> `是没什么问题的，因为图像尽管被浏览器显示， 便签对象并不能获取图像的内部数据。而Webgl使用的就是image的内部数据，就获取不到了。

有什么方法可以获取到跨域图像内部的数据呢？

##### 三、解决方式

对于访问跨域资源必须同时满足两方面的许可：

* 1 跨域站点允许跨域访问（需要服务器设置）
* 2 本地可以使用跨域的资源的内容（例如设置浏览器可跨域访问）

下面所讲的都只是解决2的问题。

###### 3.1 设置crossOrigin

最简单的方式

```javascript
const url='';
const image = new Image();
image.crossOrigin = "anonymous"; //允许
image.addEventListener('load', function() {
    //拿到image，做纹理数据
    ...
});
image.src=url;
```

crossOrigin的值有三个可选择：

* undefined：默认值，表示不需要请求许可；
* anonymous：表示请求许可，但是不发送任何信息；
* use-credentials：表示发送cookies和其它可能需要的信息，服务器会根据这些信息决定是否授予许可。

**注意：设置为其他任意值则相当于 "anonymous"**

设置完`crossOrigin`属性后，浏览器从服务器请求图像时，对于不同域名，会请求CROS许可。由于请求需求需要2个http请求，资源消耗多一些。所以同域的不需要设置请求许可，只需对跨域的资源的img标签或canvas2d设置`crossDomain`属性，这样就不会使请求变慢了。

可以添加这个一个适配函数：

```javascript
function requestCORSIfNotSameOrigin(img, url) {
  if ((new URL(url)).origin !== window.location.origin) {
    img.crossOrigin = "";//相当于anonymous
  }
}
```

###### 3.2 使用canvas

第二种方法就是先将图片绘制在画布上，然后读取画布上的数据。

```html
<canvas id="canvas"></canvas>
<div style="display:none;">
  <img id="source"
       src="https://mdn.mozillademos.org/files/5397/rhino.jpg"
       width="300" height="227">
</div>
```

```javascript
const canvas=document.getElementById('canvas');
const ctx=canvas.getContext('2d');
const image = document.getElementById('source');
image.addEventListener('load', e => {
  	ctx.drawImage(image, 33, 71, 104, 124, 21, 20, 87, 104);
    //ctx.drawImage(someImg, 0, 0);
	//const data = ctx.getImageData(0, 0, width, heigh);
});

```

在WebGL中 `gl.readPixels` 和 `ctx.getImageData` 是相似的， 所以你可能以为把这个接口封闭就好了，但事实是即使不能直接获取像素值， 也可以创建一个基于图像颜色的着色器，虽然效率低但是可以等同于获取到了图像信息。

参考：

[<https://webgl2fundamentals.org/webgl/lessons/webgl-cors-permission.html>](<https://webgl2fundamentals.org/webgl/lessons/webgl-cors-permission.html>)

[<https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage>](<https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/drawImage>)

