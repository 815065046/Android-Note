�����Test���£�

һ�������÷�

1����������URL

void loadUrl(String url)  

���������Ҫ����url����Ӧ����ҳ��ַ���������ڵ�����ҳ�е�ָ����JS����,����һ�����ע����ǣ�loadUrl()���������߳���ִ�У���������ͻᱨ��������ע�⣺����������ҳ��ַ�ǻ��õ�����permissionȨ�޵ģ�������Ҫ��AndroidManifest.xml��д�������������Ȩ�ޣ�

<uses-permission android:name="android.permission.INTERNET" />  


���֣�


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
            android:text="����URL"/>  
  
    <WebView  
            android:id="@+id/webview"  
            android:layout_width="match_parent"  
            android:layout_height="match_parent"/>  
</LinearLayout>  


��Ҫ���룺


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

		//������ҳ
                mWebView.loadUrl("http://www.baidu.com");  
            }  
        });  
    }  
}  



���������������Ĵ��룬Ч��ȴ�����������������ַ��ȴ����ʹ��webview����ַ��

���Ҫ��Ӧ���д��Ǿ�Ҫ����WebViewClient��


2�����ر���URL

һ����ԣ����ǻὫ����html�ļ�����assets�ļ�����
ֻ��Ҫ��loadurl���������Ϊ mWebView.loadUrl("file:///android_asset/web.html"); 

3��WebView��������

���������Ҫ����WebView�����ԣ���ͨ��WebView.getSettings()��ȡ����WebView��WebSettings����,Ȼ�����WebSettings�еķ�����ʵ�ֵġ� 
���webView����Ҫ�û��ֶ������û������������������webview��������֧�ֻ�ȡ���ƽ���

webview.requestFocusFromTouch();  



����JS����Java����

����һ��

����JS�������JAVA���룬��Ҫ���õ�WebView�����һ��������
public void addJavascriptInterface(Object obj, String interfaceName)  

�������������������
Object obj��interfaceName���󶨵Ķ���
String interfaceName�����󶨵Ķ�������Ӧ������
�������������WebViewע��һ��interfaceName�Ķ����������󶨵���obj����ͨ��interfaceName�Ϳ��Ե���obj�����еķ�����

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

��������Ҫ�ǵ�������䣺
mWebView.addJavascriptInterface(this, "android");  

������˼�ǰ�MyActivity����ע�뵽WebView�У���WebView�еĶ��������android�����⣬���ǻ���MyActivity�ж���д��һ������Comm(String message)�����ڵ���MSG;

��������
��ʽ2��ͨ�� WebViewClient �ķ���shouldOverrideUrlLoading ()�ص����� url��
����ԭ�� 
Androidͨ�� WebViewClient �Ļص�����shouldOverrideUrlLoading ()���� url
������ url ��Э��
�����⵽��Ԥ��Լ���õ�Э�飬�͵�����Ӧ���� 
��JS��Ҫ����Android�ķ���

<!DOCTYPE html>
<html>

   <head>
      <meta charset="utf-8">
      <title>Carson_Ho</title>

     <script>
         function callAndroid(){
            /*Լ����urlЭ��Ϊ��js://webview?arg1=111&arg2=222*/
            document.location = "js://webview?arg1=111&arg2=222";
         }
      </script>
</head>

<!-- �����ť�����callAndroid��������  -->
   <body>
     <button type="button" id="button1" onclick="callAndroid()">�������Android����</button>
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

        // ������Js������Ȩ��
        webSettings.setJavaScriptEnabled(true);
        // ��������JS����
        webSettings.setJavaScriptCanOpenWindowsAutomatically(true);

        // ����1������JS����
        // ��ʽ�涨Ϊ:file:///android_asset/�ļ���.html
        mWebView.loadUrl("file:///android_asset/javascript.html");


