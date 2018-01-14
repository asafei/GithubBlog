---
title: indexedDB的使用
date: 2018-01-14 20:07:28
tags: JavaScript
---
## 简介
IndexedDB是Indexed Database API的简称，是一套方便保存和读取JavaScript对象、同时支持查询和搜索的API。

## 一、数据库的创建和存储
### 1.1 IndexedDB对象的声明
IndexDB本作为API宿主的全局对象，但是不同浏览器常常有一些不同命名：
* IE10: msIndexedDB
* Firfox4: mozIndexedDB
* Chrome: webkitIndexedDB
所以对IndexDB的声明需要兼顾现状：

```JavaScript
var indexDB = window.indexedDB || window.msIndexedDB || window.mozIndexedDB || window.webkitIndexedDB
```

### 1.2 数据库的连接
IndexedDB与传统数据库的差别是：它使用对象保存数据。一个IndexedDB数据库可以看做时相同命名空间下的对象的集合。
对IndexedDB数据库的连接可以使用`open()`方法，
* 在回调函数中的`event.target`指向的是`request`:
* `event.target.result`找那个有一个数据库的实例对象(IDBDatabase)

```JavaScript
var request , database;
// 打开一个数据，返回一个IDBRequest对象
request=indexedDB.open("admin");
// 打开数据库发生异常时的回调函数
request.onerrro=function(event){
    alert("打开时发生了异常："+event.target.errorCode);
}
// 打开数据库成功时的回调函数
request.onsuccess=function(event){
    // 这里event.target指向的是request对象
    database=event.target.result;
}
```
### 1.3 IndexedDB版本号的设置
在Web应用中，随着对数据库结构的更新和修改，肯定会出现多个不同版本的数据库。默认情况下IndexedDB没有版本号，最好设置一下，可以使用`setVersion()`方法，以字符串的形式传递版本号，为了检验版本号是否设置成功，需要设置监听函数：

```JavaScript
if(database.version!="1.0"){
    request=database.setVersion("1.0");
    request.onerror=function(event){
        alert("打开时发生了异常："+event.target.errorCode);
    }
    request.onsuccess=function(event){
        alert("数据库初始化完成，数据库名字为："+database.name+", 版本为："+database.version);
    }
}else{
    alert("数据库已经存在，数据库名字为："+database.name+", 版本为："+database.version);
}
```

### 1.4 对象存储空间的创建（建表）
先额外看一个最简单的例子，有个简单的了解：

```JavaScript
// 初始化数据库
var dbParams=new Object();
dbParams.db_name="YYH";
dbParams.db_version="1";
dbParams.db_store_name="Test";
dbObject.init(dbParams);

// 插入三条数据
var v01=dbObject.put({title:"阿里巴巴与四十个大盗",author:"alibaba",isbn:123456},1);
var v02=dbObject.put({title:"百年身，已往回首",author:"baidu",isbn:234567},2);
var v03=dbObject.put({title:"Chrome浏览器无敌",author:"chromegroup",isbn:345678},3);
```

在数据库连接成功之后，就要创建对象存储空间（建表）了。正如前面所说，IndexedDB保存的是系列对象的集合，加入我们保存记录是用户对象如下：

```JavaScript
var user={
    username:"007",
    firstName:"Yang",
    lastName:"Afei",
    password:"iampassword"
}
```
但是建表一定要有主键，我们将username设为主键，在IndexedDB中怎么做呢,可以使用`database`中的`createObjectStore()`方法，

```JavaScript
// 第一个参数相当于表名，第二个参数执行表的主键
var store = database.createObjectStore("users", {keyPath:"username"});
```
这时，数据表已经创建好了，接下来就该插入数据了。

### 1.5 数据的插入
有了`store`这个对存储空间的引用，就可以进行数据的插入了。有两个方法进行数据的插入,它们都接收要保存的对象，区别就是处理方式的不同：
* `add()`: 要插入的对象已经在数据库中存在时，返回错误
* `put()`: 要插入的对象已经在数据库中存在时，更改数据库中的信息
示例代码如下

```JavaScript
// 假设users中保存着一批用户数据
var i=0 , len=users.length;
while(i<len){
    store.add(users[i++]);
    //或者
    //store.put(users[i++]);
}
```
这样可以不知道每一条数据插入成功与否，其实每次add或put都会创建一个新的对这个对象存储空间的更新请求，可以获取这个请求，然后添加监听：

```JavaScript
var i=0,
    request,
    requests=[],
    len=users.length;
while(i<len){
    request=store.add(users[i++]);
    request.onerror=function(event){
        //处理错误
    }
    request.onsuccess=function(event){
        //处理成功
    }
    requests.put(request);
}
```
这时候就完成了数据的插入工作，下面就是数据的查询了

