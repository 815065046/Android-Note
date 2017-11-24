代码见Test包下：

一、基本用法

1、加载在线URL

void loadUrl(String url)  

这个函数主要加载url所对应的网页地址，或者用于调用网页中的指定的JS方法,但有一点必须注意的是：loadUrl()必须在主线程中执行！！！否则就会报错！！！。注意：加载在线网页地址是会用到联网permission权限的，所以需要在AndroidManifest.xml中写入下面代码申请权限：

<uses-permission android:name="android.permission.INTERNET" />  


布局：


<?xml version="1.0" encoding="utf-8"?>  
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
              android:orientation="vertical"  
              android:layout_width="fill_parent"  
              android:layout_height="fill_parent"  
        >  
    <Button  
            android:id="@+id/btn"  
            android:layout_width="match_parent"  
            android:layout_height="wrap_content"  
            android:text="加载URL"/>  
  
    <WebView  
            android:id="@+id/webview"  
            android:layout_width="match_parent"  
            android:layout_height="match_parent"/>  
</LinearLayout>  


主要代码：


public class MyActivity extends Activity {  
  
    private WebView mWebView;  
    private Button mBtn;  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
  
        mWebView = (WebView)findViewById(R.id.webview);  
        mBtn = (Button)findViewById(R.id.btn);  
  
        mBtn.setOnClickListener(new View.OnClickListener() {  
            @Override  
            public void onClick(View v) {  

		//加载网页
                mWebView.loadUrl("http://www.baidu.com");  
            }  
        });  
    }  
}  



如果我们运行上面的代码，效果却是利用浏览器来打开网址，却不是使用webview打开网址：

如果要在应用中打开那就要设置WebViewClient；


2、加载本地URL

一般而言，我们会将本地html文件放在assets文件夹下
只需要将loadurl函数里面改为 mWebView.loadUrl("file:///android_asset/web.html"); 

3、WebView基本设置

如果我们需要设置WebView的属性，是通过WebView.getSettings()获取设置WebView的WebSettings对象,然后调用WebSettings中的方法来实现的。 
如果webView中需要用户手动输入用户名、密码或其他，则webview必须设置支持获取手势焦点

webview.requestFocusFromTouch();  



二、JS调用Java代码

方法一：

利用JS代码调用JAVA代码，主要是用到WebView下面的一个函数：
public void addJavascriptInterface(Object obj, String interfaceName)  

这个函数有两个参数：
Object obj：interfaceName所绑定的对象
String interfaceName：所绑定的对象所对应的名称
它有意义就是向WebView注入一个interfaceName的对象，这个对象绑定的是obj对象，通过interfaceName就可以调用obj对象中的方法。

public class MyActivity extends Activity {  
  
    private WebView mWebView;  
    private Button mBtn;  
  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
  
        mWebView = (WebView) findViewById(R.id.webview);  
  
        WebSettings webSettings = mWebView.getSettings();  
        webSettings.setJavaScriptEnabled(true);  
        mWebView.addJavascriptInterface(this, "android");  
        mWebView.loadUrl("file:///android_asset/web.html");  
    }  
  
     	 @JavascriptInterface
    	public void Comm(String message){
        Toast.makeText(getApplicationContext(),message,Toast.LENGTH_LONG).show();
    }  
} 

这里最主要是的下面这句：
mWebView.addJavascriptInterface(this, "android");  

这句的意思是把MyActivity对象注入到WebView中，在WebView中的对象别名叫android；另外，我们还在MyActivity中额外写了一个函数Comm(String message)，用于弹出MSG;

方法二：
方式2：通过 WebViewClient 的方法shouldOverrideUrlLoading ()回调拦截 url；
具体原理： 
Android通过 WebViewClient 的回调方法shouldOverrideUrlLoading ()拦截 url
解析该 url 的协议
如果检测到是预先约定好的协议，就调用相应方法 
即JS需要调用Android的方法

<!DOCTYPE html>
<html>

   <head>
      <meta charset="utf-8">
      <title>Carson_Ho</title>

     <script>
         function callAndroid(){
            /*约定的url协议为：js://webview?arg1=111&arg2=222*/
            document.location = "js://webview?arg1=111&arg2=222";
         }
      </script>
</head>

<!-- 点击按钮则调用callAndroid（）方法  -->
   <body>
     <button type="button" id="button1" onclick="callAndroid()">点击调用Android代码</button>
   </body>


public class MainActivity extends AppCompatActivity {

    WebView mWebView;
//    Button button;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mWebView = (WebView) findViewById(R.id.webview);

        WebSettings webSettings = mWebView.getSettings();

        // 设置与Js交互的权限
        webSettings.setJavaScriptEnabled(true);
        // 设置允许JS弹窗
        webSettings.setJavaScriptCanOpenWindowsAutomatically(true);

        // 步骤1：加载JS代码
        // 格式规定为:file:///android_asset/文件名.html
        mWebView.loadUrl("file:///android_asset/javascript.html");


// 复写WebViewClient类的shouldOverrideUrlLoading方法
mWebView.setWebViewClient(new WebViewClient() {
                                      @Override
                                      public boolean shouldOverrideUrlLoading(WebView view, String url) {

                                          // 步骤2：根据协议的参数，判断是否是所需要的url
                                          // 一般根据scheme（协议格式） & authority（协议名）判断（前两个参数）
                                          //假定传入进来的 url = "js://webview?arg1=111&arg2=222"（同时也是约定好的需要拦截的）

                                          Uri uri = Uri.parse(url);                                 
                                          // 如果url的协议 = 预先约定的 js 协议
                                          // 就解析往下解析参数
                                          if ( uri.getScheme().equals("js")) {

                                              // 如果 authority  = 预先约定协议里的 webview，即代表都符合约定的协议
                                              // 所以拦截url,下面JS开始调用Android需要的方法
                                              if (uri.getAuthority().equals("webview")) {

                                                 //  步骤3：
                                                  // 执行JS所需要调用的逻辑
                                                  System.out.println("js调用了Android的方法");
                                                  // 可以在协议上带有参数并传递到Android上
                                                  HashMap<String, String> params = new HashMap<>();
                                                  Set<String> collection = uri.getQueryParameterNames();

                                              }

                                              return true;
                                          }
                                          return super.shouldOverrideUrlLoading(view, url);
                                      }
                                  }
        );
   }
        } 

