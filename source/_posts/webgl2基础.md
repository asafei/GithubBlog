---
title: webgl2基础
date: 2019-12-24 22:46:42
tags: webgl
---
翻译自[https://webgl2fundamentals.org/webgl/lessons/webgl-fundamentals.html](https://webgl2fundamentals.org/webgl/lessons/webgl-fundamentals.html)

这些文章是讲解WebGL2的，如果你对WebGL1.0感兴趣，[点击这](https://webglfundamentals.org/)。WebGL2完全兼容WebGL1，也就是说一旦你开启WebGL2，它会像以往的方式来工作。

WebGL通常被认为是一种3D的apis。人们常说“我要使用WebGL来完成炫酷的3D效果”。实际上WebGL只是一个光栅化引擎。它基于你提供的代码来绘制点、线和三角面。想让WebGL来完成炫酷的任务就要看你掌控点线三角面运作的能力了。

WebGL在计算机的GPU中运行。所以你需要提供可以在GPU中运行的代码。这样的代码有两种函数形式。这两种函数被称为**顶点着色器（vertex shader）**和**片元着色器（fragment shader）**，它们的书写格式严格遵循一种类C/C++的名的[GLSL](https://webgl2fundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html)的语言。这两个着色器成对出现，被称为一个**程序(program)**

顶点着色器的任务是计算顶点的位置。函数可以做基于这些位置的输出（包含点、线、三角面的基元），然后WebGL就可以对这些基元进行光栅化处理。当进行光栅化处理时，就会调用片元着色器了，片元着色器的任务是计算当前绘制的每一个基元的像素颜色值。

几乎整个WebGL的API都是用来设置这两个着色器状态的。任何需要绘制的对象都需要设置一系列的状态，然后通过函数在GPU中执行这对着色器，通过`gl.drawArrays`和`gl.drawElements`函数执行。

任何这些函数可以访问的到的数据都必须提供给GPU。着色器有四种方式来接收数据：

1. Attributes(属性),Buffers(缓冲区)和Vertex Arrays(顶点数组)

   Buffers(缓冲区)是需要上传给GPU的二进制数据的数组。通常Buffers(缓冲区)包含信息：位置、法线，纹理坐标，顶点颜色等，尽管你有权传递任意数据进去。

   Attributes(属性)用来指定如何从Buffers(缓冲区)中提取数据，并且提供给顶点着色器。例如你可以向缓冲区中添加32位浮点数的位置数据，你需要告诉一个指定的Attributes(属性)包括：哪一个缓冲区来提取数据、它要提取的数据是什么类型、在缓存区中从偏移量多少的位置开始提取数据、从一个位置到下一个位置有多少个字节。

   Buffers(缓冲区)不是随机访问的。相反，顶点着色器需要指定没执行的次数。每一次执行时，都需要从指定的Buffers(缓冲区)提取下一个值分配给一个Attributes(属性)。

   Attributes(属性)的状态，会被buffers用到的，以及如何从这些buffers中提取数据，被搜集在一个顶点数组对象中（VAO），

2. Uniforms

   在你执行你的着色器程序之前，Uniforms是有效的全局变量。

3. Textures（纹理）

   纹理是数据数组，你可以在你的着色器程序中随机访问。最常见的传递给纹理的对象是image数据，但是纹理只是数据，可以包含除颜色之外的其它数据。

4. Varyings

   Varyings是顶点着色器向片元着色器传递数据的一种方式。根据要渲染的内容（点、线或三角面），当执行片元着色器时，通过顶点着色器给Varying设置的值会执行内插值计算。



## WebGL Hello World

WebGL只关心两个事情：裁剪空间坐标和颜色。作为一个WebGL程序员,你的工作就是给WebGL提供这两个东西。这些工作通过提供两个着色器来完成，顶点着色器负责提供裁剪空间坐标，片元着色器负责提供颜色。

裁剪空间坐标通常介于（-1,1）的区间，不管canvas的尺寸多大。下面是一个简单的WebGL的例子，以简单的形式展示WebGL。

以顶点着色器开始

```javascript
#version 300 es

//一个attribute是一个顶点着色器的输入
//它会从buffer中接收数据
in vec4 a_position;

//所有的着色器，都有一个main函数
void main(){
    gl_position=a_position;
}
```

当执行的时候，如果整个都是使用JavaScript编写的，而不是GLSL，你需要考虑这样书写

```javascript
var positionBuffer=[
    0,0,0,0,
  	0,0.5,0,0,
  	0.7,0,0,0,
];

var attributes={};
var gl_Psition;

drawArrays(..., offset, count){
  var  stride=4;
  var  size=4;
  
  for(var i=0;i<count;++i){
    //从positionBuffer中复制4个值给a_position attribute
    attributes.a_position=positionBuffer.slice((offset+1)*stide,size);
    runVertexShader();
    ...
    doSomethingWith_gl_Position();
  }
}
```

实际上并没有那么简单，因为`positionBuffer`需要转换成二进制数据（如下所示），所以实际从buffer中获取数据的计算会有一些不同，但是希望通过这样能让你了解到顶点着色器是如何执行的。

下面需要一个片元着色器

```javascript
#version 300es
//片元着色器没有默认的精度，所以我们需要指定一个。medium是很好的默认值
precision mediump float;

//需要给片元着色器声明一个输出
out.vec4 outColor;
void main(){
  	//将输出设置成紫色
    outColor=vec4(1,0,0.5,1);
}
```

上边声明`outColor`作为片元着色器的输出，将`outColor`设置为`1,0,0.5,1`，1表示红色，0表示绿色，0.5表示蓝色，1表示透明度。在WebGL中颜色介于(0,1)之间。

现在已经写好了两个着色器函数，接下来可以使用WebGL执行了

首先需要HTML的canvas元素

```html
<canvas id="c"></canvas>
```

在JavaScript中如下面

```javascript
var canvas=document.getElementById("c");
```

现在可以创建一个WebGL2的渲染背景了

```javascript
var gl=canvas.getContext("webgl2");
if(!gl){
    //
  	...
}
```

现在需要编译这些着色器了，将它们提供给GPU。所以首先我们需要将它们转为字符串。可以以任何正常的方式创建GLSL字符串。可以通过拼接字符串，可以使用、AJAX下载它们，通过将它们放在非JavaScript标签中，或者以如下多行模板的方式

```javascript
var vertexShaderSource='#version 300 es
in vec4 a_position;

void main(){
    gl_Position=a_position;
}
';

var fragmentShaderSource='#version 300 es
precision mediump float;
out vec4 outColor;
void main(){
    outColor=vec4(1,0,0.5,1);
}
';
```

实际上，大多数3D引擎使用各种各样的模板快速生成GLSL着色器。本网站上的例子都没有复杂到在运行时生生GLSL。

> 注意：`#verison 300 es`必须是着色器的首行。它之前不允许有任何命令或空格。`#verison 300 es`告诉WebGL2你要使用WebGL2的名为GLSL ES3.00的着色器语言。如果你不在首行放置它，着色器语言会默认使用WebGL 1.0的GLSL ES 1.00语言，它们之间有很大的不同，缺少很多特性。

下面需要一个函数来创建一个着色器，上传GLSL源码，并且编译着色器。注意饿哦并没有编写任何评论因为函数的命名清晰的展示了发生了什么

```javascript
function createShader(gl, type, source){
    var shader=gl.createShader(type);
  	gl.shaderSource(shader,source);
  	gl.compileShader(shader);
  	var success=gl.getShaderParameter(shader,gl.COMPILE_STATUS);
  	if(success){
        return shader;
    }
  	console.log(gl.getShaderInfoLog(shader));
  	gl.deleteShader(shader);
}
```

现在可以调用函数创建第二个着色器了

```javascript
var vertexShader = createShader(gl,gl.VERTEX_SHADER, vertexShaderSource);
var fragmentShader=createShader(gl,gl.FRAGMENT_SHADER,fragmentShaderSource);
```

然后将这两个着色器连接到程序（program）

```javascript
function createProgram(gl,vertexShader,fragmentShader){
    var program=gl.createProgram();
  	gl.attachShader(program,vertexShader);
  	gl.attachShader(program,fragmentShader);
  	gl.linkProgram(program);
  	var  success=gl.getProgramParameter(program,gl.LINK_STATUS);
  	if(success){
        return program;
    }
  	console.log(gl.getProgramInfoLog(program));
  	gl.deleteProgram(program);
}
```

并且调用它

```javascript
var program=createProgram(gl,vertexShader,fragmentShader);
```

现在我们在GPU中创建一个GLSL程序，并且需要给它提供数据。大部分的WebGL API都是涉及到给供给给GLSL程序提供的数据设置状态。在这个例子中给GLSL 程序提供的输入只有`a_position`，它是一个attribute。首先要做的是给我们创建的程序找到attribute的位置

```javascript
vav positionAttributeLocation=gl.getAttributeLocation(program,"a_position");
```

找到attribute的位置（唯一的位置）是在初始化阶段要完成的事情，而不是在渲染阶段。

Attribute需要从buffers中获取数据，所以需要创建一个buffer

```javascript
var positionBuffer=gl.createBuffer();
```

WebGL允许我们操纵许多全局绑定点中的WebGL资源。可以将绑定点看做是WebGL内部的去哪局变量。首先给绑定点绑定资源，然后所有的其它函数通过绑定点来绑定这些资源。所以我们绑定位置buffer

```javascript
gl.bindBuffer(gl.ARRAY_BUFFER,positionBuffer);
```

现在可以通过绑定点将数据提供给buffer了

```javascript
var positions=[
    0,0,
  	0,0.5,
  	0.7,0,
];
gl.bufferData(gl.ARRAY_BUFFER,new Float32Array(positions),gl.STATIC_DRAW);
```

在这里做了很多事情，首先我们有一个`positions`的JavaScript数组。WebGL需要强类型的数据，所以通过`new Float32Array(positions);`创建一个新的32位浮点型点数据数组，并且从`positions`拷贝数据。通过`gl.bufferData`将数据拷贝到GPU的`positionBuffer`中。它使用了位置buffer，因为我们通过绑定点将它绑定到了`ARRAY_BUFFER`。

最后一个参数`gl.STATIC_DRAW`是对WebGL的一个提示，提示如何使用数据。WebGL可以通过有这些提示来优化一些事情。`gl.STATIC_DRAW`告诉WebGL我们不太可能会改变这些数据。

现在，我们已经将需要告诉attribute如何和提取数据的数据放进了buffer中。我们需要通过调用一个顶点数组来创建一个attribute状态的集合。

```javascript
var vao=gl.createVertexArray();
```

并且需要让它成为当前的顶点数组，所以所有的attribute设置会应用在那个attribute状态集上。

```javascript
gl.bindVertexArray(vao);
```

接下来在顶点数组中设置attribute。首先我们需要开启attribute，这告诉WebGL我们想要从buffer中提取数据。如果我们没有开启attribute那么attribute就会是一个常量值。

```javascript
gl.enableVertexAttribArray(positionAttributeLocation);
```

然后我们需要指定如何将数据提取出来

```javascript
var size=4;
var type=gl.FLOAT;
var normalize=false;
var stride=0;
var offset=0;
gl.vertexAttribPointer(positionAttributeLocation,size,type,normalize,stride,offset);
```

`gl.vertexAttribPointer`的作用是将当前的`ARRAY_BUFFER`绑定给attribute。也就是说现在attribute绑定给了`positionBuffer`。这意味着我们可以绑定其它东西给`ARRAY_BUFFER`绑定点。attribute会集训使用`positionBuffer`

注意从我们的GLSL 顶点着色器视图的点，`a_position`attrubute是一个`vec4`

```javascript
in vec4 a_position;
```

`vec4`是一个4个浮点值。在JavaScript中你可以把它看做是`a_position={x:0,y:0,z:0,w:0}`。在设置`size=2`之前，attribute默认是`0,0,0,1`所以这个attribute就会从我们的buffer中获取前两个值（x和y）。z和w分别是是默认的0和1。

在绘制之前，需要重置canvas的尺寸来匹配展示尺寸。canvas们就像图像一样拥有两个尺寸。像素的数量通常位于它的内部，和它的展示尺寸是相分离的。CSS决定了canvas的展示尺寸。你应该优先选择使用css改变canvas的尺寸，因为这是最便利的方法。

为了让canvas的内部像素数量和展示尺寸相匹配。我使用了一个[辅助函数](https://webgl2fundamentals.org/webgl/lessons/webgl-resizing-the-canvas.html)。

几乎所有的例子的canvas的尺寸都是400*300像素，它会在自己的窗口上运行。如果位于iframe内部，那么它就会拉伸填充整个可用区域，就像位于这个页面之上一样。通过让CSS控制尺寸，然后调整匹配，可以轻松的处理这两种情况。

```javascript
webglUtils.resizeCanvasToDisplaySize(gl.canvas);
```

我们需要告诉WebGL如何进行剪裁空间值的转换，需要设置`gl_Position`返还给像素，通常被称为屏幕空间。为了做到它，可以调用`gl.viewport`并且给它传递canvas的当前尺寸。

```javascript
gl.viewport(0,0,gl.canvas.width,gl.canvas.height);
```

它告诉WebGL将（-1，1）的剪裁空间映射到（0，gl.canvas.width）给x轴，（0,gl.canvas.height）给y轴

我们清空画布。`0,0,0,0`分别代表红、绿、蓝、透明度。所以在这个例子中，我们让画布透明

```glsl
gl.clearColor(0,0,0,0,);
gl.clear(gl.COLOR_BUFFER_BIT);
```

接下来我们需要告诉WebGL执行哪一个着色器程序

```JavaScript
gl.useProgram(program);
```

然后我们需要告诉它需要用到哪一个buffers集，并且如何从这些buffers中提取数据给attributes

```javascript
gl.bindVertexArray(vao);
```

最后需要请求WebGL来执行GLSL程序

```javascript
var primitiveType=gl.TRIANGLES;
var offset=0;
var count=3;
gl.drawArrays(primitiveType,offset,count);
```

因为count是3，所以会执行顶点着色器3次。第一次中顶点着色器attribute的`a_position.x`和`a_position.y`会被设置成positionBuffer的前两个值；第二次中`a_position.xy`会别设置成随后的两个值；最后一次中会别设置成最后两个值。

因为我们将`primitiveType`设置为了`gl.TRIANGLES`，每次我们的顶点着色器都执行三次，WebGL会基于我们设置给`gl.Position`de 三个值来绘制三角形。无论canvas的尺寸是多少，在剪裁空间中的这些值在每个方向上都是（-1,1）的区间。 

由于顶点着色器只是简单的将positionBuffer中的值拷贝到`gl_Position`，三角形会在剪裁空间坐标中绘制。

```javascript
0,0,
0,0.5,
0.7,0
```

如果canvas的尺寸正好是400*300，从剪裁空间向屏幕坐标的转化会如下：

| clip space |      | screen space |
| ---------- | ---- | ------------ |
| 0,0        | ---> | 200,150      |
| 0,0.5      | ---> | 200,225      |
| 0.7,0      | ---> | 340,150      |

WebGL现在就回渲染这个三角形面了。对于每一个要绘制的像素，WebGL都会调用片元着色器。我们的片元着色器仅仅设置了`outColor`为`1,0,0.5,1`。由于Canvas在每个通道都是8位的，意味着WebGL可以将`[255,0,127,255]`写进canvas、
