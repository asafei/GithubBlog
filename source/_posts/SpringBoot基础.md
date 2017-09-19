---
title: SpringBoot基础
date: 2017-09-19 22:27:04
tags: SpringBoot
---


本笔记是慕课网视频记录的[http://www.imooc.com/learn/767](http://www.imooc.com/learn/767)

#### 一、 创建页面

常见一个Spring Boot工程，然后在里面创建一个类用`@RestController`标示；然后创建一个方法用`@RequestMapping(value = "hello" , method = RequestMethod.GET)`，其中value表示延长路径，method表示请求方法。
如下

```
@RestController
public class HelloController {
    @RequestMapping(value = "hello" , method = RequestMethod.GET)
    public String say(){
        return "Hello Spring Boot，我是你爹";
    }
}
```
### 二、启动方式。

* 第一种是直接通过ide进行启动；
* 第二种是进入项目所在目录输入`mvn spring-boot:run`;
* 第三种启动方式，使用maven先编译`mvn install`，然后进入target目录下，会发现又一个****.jar 的文件，然后编译这个jar文件即可`java-jar ****.jar`.
* 就可以通过`localhost:8080/girl`访问了;

### 三、属性配置

```
spring.datssource.url:jdbc:mysql://127.0.0.1:3306
spring.datssource.username:root
spring.password:123456
spring.datasource.driver-class-name:com.mysql.jdbc
```

##### 3.1 改变端口号，增加路径 

在‘application.properities’文件中添加

```
server.port=8081
server.context-path=/girl
```
此时访问端口就变成了8081，并且路径前添加了girl，这时候就要通过如下地址才能访问刚才的页面`localhost:8081/girl/hello`
推荐使用yml文件代替原来的properities


##### 3.2 将配置文件中的属性，引到变量中去

在application.yml中添加

```
cupSize: B
```

然后在HelloController文件中添加变量

```
@Value("${cupSize}")
    private String cupSize;
```
这样配置文件中的cupSize就引入变量中了

也可以在配置文件中，使用变量

```
server:
  port: 8085
  context-path: /girl
cupSize: B
age: 18
content: "罩杯是: ${cupSize} , 年龄是：${age} 。"
```

需要说明的是，在配置文件中不需要说明类型，只需要在变量处给出声明就可以；


##### 3.3 使用分类来代替配置文件

如果属性较多，在配置文件中一个一个写会过于繁琐，这时候需要创建一个类GirlProperties，获取前缀是girl的配置,同时需要在类GirlProperties上添加`@Component`的注解.

```
@Component
@ConfigurationProperties(prefix = "girl")
public class GirlProperties {

    private String cupSize;
    private Integer age;

    public String getCupSize() {
        return cupSize;
    }

    public void setCupSize(String cupSize) {
        this.cupSize = cupSize;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```
然后在application.yml中添加

```
girl:
  cupSize: D
  age: 28
```
如何使用呢，在HelloController中添加`@Autowired`,就可以使用了

```
	@Autowired
    private GirlProperties girlProperties;
    
    @RequestMapping(value = "hello" , method = RequestMethod.GET)
    public String say(){
        //return cupSize;
        return girlProperties.getCupSize();
    }
```

但是有多个环境，比如开发环境和生产环境配置不同的时候，频繁的修改application.yml配置文件吗？不要咬的，我们可以创建一个application-dev.yml在里面放置开发环境的配置，然后再创建一个application-prod.yml的文件，在里面放置生产环境的配置。最后只需要在application.yml文件中作选择即可，比较选择开发环境，只需写入：

```
spring:
  profiles:
    active: dev
```
而如果使用命令行的方式启动，需要：

```
java -jar ****.jar --spring.profiles.active=dev
```

### 四、Controller的使用

| 注解 | 描述 |
| --- | ---  |
| @Controller | 处理http请求 |
| @RestController | Spring4之后新加的注解，原来返回json需要@ResponseBody配合@Controller |
| @RequestMapping | 配合url映射 |

##### 4.1 @Controller和@RestController的区别

将项目中的HelloController的注解@RestController换做@Controller，再次运行就会报错，这时候需要添加模版，首先在porm中添加注解 

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

然后再resource目录下新建目录templates，并新建文件indexhtml，随意写入东西。然后在HelloController中写入：

```
 @RequestMapping(value = "hello" , method = RequestMethod.GET)
    public String say(){
        return "index.html";
    }
```
这时候就可以访问这个借口了。虽然@Controller配合模版可以实现与@RestController同样的效果，但是使用模版会带来性能上的损耗，不推荐使用。

##### 4.2 如何做到两个借口是同一个方法

使用@RequestMapping的注解，在value中使用集合即可：

```
@RequestMapping(value = {"hello","hi" }, method = RequestMethod.GET)
    public String say(){
        //return cupSize;
        return girlProperties.getCupSize();
    }
```
这样的就既可以使用`ip:port/path/hello`，又可以`ip:port/path/hi`来访问相同的请求了。
也可以给整个类设定一个url，如下：

```
@RestController
@RequestMapping(value = "/hello")
public class HelloController {

    @Autowired
    private GirlProperties girlProperties;


    @RequestMapping(value = {"/say"}, method = RequestMethod.GET)
    public String say(){
        //return cupSize;
        return girlProperties.getCupSize();
    }

}

```
这样就可以使用`ip:port/path/hello／say`来访问girl的属性了。

`@RequestMapping(value = {"hello","hi" }, method = RequestMethod.GET)`中的请求方式可以是GET也可以是POST，如果不写的话，两种都可以，但是并不推荐这么做。


##### 4.3 如何处理url中的参数

| 注解 | 描述 |
| --- | ---  |
| @PathVariable | 获取url中的数据 |
| @RequestParam | 获取请求参数的值 |
| @GetMapping | 组合注解 |

**4.3.1 不带等号的传餐方式**

如何使用@PathVariable传递参数呢，在@RequestMapping的value中添加参数用大括号括起来，然后在具体函数的传入参数用@PathVarable来声明，如下

```
@RequestMapping(value = {"/say/{id}"}, method = RequestMethod.GET)
    public String say(@PathVariable("id") Integer id){
        return "id="+id;
    }
```
这样就可以通过`ip:port/path/hello／say/36`来传递id=36了。当然在@RequestMapping中的"{id}"可以写在任何路径。

**4.3.2 带等号的传参方式**

但是如何获取`ip:port/path/hello／say/id=100`这种方式中的id的值的，使用@RequestParam，修改如下：

```
@RequestMapping(value = {"/say"}, method = RequestMethod.GET)
    public String say(@RequestParam("id") Integer id){
        return "id="+id;
    }

```

如何设置要传递的参数是否必传，以及如何设定默认值呢，只需要使用required和defaultValue在RequestMapping的value中设置一下即可

```
@RequestMapping(value = {"/say"}, method = RequestMethod.GET)
    public String say(@RequestParam("id" , required=false , defaultValue= “2”) Integer id){
        return "id="+id;
    }
```

**4.3.3 如何简化@RequestMapping(value = {"/say"}, method = RequestMethod.GET)呢**

可以使用`@GetMapping(value="/say")`代替


### 五、jpa

##### 5.1 什么是jpa

JPA（Java Persistcence API）定义了一系列对象持久化的标准。目前实现这一规范的产品有Hibernate、TopLink等。

使用mysql数据哭需要添加两个组件,在porm中添加

```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
```
然后在application.yml中添加mysql驱动和jpa

```
spring:
  profiles:
    active: dev
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/dbgirl
    username: root
    password: 123456
  jpa:
    hibernate:
      ddl-auto: create
    show-sql: true

```
的ddl-auto　的值为create时，表示每次启动都创建一个新的Girl表（删除老表）；当为update时，不会删除老表中的数据，会保留下来；当为create-drop时，当停下来时会删除表；当为validate时会验证表中的结构，如果不一致的化会报错

RESTful API的设计
| 请求类型 | 请求路径 | 功能 |
| --- | ---  | --- |
| GET | `/girls` | 获取女生列表 |
| POST | `/girls`| 创建一个女生 |
| GET | `/girls/id` | 通过id查询一个女生 |
| PUT | `/girls/id` | 通过id更新一个女生 |
| Delete | `/girls/id` | 通过id删除一个女生 |


##### 5.2 创建数据库

比如要在数据库中创建一个dbgirl的表。不用使用sql语句的。可以直接在工程中创建一个Girl的类，并且加上@Entity；必须加一个无参的构造函数，如下

```
@Entity
public class Girl{
    @Id
    @GeneratedValue
    private Integer id;
    private String cupSize;
    private Integer age;
    
    public Girl(){
    }
    
    ^^get和set方法
}
```

这样运行一下，就直接在数据库中擦黄建了一个Girl的表，里面有相应的字段了。

##### 5.3 数据库查询

如何通过spring boot查询数据库中的所有内容呢，首先我们需要建立一个接口继承JpaRepository

```
public interface GirlRepository extends JpaRepository<Girl,Integer> {

}
```
然后新建一个GirlController用@RequestController注解，然后在里面声明一个GirlRepository，最后给出一个girlList的函数即可，如下所示：

```
@RestController
public class GirlController {
    @Autowired
    private GirlRepository girlRepository;

    @GetMapping(value = "/girls")
    public List<Girl> girlList(){
        return girlRepository.findAll();
    }
}
```
这样就可以通过`ip:port/path／girls`来查询数据库中所有的女孩列表了

##### 5.4 添加一个女生到数据库

在GirlController中声明一个函数即可，接受一系列的参数

```
     /**
     * 添加一个女生
     * @param cupSize
     * @param age
     * @return
     */
    @PostMapping(value = "/girls")
    public Girl girlAdd(@RequestParam("cupSize") String cupSize,
                          @RequestParam("age") Integer age){

        Girl girl=new Girl();
        girl.setCupSize(cupSize);
        girl.setAge(age);

        return girlRepository.save(girl);
    }
```
这样就可以通过post请求传入相关参数来入库了，不需要一句sql语句。同理对女生的查询、更新、删除如下：

```
    /**
     * 根据id查询一个女生
     * @param id
     * @return
     */
    @GetMapping(value = "/girl/{id}”)
    public Girl girlFindOne(@RequestParam("id") Integer id){
        return girlRepository.findOne(id);
    }

    /**
     * 更新一个女生
     * @param id
     * @param cupSize
     * @param age
     * @return
     */
    @GetMapping(value = "/girls/{id}”)
    public Girl girlUpdate(@RequestParam("id") Integer id, @RequestParam("cupSize") String cupSize,
                           @RequestParam("age") Integer age){
        Girl girl=new Girl();
        girl.setId(id);
        girl.setCupSize(cupSize);
        girl.setAge(age);
        return girlRepository.save(girl);
    }

    /**
     * 删除一个女生
     * @param id
     */
    @GetMapping(value = "/girl/{id}”)
    public void girlDelete(@RequestParam("id") Integer id){
        girlRepository.delete(id);
    }
```

##### 5.5 自定义数据库查询接口

如果想要根据年龄查询数据库中的女生，怎么做呢。首先在GirlRepository中声明一个接口findByAge(注意这种格式)：

```
public interface GirlRepository extends JpaRepository<Girl,Integer> {

    /**
     * 通过年龄查询
     * @param age
     * @return
     */
    public List<Girl> findByAge(Integer age);
}
```
然后在GirlController中添加根据年龄查询的函数即可：

```
/**
     * 通过年龄查询女生列表
     * 需要在GirlRepositiry中声明接口
     * @param age
     * @return
     */
    @GetMapping(value = "/girls/age/{age}")
    public List<Girl> girlFindByAge(@RequestParam("age") Integer age){
        return girlRepository.findByAge(age);

    }
```

### 六、事务管理

作为单个逻辑单元执行的一系列的操作，要么完全执行，要么完全不执行
比如插入两个女生信息，要么同时成功，要么就同时都不成功；
创建一个Service,在里面声明GirlRepository，

```
@Service
public class GirlService{
    @Autiwired
    private GirlRepository girlRepository;


    @Transactional  
    public void insertTwo(){
        Girl girlA=new Girl();
        girlA.setCupSize("A");
        girlA.setAge(18);
        girlRepository.save(girlA);
        
        Girl girlB=new Girl();
        girlB.setcupSize("B");
        girlB.setAge(19);
        girlRepository.save(girlB);
        
    }
}
```
加入的注解@Transactional  为事务注解，表示要么同时成功要么同时失败。不可能只成功一个
然后在GirlController中添加：

```
@Autowired 
private GirlService girlService；

@postMapping(value="/girls/two")
public void girlTwo(){
    girlService.insertTwo();
}
```
此时在浏览器输入`localhost:8080/girls/two`即可插入insertTwo()中的两个数据。要么同时成功，要么同时失败。




















