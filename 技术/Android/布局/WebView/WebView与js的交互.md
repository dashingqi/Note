##WebView与Js的交互

> WebView与js的交互，实则就是Android与JS之间的交互
>
> 只不过WebView是两者之间调用的桥梁

#### 交互方式

###### Android调用JS

- WebView的loadUrl()方法

  - 该方法去调用JS的代码，一定要在onPageFinished()方法结束之后在调用。否则无效！

  - 在 main下新建assets文件夹 将html文件放入其中

    ```html
    <!DOCTYPE html>
    <html>
    
    <head>
        <meta charset="utf-8">
        <title>heihei</title>
    
    
        <script>
         // Android需要调用的方法
         function callJS(){
            alert("Android调用了JS的callJS方法");
            return 1;
         }
        </script>
    
    </head>
    <body>
    // JS代码
    这里是body代码
    </body>
    
    </html>
    ```

    

    ```java
     mWebView = findViewById(R.id.web_view);
            mCalljs = findViewById(R.id.btn_call_js);
    
            //获取到WebSettings
            mSettings = mWebView.getSettings();
            //开启与JS的交互
            mSettings.setJavaScriptEnabled(true);
            //设置允许JS的弹窗
            mSettings.setJavaScriptCanOpenWindowsAutomatically(true);
            //加载html文件
            mWebView.loadUrl("file:///android_asset/javascript.html");
    
            mCalljs.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    mWebView.post(new Runnable() {
                        @Override
                        public void run() {
                            //调用JS中的代码
                            mWebView.loadUrl("javascript:callJS()");
                        }
                    });
                }
            });
    
            //WebView只是载体，真正的内容渲染需要WenChromeClient来完成
            //通过onJsAlert()函数来响应JS的alert函数
            //拿到alert的数据
            mWebView.setWebChromeClient(new WebChromeClient() {
                @Override
                public boolean onJsAlert(WebView view, String url, String message, final JsResult result) {
                    AlertDialog.Builder b = new AlertDialog.Builder(MainActivity.this);
                    b.setTitle("Alert");
                    b.setMessage(message);
                    b.setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
                        @Override
                        public void onClick(DialogInterface dialog, int which) {
                            result.confirm();
                        }
                    });
                    b.setCancelable(false);
                    b.create().show();
                    return true;
                }
            });
    ```

    

  

- WebView的evaluateJavaScript()方法

  - 该方式的调用，效率更高。

  - 该方式去执行代码不会使得页面刷新，loadUrl()会刷新页面

  - Android4.4之后才能使用

    ```java
    //替换loadUrl("javascript:callJS()")
    mWebView.evaluateJavascript("javascript:callJS()", new ValueCallback<String>() {
                                @Override
                                public void onReceiveValue(String value) {
                                    //此处为调用JS方法的返回值。
                                  	//比如调用的JS方法内，返回了 此处打印就是1
                                    Log.d(TAG, "onReceiveValue: value = " + value);
                                }
                            });
    ```

- 以上两种方法的使用建议

  - Android4.4以下使用第一种。
  - Android4.4以上版本的使用第二种。
  - 主要是做下手机版本的判断。



###### JS调用Android