// ��дWebViewClient���shouldOverrideUrlLoading����
mWebView.setWebViewClient(new WebViewClient() {
                                      @Override
                                      public boolean shouldOverrideUrlLoading(WebView view, String url) {

                                          // ����2������Э��Ĳ������ж��Ƿ�������Ҫ��url
                                          // һ�����scheme��Э���ʽ�� & authority��Э�������жϣ�ǰ����������
                                          //�ٶ���������� url = "js://webview?arg1=111&arg2=222"��ͬʱҲ��Լ���õ���Ҫ���صģ�

                                          Uri uri = Uri.parse(url);                                 
                                          // ���url��Э�� = Ԥ��Լ���� js Э��
                                          // �ͽ������½�������
                                          if ( uri.getScheme().equals("js")) {

                                              // ��� authority  = Ԥ��Լ��Э����� webview������������Լ����Э��
                                              // ��������url,����JS��ʼ����Android��Ҫ�ķ���
                                              if (uri.getAuthority().equals("webview")) {

                                                 //  ����3��
                                                  // ִ��JS����Ҫ���õ��߼�
                                                  System.out.println("js������Android�ķ���");
                                                  // ������Э���ϴ��в��������ݵ�Android��
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

��ʽ3��ͨ�� WebChromeClient ��onJsAlert()��onJsConfirm()��onJsPrompt���������ص�����JS�Ի���alert()��confirm()��prompt���� 3333333
��Ϣ


����Android����JS����

��ʽ1��ͨ��WebView��loadUrl����

 mWebView.loadUrl("javascript:callJS()");

�ر�ע�⣺JS�������һ��Ҫ�� onPageFinished���� �ص�֮����ܵ��ã����򲻻���á�
onPageFinished()����WebViewClient��ķ�������Ҫ��ҳ����ؽ���ʱ����.

��ʽ2��ͨ��WebView��evaluateJavascript����

�ŵ㣺�÷����ȵ�һ�ַ���Ч�ʸ��ߡ�ʹ�ø���ࡣ
��Ϊ�÷�����ִ�в���ʹҳ��ˢ�£�����һ�ַ�����loadUrl ����ִ����ᡣ
Android 4.4 ��ſ�ʹ��

// ֻ��Ҫ����һ�ַ�����loadUrl()��������÷�������
    mWebView.evaluateJavascript��"javascript:callJS()", new ValueCallback<String>() {
        @Override
        public void onReceiveValue(String value) {
            //�˴�Ϊ js ���صĽ��
        }
    });
}

WebViewClient�к�������:

/** 
 * �ڿ�ʼ������ҳʱ��ص� 
 */  
public void onPageStarted(WebView view, String url, Bitmap favicon)   
/** 
 * �ڽ���������ҳʱ��ص� 
 */  
public void onPageFinished(WebView view, String url)  
/** 
 * ���� url ��ת,�������ӵ��������ת���߲��� 
 */  
public boolean shouldOverrideUrlLoading(WebView view, String url)  
/** 
 * ���ش����ʱ���ص��������п����������������������һ�Σ�������ʾ404�Ĵ���ҳ�� 
 */  
public void onReceivedError(WebView view, int errorCode,String description, String failingUrl)  
/** 
 * �����յ�https����ʱ����ص��˺����������п����������� 
 */  
public void onReceivedSslError(WebView view, SslErrorHandler handler,SslError error)  
/** 
 * ��ÿһ��������Դʱ������ͨ������������ص� 
 */  
public WebResourceResponse shouldInterceptRequest(WebView view,  
        String url) {  
    return null;  
}  

WebViewClient֮onPageStarted��onPageFinished

onPageStarted��֪ͨ������ҳ�浱ǰ��ʼ���ء��÷���ֻ���ڼ���main frameʱ����һ�Σ����һ��ҳ���ж��frame��onPageStartedֻ�ڼ���main frameʱ����һ�Ρ�Ҳ��ζ��������frame�����仯��onPageStarted���ᱻ���ã��磺��iframe�д�url���ӡ� 
onPageFinished��֪ͨ������ҳ����ؽ���������ֻ��main frame����һ�Ρ� 
���Ǿ�����������뷨���ٸ����ӣ���ʼ����ʱ��ʾ����ԲȦ����������ʱ���ؼ���ԲȦ

public boolean shouldOverrideUrlLoading(WebView view, String url)  
����������ڼ��س�����ʱ�ص�����������ͨ����дshouldOverrideUrlLoading������ʵ�ֶ���ҳ�г����ӵ����أ� 
����ֵ��boolean���ͣ���ʾ�Ƿ�����WebView��������URL��Ĭ����Ϊ����Ϊ���������WebView����URLǰ�ص��ģ������������return true����WebView�������Ͳ����ټ������URL�ˣ����д�����Ҫ��WebView�в������������ء��������return false����ϵͳ����Ϊ�ϲ�û�����������������ǻ�����������URL�ġ�WebViewClientĬ�Ͼ���return false��.

WebViewClient֮shouldInterceptRequest
k����ʵ�ֶ���Դ������

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

	//����ʵ��������Դ�Ĺ���

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