方式3：通过 WebChromeClient 的onJsAlert()、onJsConfirm()、onJsPrompt（）方法回调拦截JS对话框alert()、confirm()、prompt（） 3333333
消息


三、Android调用JS代码

方式1：通过WebView的loadUrl（）

 mWebView.loadUrl("javascript:callJS()");

特别注意：JS代码调用一定要在 onPageFinished（） 回调之后才能调用，否则不会调用。
onPageFinished()属于WebViewClient类的方法，主要在页面加载结束时调用.

方式2：通过WebView的evaluateJavascript（）

优点：该方法比第一种方法效率更高、使用更简洁。
因为该方法的执行不会使页面刷新，而第一种方法（loadUrl ）的执行则会。
Android 4.4 后才可使用

// 只需要将第一种方法的loadUrl()换成下面该方法即可
    mWebView.evaluateJavascript（"javascript:callJS()", new ValueCallback<String>() {
        @Override
        public void onReceiveValue(String value) {
            //此处为 js 返回的结果
        }
    });
}

WebViewClient中函数概述:

/** 
 * 在开始加载网页时会回调 
 */  
public void onPageStarted(WebView view, String url, Bitmap favicon)   
/** 
 * 在结束加载网页时会回调 
 */  
public void onPageFinished(WebView view, String url)  
/** 
 * 拦截 url 跳转,在里边添加点击链接跳转或者操作 
 */  
public boolean shouldOverrideUrlLoading(WebView view, String url)  
/** 
 * 加载错误的时候会回调，在其中可做错误处理，比如再请求加载一次，或者提示404的错误页面 
 */  
public void onReceivedError(WebView view, int errorCode,String description, String failingUrl)  
/** 
 * 当接收到https错误时，会回调此函数，在其中可以做错误处理 
 */  
public void onReceivedSslError(WebView view, SslErrorHandler handler,SslError error)  
/** 
 * 在每一次请求资源时，都会通过这个函数来回调 
 */  
public WebResourceResponse shouldInterceptRequest(WebView view,  
        String url) {  
    return null;  
}  

WebViewClient之onPageStarted与onPageFinished

onPageStarted：通知主程序页面当前开始加载。该方法只有在加载main frame时加载一次，如果一个页面有多个frame，onPageStarted只在加载main frame时调用一次。也意味着若内置frame发生变化，onPageStarted不会被调用，如：在iframe中打开url链接。 
onPageFinished：通知主程序页面加载结束。方法只被main frame调用一次。 
我们就利用上面的想法来举个例子：开始加载时显示加载圆圈，结束加载时隐藏加载圆圈

public boolean shouldOverrideUrlLoading(WebView view, String url)  
这个函数会在加载超链接时回调过来；所以通过重写shouldOverrideUrlLoading，可以实现对网页中超链接的拦截； 
返回值是boolean类型，表示是否屏蔽WebView继续加载URL的默认行为，因为这个函数是WebView加载URL前回调的，所以如果我们return true，则WebView接下来就不会再加载这个URL了，所有处理都需要在WebView中操作，包含加载。如果我们return false，则系统就认为上层没有做处理，接下来还是会继续加载这个URL的。WebViewClient默认就是return false的.

WebViewClient之shouldInterceptRequest
k可以实现对资源的拦截

public class MyActivity extends Activity {  
    private WebView mWebView;  
  
    private ProgressDialog mProgressDialog;  
    private String TAG = "qijian";  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
  
        mWebView = (WebView)findViewById(R.id.webview);  
        mProgressDialog = new ProgressDialog(this);  
        mWebView.getSettings().setJavaScriptEnabled(true);  
  
  
        mWebView.setWebViewClient(new WebViewClient(){  
            @Override  
            public WebResourceResponse shouldInterceptRequest(WebView view, String url) {  
                try {  
                    if (url.equals("http://localhost/qijian.png")) {  
                        AssetFileDescriptor fileDescriptor =  getAssets().openFd("s07.jpg");  
                        InputStream stream = fileDescriptor.createInputStream();  
                        WebResourceResponse response = new WebResourceResponse("image/png", "UTF-8", stream);  
                        return response;  
                    }  
                }catch (Exception e){  
                    Log.e(TAG,e.getMessage());  
                }  
                return super.shouldInterceptRequest(view, url);  
            }  
        });  
  
        mWebView.loadUrl("file:///android_asset/web.html");  
}   

	//可以实现拦截资源的功能

          @Override
            public WebResourceResponse shouldInterceptRequest(WebView view, String url) {
                WebResourceResponse webResourceResponse=null;
                try{
                    AssetFileDescriptor fileDescriptor =  getAssets().openFd("ic_launcher.png");
                    InputStream inputStream=fileDescriptor.createInputStream();
                     webResourceResponse=new WebResourceResponse("img/png","UTF-8",inputStream);
                    
                }catch (IOException e){

                }

                return webResourceResponse;
            }



