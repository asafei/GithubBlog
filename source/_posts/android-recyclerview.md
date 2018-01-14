---
title: Android之recyclerview
date: 2018-01-14 19:43:28
tags: Android
---
## RecyclerView的使用
-----
### 一、引用与布局

#### 1、添加依赖

为了让RecyclerView兼容所有的版本，Android团队将RecyclerView定义在了support库中了。所以首先在`build.gradle`文件的dependencies中添加依赖：

```
compile 'com.android.support:recyclerview-v7:24.2.1'
```

#### 2、添加布局

引用依赖之后，就可以在layer中添加该控件了，如下就完成了对recyclerview的添加：

```
<android.support.v7.widget.RecyclerView
	android:id="@+id/recycler_view"
	android:layout_width="match_parent"
	android:layout_height="match_parent"/>
```

### 二、数据的准备

#### 1、模拟数据

以一个数组来模拟要展示的数据

```
private String[] data={
	"Apple","Banana","Orange","Watermelon","Pear",
	"Grape","Pineapple","Strawberry","Cherry","Mango",
	"Apple","Banana","Orange","Watermelon","Pear",
	"Grape","Pineapple","Strawberry","Cherry","Mango",
}
```
#### 2、创建一个水果类

```
public class Fruit{
	private String name;
	private int imageId;
	public Fruit(String name,int imageId){
		this.name=name;
		this.imageId=imageId;
	}

	public String getName(){
		return name;
	}

	public int getImageId(){
		return imageId;
	}
}
```

#### 3、创建一个item的布局文件

每一条数据以何种样式展示，加入是一张图片配一个名字。在layout文件夹下创建一个`fruit_item.xml`的布局文件

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android
	android:layout_width="match_parent"
	android:layout_height="wrap_content">
	<ImageView
		android:id="@+id/fruit_image"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content">
	<IextView
		android:id="@+id/fruit_name"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:layout_gravity="center_vertical"
		andriod:layout_marginLeft="10dp">
</LinearLayout>
```


### 三、适配器的编写

有了上面的准备就可以开始编写适配器了，需要继承自RecyclerView.Adapter类

```
public class FruitAdapter extends RecyclerView.Adapter<FruitAdapter.ViewHolder>{
	private List<Fruit> fruitList;
	static class ViewHolder extends RecyclerView.ViewHolder{
		ImageView fruitImage;
		TextView fruitName;
		public ViewHolder(View v){
			super(v);
			fruitImage=(ImageView)view.findViewById(R.id.fruit_image);
			fruitName=(TextView)view.findViewById(R.id.fruit_name);
		}
	}

	//构造函数接收数据源
	public 	FruitAdapter(List<Fruit> fruitList){
		this.fruitList=fruitList;
	}

	//通过架子啊fruit_item布局，创建ViewHolder实例，并返回该实例
	@Override
	public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType){
		View view=LayoutInflater.from(parent.getContext())
			.inflate(R.layout.fruit_item,parent,false);
		ViewHolder holder=new ViewHolder(view);
		return holder;
	}

	//将RecyclerView的子项进行赋值
	@Override
	public void onBindViewHolder(ViewHolder holder,int position){
		Fruit fruit=fruitList.get(position);
		holder.fruitImage.setImageResource(fruit.getImageId());
		holder.fruitName.setText(fuit.getName());
	}

	//获取一共有多少项
	@Override
	public int getItemCount(){
		return fruitList.size();
	}
}
```

### 四、RecyclerView的加载

现在就可以展示数据了

```
public class MainActivity extends AppcompatActivity{
	private List<Fruit> fruitList=new ArrayList<>();
	
	@Override
	public void onCreate(Bundle savedInstanceState){
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		//初始化数据
		initFruits();
		RecyclerView recyclerView=(RecyclerView)findViewById(R.id.recycler_view);

		//指定RecyclerView的布局方式
		LinearLayoutManager layoutManager=new LinearLayoutManager(this);
		recyclerView.setLayoutManager(layoutManager);
		FruitAdapter adapter=new FruitAdapter(fruitList);
	
		recyclerView.setAdapter(adapter);
	}

