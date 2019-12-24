---
title: drawArrays和drawElements
date: 2019-12-24 22:25:29
tags: webgl
---

在使用`drawArrays`时，我们通常是

```javascript
const bufferPosition = this.gl.createBuffer();
gl.bindBuffer(this.gl.ARRAY_BUFFER, bufferPosition);
gl.bufferData(this.gl.ARRAY_BUFFER,new Float32Array(positions),this.gl.STATIC_DRAW);
```

然后再绑定vao时，指定一下数据就可以了

```javascript
gl.enableVertexAttribArray(positionLocation);
gl.vertexAttribPointer(positionLocation, size, type, normalize, stride, offset);
```

这里说明一下：`ARRAY_BUFFER`绑定的数据属于全局状态。



而使用`drawElements`时，通常配合绑定`ARRAY_BUFFER`，还需要创建索引buffer

```javascript
//创建索引
const indexBuffer = gl.createBuffer();
gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, indexBuffer);
const indices = this.earth.getFillIndices(64,64);
gl.bufferData(
    gl.ELEMENT_ARRAY_BUFFER,
    new Uint16Array(indices),
	gl.STATIC_DRAW
);
const indexCount=indices.length;
```

这里的`ELEMENT_ARRAY_BUFFER`是当前定点数组的一部分

