

## 本文主要从如下几点介绍WebView

- WebView是啥
- WebView在Android中起到什么作用
- WebView的使用介绍
- WebView的内存泄漏避免
- WebView的代码实操

#### WebView是啥

- Android中WebView是一个控件，用来展示Web页面的
- 它是基于Webkit引擎的
- 在Android4.4之后使用的webkit内核是chrome。

#### WebView在Android中起到什么作用

- 用来渲染和显示Web页面
- 直接使用html文件作布局
- 可以和JavaScript交互调用

#### WebView的使用

##### WebView的常用方法

###### WebView状态

- onResume()
  
  - 激活WebView为活跃状态，能正常执行网页的响应。
- onPause()
  - 当页面被切换或者切换到后台，当前页面就变成不可见了，需要执行onPause()
  - 通过onPause方法的执行，可以通知内核暂停所有动作
    - DOM的解析
    - plugin的执行
    - JavaScript的执行
- pauseTimers()
  - 当存在WebView的应用程序被切换到后台时，该方法不仅仅正对当前的webview，对于应用内的全局webview，它都会暂停所有的layout，parsing，JavaScriptTimer。
  - 主要就是降低CPU功耗。
- resumeTimers()
  
- 用来恢复pauseTimers状态
  
- destroy()

  - 销毁WebView
  - 当WebView调用了destroy()时，WebView还是绑定在Activity上
    - 因为自定义的WebView构建的时候传入了Actiivty的context对象。
    - 所以需要从父容器中移除WebView，然后在销毁WebView
    - 也就是如下代码操作
      - rootView.removeView(webView);
      - webView.destroy()
  - 所以当Activity销毁的时候，如果WebView还在播放着视频，就必须销毁WebView。不然就会发生内存泄漏。

  ###### 网页的前进和后退

  - goBack()
    - 后退网页
  - canGoBack()
    - 是否可以后退
  - goForward()
    - 前进页面
  - canGoForward()
    - 是否可以前进页面
  - goBackOrForward(int steps)
    - 以当前index为基准，向前或者向后跳转steps
    - steps为正就是向前
    - steps为负就是向后

  ###### 清除缓存数据

  - clearCache(true)
    - 清除网页访问留下的缓存
    - 内核的缓存时全局的，该方法不但针对当前WebView，而是针对整个应用程序。
  - clearHistroy()
    - 清除当前WebView的访问记录
    - 针对当前WebView的访问记录的。
  - clearFormData()
    - 这个api仅仅清除自动完成填充的表单数据，不会清除WebView存储到本地的数据。

##### 常用类

###### WebSettings

- 对WebView进行配置管理

- 获取到WebSettings的实例

  ```java
  //一般我们都是子xml文件中声明WebView
  WebView mWebView = findViewById(R.id.web_view);
  //获取到WebSettings的实例
  WebSettings mWebSettings = mWebView.getSettings();
  ```

- 使用WebSettings配置WebView

  ```java
  //访问的页面需要与JS进行交互。那么WebView需要支持JS
  mWebSettings.setJavaScriptEnabled(true);
  //支持插件
  mWebSettings.setPluginsEnabled(true);
  //设置自适应屏幕，两者合用
  mWebSettings.setUseWideViewPort(true)//将图片调整到适合WebView的大小
  mWebSettings.setLoadWithOverviewMode(true);//缩放到屏幕的大小
  // 缩放操作
  mWebSettings.setSupportZoom(true);//支持缩放，默认为true是下面那个的前提
  mWebSettings.setBuiltInZoomControls(true);//设置内置的缩放空间，如果为false，那么WebView不可缩放
  mWebSettings.setDisplayZoomControls(false);//隐藏原生的缩放控件
  
  //其他的操作
  mWebSettings.setCacheMode(WebSettings.LOAD_NO_CACHE);//不使用缓存
  mWebSettings.setAllowFileAccess(true);//设置可以访问文件
  mWebSettings.setJavaScriptCanOpenWindowsAutomatically(true);//支持通过JS打开新窗口
  mWebSettings.setLoadsImagesAutomatically(true);//支持自动加载图片
  mWebSettings.setDefaultTextEncodingName("utf-8");//设置编码格式
  ```

