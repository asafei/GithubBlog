---
title: WebGIS中切片获取和展示的原理
date: 2017-07-14 01:50:02
tags: GIS
---
## 前言
 在北京实习的时候一次跟强哥聊天，做GIS这么长时间了，自己能实现切片数据的展示，就没有那么多的迷茫了。一直以来对切片的展示都是一知半解的，现在静下来好好的梳理一下。

### 一、切片简介
 说起切片，就离不开影像金字塔的概念，这里不予详细讲解。只需要明白一点：切片数据就是将不同比例尺下的地图进行切割，将切割后的切片文件按照一定的规则进行存储（当然切割方式，以及存储规则都各有讲究）。切片的展示就是按照一定的相应的规则取出这些切片，在相应的端口进行拼接展示。切片数据可以放在本地，也可以放在服务端。这里我们抛开一切个地理相关的细节，重点说明切片的展示方案。

### 二、切片数据获取流程
1 获取要展示的地理中心点位置point_screenCenter,即在屏幕中展示地图的中心点坐标；<br />
2 根据中心点坐标，以及屏幕的大小screenSize,计算出屏幕内要呈现地图的范围；<br />
3 根据屏幕边界处的坐标，计算出边界所在的切片行列号<br />
4 计算边界处所在切片的边界坐标，计算切片边界与屏幕边界的偏移量offset<br />
5 获去要求请的所有切片，按照计算的值进行排列展示

#### 2.1 获取展示中心点的地理坐标
所谓展示中心点的坐标，就是当你准备在屏幕上展示地图时，首先你得对地图的范围进行初始化，比如设置地图中心点、地图的坐标系、显示级别等参数。就是这里的地图中心点，我们暂时命名为point_screenCenter，（属性X，Y表示地理坐标）。假设地图的坐标原点为point_original,则该点在x轴和y轴上距离为
```
x轴地理距离：(point_screenCenter.X-point_original.X)
y轴地理距离：(point_screenCenter.Y-point_original.Y)
```
注意：这里我们假设坐标原点在地图图片的左上角，水平向右为x轴正方向，水平向下为y轴正方向，当然不同的地图服务坐标原点和设定规则可能会有区别；
再说一下像素距离，像素距离指的是在一定比例尺的地图图片上的距离间隔，如果要计算像素距离，就需要用到**分辨率（resolution）**的概念了，resolution是指地图图片上一个像素所代表的实际距离，point_screenCenter距离坐标原点的像素距离的计算公式为
```
x轴方向像素距离：(point_screenCenter.X-point_original.X)/resolution
y轴方向像素距离：(point_screenCenter.Y-point_original.Y)/resolution
```

#### 2.2 计算出屏幕内要呈现地图的范围
有了展示中心点（point_screenCenter）和屏幕的大小（screen_size,像素数量表示，如960*640）就可以计算屏幕内能展示的地图范围了，屏幕左上角的点为point_screenLT,它的地理坐标如下：
```
屏幕左上角的x坐标：point_screenLT.X=point_screenCenter.X-(resolution*screen_size.Width)/2
屏幕左上角的y坐标：point_screenLT.Y=point_screenCenter.Y-(resolution*screen_size.Height)/2
```
#### 2.3 计算出边界所在的切片行列号
确定的屏幕左上角能展示的地理位置，就可以根据这个地理位置坐标计算该位置点所位于的具体切片了，给该切片命名为tileLT,则该切片的行列号的计算如下：
```
切片的行号：tileLT.Row=Math.floor((Math.abs(point_screenLT.X-point_original.X))/(resolution*tileSize.Width))
切片的列号：tileLT.Col=Math.floor((Math.abs(point_screenLT.Y-point_original.Y))/(resolution*tileSize.Height))
```
这里用到了Math.floor()函数是为了获取离坐标原点更近的点，因为我们获取的切片范围要大于在屏幕上展示的范围。
找到了行列号，还有一个小细节需要处理，处理这个细节前首先了解一个常识，一般情况下，切片的在展示时是按照切片左上角的位置进行排列的，此时我们只是确定了屏幕左上角位于该切片中，但是该切片的左上角并不一定与屏幕的左上角重合啊，怎么办？，计算该切片左上角的实际地理坐标为：
```
切片左上角x坐标：tileLT.LTX = tileLT.Col*tileSize.Width*resolution+point_original.X
切片左上角y坐标：tileLT.LTY = tileLT.Raw*tileSize.Height*resolution+point_original.Y
```
#### 2.4 计算切片边界与屏幕边界的偏移量offset
有了切片的左上角的地理坐标，和屏幕左上角的地理坐标位置，就可以计算偏移量了，假设其在x轴和y轴上的偏移量为offSetX和offSetY，则偏移值为：
```
切片在x轴偏移像素值：offSetWidth=（point_screenLT.X-tileLT.LTX）/resolution
切片在y轴偏移像素值：offSetHeight=（point_screenLT.Y-tileLT.LTY）/resolution
```
有了这个值就可以将该切片展示在正常的位置了。
#### 2.5 所有要展示切片的获取
最后就可以计算在x和y轴上要请求的切片数量了，获取个数时要用Math.ceil()函数了，原因是的切片的总范围要比屏幕上展示的范围要大一些，这里要获取离坐标原点尽量远的切片。
```
x轴切片数量：tileNumsX=Math.ceil((screenSize.Width + Math.abs(offSetWidth))/tileSize.Width);
y轴切片数量：tileNumsY=Math.ceil((screenSize.Height + Math.abs(offSetHeight))/tileSize.Width);
```

### 三、小结
到现在已经完成了切片的获取工作，这是一个简化的过程。在实际获中不同的切片种类（如TMS、WMS、WMTS等）有不同的切片大小和不同的切片规则，不同的坐标系对应不同单位，还会涉及到一些中间术语比如显示级别、比例尺等。在以后的文章中会介绍。因为我们要做一个自己的***。