## 二、事务
对IndexedDB中数据的查询（事实上时除了数据插入外的所有操作）都是通过事务来完成的。可以通过`transaction()`来创建事务：
* 如果没有参数传递，该事务只能读取数据库中保存的对象
* 只传一个对象存储空间名（表名），该事务只加载该‘表’中的数据
* 传递数组形式存储的多个对象存储空间名（表名），该事务只加载该‘表’数组中的数据
* 传递两个参数，第二个参数表示访问模式
* * READ_ONLY(0)      ：表示只读
* * READ_WRITE(1)     ：表示读写
* * VERSION_CHANGE(2) ：表示改变

**注意：在IE10+和Firfox4+中实现的是IDBTransaction，而Chrome中则交webkitIDBTransaction，使用时应该做统一接口：**

```JavaScript
var IDBTransaction = window.IDBTransaction || window.webkitIDBTransaction;
```
相关例子代码如下：

```JavaScript
// 读取数据库中保存的对象
var transaction1=database.transaction();

// 读取数据库中指定对象存储空间（单个）保存的对象
var transaction2=database.transaction("users");

// 读取数据库中指定对象存储空间（多个）保存的对象
var transaction3=database.transaction(["users","anotherStore"]);

// 获取读写模式下的事务，此种模式不仅可以读，还可以写
var transaction4=database.transaction("users",IDBTransaction.READ_WRITE);
```

创建好事务后，可以使用`objectStore()`方法传入特定的存储空间名称访问特定的存储空间，该进行实质性的查找工作了，
* add()和put():
* get():接收键作为参数，取得值
* delete():接收键作为参数，删除对象
* clear():删除所有对象

```JavaScript
// 获取users对象存储空间中username为007的记录
var request=database.transaction("users").objectStore("users").get("007");
request.onerror = function(event){
    alert("没有获取到对象！");
};

request.onsuccess = function(event){
    //获取查询到的结果
    var result=event.target.result;
    alert(result.firstName);
}
```

事务主要是完成多个请求的，所以事务本身也有时间处理程序：`onerror`,`oncomplete`。

```JavaScript
// 事务执行出错
transaction.onerror=function(){
    // 整个事务被取消
}

// 事务执行全部完成，
transaction.oncomplete=function(){
    // 整个事务都成功完成了,但是这里访问不到get请求回的任何数据
    // 要想访问到get请求回的数据，需要去相应请求中的onsuccess方法中获取
}

```

## 三、游标和键范围
### 3.1 游标
使用事务可以直接通过已知的键检索单个对象，**如果要检索多个对象呢，这时候就要在事务内部创建游标了**。游标是一个执行结果集的指针。与传统数据库查询不同，游标并不提前搜集结果。游标指针会先指向结果中的第一项，在接到查找下一项指令时才会指向下一项。
<p>游标的创建可以使用`openCursor()`方法，该方法返回的是请求对象，因此也可以指定`onsuccess`和`onerror`事件处理程序：</p>

```JavaScript
var store=database.transaction("users").objectStore("users"),
    request=store.openCursor();
    
request.onsuccess(event){
    // 处理成功，在这里可以使用event.target.result获取存储空间的下一个对象
}

request.onerror(event){
    // 处理失败
}
```
在`onsuccess`方法中可以使用event.target.result获取存储空间的下一个对象，结果集中如果有下一项时，这个属性保存一个IDBCursor实例，没有下一项时，这个属性值为null，IDBCursor的属性有如下：
* direction: 游标移动的方向，默认值是：IDBCursor.NEXT(0)表示下一项；IDBCursor.NEXT_NO_DUPLICATE(1)表示下一个不重复的项；IDBCursor.PREV(2)表示前一项；IDBCursor.PREV_NO_DUPLICATE(3)表示前一个不重复的项。
* key:对象的键
* value:对象的值，实际的对象
* primaryKey:游标使用的键，可能是对象键，也可能是索引键（后面会有讨论）

```JavaScript
request.onsuccess=function(event){
    //获取存储空间的下一个对象
    var cursor=event.target.result;
    if(cursor){//必须检查
        // 因为cursor.value是一个对象，需要先将它转化为json字符串
        console.log("Key: "+cursor.key+" , value: "+ JSON.stringfy(cursor.value));
    }
}
```
**使用游标更新和删除对象**
* update() : 用指定对象更新当前游标的value，返回一个请求可以指定onsuccess和onerror
* delete() : 返回一个请求可以指定onsuccess和onerror
如果当前事务没有修改对象存储空间权限，update和delete就会报错。