	//初始化数据
	public void initFruits(){
		Fruit apple=new Fruit("apple",R.drawable.apple_pic);
		fruitList.add(apple);
		//...

	}
}
```



### 五、改变RecyclerView的布局

通过上边的步骤就可以成功展示列表数据了，排列方式是纵向排列。如果要实现横向布局怎么办呢？
第四部中出现了`LinearLayoutManager`，表示纵向布局，可以给这个类添加朝向,通过
`setOrientation`方法。

```
	@Override
	public void onCreate(Bundle savedInstanceState){
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		//初始化数据
		initFruits();
		RecyclerView recyclerView=(RecyclerView)findViewById(R.id.recycler_view);

		//指定RecyclerView的布局方式
		LinearLayoutManager layoutManager=new LinearLayoutManager(this);
		//横向布局 
		layoutManager.setOrientation(LinearLayoutManager.HORIZONTAL);
		recyclerView.setLayoutManager(layoutManager);
		FruitAdapter adapter=new FruitAdapter(fruitList);
	
		recyclerView.setAdapter(adapter);
	}
```

这需要改一下`fruit_item.xml`文件的布局

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android
	android:orientation="vertical"
	android:layout_width="100dp"
	android:layout_height="wrap_content">
	<ImageView
		android:id="@+id/fruit_image"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:layout_gravity="center_horizontal">
	<IextView
		android:id="@+id/fruit_name"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content"
		android:layout_gravity="center_horizontal"
		andriod:layout_marginTop="10dp">
</LinearLayout>
```
这是就实现了横向布局。





### 六、其它的布局类
* GridLayoutManager  网格状布局
* StaggeredGridLayoutManager 瀑布流式布局

比如瀑布流式布局，只需修改layoutManager即可

```
//第一个参数表示列数；第二个参数指定布局的排列方向
StaggeredGridLayoutManager layoutManager=new StaggeredGridLayoutManager(3,StaggeredGridLayoutManager.VERTICAL);
```

同样也需要修改`fruit_item.xml`文件，不再赘述

### 七、点击事件

RecyclerView的所有的点击事件都由具体的view去注册，这样就可以给整个子item注册事件，也可以给item中的子控件注册事件。是在Adaper中实现的


```
public class FruitAdapter extends RecyclerView.Adapter<FruitAdapter.ViewHolder>{
	private List<Fruit> fruitList;
	static class ViewHolder extends RecyclerView.ViewHolder{
		//1
		View fruitView;

		ImageView fruitImage;
		TextView fruitName;
		public ViewHolder(View v){
			super(v);

			//2
			fruitView=view;
	
			fruitImage=(ImageView)view.findViewById(R.id.fruit_image);
			fruitName=(TextView)view.findViewById(R.id.fruit_name);
		}
	}

	//构造函数接收数据源
	public 	FruitAdapter(List<Fruit> fruitList){
		this.fruitList=fruitList;
	}

	//通过架子啊fruit_item布局，创建ViewHolder实例，并返回该实例
	@Override
	public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType){
		View view=LayoutInflater.from(parent.getContext())
			.inflate(R.layout.fruit_item,parent,false);
		//3		
	    final ViewHolder holder=new ViewHolder(view);
		//注册点击事件
		holder.fruitView.setOnclickListener(new OnClickListener(){
			@Override
			public void onClick(View v){
				int position=holder.getAdapterPosition();
				Fruit fruit=fruitList.get(position);
				//...
			}
		});

		holder.fruitImage.setOnclickListener(new OnClickListener(){
			@Override
			public void onClick(View v){
				int position=holder.getAdapterPosition();
				Fruit fruit=fruitList.get(position);
				//...
			}
		});

		return holder;
	}

	//将RecyclerView的子项进行赋值
	@Override
	public void onBindViewHolder(ViewHolder holder,int position){
		Fruit fruit=fruitList.get(position);
		holder.fruitImage.setImageResource(fruit.getImageId());
		holder.fruitName.setText(fuit.getName());
	}

	//获取一共有多少项
	@Override
	public int getItemCount(){
		return fruitList.size();
	}
}
```

### 八、RecyclerView的更新
1、如果对数据源传入了新数据，需要更新RecyclerView的显示，可以通过调用adapter的`NotifyItemInserted`方法来实现

```
adatper.notufyItemInsert(msgList.size()-1);
recyclerView·scrollToPosition(msgList.size()-1);
```
