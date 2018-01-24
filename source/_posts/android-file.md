---
title: Android数据持久化---文件存储
date: 2018-01-24 20:54:03
tags: Android
---
文件存储不对数据做任何格式化的处理，原封不动的保存数据，适合存储一些简单的文本数据，创建的文件位于`/data/data/<package-name>/fiele/`目录下

###  一、存储数据

Context类提供了`openFileOutput()`方法，可以将数据保存。它接收两个参数

* 第一个参数： 文件名

* 第二个参数： 文件的操作模式

  > 文件的操作模式主要有两种，分别是`MODE_PRIVATE`和`MODE_APPEND`,其中
  >
  > * MODE_PRIVATE：指定同样文件名的时候，会覆盖原文件的内容；
  > * MODE_APPEND：指定同样文件名的时候，会往文件中追加内容；
  >
  > 还有其它的模式，如：MODE_WORLD_READABLEH和MODE_WORLD_WRITEABLE表示允许其它应用对我们程序中的文件进行读写操作，存在安全漏洞，在Android4.2版本中被废弃了。

保存文件的步骤如何呢：

1. 通过`openFileOutput()`方法得到一个`FileOutputStream`对象；
2. 通过`FileOutputStream`对象构建一个`OutputStreamWriter`对象；
3. 通过`OutputStreamWriter`对象构建一个`BufferedWriter`对象；
4. 通过`BufferedWriter`对象就可以将文本写入文件了

具体代码如下：

```Java
public void save(){
    String data="我是要保存的内容";
  	FileOutputStream out=null;
  	OutputStreamWriter wsw=null;
  	BufferedWriter writer=null;
  	try{
        out=openFileOutput("data",Context.MODE_PRIVATE);
      	osw=new OutputStreamWriter(out);
      	writer=new BufferedWriter(osw);
      	writer.writer(data);
    }catch(IOException e){
        //...
    }finally{
        try{
            if(writer!=null){
                writer.close();
            }
        }catch(IOException e){
            //..
        }
    }
}
```

这时就可以在`onDestroy()`方法中将相应的数据保存起来了，以便再次启动时展示。

如何查看我们已经保存的数据呢？在Android studio的Tools-Android-Android Device Monitor，就可以进入File Explorer标签页，从而找到`/data/data/<package-name>/fiele/`下相应的文件了

### 二、读取数据

如何读取保存在`/data/data/<package-name>/fiele/`的文件数据呢，Context有一个`openFileInput()`方法，按照下面步骤即可：

1. 通过`openFileInput()`方法得到一个`FileInputStream`对象；
2. 通过`FileInputStream`对象构建一个`InputStreamWriter`对象；
3. 通过`InputStreamWriter`对象构建一个`BufferedReader`对象；
4. 通过`BufferedReader`对象就读取文件了

代码如下：

```Java
public void load(){
  	FileInputStream in=null;
  	InputStreamWriter isw=null;
  	BufferedReader reader=null;
  	StringBuilder content=new StringBuilder();
  	try{
        in=openFileInput("data");
      	isw=new InputStreamWriter(in);
      	reader=new BufferedReader(isw);
      	String line="";
      	while((line=reader.readerLine())!=null){
            content.append(line);
        }
    }catch(IOException e){
        //...
    }finally{
        try{
            if(reader!=null){
                reader.close();
            }
        }catch(IOException e){
            //..
        }
    }
}
```

这时就可以在`onCreate()`方法中将相应的数据读取起来进行展示了。
