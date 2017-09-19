---
title: Java反射
date: 2017-09-19 22:45:20
tags: java
---

#### 1、Class类
<p>java语言中万物皆对象，（基础类型有包装类，静态成员除外）。</p>
<p>类也是对象，类是谁的对象呢？--java.lang.Class的实例对象</p>

eg：

```
public class ClassDemo(){
	public static void main(String[] args){
		//Foo的实例对象如何表示
		Foo foo=new Foo();
		
		//Foo这个类也是一个实例对象，Class类的实例对象，如何表示呢
		//任何一个类都是Class的实例对象，这个实例对象有三种表示方式
		
		//第一种表示方式-->任何一个类都有一个隐含的静态成员变量
		Class c1=Foo.class;
		
		//第二种表达方式，已经知道的该类的对象通过getClass方法
		Class c2=foo1.getClass();
		
		/*官网c1，c2表示了Foo类的类类型（class type）
		 *万物皆对象，
		 *类也是对象
		 *这个对象我们成为该类的类类型
		 */
		
		//不管c1，c2都代表了Foo类的类类型，一个类只可能是Class类的一个实例对象
		System.out.println(c1==c2);
		
		//第三种表达方式
		Class c3=null;
		try{
			c3=Class.forName("com.afei.reflect.Foo");
		}catch(ClassNotFoundException e){
			e.printStaticTrace();
		}
		System.out.println(c3==c2);
		
		//完全可以通过类的类类型创建该类的对象实例，-->通过c1，c2，c3创建Foo的实例
		try{
			Foo foo=c1.newInstance();//需要有无参数的构造方法
			foo.print();
		}catch(InstantictionException e){
			e.printStaticTrace();
		}catch(IllegalAccessException e){
			e.printStaticTrace();
		}
		
		
		
	}
}

class Fool{
	void print(){
		System.out.println("Foo");
	}
}
```


#### 动态加载类

Class.forName("类的全称");

* 不仅表示了类的类类型，还代表了动态加载类
* 
* 请区分编译，运行
* 
* 编译时刻加载类是静态加载类，运行时刻加载类hi动态加载类
* 

```
class Office{
	public static void main(String[] args){
		
		//new 创建的对象，是静态加载类，在编译时刻就需要加载所有的可能使用到的类
		//通过动态加载类可以解决该问题
		if("World".equals(args[0])){
			World w=new World();
			w.start();
		}
		
		if("Excell".equals(args[0])){
			Excell e=new Excell();
			e.start();
		}
		
	}
}

class World{
	void start(){
		System.out.println("我是world！");
	}
}

class Excell{}

```

```
class OfficeBetter{
	public static void main(String[] args){
		
		try{
			//动态加载类，在运行时刻加载
			Class c=ClassforName(args[0]);
			
			OfficeAble oa=(OfficeAble)c.newInstance();
			c.start();
		}catch(Exception e){
			e.printStaticTrace();
		}		
	}
}


interface OfficeAble{
	void start();
}
class World implements OfficeAble{
	public void start(){
		System.out.println("我是world！");
	}
}

class Excell implements OfficeAble{
	public void start(){
		System.out.println("我是Excell！");
	}
}

```

可以使用 `java OfficeBetter Excell`  研验证

#### Java 获取方法信息

基本的数据类型
* 
* void关键字 都存在类类型
* 

```
public class ClassDemo2(){
	public static void main(String[] args){
		//
		Class c1=int.class;//int的类类型
		Class c2=String.class;//String类的类类型，String类字节码
		
		Class c3=double.class;//
		Class c4=Double,class;//
		Class c5=void.class;//
		
		
		//返回不包含包名的类的名称
		System.out.println(c1.getName());
		System.out.println(c2.getName());
		System.out.println(c3.getName());

		
	}
}

```

Class类的基本操作

