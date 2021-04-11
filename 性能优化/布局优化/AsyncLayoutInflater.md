## 本文主要从如下几点学习AsyncLayoutInflater

- AsyncLayoutInfalter是啥
- AsyncLayoutInflater代码实操
- AsyncLayoutInflater的源码分析
- AsyncLayoutInflater的问题

## AsyncLayoutInflater是啥

#### 官方源码定义

- 如下

  ```java
  Helper class for inflating layouts asynchronously
  //用于异步加载布局的帮助类
  ```

  

## AsyncLayoutInflater代码实操

- 代码如下

  ```java
   new AsyncLayoutInflater(this).inflate(R.layout.activity_main, null, new AsyncLayoutInflater.OnInflateFinishedListener() {
              @Override
              public void onInflateFinished(@NonNull View view, int resid, @Nullable ViewGroup parent) {
                  Log.d(TAG, "currentThread = " + Thread.currentThread().getName());
                  setContentView(view);
              }
          });
  ```

  

## AsyncLayoutInflater源码解析

#### AsyncLayoutInflater的构造方法

- 代码如下

  ```java
  public AsyncLayoutInflater(@NonNull Context context) {
    			//创建BasicInflater对象，BasicInflater继承至 LayoutInflater
          mInflater = new BasicInflater(context);
         //创建一个Handler对象，目的是线程的切换，从布局加载的工作线程切换到主线程中。
          mHandler = new Handler(mHandlerCallback);
    			//创建一个InflateThread对象，该类是继承至Thread类，
          mInflateThread = InflateThread.getInstance();
      }
  ```

#### AsyncLayoutInflater的inflate方法

- 代码如下

  ```java
  @UiThread
      public void inflate(@LayoutRes int resid, @Nullable ViewGroup parent,
              @NonNull OnInflateFinishedListener callback) {
          if (callback == null) {
              throw new NullPointerException("callback argument may not be null!");
          }
        // 构建InflateRequest对象，将resid、parent、callback等变量存储到这个变量中。
          InflateRequest request = mInflateThread.obtainRequest();
        	
          request.inflater = this;
          request.resid = resid;
          request.parent = parent;
          request.callback = callback;
        	// 然后将带有参数的InflateRequest对象，通过Inflatethread类中的enqueue方法保存到ArrayBlockingQueue队列中
          mInflateThread.enqueue(request);
      }
  ```

#### AsyncLayoutInflater中的InflateThread类

- 代码如下

  ```java
   // InflateThread是AsyncLayoutInfalter类中的一个静态内部类
  private static class InflateThread extends Thread {
          private static final InflateThread sInstance;
    			// 当类加载的时候，会初始化该类并且开启线程
          static {
              sInstance = new InflateThread();
              sInstance.start();
          }
  				//对外提供获取实例的方法
          public static InflateThread getInstance() {
              return sInstance;
          }
  				//生产者-消费者的模型，阻塞队列
          private ArrayBlockingQueue<InflateRequest> mQueue = new ArrayBlockingQueue<>(10);
    //使用了对象池，来缓存创建的InflateRequest对象，防止重复创建
          private SynchronizedPool<InflateRequest> mRequestPool = new SynchronizedPool<>(10);
          public void runInner() {
              InflateRequest request;
              try {
                //从队列中取出一个请求，
                  request = mQueue.take();
              } catch (InterruptedException ex) {
                  // Odd, just continue
                  Log.w(TAG, ex);
                  return;
              }
  
              try {
                //LayoutInfalter加载view的操作
                //获取到View对象
                
                  request.view = request.inflater.mInflater.inflate(
                          request.resid, request.parent, false);
              } catch (RuntimeException ex) {
                  // Probably a Looper failure, retry on the UI thread
                  Log.w(TAG, "Failed to inflate resource in the background! Retrying on the UI"
                          + " thread", ex);
              }
            //无论inflate方法的失败或者成功，都将request发送到主线程中。
             Message.obtain(request.inflater.mHandler, 0, request)
                      .sendToTarget();
          }
  
          @Override
          public void run() {
            //死循环
              while (true) {
                  runInner();
              }
          }
  				
    			//主要创建一个InflateRequest对象
          public InflateRequest obtainRequest() {
            //先从对象缓存池中查询
              InflateRequest obj = mRequestPool.acquire();
            //如果当前InflateRequest对象没有创建过，就创建一个
              if (obj == null) {
                  obj = new InflateRequest();
              }
            //否则的话就将当前内存中存在的返回；
              return obj;
          }
  
    			//将对象缓存池中的对象数据清空，方便对象的复用。
          public void releaseRequest(InflateRequest obj) {
              obj.callback = null;
              obj.inflater = null;
              obj.parent = null;
              obj.resid = 0;
              obj.view = null;
              mRequestPool.release(obj);
          }
  				
    			//将inflate请求中的request存储到队列中
          public void enqueue(InflateRequest request) {
              try {
                  mQueue.put(request);
              } catch (InterruptedException e) {
                  throw new RuntimeException(
                          "Failed to enqueue async inflate request", e);
              }
          }
      }
  ```

#### AsyncLayoutInflater中的InflateRequest类