```JavaScript
rquest.onsuccess=function(event){
    var cursor=event.target.result,
        value,
        updateRequest;
        
    if(cursor){//必须要检查
        if(cursor.key=="foo"){
            value=cursor.value; // 取得当前的值
            value.password="iamanotherpw"; // 更新密码
            
            updateRequest=cursor.update(value); //请求保存更新
            updateRequest.onsuccess=function(event){
                // 更新成功
            };
            updateRequest.onerror=function(event){
                // 更新失败
            }
        }
    }
}
```

```JavaScript
rquest.onsuccess=function(event){
    var cursor=event.target.result,
        value,
        deleteRequest;
        
    if(cursor){//必须要检查
        if(cursor.key=="foo"){
            deleteRequest=cursor.delete(); //请求删除当前项
            deleteRequest.onsuccess=function(event){
                // 删除成功
            };
            deleteRequest.onerror=function(event){
                // 删除失败
            }  
        }
    }
}
```
默认情况下，每个游标只发起一次请求，想要发送另一次请求就需要`cursor`调用下面的方法了：
* `continue(key)`:移动到结果集中的下一项。参数key可选，不指定表示移动到下一项，指定key参数表示移动到指定键位置；
* `advance(count)`:向前移动count指定的项数；
它们都会导致游标使用相同的请求，因此onsuccess和onerror函数也会得到重用。

```JavaScript
rquest.onsuccess=function(event){
    var cursor=event.target.result;
    if(cursor){//必须要检查
       console.log("Key: "+cursor.key+" , value: "+ JSON.stringfy(cursor.value));
       
       cursor.continue(); //移动到下一项
    }else{
        //没有更多选项可以迭代时，cursor为null
        console.log("完成！");
    }
}
```

### 3.2 键范围
使用游标查找数据的方式太受限了，不太理想。键范围（key range）为使用游标添加了一些灵活性。键范围由IDBKeyRange的实例表示，IE10+和Firfox4+ chrome都支持，但是chrome中名字为webkitIDBKeyRange，使用时最好先声明一个本地类型，这样可以兼顾到浏览器的差异

```JavaScript
var IDBKeyRange=window.IDBKeyRange || window.webkitIDBKeyRange;
```
那么键范围如何指定呢，如下四种方法：
* `only(key)`:范围保证只取得键"007"的对象
* `lowerBound(key)`：指定结果集的下界，保证游标从键007的对象开始，然后继续向前移动，直到最后一个对象；
* `lowerBound(key,boolean)`：第二个参数如果是true表示忽略key对象，为false表示包含key对象
* `upperBound(key)`：指定结果集的上界，游标从头开始，到key的对象为止
* `upperBound(key,boolean)`：第二个参数如果是true表示忽略key对象，为false表示包含key对象
* `bound(key1,key2)`：从key1开始，到key2的范围
* `bound(key1,key2,boolean1)`：boolean1为true表示从key1的下一个对象开始，到key2的范围
* `bound(key1,key2,boolean1,boolean2)`：boolean1和boolean2为true表示从key1的下一个对象开始，到key2的上一个对象为止范围
然后将它给`openCursor()`方法即可，下面的例子表示输出从007到ace的对象：

```JavaScript
//
var onlyRange=IDBKeyRange.only("007")
var store=database.transaction("users").objectStore("users"),
    range=IDBKeyRange.bound("007","ace"),
    request=store.openCursor(range);
request.onsuccess=function(event){
    var cursor=event.target.result;
    if(cursor){
        console.log("Key: "+cursor.key+" , value: "+ JSON.stringfy(cursor.value));
        
        cursor.continue();
    }else{
        console.log("完成");
    }
}
```
### 3.3 游标方向
上边在讲游标时讲到了`IDBCursor`对象，在`openCursor()`方法时会得到一个`IDBCursor`实例。
其实`openCursor()`也可以接收两个参数：
* IDBKeyRange：
* 方向的数值常量：即前边查询时讲的IDBKeyCursor的常量
首先考虑不同浏览器的差异：

```JavaScript
var IDBKeyCursor=window.IDBKeyCursor || window.webkitIDBCursor;
```
正常情况下，游标是从对象存储空间的第一项开始，通过调用continue()或advance()前进到最后一项。游标默认方向值时IDBCursor.NEXT,如果对象存储空间中有重复的项而又想跳过重复的项，就可以使用下面的：

```JavaScript
var store=database.transaction("users").objectStore("users"),
    request=store.openCursor(null,IDBCursor.NEXT_NO_DUPLICATE);
```
我们要说明的就是一个参数null，null表示默认的键范围，即包含所有对象。如果想让游标在指定的范围内移动，怎么办呢，将第一个参数换位IDBKeyRange对象即可。

