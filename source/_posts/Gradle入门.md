---
title: Gradle入门
date: 2017-09-19 22:48:53
tags: Gradle
---

## 一、构建工具

1、构建工具的作用
* 依赖管理
* 测试、打包、发布
* 机器能干的活，绝不自己动手
*

2、主流构建工具
* Ant：编译、测试、打包
* Maven： 依赖管理、发布
* Gradle：

3、Gradle是什么
一个开源的 **项目自动化构架工具** ，建立在Apache Ant和Apache Maven概念的基础之上，并且引入基于Groovy的特定领域语言（DSL），而不再使用XML形式管理构建脚本。

## 二、Gradle的安装

1. 确保本地已经安装JDK
2. 从Gradle官网下载[Gradle](https://gradle.org)
3. 配置环境变量：GRADLE_HOME，并且添加到path，%GRADLE_HOME%\bin
4. 验证是是否安装成功，gradle -v

## 三、Groovy是什么

Groovy是用于java虚拟机的一种敏捷的动态语言，它是一种成熟的面向对象编程语言，既可以用于面向对象编程，又可以用作纯粹的脚本语言。使用该语言不必编写过多的代码，同时又具有闭包和动态语言中的其它特性。 

##### 3.1 与java比较

* Groovy完全兼容java语法
* 分号可选
* 类、方法默认是public
* 编译器给属性自动添加getter／setter方法
* 属性可以直接用点号获取
* 最后一个表达式的汁会被作为返回值（也就是return字符串可以不写）
* == 等同于equals()，不会有NullPointerExceptioins

##### 3.2 高效的grovvy特性

* 可选类型定义 

```
 def version=1
```
* assert语句

```
 assert version==2
```
* 可选的括号

```
//有没有括号，效果是一样的
 println(version)  // 1
 println version   // 1
```
* 字符串（单引号、双引号、三引号）

```
//单引号就是字符串
def s1='com'

//双引号可以插入字符串,${}
def s2="afei"
def s22="afei is a ${version}"

//三引号可以换行
def s3=''' wo shi ni die'''

def s32='''wo 
shi 
ni
ma 
'''
```
* 集合API

```
//list
def buildTools=['ant','maven']
buildTools<< 'gradle'
assert buildTools.getClass()== ArrayList
assert buildTools.size() == 3

//map
def buidYears=['ant':2000,'maven':2004] 
buildYears.gradle=2009

println buildYears.ant  //2000
printlb=n buildYears['gradle'] //2009

println buildYears.getClass()
```
* 闭包

```
def c1={
	v ->
		println v
}

def c2={
	println 'hello'
}

def method1(Closure closure){
	closure('param');
}

def method2(Closure closure){
	closure();
}

method1(c1) //param
method(c2)  //hello
```

##### 3.3 grovvy基础知识

```
//构建脚本默认都有个Project实例

apply  plugn : 'java' //命名参数 

version='0.1'

repositories{
	mavenCentral()
}

dependencies{
	compile 'common-codec:commons-codec:1.6'
}
```

## 四、Gradle项目

##### 4.1 Gradle目录结构

* src
* 	main
*    java
*    resources   配置文件
*    webapp      web资源
*  test
*  	 java
*  	 resources

## 五、构建脚本

##### 项目 project

一个项目代表一个正在构建的组件（比如一个jar文件），当构建启动后，Gradle会基于build.gradle实例化一个org.gradle.api.Project类，并且能够通过project变量使其隐式可用。

* group 、 name 、 version
* apply（应用一个插件）、 dependencies（用来声明依赖，）、 repositories（声明仓库，去哪个仓库找jar包）、 task（声明项目中有哪些任务）
* 属性等其它配置方式：ext、gradle.properties(键值对的方式声明属性)

##### 任务 task

任务对应org.gradle.api.Task。主要包括任务动作和任务依赖。任务动作定义了一个最小的工作单元。可以定义依赖于其它任务、动作序列和执行条件。  

* dependsOn
* doFirst(在动作列表最前边添加动作)、doLast（在动作列表最后边添加动作） <<

##### 自定义一个task

```
def cresteDir={
	path ->
		File dir=new File(path);
		if(!dir.exists()){
			dir.mkdirs();
		}
		
}

task makeJavaDir(){
	def paths=['src/main/java','src/main/resources','src/test/java','src/rest/resources'];
	doFirst{ //动作代码，执行阶段会执行
		path.forEach(createDir);
	}
}

task makeWebDir(){
	dependsOn 'makeJavaDir'  //声明依赖makeJavaDir,执行makeWebDir时先执行makeJavaDir
	def paths=['src/main/webapp','src/test/webapp'];
	doLast{//动作代码，执行阶段会执行
		paths.forEach(createDir);
	}
}
```

## 依赖管理

#####  概述

几乎所有的基于JVM的软件项目都需要依赖外部类库来重用现有的功能。自动化的依赖管理可以明确依赖的版本，可以解决因传递性依赖带来的版本冲突。

##### 工件坐标

group、name、version

##### 常用仓库

* mavenLocal（本地仓库）／mavenCentral（公网仓库）／jcenter（公网仓库）
* 自定义的maven仓库
* 文件仓库

##### 依赖的传递性

* B依赖A，如果C依赖B，那么C依赖A

##### 依赖阶段配置

* compile、runtime
* testCompile、testRuntime

配置maven仓库地址

```
respositories{
	mavenLocal()     //配置本地仓库
	mavenCentral()   //配置公网仓库
	maven{           //配置私服仓库
		url '私服仓库地址'
	}
}
```

###### 解决版本冲突

传递依赖可能产生版本冲突

解决版本冲突
* 产看依赖报告
* * 修改默认解决策略（gradle默认使用最新的版本）
```
configurations.all{
	resolutionStrategy{
		failOnVersionConflict()  //强制冲突报错
	}
}
```

* 排除传递性依赖
* * 修改默认解决策略
```
compile('org.hibernate:hibernate-core:3.6.3.Final'){
	exclude group:"org.sef4j",module:"slf4j-api" //module就是坐标中的name属性
	//transitive=false //排除所有的传递性依赖
}
```
* 强制一个版本
* ```
configurations.all{
	resolutionStrategy{
		force 'org.sef4j:slf4j-api:1.7.24'	}
}
```
## 多项目构建

##### 项目模块化

settings.gradle就是用来多项目构建的
```
rootProject.name = 'todo'
include 'respository'
include 'model'
include 'web'
```

##### 如果每个module都配置相同的东西，可以将它们删除，然后在根build.gradle文件中添加：

```
allprojects{
	apply plugn: 'java'
	sourceCompactibility=1.8
}
```

##### 如果需要将某个模块大包成war包，需要在改模块下面的build.gradle下添加

```
apply plugn: 'war'
```

##### 如果要依赖另一个modele

```
dependencies {
	compile project(": repository")
}
```

##### 在根build.gradle文件中给子module添加依赖

```
subprojects {
	respositories{
		mavenCentral()
	}
	
	dependencies{
		compile 'ch.qos.logback:logback-class^^^'
		testCompile group: 'junit', name: 'jnuit'
	}
}
```

##### 如果要版本统一

可以先将子模块中的版本号都取消，然后新建一个gradle.properites，在里面写入要添加的版本号：

```
group = ‘com.afei.gradle’
version=1.0
```
## 自动化测试

测试配置

```
dependencies{
	testCompile 'junit:junit:4.11'
}
```

##### 测试发现

* 任何即成自junit.framework.TestCase或groovy.util.GroovyTestCase
* 任何使用@RunWith注解的类
* 任何至少包含一个被@Test注解的类


## 发布

gradle没有自己的仓库，一般发布到maven仓库。
gradle使用插件发布，

```
apply plugn:'maven-pulish'
publishing{
	publications{
		myPublish(MavenPulication){
			from components.java
		}
	}
	
	repositories{
		maven{
			name "myrepo"
			url "私服地址"
		}
	}
}
```