- WebView的addJavaScriptInterface() 进行对象的映射

  - 定义一个与JS对象映射关系的Android类：AndroidToJS

    ```java
    /**
     * 定义哥映射类必须继承Object
     */
    public class AndroidToJS extends Object {
        private static final String TAG = "AndroidToJS";
    
        /**
         * 定义JS需要调用的方法
         * 被JS调用的方法必须加入@JavascriptInterface注解
         * @param msg
         */
        @JavascriptInterface
        public void hello(String msg) {
            Log.d(TAG, "JS调用了Android的Hello方法");
        }
    }
    ```

    

  - 将需要加载的JS代码以.html格式放到assets文件夹里

    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8">
        <title>heihei</title>
        <script>
    
    
             function callAndroid(){
            // 由于对象映射，所以调用test对象等于调用Android映射的对象
                test.hello("js调用了android中的hello方法");
             }
    
        </script>
    </head>
    <body>
    //点击按钮则调用callAndroid函数
    <button type="button" id="button1" onclick="callAndroid()"></button>
    </body>
    </html>
    ```

    

  - 在Android里通过WebView设置Android类与JS代码的映射

    ```java
     @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            mWebView = findViewById(R.id.web_view);
            mSettings = mWebView.getSettings();
            //设置与JS交互
            mSettings.setJavaScriptEnabled(true);
    
            //通过addJavaScriptInterface()将java对象映射到JS对象
            //将AndroidToJS类对象映射到js的test对象
            mWebView.addJavascriptInterface(new AndroidToJS(), "test");
    
            //加载html文件
            mWebView.loadUrl("file:///android_asset/javascript.html");
        }
    ```

    

  - 有点儿

    - 使用起来简单

  - 缺点

    - 存在严重的漏洞

- WebViewClient的shouldOverrideLoading()方法，在回调方法里面进行拦截url

  - Android通过WebViewClient的shouldOverrideUrlLoading()拦截url，解析这个url如果该url是我们提前预定好的协议，就调用相应的方法（JS调用Android方法）

  - 在JS代码里面约定需要的Url协议

    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="utf-8">
        <title>heihei</title>
        <script>
             function callAndroid(){
                //下面为约定好的协议
                document.location = "js://webview?arg1=111&arg2=222";
             }
    
        </script>
    </head>
    <body>
    <button type="button" id="button1" onclick="callAndroid()" value="点击按钮则调用callAndroid函数">
        点击按钮则调用callAndroid函数
    </button>
    </body>
    </html>
    ```

    

  - 复写WebViewClient的shouldOverrideUrlLoading()方法在里面解析url，看是否是约定好的url协议

    ```java
     /**
         * 通过重写WebViewClient的shouldOverrideUrlLoading方法
         * 判断返回的url的协议，来进行Android代码的调用
         */
        private void js2Android2() {
            //1. 加载html代码
            mWebView.loadUrl("file:///android_asset/js2.html");
            mWebView.setWebViewClient(new WebViewClient() {
                @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
                @Override
                public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
                    Uri uri = request.getUrl();
                    //2. 判断协议格式
                    if (uri.getScheme().equals("js")) {
                        //判断协议名字
                        if (uri.getAuthority().equals("webview")) {
                            //3. 执行JS所需要调用的逻辑
                            Log.d(TAG, "js 调用 Android的代码");
                            Set<String> queryParameterNames = uri.getQueryParameterNames();
                            for (String str : queryParameterNames) {
                                Log.d(TAG, "str = " + str);
                            }
                        }
                        return true;
    
                    }
    
                    return super.shouldOverrideUrlLoading(view, request);
                }
            });
        }
    ```

  - 特点

    - 不存在方式1的漏洞

    - JS获取到Android方法的返回值复杂

      - 如果JS想要得到Android方法的返回值，只能执行Android调用JS的过程，通过loadUrl方法去执行的代码将返回值传递回去。

        ```java
        //html文件中，写一个用于接收数据的JS方法。
        function getResult(result){
          //处理这个result
        }
        
        //在Android中调用loadUrl()执行该方法将返回值传递回去。
        webView.loadUrl("js2:getResult(""+result+"")");
        //进行如上操作就可以了。
        ```

        

- WebChromeClient的onJsAlert()、onJsConfirm()、onJsPrompt()方法回调拦截JS的alert()、confirm()、prompt()

  - alert

    - 弹出警告框
    - 没有返回值

  - confirm

    - 弹出确认框
    - 有两个返回值
    - 返回boolean值，通过该值可以确认点击的是确认还是取消

  - prompt

    - 弹出输入框
    - 可以设置任意返回值
    - 点击确认，返回输入框中的值，点击取消返回null

  - 该方式的原理就是通过复写WenChromeClient的onJsAlert拦截alert方法，复写onJsConfirm拦截confirm方法，复写onJsPrompt()拦截prompt方法，从而得到他们的消息内容然后进行解析。

  - 代码实操： 拦截js的prompt()方法 也就是输入框

    - 将加载的html文件放入到assets文件中

      ```html
      <!DOCTYPE html>
      <html>
      <head>
          <meta charset="utf-8">
          <title>hahaha</title>
          <script>
      		function clickprompt(){
          	// 调用prompt（）
          	var result=prompt("js://demo?arg1=111&arg2=222");
          	alert("demo " + result);
      		} 
          </script>
      </head>
      
      <!-- 点击按钮则调用clickprompt()  -->
      <body>
      <button type="button" id="button1" onclick="clickprompt()">点击调用Android代码</button>
      </body>
      </html>
      ```

    - 在Android中通过WebChromeClient复写onJsPrompt()方法

      - 当我们加载上面的html文件，点击按钮会触发prompt方法，此时我们Android通过复写WebChromeClient中的onJsPrompt()回拦截到本次调用

      ```java
      private void js2Android3() {
              mWebView.loadUrl("file:///android_asset/js3.html");
              mWebView.setWebChromeClient(new WebChromeClient() {
                  @Override
                  public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
                      Uri uri = Uri.parse(message);
                      //判断协议格式
                      if (uri.getScheme().equals("js")) {
                          //协议的名称相同
                          if (uri.getAuthority().equals("demo")) {
                              //拦截url，下面JS开始调用Android需要的方法
                              System.out.println("JS调用了Android方法");
                          }
                          return true;
                      }
      
                      return super.onJsPrompt(view, url, message, defaultValue, result);
                  }
              });
          }
      ```

