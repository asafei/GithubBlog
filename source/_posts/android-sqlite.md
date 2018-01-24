---
title: Android数据持久化---SQLite
date: 2018-01-24 20:57:25
tags: Android
---
无论是文件存储还是sharedPreference存储，都只能存储简单的数据或键值对。如果要存储复杂的关系型数据时该怎么办呢？SQLite就派上用场了，它是内置在Android中的一个轻量级的关系型数据库，且不用设置用户名和密码就可以使用，功能强大。

### 一、创建数据库

为了方便的管理数据库，android专门提供了一个SQLiteOpenHelper帮助类，用它可以完成数据库的创建和升级。

SQLiteOpenHelper是一个抽象类，在使用时需要我们创建一个类去继承它，并实现它的两个抽象方法：

* `onCreate()`： 首次创建数据库时会调用这个方法
* `onUpgrade() `：对数据库进行更新时会调用该方法

另外SQLiteOpenHelper还有两个方法用来打开或创建一个现有的数据库，如果数据库已存在则打开，不存在则新建数据库。并且返回一个可对数据库进行读写操作的对象。

* `getReadableDatabase()`： 当不可写入时，只读方式打开
* `getWritableDatabase()`： 当不可写入时，出现异常

SQLiteOpenHelper有两个参数可供重写，参数较少的构造方法需要提供四个参数：

* Context：只有它才能进行数据库操作
* 数据库名：指定要创建的数据库名字
* 自定义的Curfor：运行查询数据时返回的一个自定义的Cursor，一般传入null即可
* 当前数据库版本号：用来对把数据库升级操作

创建好的数据库存放在`/data/data/<package-name>/databases/`目录下，一般使用adb进行查看，具体后边会有介绍。

如何创建一个数据库，并创建表呢？

##### 1.1 创建一个类继承自SQLiteOpenHelper

创建一个BookStore的数据库，并且在里面创建一个Book的表，其中：

* integer表示整形
* real表示浮点型
* text表示文本型
* blob表示二进制类型

primary key表示将id设为主键，autoincrement表示该列自增，这样的化以后增加数据时就不用传递这一列的参数了

```Java
public class MyDatabaseHelper extends SQLiteOpenHelper{
  public static final String CREATE_BOOK="create table Book(id integer primary key autoincrement,author text,price real,pages integer,name text)"
  private Context mContext;
  public MyDatabaseHelper(Context context,String name,SQLiteDatabase.CursorFactory factory, int version){
    super(context,name,factory,version);
    mContext=context;
  }
  
  //首次创建数据库时执行的语句
  @Override
  public void onCreate(SQLiteDatabse db){
     //执行创建表的操作 
    db.execSQL(CREATE_BOOK);
  }
  
  //升级数据库时执行的语句
  @Override
  public void onUpgrade(SQLiteDatabse db,int oldVersion,int newVersion){
      //...
  }
  
}
```

然后在适当的位置就可以调用该类进行数据库和表的创建了，可以给一个按钮设置该监听

```Java
//创建一个名为BookStore.db的数据库
MyDatabaseHelper mdb=new MyDatabaseHelper(this,"BookStore.db",null,1);
//如果点击该按钮时，不存在名为BookStore.db的数据库，就会执行mdb内部的onCreate方法进行创建Book表的操作
btn_catete.setOnClickListener(new View.OnClickListener(){
    @Override
  	public void onClick(View v){
        mdb.getWritableDatabase();
    }
});
```

##### 1.2 数据库的升级

现在数据库下已经有一个Book表了，如果要对数据库进行升级操作（比如增加一个），该怎么办呢，这就要用到`onUpgrade()`方法了，想让mdb执行这个方法就需要在创建（开发）数据库时传递一个更高的版本号。

在`onUpgrade()`方法中添加相关的代码

```Java
public class MyDatabaseHelper extends SQLiteOpenHelper{
  ...
  public static final String CREATE_CATEGORY="create table Category(id integer primary key autoincrement,category_name text,category_code integer)"
  
  //首次创建数据库时执行的语句
  @Override
  public void onCreate(SQLiteDatabse db){
     //执行创建表的操作 
    db.execSQL(CREATE_BOOK);
    db.execSQL(CREATE_CATEGORY);
  }
  
  //升级数据库时执行的语句
  @Override
  public void onUpgrade(SQLiteDatabse db,int oldVersion,int newVersion){
      	//如果Book和Category表已经存在，则删除它们
    	db.execSQL("drop table if exists Book");
    	db.execSQL("drop table if exists Category");
    	onCreate(db);
  }
  
}
```

然后在适当位置给某按钮添加监听，

```Java
//将版本号提高为2，
MyDatabaseHelper mdb=new MyDatabaseHelper(this,"BookStore.db",null,2);
//如果此时已经存在BookStore.db的数据库，就会执行mdb内部的onCUpgrade方法
btn_upgrade.setOnClickListener(new View.OnClickListener(){
    @Override
  	public void onClick(View v){
        mdb.getWritableDatabase();
    }
});
```

这时，BookStore数据库下就有两个表了。当然具体升级的思路多种多样，不再多述。

##### 1.3 添加数据

可以通过SQLiteOpenHelper的`getReadableDatabase()`或`getWritableDatabase()`

方法获取到一个SQLiteDatabse对象，可以借助于它来实现CRUD操作（增删改查）。对于插入操作SQLiteDatabse对象提供了`insert()`方法，接收三个参数：

* 第一个参数是表名：
* 第二个参数：直接传null即可。一般不用，在未指定添加数据的情况下可为空的列赋值NULL。
* 第三个参数是ContentValues对象：可通过put方法向ContentValues中添加数据

