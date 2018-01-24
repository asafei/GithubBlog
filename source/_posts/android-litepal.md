---
title: Android数据持久化---LitePal
date: 2018-01-24 20:58:43
tags: Android
---
记得很久之前就使用过一些第三方工具进行sqlite数据库操作，只觉得分外简单。入职以来进行了SpringBoot等的训练，发现它们所做的工作如此之像，一精则百通吧，现在看这本书，笔记还是继续做吧，梳理一下才是自己的东西。

### 一、LitePal创建数据库

##### 1.1 添加依赖

在相应module的`build.gradle`的dependencies中添加依赖引用：

```groovy
dependencies{
  	//...
  	compile 'org.litepal.android:core:1.4.1'  
}
```

##### 1.2 添加配置文件

在`src/main`目录下新建一个`assets`的目录，然后在下面新建一个litepal.xml的文件，添加内容如下：

其中

```xml
<?xml version="1.0" encoding="utf-8">
<litepal>
	<!--数据库名称-->
	<dbname value="BookStore"></dbname>
	<!--数据库版本-->
	<version value="1"></value>
	
	<!--数据库的表对应的类-->
	<list>
	</list>
</litepal>
```

##### 1.3 配置AndroidManifest.xml文件

 在AndroidManifest.xml中的application标签中添加一个name属性

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.example.litepaltest">
  <application
  		android:name="org.litepal.LitePalApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportRtl="true"
        android:theme="@style/AppTheme">
    	...
    </application>
</manifest>
```

##### 1.4 创建对象类

LitePal采取的是对象关系映射（ORM）模式，将面向对象语言与面向关系的数据库之间建立一种映射关系。我们只需要创建一个实体类，LitePal就可以将它转变成数据库中的表。

创建一个Book的实体类：

Book类会对应数据库中的Book表，类中的字段对应表中的列。这就是关系对象映射。

```Java
public class Book{
    private int id;
  	private String author;
  	private double price;
  	private int pages;
  	private String name;
  
  	public int getId(){
        return id;
    }
  	public void setId(int id){
        this.id=id
    }
  	...
      
}
```

##### 1.5 修改litepal.xml文件

刚才创建的litepal.xml文件中有一个list的标签，里面就是放对应的实体类，可以通过map标签建立与Litepal的关联，注意mapping中的class必须是实体类的完整类名

```xml
<?xml version="1.0" encoding="utf-8">
<litepal>
	<!--数据库名称-->
	<dbname value="BookStore"></dbname>
	<!--数据库版本-->
	<version value="1"></value>
	
	<!--数据库的表对应的类-->
	<list>
  <mapping class="com.example.litepaltest.Book"></mapping>
	</list>
</litepal>
```

##### 1.6 执行数据库的创建

和直接使用SQLiteOpenHelper一样，只要获取数据库实例就会自动执行数据库和表的创建工作。只不过这里使用LitePal自带的获取方法，更加简单。

```Java
btn_create.setOnClickListener(new View.OnClickListener(){
    @Override
  	public void onClick(View v){
        LitePal.getDatabase()
    }
});
```

这样点击btn_create按钮就会执行数据库和Book表的创建工作了。

### 二、升级数据库

如果想升级数据库，比如给Book表增加一个出版社（press）的列，或者想添加一个新的表，怎么做呢?非常简单

（1）只需要给Book实体类添加新的press字段，并设置getter和setter方法

```Java
public class Book{
    ...
    private String press;
  
  	public String getPress(){
        return press;
    }
  	public void setPress(String press){
        this.press=press
    }
  	...
      
}
```

（2）新建一个实体类Category

```Java
public class Category{
    private int id;
  	private String categoryName;
  	private int categoryCode;
  	
  	//getter和setter方法
  	...
      
}
```

（3）修改litepal.xml文件，修改里面数据库版本，添加新增的Category类mapping

```xml
<?xml version="1.0" encoding="utf-8">
<litepal>
	<!--数据库名称-->
	<dbname value="BookStore"></dbname>
	<!--数据库版本-->
	<version value="2"></value>
	
	<!--数据库的表对应的类-->
	<list>
  <mapping class="com.example.litepaltest.Book"></mapping>
	<mapping class="com.example.litepaltest.Category"></mapping>
	</list>
