---
title: Java之登陆验证功能
date: 2018-01-14 20:01:53
tags: java
---
# 登录验证
------
参考：
[https://www.zhihu.com/question/20085241](https://www.zhihu.com/question/20085241)
[http://blog.csdn.net/u012920206/article/details/52518622](http://blog.csdn.net/u012920206/article/details/52518622)
先说一下需求：

* 首先可以实现用户名密码的登录当然数据传输需要进行加密传输（推荐使用sha256）；
* 用户登录后，需要保存用户的登录状态，以便请求后续功能（可以考虑sessionID，也可结合Token）；
* 用户每次请求其它功能时，提前予以拦截验证（和具体功能函数解耦）；

### 一、如何在java中创建session
1.1 使用例子

首先你需要导入javax.servlet.http这个包，对应的jar文件是servlet-api.jar，在此基础上调用该包里面的HttpSession就能实例化session了

```
//创建session
HttpSession session=ServletActionContext.getRequest().getSession();
//保存
ActionContext.getContext().getSession.put("msg","hello world from Session");
session.setAttribute("softtypeid",softtypeid);

//获取
if(session.getAttribute("softtypeid")!=null){
	if(!softtypeid.equals(session.getAttribute("softtypeid"))){
		pager_offset=1;//如果不是同一种分类，返回第一页
	}
}


HttpServletRequest request=ServletActionContext.getRequest();
HttpServletReponse response=ServletActionContext.getResponse();
HttpSession session=request.getSession();

```

1.2 session、和cookie

cookie特点：
 
* 会话数据存放在浏览器
* 数据类型只能是String，而且有大小限制
* 数据存放不安全

session特点： 

* 会话数据存放在服务器（服务器内存）
* 数据类型任意，没有大小限制
* 相对安全


cookie技术原理：

* 服务器创建cookie对象，保存会话数据，把cookie数据发送给浏览器
* 浏览器获取cookie数据，保存到浏览器缓存区，然后在下次访问服务器携带cookie数据
* 服务器获取浏览器发送的cookie数据


cookie简单使用

```
	Cookie cookie=new Cookie("name","nidie");//创建Cookie对象，保存会话数据
	response.addCookie(cookie);// 通过响应头携带cookie给浏览器
	//或者 response.setHeader("set-cookie","name=nidie");

	
	//浏览器下次访问的时候，在请求头将cookie发送给服务器
	


	//服务器获取浏览器发送的cookie
	//String name=request.getHeader("cookie");
	
	Cookie[] cookies=request.getcookies();
	for(Cookie cookie:cookies){
		String name=cookie.getName();
		String value=cookie.getValue();
	}
	

```

cookie细节

* cookie的数据类型一定是字符串，如果发送中文，需要通过URLEncoder进行加密，和URlDecoder进行解密。
* setPath(path):默认情况下，是当期项目的根目录下，可以通过setpath更改，如果把该cookie设置到某个有效路径下，只有当访问该有效路径的时候才会携带该cookie信息
* setMaxAge(整数): 设置cookie的有效时间。 
* * 正整数：表示超过了正该值cookie会丢失（cookie保存到浏览器的缓存中），单位：秒。
* * 负整数：表示如果浏览器关闭了，cookie就会消失（cookie保存在浏览器的内存中）
* * 0:表示删除同名的cookie
* cookie可以有多个，浏览器一般存放300个cookie，每个站点最多存放20个，每个cookie大小限制为4K


Session使用步骤

* 创建HttpSession对象，用于保存会话数据。 
request.getSession();//创建或获取session
* 修改HttpSession对象 
setMaxInactiveInterval（）;
* 保存会话数据（作为域对象） 
session.setAttribute(“name”, “yangqing”);

```
		//创建或者获取session对象

        HttpSession session = request.getSession();
        //修改session
        session.setMaxInactiveInterval(20);//20秒后session对象将要被销毁
        //保存会话数据（作为域对象）
        session.setAttribute("name", "yangqing");
```

session细节
* 1 setMaxInactiveInterval(秒)：设置session对象的有效时间 
* 2 设置JSESSIONID不会随着浏览器的关闭而关闭
* 3 通过invalidate()方法直接销毁session对象
* 4 request.getSession()//request.getSession(true):查询session对象，如果没有session对象，创建新的 
request.getSession(false):如果没有session对象，返回null。


## 二、sha256加密处理
参考：[http://blog.csdn.net/u012188107/article/details/69267054](http://blog.csdn.net/u012188107/article/details/69267054)

2.1 方式1

利用Apache的工具类实现：
maven添加依赖：

```
 <dependency>
      <groupId>commons-codec</groupId>
      <artifactId>commons-codec</artifactId>
      <version>${common-codec.version}</version>
  </dependency>
```
实现代码：

```

    /**
     *  利用Apache的工具类实现SHA-256加密
     * @param str 加密后的报文
     * @return
     */
    public static String getSHA256Str(String str){

        MessageDigest messageDigest;
        String encdeStr = "";
        try {
            messageDigest = MessageDigest.getInstance("SHA-256");
            byte[] hash = messageDigest.digest(str.getBytes("UTF-8"));
            encdeStr = Hex.encodeHexString(hash);
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return encdeStr;
    }
```

2.2 方式2
利用java自带的实现加密

```

   	/**
     *  利用java原生的摘要实现SHA256加密
     * @param str 加密后的报文
     * @return
     */
    public static String getSHA256StrJava(String str){
        MessageDigest messageDigest;
        String encodeStr = "";
        try {
            messageDigest = MessageDigest.getInstance("SHA-256");
            messageDigest.update(str.getBytes("UTF-8"));
            encodeStr = byte2Hex(messageDigest.digest());
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return encodeStr;
    }

    /**
     * 将byte转为16进制
     * @param bytes
     * @return
     */
    private static String byte2Hex(byte[] bytes){
        StringBuffer stringBuffer = new StringBuffer();
        String temp = null;
        for (int i=0;i<bytes.length;i++){
            temp = Integer.toHexString(bytes[i] & 0xFF);
            if (temp.length()==1){
                //1得到一位的进行补0操作
                stringBuffer.append("0");
            }
            stringBuffer.append(temp);
        }
        return stringBuffer.toString();
    }
```