- 常见用法

  - 设置WebView缓存

    - 当加载html页面时，WebView会在/data/data/包名的目录下生成database与cache两个文件夹

    - 把请求的URL记录保存在WebViewCache.db，而URL的内容是保存在WebViewCache文件下

    - 是否启用缓存:

      ```java
      //优先使用缓存
      mWebViewSettings.setCache(mode)
      //该mode存在如下几种
      // LOAD_CACHE_ONLY: 不使用网络，只读取本地缓存
      // LOAD_DEFAULT: (默认) 根据cache-control决定是否从网络上获取数据
      // LOAD_NO_CACHE: 不使用缓存，只从网络获取数据
      // LOAD_CACHE_ELSE_NETWORK,只要本地有，无论是否过期，都使用混存中的数据
      ```

###### WebViewClient类

- 处理各种通知&请求事件

- 常见方法

  - shouldOverrideUrlLoading()

    - 该方法就是打开网页的时候不需要调用系统浏览器，而是在该WebView中显示。

      ```java
      //获取到WebView的组件
      WebView webView = findViewById(R.id.webView1);
      
      // 加载一个网页
      webView.loadUrl("http://...");
      //加载apk包中的html页面
      	webView.loadUrl("file:///... /name.html");
      //加载手机本地的html页面
      	webView.loadUrl("content:///... /name.html")
          
      // 通过复写shouldOverrideUrlLoading()方法，控制打开的网页不使用手机本地浏览器，是在该WebView中显示
         webView.setWebViewClient(new WebViewClient(){
           @Override
           public boolean shouldOverrideUrlLoading(WebView view,String url){
             view.load(url);
             return true;
           }
         })
      ```

  - onPageStarted()

    - 开始载入页面的时候调用，我们可以设定一个loading页面，告知用户正在开始加载

      ```java
      webView.setWebViewClient(new WebViewClient(){
      	@Override
        public void onPageStarted(WebView view,String url,Bitmap favicon){
          //设定加载开始的操作
        }
      })
      ```

  - onPageFinished()

    - 在页面加载结束时调用，我们可以关闭这个loading页面。

      ```java
      webView.setWebViewClient(new WebViewClient(){
        @Override
        public void onPageFinished(WebView view,String url){
          //设置加载结束的操作
        }
      })
      ```

  - onLoadResource()

    - 在加载页面资源时会调用，每一个资源的加载都会调用一次（比如图片）

      ```java
      webView.setWerbViewClient(new WebViewClient(){
      	@Override
        public boolean onLoadResource(WebView view,String url){
          //加载某个资源时的操作
        }
      })
      ```

  - onReceiveError()

    - 加载页面的服务器出现错误时调用

      - 比如我们加载一个页面出现404了，我们这是通过这个方法可以加载我们本地准备好的一个404html页面，进行加载

      ```java
      //步骤一：写一个html文件，用于出错时展示给用户看
      //步骤二:将该html文件，放置到assets文件夹下
      //步骤三: 通过复写onReceiveError方法，进行不同错误的处理
      	webView.setWebViewClient(new WebViewClient(){
      		@Override
          public void onReceivedError(WebView view,int errorCode,String description,String failingUrl){
            switch(errorCode){
              case 404:
                //进行一些列的骚操作
                view.loadUrl("path");
                break;
            }
          }
      	})
      ```

  - onReceivedSslError()

    - 处理https请求

      - WebView默认时不能处理https请求的，页面显示空白，可以进行如下操作

      ```java
      webView.setWebViewClient(new WebViewClient(){
      	@Override
        public voiud onReceivedSslError(WebView view,SslErrorHandler handler ,SslError error){
          //表示等待证书响应
          handler.proceed();
          // 表示挂起连接，为默认方式
          handler.cancel();
          //可做其他处理
          handler.handleMessage(null)
        }
      })
      ```

  ###### WebChromeClient类

  - 辅助WebView处理JavaScript的对话框，网站图标，网站标题等等。

  - 常见的使用方法

    - onProgressChanged()

      - 获取到网页加载进度并显示

        ```java
        webView.setWebChromeClient(new WebChromeClient(){
        	@Override
          public void onProgressChanged(WebView view ,int newProgress){
            //更新进度
          }
        })
        ```

    - onReceivedTitle()

      - 获取到加载网页中的标题

      - 每个加载的网址都会有一个标题（<title></title>）,该方法就是用来获取的

        ```java
        webView.setWebChromeClient(new WebChromeClient(){
          @Override
          public void onReceivedTitle(WebView view,String title){
            //获取到标题
          }
        })
        ```

#### WebView的内存泄漏避免

- 不在xml中定义WebView，在需要的时候通过Java代码在Activity中创建，并且创建的WebView时传入的Context，不是Activity的，而是Application的。

  ```java
  WebView webView = new WebView(getApplicationContext());
  new LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT,ViewGroup.LayoutParams.MATCH_PARENT);
  webView.setLayout(params);
  ```

  