实例如下：

```Java
btn_addBook.setOnClickListener(new View.OnClickListener(){
    @Override
  	public void onClick(View v){
        mdb.getWritableDatabase();
      
      	//
      	ContentValues values=new ContentValues();
      	values.put("name","西游记");
      	values.put("author","吴承恩");
      	values.put("pages",10000);
      	values.put("price",100.00);
      	//执行添加操作
      	mdb.insert("Book",null,values);
      
      	values.clear();//清空values
      	values.put("name","红楼梦");
      	values.put("author","曹雪芹");
      	values.put("pages",20000);
      	values.put("price",200.00);
      	//插入第二条数据
      	mdb.insert("Book",null,values);
      	
    }
});
```

在创建Book表的时候还有一个id的列，在这里没有赋值，因为它是自增长的，会在入库时自动生成，就不需要手动赋值了。



##### 1.4 更新数据

SQLiteDatabse对象提供了`update()`方法来更新数据，它接收四个参数

* 第一个参数是表名：
* 第二个参数是ContentValues对象：
* 第三个参数是约束行：即sql语句where后面的部分
* 第四个参数是数据：与第三个参数对应的具体数据（为where中的占位符提供的具体的值），共同起到约束行的作用

示例如下：如果不传第三个和第四的数据的话，则会更新所有行数据中相关的列的值。

其中`?`是一个占位符，具体内容由第四个参数来填充。

```Java
btn_updateBook.setOnClickListener(new View.OnClickListener(){
    @Override
  	public void onClick(View v){
        mdb.getWritableDatabase();
      
      	ContentValues values=new ContentValues();
      	//要更新的数据列
      	values.put("price",150.00);
      	//将书名为西游记的数的价格改为150.00
      	mdb.update("Book",values,"name= ? ",new String[]{"西游记"});
      	
    }
});
```

##### 1.5 删除数据

SQLiteDatabse对象提供了`delete()`方法来删除数据，它接收三个参数

- 第一个参数是表名：
- 第二个参数是约束行：即sql语句where后面的部分
- 第三个参数是数据：与第二个参数对应的具体数据，共同起到约束行的作用

如果不指定第二和第三个参数的话，默认删除所有行数据

```Java
btn_deleteBookData.setOnClickListener(new View.OnClickListener(){
    @Override
  	public void onClick(View v){
        mdb.getWritableDatabase();
      
      	ContentValues values=new ContentValues();
      	//删除页数大于10000页的书籍
      	mdb.delete("Book","page > ? ",new String[]{"10000"});
      	
    }
});
```

#### 1.6 查询数据

查询数据是最难的，SQLiteDatabse对象提供了`query()`方法来查询数据，最短的一个重载方法也有7个参数

* 第一个：table -- 表名
* 第二个：colunms -- 查询哪几列
* 第三个：selections -- 约束查询的行（指定where的约束条件）
* 第四个：selectionArgs -- 约束查询的行的数值（为where中的占位符提供的具体的值）
* 第五个：groupBy -- 需要group by的列
* 第六个：having -- group by之后需要近一步过滤的数据
* 第七个：orderBy -- 指定查询结果的排序方式

示例代码如下：后边的参数全为null，表示查询表中的所有数据

```Java
btn_queryBook.setOnClickListener(new View.OnClickListener(){
    @Override
  	public void onClick(View v){
        mdb.getWritableDatabase();
     	Cursor cursor=mdb.query("Book",null,null,null,null,null,null);
      	if(cursor.moveToFirst()){
            do{
                String name=cursor.getString(cursor.getColumnIndex("name"));
              	String author=cursor.getString(cursor.getColumnIndex("author"));
             	int pages=cursor.getInt(cursor.getColumnIndex("pages"));
              	doubole price=cursor.getDouble(cursor.getColumnIndex("price"));
              	//...
            }while(cursor.moveToNext())
        }
        //注意不要忘了关闭cursor
      	cursor.close();
    }
});
```



### 二、使用SQL语句操作数据库

上边介绍的是Android提供的API操作数据库，还可以通过常规的SQL语句进行数据库的操作

eg:

```Java
//插入数据
mdb.execSQL("insert into Book (name,author,pages,price) valuse(?,?,?,?)",new String[]{"三国演义","佚名","800","50.00"});

//更新数据，三国演义的价格改为600.00
mdb.execSQL("update Book set price = ? where name = ? ",new String[]{"600.00","三国演义","800","50.00"});

//删除数据
mdb.execSQL("delete from Book where pages > ? ",new String[]{"800"});


//查询数据
mdb.rawQuery("select * from Book",null);



```

### 三、使用adb查看数据库

上边提到了使用adb可以查看SQLite创建的数据库。

adb存放在sdk的platform-tools的文件夹下，可以用来调试连接在电脑上的手机或模拟器。如果想通过命令行使用它，可以配置环境变量：

* window ： 在Path中 将platform-tools的目录配置进去
* Linux和Mac： 在home路径下的.bashenjain中，将platform-tools的目录配置进去

配置好之后在命令行输入`adb shell`就可以进入设备的控制台了，`#`表示超级管理员，`$`表示普通用户，可以通过`su`命令切换到超级管理员；

使用`cd /data/data/<package-name>/databases/ `目录下，就可以ls里面的所有文件了

* 打开数据库： `sqlite3 BookStore.db`
* 查看表目录：`.table`
* 查看建表语句：`.schema`
* 操作数据库：SQL语句，如`select * from Book `
* 退出数据库编辑：`.exit`或`.quit`