</litepal>
```

这时重新运行程序，单价btn_create按钮，就完成了数据库的更新操作



### 三、操作数据库

##### 3.1 添加数据

有了LitePal就不再需要ContentValue对象了，我们可以执行使用对象实体类来添加数据，只是需要修改一些实体类文件，让实体类继承自LitePal中的DataSupport类

```Java
public class Book extends DataSupport{
    ...
}
```

添加数据，只需要创建实体类，设置参数，然后调用实体类的`save()`方法即可，当然`save()`方法是DataSupport内部的方法

```Java
btn_addBookData.setOnClickListener(new View.OnClickListener(){
    @Override
  	public void onClick(View v){
        Book b=new Book();
      	b.setName("书名");
      	b.setAuthor("作者");
      	b.setPages(300);
      	b.setPrice(100.00);
      	b.setPress("出版社");
      	//保存
      	b.save();
    }
});
```

这时候数据就添加成功了，可以通过adb进行查看

##### 3.2 更新数据

更新数据，可以对已存储的对象重新复制，然后重新调用`save()`方法，就完成了更新操作。

那么如何确定对象是否是已存储对象呢？可以调用`model.isSave()`方法来判断，只有在两种情况下该方法才会返回true

* 一种情况是已经调用过model.save()方法去添加数据了，此时model是已存储的对象
* 另一种是model是通过LitePal的api查询出来的对象，也会被认为是已存储的对象

实例代码

```Java
btn_updateBookData.setOnClickListener(new View.OnClickListener(){
    @Override
  	public void onClick(View v){
        Book b=new Book();
      	b.setName("书名");
      	b.setAuthor("作者");
      	b.setPages(300);
      	b.setPrice(100.00);
      	b.setPress("出版社");
      	//保存
      	b.save();
      
      	//修改数据
      	b.setPrice(99.00);
      	b.save();
    }
});
```

此时数据库新增了一条数据，但是书的价格不是原先的100.00，而是99.00了。

但是这种更新的方式限制性比较大，如何快速对多条数据进行更新的，可以使用`updateAll()`方法，这个方法与SQLiteDatabase中的update方法where参数有些类似，但是更加简洁，同样参数若不指定表示更新所有

```Java
btn_updateBooks.setOnClickListener(new View.OnClickListener(){
    @Override
  	public void onClick(View v){
        Book b=new Book();
      
      	//修改数据
      	b.setPrice(9.99);
      	b.setPress("大白出版社");
      	b.updateAll("name = ? and author = ? ", "红楼梦","曹雪芹");
    }
});
```

此时就会把所有作者为曹雪芹并且名字为红楼梦的数的价格改为9.99，出版社改为大白出版社。

需要注意的是，如果想把某一字段改为默认值，是不可以使用上面的set方法的。比如Java中默认的int类型值为0，boolean值为false，String值为null。

如果想把pages设为0，通过b.setPages(0)，执行updateAll()操纵是不会更新数据的，对于想要将数据更改为默认值的操作，LitePal提供了`setToDefault()`的方法，将要更改的字段作为参数传进去即可

```Java
Book n=new Book();
b.setToDefault("pages");
b.updateAll();
```

这样就能将相应的字段更新为默认的数值了。

##### 3.3 删除数据

使用LitePal删除吧数据主要两种：

* 直接调用已存储对象的`delete()`方法即可，对于LitePal提供的api查询出来的对象都可以使用该方法
* 另一种方法是使用DataSupport的`deleteAll()`方法，参数第一个是实体类，后面的同where语法

方法二的代码，删除Book表中所有价格小于15的数据。

```Java
btn_deleteData.setOnClickListener(new View.OnClickListener(){
    @Override
  	public void onClick(View v){
        DataSupport.deleteAll(Book.clas,"price < ?" , "15");
    }
});
```

当然如果不指定约束条件，就是删除所有的记录。

##### 3.4 查询数据

查询操作同样是DataSupport类提供的，常用的有

* findAll() ：查询所有数据
* findFirst() ：查询首条记录
* findLast() ： 查询末条记录
* select().find() ： 查询指定列的数据
* where().find() ： 查询指定约束条件的数据
* order().find() ： 指定结果的排序方式
* limit().find() ： 指定查询结果的数量
* limit().offset().find() ： 指定查询结果的偏移量

当然可以任意进行组合

代码如下：

```Java
//查询所有数据
List<Book> books=DataSupport.findAll(Book.class);

//查询第一条数据
Book b=DataSupport.findFirst(Book.class);

//查询最后一条数据
Book b=DataSupport.findlast(Book.class);

//查询指定列的数据
Book b=DataSupport.select("name","author").find(Book.class);

//查询满足约束条件的数据
Book b=DataSupport.where("pages > ?","100").find(Book.class);

//查询结果按指定方式排序
Book b=DataSupport.order("price desc").find(Book.class);

//指定查询结果的数量,返回第1、2、3条数据
Book b=DataSupport.limit(3).find(Book.class);

//指定查询结果的偏移量,返回第2、3、4条数据
Book b=DataSupport.limit(3).offset(1).find(Book.class);
```



##### 3.5 使用原生sql语句

DataSupport同样支持原生SQL语句的操作，提供了`findBySQL()`方法

```Java
String sql="select * from Book where pages > ? and price < ?";
Cursor cursor=DataSupport.findBySQL(sql,"100","100.00");
```
