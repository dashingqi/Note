- Factory

- Factory2

- 通过LayoutInfalter.setFactory()可以替换我们在xml文件中控件

  ```java
   LayoutInflaterCompat.setFactory2(LayoutInflater.from(this), new LayoutInflater.Factory2() {
              @Nullable
              @Override
              public View onCreateView(@Nullable View parent, @NonNull String name, @NonNull Context context, @NonNull AttributeSet attrs) {
                  if (name.equals("TextView")){
                      EditText editText = new EditText(MainActivity.this);
                      editText.setHint("我是EditText");
                      return editText;
                  }
                  return getDelegate().createView(parent, name, context, attrs);
              }
  
              @Nullable
              @Override
              public View onCreateView(@NonNull String name, @NonNull Context context, @NonNull AttributeSet attrs) {
                  return null;
              }
          });
  ```

  

- 我们在LayoutInfalter在加载View的时候会先调用tryCreateView，该方法中会先调用Factory的onCreateView方法来创建View，并且将创建好的view返回回来。

- 在super.onCreate()方法之后setContentView之前调用上面的代码，报下面的错误

  ```java
  java.lang.IllegalStateException: A factory has already been set on this LayoutInflater
          at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2646)
          at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2707)
          at android.app.ActivityThread.-wrap12(ActivityThread.java)
          at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1460)
          at android.os.Handler.dispatchMessage(Handler.java:102)
          at android.os.Looper.loop(Looper.java:154)
          at android.app.ActivityThread.main(ActivityThread.java:6077)
          at java.lang.reflect.Method.invoke(Native Method)
          at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:865)
          at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:755)
       Caused by: java.lang.IllegalStateException: A factory has already been set on this LayoutInflater
          at android.view.LayoutInflater.setFactory2(LayoutInflater.java:317)
          at androidx.core.view.LayoutInflaterCompat.setFactory2(LayoutInflaterCompat.java:139)
          at com.dashingqi.asynclayoutinflaterproject.MainActivity.onCreate(MainActivity.java:44)
          at android.app.Activity.performCreate(Activity.java:6664)
          at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1118)
          at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2599)
         
         // 重点是 A factory has already been set on this LayoutInflater
  ```

- LayoutInfalter.setFactory2()方法中

  ```java
  public void setFactory2(Factory2 factory) {
    			// mFactorySet为true 抛出异常，说明已经调用过这个setFactory2方法 才会为true
          if (mFactorySet) {
              throw new IllegalStateException("A factory has already been set on this LayoutInflater");
          }
          if (factory == null) {
              throw new NullPointerException("Given factory can not be null");
          }
          mFactorySet = true;
          if (mFactory == null) {
              mFactory = mFactory2 = factory;
          } else {
              mFactory = mFactory2 = new FactoryMerger(factory, factory, mFactory, mFactory2);
          }
      }
  ```

- 在Activity中调用setFactory2之前 仅仅调用了super.onCreate()

  - AppCompatActivity中的onCreate方法中

    ```java
    protected void onCreate(@Nullable Bundle savedInstanceState) {
            final AppCompatDelegate delegate = getDelegate();]
           	//该方法会最终调用到 AppCompatDelegateImpl中的installViewFactory()方法
            delegate.installViewFactory();
            delegate.onCreate(savedInstanceState);
            super.onCreate(savedInstanceState);
        }
    ```

  - AppCompatDelegateImpl中的installViewFactory()方法

    ```java
    public void installViewFactory() {
            LayoutInflater layoutInflater = LayoutInflater.from(mContext);
            if (layoutInflater.getFactory() == null) {
                LayoutInflaterCompat.setFactory2(layoutInflater, this);
            } else {
                if (!(layoutInflater.getFactory2() instanceof AppCompatDelegateImpl)) {
                    Log.i(TAG, "The Activity's LayoutInflater already has a Factory installed"
                            + " so we can not install AppCompat's");
                }
            }
        }
    ```

    