## 四、索引和并发
### 4.1 索引
前边讲到创建对象存储空间（表）时，需要指定键（主键），但是有时候可能需要为一个对象存储空间指定多个键（比如通过用户ID和用户名两种方式保存用户资料）。这是就要用到索引了。可以考虑将用户ID作为主键，然后为用户名创建索引。
**4.1.1 如何创建索引呢：**
> (1) 首先引用对象存储空间，然后调用`createIndex()`方法，返回的是一个IDBKIndex实例
>
> ```JavaScript
> var store=database.transaction("users").objectStore("users"),
>       //第一个参数索引的名字，第二个参数索引的属性的名字，第三参数包含unique的可选项（必须指定，它标书键所在所有记录中是否唯一，因为usename可能有重复，所以这个索引不唯一）
>       index=store.createIndex("username","username",{unique:false});
> ```
> 也可以对对象存储空间调用`index()`方法，获取同样的实例
>
> ```JavaScript
> var store=database.transaction("users").objectStore("users"),
>       index=store.index("username");
> ```
> (2) 然后对索引就可以通过`openCursor()`方法创建游标了。这里会把索引键保存在`event.target.result`中（与存储空间的区别：对象存储空间会把主键保存在`event.target.result中），其余的都一样。
>
> ```JavaScript
> var store=database.transaction("users").objectStore("users"),
>       index=store.index("username"),
>       request=index.openCursor();
> request.onsuccess=function(event){
>       //处理成功
> }
> ```
> (3) 也可以在索引上创建一个只返回每条记录主键的游标，这时调用`openKeyCursor()`方法，它接收参数与`openCursor()`方法相同。此时在`event.result.key`中仍保存着索引键，`event.result.vaule`保存的则是主键（而不再是整个对象）
>
> ```JavaScript
> var store=database.transaction("users").objectStore("users"),
>       index=store.index("username"),
>       request=index.openKeyCursor();
> request.onsuccess=function(event){
>       //处理成功，event.result.key保存索引键，event.result.vaule保存主键
> }
> ```

**4.1.2 对索引进行查询**
索引创建好了，接下来就可以通过它进行查询了。
(1) 通用使用`get()`方法从索引中取得一个对象，传入一个索引键参数即可

```JavaScript
var store=database.transaction("users").objectStore("users"),
      index=store.index("username"),
      request=index.get("007");
request.onsuccess=function(event){
      //处理成功
};
request.onerror=function(event){
     //处理失败
};   
```
(2) 同样也可以通过`getKey()`方法根据索引键取得主键

```JavaScript
var store=database.transaction("users").objectStore("users"),
      index=store.index("username"),
      request=index.getKey("007");
request.onsuccess=function(event){
      //处理成功，event.result.key保存索引键，event.result.vaule保存主键（用户ID）
};  
```
(3) 如果想得到索引相关信息呢 怎么办呢，可以通过IDBCursor对象获取到索引的额如下信息：
* name：索引的名字
* keyPath：传入`createIndex()`中的属性路径
* objectStore：索引的对象存储空间
* unique：表示索引键是否唯一的boolean值

(4) 可以通过indexName属性访问到为该空间建立的所有索引

```JavaScript
var store=database.transaction("users").objectStore("users"),
    indexNames=store.indexNames,
    i=0,
    len=indexNames.length;
while(i<len){
    index=store.index(indexNames[i++]);
    console.log("索引名字："+index.name+" , KeyPath:"+index.keyPath+" , Unique: "+index.unique);
} 
```
**4.1.3 删除索引**
通过`deleteIndex()`方法删除索引

```JavaScript
var store=database.transaction("users").objectStore("users");
store.deleteIndex("username");
```

### 4.2 并发
异步API也会存在并发问题，一个浏览器多个标签操作数据库时就会发生。当浏览器中仅有一个标签页使用数据库时，才能调用`setVersion()`方法完成操作。
**注意刚打开数据库时要记着指定`onversionchange`事件**，这样当同一个来源的另一个标签调用`setVersion()`时就会执行这个回调函数，最佳处理方式，是在里面执行立即关闭数据库，从而保证版本的顺利更新

```JavaScript
var request,database;
request=indexDB.open("admin");
request.onsuccess=function(event){
    database=event.target.result;
    
    //当其它标签页对数据库版本有更新时，执行数据库关闭
    database.onversionchange=function(event){
        database.close();
    }
}
```
当你要调用`setVersion()`更新数据库版本时，但是另一个标签页已经打开数据库了，这时候通知用户关闭其它标签页页很有用。可以使用`onblocked`事件：

```JavaScript
var request=database.setVersion("2.0");
request.onblock=function(){
    alert("请关闭其它标签页，然后重新执行");
};

request.onsuccess=function(){
    //处理成功，继续
}
```
