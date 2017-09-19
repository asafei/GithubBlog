---
title: SpringBoot进阶
date: 2017-09-19 22:33:32
tags: SpringBoot
---

## 内容

* 1、使用@Valid表单验证
* 2、使用AOP处理请求
* 3、统一一场处理
* 4、单元测试

#### 一、表单验证

首先说一下git的使用，将项目clone下来之后，使用`git check -b web-2 web-2`就可以在本地使用分枝web-2了。
当参数过多的时候，在函数里面的参数很多就显得很业余，如果还要往里面添加属性，怎么办呢？可以将单个的属性参数换作一个类，这样的话就可以不更新函数了。
比如girlAdd方法原来是这样的，

```

    @PostMapping(value = "/girls")
    public Girl girlAdd(@RequestParam("cupSize") String cupSize,
                          @RequestParam("age") Integer age){

        Girl girl=new Girl();
        girl.setCupSize(cupSize);
        girl.setAge(age);

        return girlRepository.save(girl);
    }
```
可以改为

```
 @PostMapping(value = "/girls")
    public Girl girlAdd(Girl g){

        Girl girl=new Girl();
        girl.setCupSize(g.getCupSize());
        girl.setAge(g.getAge());

        return girlRepository.save(girl);
    }
```

如果想让未满18岁的少女插入不成功，怎么做呢，首先进入Girl类内部，在age的属性上添加注解@Min，value写入最小年龄，message写入错误信息，如下

```
@Min(value = 18, message = "未成年少女禁止入内")
    private Integer age;
```
然后在GirlController中的函数girlAdd参数钱加上注解@Valid即可，还要考虑到如何获取验证结果呢，这是虎需要用到BindingResult添加到参数重，如下

```
@PostMapping(value = "/girls")
    public Girl girlAdd(@Valid Girl g , BindingResult bindingResult){
        if (bindingResult.hasErrors()){
            System.out.println(bindingResult.getFieldError().getDefaultMessage());
        }

        Girl girl=new Girl();
        girl.setCupSize(g.getCupSize());
        girl.setAge(g.getAge());

        return girlRepository.save(girl);
    }
```

总结：表单验证，首先在类（Girl）中添加@Min注解，设置阈值和错误信息；在Controller的类（GirlController）中相应函数（girlAdd）的参数用@Valid注解响应的参数类（Girl g）；并且用BindingResult来获取验证信息。

#### 二、AOP统一处理请求日志

* AOP是一种编程范式
* * 与语言无关，是一种程序设计思想
* * 面向切面（AOP）Aspect Oriented Programming
* * 面向对象（OOP）Object Oriented Programming
* * 面向过程（POP）Procedure Oriented Programming

AOP将通用逻辑从业务逻辑中分离出来。

2.1

在获取请求之前判断用户是否已登陆，难道需要在每一个方法中都做判断吗？不需要。使用aop就可以解决。
首先在porm中添加依赖

```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-aop</artifactId>
		</dependency>
```
2.2

然后新建一个类HttpAspect添加注解@Aspect，并且添加@Component将这个类引入到spring容器中去。然后在里面使用@Before注解，规定要拦截的函数，这样就可以在每次调用这个函数之前都做响应处理了。如果是想对所有的函数度做拦截，可以将@Before中的函数换作*；如果想让函数执行之后再调用某个方法，可以使用@After注解

```
@Aspect
@Component
public class HttpAspect {

    /**
     * 拦截girlList方法
     * girllist只要是这个方法，不论是任何参数都会被拦截。
     * 在girlList放置执行之前被拦截
     *
     * 如果想拦截所有的方法，只需要吧girlList换为*即可
     */
    @Before("execution(public * com.afei.controller.GirlController.girlList())")
    public void log(){
        System.out.println("1111111111");
    }
    
    /**
     * 同理这个方法，会在拦截函数执行之后触发
     */
    @After("execution(public * com.afei.controller.GirlController.girlList())")
    public void doAfter(){
        System.out.println("222222222");
    }
}
```
2.3

可是这时候再@Before和@After之中的拦截函数有重复，这时候可以做以下改进：

```
@Aspect
@Component
public class HttpAspect {

    @Pointcut("execution(public * com.afei.controller.GirlController.girlList())")
    public void log(){
    }

    /**
     * 拦截girlList方法
     * girllist只要是这个方法，不论是任何参数都会被拦截。
     * 在girlList放置执行之前被拦截
     *
     * 如果想拦截所有的方法，只需要吧girlList换为*即可
     */
    @Before("execution(public * com.afei.controller.GirlController.girlList())")
    public void doBefore(){
        System.out.println("1111111111");
    }

    /**
     * 同理这个方法，会在拦截函数执行之后触发
     */
    @After("execution(public * com.afei.controller.GirlController.girlList())")
    public void doAfter(){
        System.out.println("222222222");
    }
}
```
2.4

通常使用System.out.println()方法不如使用日志打印，可以使用` private final static Logger logger= LoggerFactory.getLogger(HttpAspect.class);
`来替代，只需要在相应的地方打印信息`logger.info("222222222"));`即可

2.5 如何获取方法的http请求参数呢

只需要在HttpAspect类中添加

```
@Before("log()")
    public void doBefore(JoinPoint joinPoint){
        //System.out.println("1111111111");
        //logger.info("1111111111");
        ServletRequestAttributes attributes= (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request=attributes.getRequest();

        //获取http请求的内容

        logger.info("url={}",request.getRequestURI());

        logger.info("method={}",request.getMethod());

        logger.info("ip={}",request.getRemoteAddr());

        logger.info("class_method={}",joinPoint.getSignature().getDeclaringTypeName());

        logger.info("args={}",joinPoint.getArgs());
    }
```

如何获取http请求的结果呢，需要使用到@AfterReturning的注解，