- 在Activity销毁的时候，先让WebView加载null内容，然后移除WebView，在销毁WebView，最后置空

  ```java
  @Override
  protected void onDestroy(){
    if(webView !=null){
      webView.loadDataWithBaseURL(null,"","text/html","utf-8",null);
      webView.clearHistory();
      
      webView.getParent().removeView(webView);
      webView.destroy();
      webView = null;
    }
    super.onDestroy();
  }
  ```

#### 代码实操

> 需求如下，获取到www.baidu.com 该连接的 标题以及加载进度 提示加载开始和结束，不准用自带浏览器显示要用WebView加载

- 布局文件

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:tools="http://schemas.android.com/tools"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      tools:context=".MainActivity">
  
      <TextView
          android:id="@+id/tv_title"
          android:text="@string/app_name"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          app:layout_constraintLeft_toLeftOf="parent"
          app:layout_constraintRight_toRightOf="parent"
          app:layout_constraintTop_toTopOf="parent" />
  
      <TextView
          android:id="@+id/tv_progress"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@string/app_name"
          app:layout_constraintLeft_toLeftOf="parent"
          app:layout_constraintRight_toRightOf="parent"
          app:layout_constraintTop_toBottomOf="@+id/tv_title" />
  
      <TextView
          android:id="@+id/tv_start"
          android:layout_width="wrap_content"
          android:text="@string/app_name"
          android:layout_height="wrap_content"
          app:layout_constraintLeft_toLeftOf="parent"
          app:layout_constraintRight_toRightOf="parent"
          app:layout_constraintTop_toBottomOf="@+id/tv_progress" />
  
      <TextView
          android:id="@+id/tv_end"
          android:text="@string/app_name"
          app:layout_constraintLeft_toLeftOf="parent"
          app:layout_constraintRight_toRightOf="parent"
          app:layout_constraintTop_toBottomOf="@+id/tv_start"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"/>
  
      <WebView
          android:id="@+id/web_view"
          android:visibility="visible"
          app:layout_constraintLeft_toLeftOf="parent"
          app:layout_constraintTop_toBottomOf="@+id/tv_end"
          app:layout_constraintRight_toRightOf="parent"
          app:layout_constraintBottom_toBottomOf="parent"
          android:layout_width="match_parent"
          android:layout_height="0dp"/>
  
  </androidx.constraintlayout.widget.ConstraintLayout>
  ```

- Java代码

  ```java
  public class MainActivity extends AppCompatActivity {
      private static final String TAG = "MainActivity";
  
      private WebView mWebView;
      private TextView mTvStart;
      private TextView mTvEnd;
      private TextView mTvProgress;
      private TextView mTvTitle;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
          mWebView = findViewById(R.id.web_view);
          mTvStart = findViewById(R.id.tv_start);
          mTvEnd = findViewById(R.id.tv_end);
          mTvProgress = findViewById(R.id.tv_progress);
          mTvTitle = findViewById(R.id.tv_title);
  
          mWebView.loadUrl("http://www.baidu.com/");
  
  
          mWebView.setWebViewClient(new WebViewClient() {
  
              @Override
              public boolean shouldOverrideUrlLoading(WebView view, String url) {
                  view.loadUrl(url);
                  return true;
              }
  
              @Override
              public void onPageStarted(WebView view, String url, Bitmap favicon) {
                  mTvStart.setText("开始加载了");
              }
  
              @Override
              public void onPageFinished(WebView view, String url) {
                  mTvEnd.setText("结束加载了");
              }
          });
  
          mWebView.setWebChromeClient(new WebChromeClient() {
              @Override
              public void onProgressChanged(WebView view, int newProgress) {
                  Log.d(TAG, Thread.currentThread().getName());
                  if (newProgress < 100) {
                      mTvProgress.setText(newProgress + "%");
                  }else if (newProgress == 100){
                      mTvProgress.setText(newProgress + "%");
                  }
              }
  
              @Override
              public void onReceivedTitle(WebView view, String title) {
                  mTvTitle.setText(title);
              }
          });
      }
    @Override
      protected void onDestroy() {
          if (mWebView!=null){
              //加载一个空url
              mWebView.loadDataWithBaseURL(null, "", "text/html", "utf-8", null);
              // 清除当前WebView访问记录
              mWebView.clearHistory();
              //从父布局中清除这个webview
              ViewGroup parent = (ViewGroup) mWebView.getParent();
              parent.removeView(mWebView);
              //销毁WebView
              mWebView.destroy();
              //置空
              mWebView = null;
          }
          super.onDestroy();
      }
  }
  ```

  

