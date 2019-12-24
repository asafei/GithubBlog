---
title: WebGL绑定多纹理
date: 2019-12-24 22:03:48
tags: webgl
---

我们的场景时实时渲染网络切片地图栅格数据。如果每次在渲染阶段进行数据的绑定，必定会降低帧率，什么卡顿。该如何优化呢

### 一、思考

对于非图片数据，我们的处理方式是：

* 将各attribute数据绑定到vbo中；
* 将各vbos绑定给指定的vao中；
* 绘制时，切换vao即可。

但是这个方式对绘制大量图片并不适用，原因是webgl中可绑定的纹理数量一定。这就导致对于有限树数量的纹理单元（textureUnit）我们必须要重复使用。

如何破解这个问题呢？

虽然可绑定的纹理单元数量有限，但是我们可创建的纹理却不受限制。

这样我们就可以继续将各vbo绑定给vao，然后在绘制时，实时将已经创建好纹理绑定给指定的纹理单元即可。

### 二、方案

* 计算要渲染切片的position、textureCoord、index等数据
* 创建各个attribute的buffer
* 创建一个空纹理对象texture，可以是纯色
* 将各buffers绑定给vao，并且将空纹理texture与该vao关联起来
* 异步请求img的实际数据，然后更新纹理的真实数据，并给texture标记需update
* 绘制时，若texture的update为false，则直接绑定空纹理数据绘制；若update为true，则重新绑定纹理数据，进行绘制



### 三、其它

需要注意：

* 请求切片可以放在子线程中执行；
* 这个频繁切换的纹理，要使用可变纹理的方式

### 四、创建空纹理的代码

对于网络图片，首先需要先把图片数据下载下来：

```javascript
// creates a texture info { width: w, height: h, texture: tex }
// The texture will start with 1x1 pixels and be updated
// when the image has loaded
function loadImageAndCreateTextureInfo(url) {
  //创建纹理对像，默认设置填充为蓝色
  var tex = gl.createTexture();
  gl.bindTexture(gl.TEXTURE_2D, tex);
  // Fill the texture with a 1x1 blue pixel.
  gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, 1, 1, 0, gl.RGBA, gl.UNSIGNED_BYTE,
                new Uint8Array([0, 0, 255, 255]));
 
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
 
  var textureInfo = {
    width: 1,   // we don't know the size until it loads
    height: 1,
    texture: tex,
  };
  var img = new Image();
  img.addEventListener('load', function() {
    textureInfo.width = img.width;
    textureInfo.height = img.height;
 
    gl.bindTexture(gl.TEXTURE_2D, textureInfo.texture);
    gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, img);
    gl.generateMipmap(gl.TEXTURE_2D);
  });
  img.src = url;
 
  return textureInfo;
}
```



* 思路1

  先将顶点坐标、纹理坐标、纹理预处理好，可以直接绘制。并且绑定给vao。在绘制时，默认绘制vao，直到真实纹理加载完成，直接再次绑定新的纹理即可。



课外：[<https://webgl2fundamentals.org/webgl/lessons/webgl-texture-units.html>](<https://webgl2fundamentals.org/webgl/lessons/webgl-texture-units.html>)