- 代码如下

  ```java
  //该类主要是携带数据，赋给Message.obj变量，通过Message将数据发送到主线程中
  private static class InflateRequest {
          AsyncLayoutInflater inflater;
          ViewGroup parent;
          int resid;
          View view;
          OnInflateFinishedListener callback;
  
          InflateRequest() {
          }
      }
  ```

#### AsyncLayoutInflater的BasicInflater类

- 代码如下

  ```java
  //该类是继承至LayoutInflater
  //重写了onCreateView
  private static class BasicInflater extends LayoutInflater {
          private static final String[] sClassPrefixList = {
              "android.widget.",
              "android.webkit.",
              "android.app."
          };
  
          BasicInflater(Context context) {
              super(context);
          }
  
          @Override
          public LayoutInflater cloneInContext(Context newContext) {
              return new BasicInflater(newContext);
          }
  
          @Override
          protected View onCreateView(String name, AttributeSet attrs) throws ClassNotFoundException {
            //在onCreateView中优先加载 android.widget.", "android.webkit.","android.app."中的控件，之后在按照常规的加载方式去加载
             
              
              for (String prefix : sClassPrefixList) {
                  try {
                      View view = createView(name, prefix, attrs);
                      if (view != null) {
                          return view;
                      }
                  } catch (ClassNotFoundException e) {
                      // In this case we want to let the base class take a crack
                      // at it.
                  }
              }
  
              return super.onCreateView(name, attrs);
          }
      }
  ```

#### AsyncLayoutInflater的Callback

- 代码如下

  ```java
  private Callback mHandlerCallback = new Callback() {
          @Override
          public boolean handleMessage(Message msg) {
            //通过Handler从Message中获取InflateRequest中的数据带到主线程
              InflateRequest request = (InflateRequest) msg.obj;
              if (request.view == null) {
                  request.view = mInflater.inflate(
                          request.resid, request.parent, false);
              }
              request.callback.onInflateFinished(
                      request.view, request.resid, request.parent);
            //置空InflateRequest中携带的数据
              mInflateThread.releaseRequest(request);
              return true;
          }
      };
  ```

  

## AsyncLayoutInflater的问题

#### 同样看源码定义

- 定义如下

  ```java
   
  /**
   * <p>For a layout to be inflated asynchronously it needs to have a parent
   * whose {@link ViewGroup#generateLayoutParams(AttributeSet)} is thread-safe
   * and all the Views being constructed as part of inflation must not create
   * any {@link Handler}s or otherwise call {@link Looper#myLooper()}. If the
   * layout that is trying to be inflated cannot be constructed
   * asynchronously for whatever reason, {@link AsyncLayoutInflater} will
   * automatically fall back to inflating on the UI thread.
   */
     
   // 1. 对于异步加载的布局需要它的父View的generateLayoutParams(AttributeSet attrs)方法是线程安全的。
     
   // 2. 所有被创建的View不能在内部创建Handler或者是调用Looper.myLooper()方法。
     
   // 如果使用AsynclayoutInflater.inflate()的方法异步加载失败，不管是什么原因，都会会退到主线程中就加载布局。
  ```

  ```java
   
  /**
   * <p>NOTE that the inflated View hierarchy is NOT added to the parent. It is
   * equivalent to calling {@link LayoutInflater#inflate(int, ViewGroup, boolean)}
   * with attachToRoot set to false. Callers will likely want to call
   * {@link ViewGroup#addView(View)} in the {@link OnInflateFinishedListener}
   * callback at a minimum.
   */
  
  // 3. 异步加载布局获取的View没有添加到父View中，因为源码中request.inflater.mInflater.inflate(
  //    request.resid, request.parent, false);是这样调用去加载布局的 attachToRoot 为 false
  //    所以如果希望被添加到父View中，需要在onInflateFinishedListener方法中去手动添加View
  ```

  ```java
  /**
   * <p>This inflater does not support setting a {@link LayoutInflater.Factory}
   * nor {@link LayoutInflater.Factory2}. Similarly it does not support inflating
   * layouts that contain fragments.
   */
  
  // 4. 不支持设置LayoutInflater.Factory和Factory2
  // 5. 不支持加载包含Fragment的布局
  ```

#### 为什么会有这些问题

###### 针对第一点：对于异步加载的布局需要它的父View(ViewGroup)的generateLayoutParams(AttributeSet attrs)方法是线程安全的。

- 看ViewGroup的generateLayoutParams(AttributeSet attrs)f方法（注意参数哟，ViewGroup中有很多同名的重载方法）

  ```java
  public LayoutParams generateLayoutParams(AttributeSet attrs) {
          return new LayoutParams(getContext(), attrs);
      }
  // 可以看到，直接new了一个对象出来，如果是在非线程安全的情况下去调用，会创建多个对象。
  ```

###### 针对第二点:所有被创建的View不能在内部创建Handler或者是调用Looper.myLooper()方法。

- 因为是异步加载，在子线程中如果使用Handler必须在同线程下创建一个looper对象，不然会报错的。同样Looper.myLooper是获取同线程下的looper对象，你的有才能用，才能获取。