```
/**
 * 打印类的信息，包括类的成员函数，成员变量
 */
public class ClassUtil(){
	/**
	 * 打印成员函数
	 */
	public static void printClassMessage(Object obj){
		//获取类的信息，首先要获取类的类类型
	
		Class c=obj.getClass();//传递的是哪个字类的对象，c就是改字类的类类型
	
		//获取类的名称
		System.out.println("类的名称是："+c.getName());
	
		/**
		 * Method类，方法和对象
		 * 一个成员方法就是一个Method对象
		 * getMethods()方法获取的是所有public的函数
		 * getDeclareMethods()获取的是该类自己声明的方法，不问访问权限
		 */
		 Method[] ms=c.getMethods();//c.getDeclareMethos();
		 
		 for(int i=0,int len=ms.length;i<len;i++){
			 //得到方法的返回值类型-->返回值类型的类类型
			 Class returnType=ms[i].getReturnType();
			 System.out.print(returnType.getName()+" ");
			 
			 //得到方法的名称
			 System.out.print(ms[i].getName()+" ( ");
			 
			 //得到参数类型-->得到的是参数列表的类型的类类型
			 Class[] paramTypes=ms[i].getParameterType();
			 
			 //
			 for(Class class1:paramTypes){
			 	System.out.print(class1.getName()+" , ");
			 }
			 
			 System.out.println(" ) ");
	
		}
		
		
		/**
		 * 获取字段的函数
		 */
		public static void getFieldMessage(Object obj){
			Class c=obj.getClass();
		
			 /**
			  * 成员变量也是对象
			  * java.lang.reflect.Field
			  * Field类封装了关于成员变量的操作
			  * getFields()获取的是所有的public的成员变量的信息
			  * getDeclaredFields()获取所有该类自己声明的成员变量（可以是公有，也可以是私有）
			  */
			  //Field[] fs=c.getFields();
			  Field[] fs=c.getDeclaredFields();
			  for(Field field:fs){
			  	//得到成员变量的类型的类类型
			  	Class fieldType=field.getType();
			  	String typeName=fieldType.getName();
			  	
			  	//得到成员变量的名称
			  	String fieldName=field.getName();
			  	System.out.println(typeName+" "+ fieldName);
			  }
		}
		
		/**
		 * 获取构造函数
		 */
		 public static void getConMessage(Object obj){
			Class c=obj.getClass();
			/**
			 *构造函数也是对象
			 *java.lang.Constructor中封装了构造函数的信息
			 *getConstructor获取所有的public的构造函数
			 *getDeclareConstrutor获取所有的构造函数
			 */
			//Constructor[] cs=c.getConstructor();
			Constructor[] cs=c.getDeclareConstructor();
			for(Constructor constructor: cs){
				System.out.print(constructor.getName+"(");
				
				//获取构造函数的参数列表-->得到的是参数列表的类类型
				Class[] paramTypes=constructor.getParameterTypes();
				for(Class class1:paramTypes){
					System.out.print(class1.getName+",");
				}
				System.out.print(")");
				
			}
		
		 }
		
	}

```

## 方法反射的基本操作

<p>如何获取某个方法：方法的名称和方法的参数列表才能为宜决定某了列表</p>
<p>方法的反射操作：`method.invoke(对象，参数列表)`</p>

```
public class MethodDemo1{
	public static void main(String[] args){
		//获取print(int , int)方法,1、要获取一个方法就是获取类的信息，获取类的信息首先要获取类的类类型
		A a1=new A();
		Class c=a1.getClass();
		/**
		 *2 获取方法，名称和参数列表来决定
		 * getMethod获取的是public的方法
		 * getDelcaredMethod自己声明的方法
		 */
		try{
			//Method m=c.getMethod("print",new Class[]{int.class,int.class});
			Method m=c.getMethod("print",int.class,int.class);
			
			
		//方法的反射操作；
		//a1.print(10,20); 方法的反射操作时用m对象来进行方法调用，和a1.print效果相同
		//方法如果没有返回值返回null，有返回值返回具体的返回值
		//Object obj=m.invoke(a1,new Object[]{10,20});
		Object obj=m.invoke(a1,10,20);
		
		//获取方法print(String,String)
		Method m1=c.getMethod("print",String.class,String.class);
		//用方法进行反射操作
		o=m1.invoke(a1,"hello","world");
		}catch(Exception e){
			e.printStaticTrace();
		}
		
		
		//获取没有参数的方法
		//Method m2=c.getMethod("print",new Class[]{});
		Method m2=c.getMethod("print");
		//m2.invoke(a1,new Object[]{});
		m2.invoke(a1);		
		
	}
}

class A{
	public void print(){
		System.out.println("我没有参数");
	}

	public void print(int a,int b){
		System.out.println(a+b);
	}
	public void print(String a,String b){
		System.out.println(a.toUpperCase()+","+b.toLowerCase());
	}
}

```

## 通过反射了解集合泛型的本质

* 通过Class, Method来认识泛型的本质
* 


```
public class MethodDemo4{
	public static void main(String[] args){
		ArrayList list=new ArrayList();//没有泛型，可以放任何数据；
		
		ArrayList<String> list1=new ArrayList<String>();//泛型指定只能放指定的类型
		list1.add("hello");
		//list1.add(20);//错误
		
		
		Class c1=list.getClass();
		Class c2=list.getClass();
		System.out.println(c1==c2);//true
		//反射的操作都是编译之后的操作
		
		/**
		 *c1==c2结果返回true，说明编译之后的集合的泛型时去泛型化的
		 * java中集合的泛型，是防止错误输入的，只在编译阶段有效，绕过编译就无效了
		 * 验证：通过反射操作，绕过编译
		 */
		 try{
		 	Method m=c2.getClass("add",Object.class);
		 	m.invoke(list1,100);//绕过编译操作就绕过了泛型
		 	System.out.println(list1.size());
		 	System.out.println(list1);
		 	
		 	/*for(String string:list1){
			 		System.out.println(string);
			 	}
			 }catch(Exception e){
			 	e.printStaticTrace();
		  }*///现在不能这样遍历，因为100不能转化为String
		 
		
	}
}
```