```
 //获取请求返回的结果 @AfterReturning(returning="object" ,pointcut = "log()")
    public void doAfterReturning(Object object){
        logger.info("response={}",object);
    }
```

#### 三、统一异常处理

 需求：
 获取某个女生的年龄并判断
 小于10，返回“应该上小学”
 大于10且小于16，返回“可能在上高中”
 
 如何在有异常发生时，按照指定的数据格式处理回应呢

 3.1 异常的捕获处理：

 首先新建一个类ExceptionHandle用@ControllerAdvice注解，在后在里面声明一个方法handler并且用@ExceptionHandler注解，value值为Exception.class，和@ReponseBody注解。
 
 ```
 @ControllerAdvice
public class ExceptionHandle {
    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public Result handler(Exception e){
        return ResultUtil.error(100,e.getMessage());
    }
 ```
 然后在一些位置抛异常，比如girlService中的getAge方法
 ```
 public void getAge(Integer id) throws Exception{
        Girl girl=girlRepository.findOne(id);
        Integer age=girl.getAge();
        if (age<10){
            //
            throw new Exception("你还在上小学吧");
        }else if(age>10 && age<16){
            throw new Exception("你可能上初中");
        }

    }
 ```
 这样抛的异常就全部交给了ExceptionHandle来处理,此时用户再传入相应的参数的就可以按照指定的Result的方式来输出了。
 
 3.2 自定义异常

 如果我想你上小学的时候返回100，上初中的时候返回101,但是抛异常时又没有自定义错误码的方法，这时就需要我们自定义异常类了。
 新建一个GirlException的类继承自RuntimeException。spring只对RuntimeException进行回滚，对Exception是不会进行回滚的。在GirlException中增加错误码的属性：
 
 ```
 public class GirlException extends RuntimeException{

    private Integer code;
    public GirlException(Integer code,String message){
        super(message);
        this.code=code;
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }
}
 ```
 然后就可以在GirlService中抛异常时使用自定义的异常了：
 
 ```
 public void getAge(Integer id) throws Exception{
        Girl girl=girlRepository.findOne(id);
        Integer age=girl.getAge();
        if (age<10){
            //
            throw new GirlException(100,"你还在上小学吧");
        }else if(age>10 && age<16){
            throw new GirlException(101,"你可能上初中");
        }

    }
 ```
 最后在ExceptionHandle类中的handler方法中对GirlException进行处理：
 
 ```
 @ControllerAdvice
public class ExceptionHandle {
    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public Result handler(Exception e){

        if (e instanceof GirlException){
            GirlException girlException=(GirlException) e;
            return ResultUtil.error(girlException.getCode(),girlException.getMessage());
        }
        return ResultUtil.error(100,e.getMessage());
    }
}
 ```
 此时传入相应的请求就会安规定返回相应的数据了.
 要过要抛出系统异常，但是如何跟踪异常呢，这时需要使用日志工具了：
 
```
 @ControllerAdvice
public class ExceptionHandle {
    private final static Logger logger= LoggerFactory.getLogger(Exception.class);

    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public Result handler(Exception e){

        if (e instanceof GirlException){
            GirlException girlException=(GirlException) e;
            return ResultUtil.error(girlException.getCode(),girlException.getMessage());
        }else{
            logger.error("[系统异常]{}",e);
            return ResultUtil.error(-1,"未知错误");
        }

    }
}
```
这样发生系统异常时，就可以在控制台打印消息了

额外的错误码管理

此时异常捕获已经差不多，有个小地方可以优化，就是错误码管理，可以将code和msg用枚举进行管理：

```
public enum ResultEnum {

    UNKONW_ERROR(-1,"未知错误"),
    SUCCESS(0,"成功"),
    PROMARY_SCHOOL(100,"你在上小学吧"),
    MIDDLE_SCHOOL(101,"你可能在上初中");


    private Integer code;
    private String msg;

    ResultEnum(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public Integer getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }
}
```
然后在GirlService中修改相应的异常：

```
public void getAge(Integer id) throws Exception{
        Girl girl=girlRepository.findOne(id);
        Integer age=girl.getAge();
        if (age<10){
            //
            throw new GirlException(ResultEnum.PROMARY_SCHOOL);
        }else if(age>10 && age<16){
            throw new GirlException(ResultEnum.MIDDLE_SCHOOL);
        }

    }
```
此时会有错误，将GirlException中的构造函数进行改造:

```
    public GirlException(ResultEnum resultEnum){
        super(resultEnum.getMsg());
        this.code=resultEnum.getCode();
    }
```

#### 四、单元测试

首先对要测试的类（如GirlService）中添加一个可以用的方法，

```
………………
```
然后在test目录下床及亲一个测试类用@RunWith(SpringRunner.class)注解，还需要加上@SpringBootTest的注解.然后使用@Autowired注入GirlService，就可以写测试函数了

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class GirlServiceTest{
	@Autowired
	private GirlService girlService;
	
	@Test
	public void findOneTest(){
		Girl girl=girlService.findOne(73);
		Assert.assertEquals(new Integer(13),girl.getAge());
	}
}
```

也可以使用ide自动生成，gotoTest，但是还需要自己添加相应的注解

如何给api进行测试呢，希望想postman一样使用url测试，这时需要给类额外用到@AutoConfigureMockTest注解：

```
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockTest
public class GirlControllerTest{
	@Autowired
	private MockMvc mvc;
	
	@Test
	public void girlList() throws Exception{
		 //get 使用get方法，post使用post方法
		mvc.perform(MockMvcRequestBuilders.get("/girls")).andExcept(MockMvcResultMatchers.status().isOk())
		.andExcept(MockMvcResultMatchers.content().string("abc"));//判断内容
	}
}

```





