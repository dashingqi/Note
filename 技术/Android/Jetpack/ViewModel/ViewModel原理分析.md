#### 简介
- ViewModel是用来管理Activity和Fragment的数据。
- 它是以生命周期意识的方式存储和管理用户界面相关的数据。
- 当我们在Fragment或者Activity中创建了相关联的ViewModel，只要Fragemnt或者Activity处于活动状态，ViewModel就不会被销毁，即使旋转屏幕ViewModel也不会发生被销毁依然持有着数据
- 这里附上一张Google官方ViewModel的生命周期图
![viewmodel-lifecycle.png](https://upload-images.jianshu.io/upload_images/4997216-1f00c77329fab420.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 注意
- ViewModel中不要持有View、Fragment或者Activity，我们知道ViewModel的生命周期要比它们要长（发生屏幕旋转的情况下），所以不能在ViewModel内部持有它们的引用，不然会发生内存泄漏。

#### 简单使用（基于Lifecycle 2.2.0）
- 新建一个ViewModel 

```kotlin
class ViewModelDemo : ViewModel() {

    var liveData = MutableLiveData<Int>(4)
}

```

- 在Activiyt中获取到这个ViewModel对象

```kotlin
class ViewModelActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_view_model)
        //获取到ViewModel
      	//在之前的版本中，可能这样获取到ViewModel 
      	//var viewModelDemo = ViewModelProviders.of(this).get(ViewModelDemo::class.java) 这种在lifecycle2.2.0之后就过期了。
        var viewModelDemo = ViewModelProvider(this)[ViewModelDemo::class.java]
        Log.d("viewModelDemo -->", viewModelDemo.toString())
        Log.d("liveData --> ", viewModelDemo.liveData.toString())
        text.text = viewModelDemo.liveData.value.toString()
    }
}

```

###### 旋转屏幕，观察现象
- 旋转前

```
2020-06-20 15:26:32.209 13837-13837/com.dashingqi.viewmodeldemo D/viewModelDemo -->: com.dashingqi.viewmodeldemo.ViewModelDemo@a9d570
2020-06-20 15:26:32.209 13837-13837/com.dashingqi.viewmodeldemo D/liveData -->: androidx.lifecycle.MutableLiveData@bf830e9
```

- 旋转后

```
2020-06-20 15:27:28.799 13837-13837/com.dashingqi.viewmodeldemo D/viewModelDemo -->: com.dashingqi.viewmodeldemo.ViewModelDemo@a9d570
2020-06-20 15:27:28.799 13837-13837/com.dashingqi.viewmodeldemo D/liveData -->: androidx.lifecycle.MutableLiveData@bf830e9
```
- 总结：我们发现旋转前后获取到的ViewModel是同一个对象，并且屏幕上显示的数据也是一样。对的就像之前介绍那样，即使屏幕发生旋转，Activity重建了ViewModel还是持有之前的数据。

###### 传递数据到ViewModel中
> 在我们新建ViewModel的时候，如何传递数据到ViewModel中呢？这就的依靠ViewModelProvider内部的工厂类Factory
- 比如我们要传递一个字符串到ViewModel中我们可以这样操作

```kotlin
// 创建一个ViewModel类，带有参数
class ViewModelDemo(var str:String) : ViewModel() {

    var liveData = MutableLiveData<Int>(4)
}

//通过ViewModelProvider的工厂类创建一个带有参数的ViewModel
   var viewModelDemo = ViewModelProvider(this, object : ViewModelProvider.Factory {
            override fun <T : ViewModel?> create(modelClass: Class<T>): T {
                return ViewModelDemo("test") as T
            }

        })[ViewModelDemo::class.java]

```
##### 思考
###### 都是能提供数据恢复功能，那么ViewModel与onSaveInstanceState()有什么区别呢

- onSaveInsatnceState:
  
    - 因为配置导致Activity重建的话，可以恢复之前的数据
    - 因为资源紧张导致Activity重建的话，可以恢复之前的数据
    - 保存的数据量不能过大，因为它是使用Bundle来保存数据的，在恢复数据时会阻塞UI线程，这样的话会影响Activiyt生命周期方法的执行

- ViewModel

    - 因为配置导致Activity重建的话，可以恢复之前的数据
    - 可以保存大量数据
    
#### 源码分析
##### ViewModel的创建
- 通过使用部分的介绍，我们可以通过ViewModelProvider来获取到ViewModel，实则是分两部分的，1.创建ViewModelProvider；2.调用其get方法
###### ViewModelProvider构造方法

```java
//通过构造方法的调用链，我们可以看到最终都是调用了第三个构造方法
public ViewModelProvider(@NonNull ViewModelStoreOwner owner) {
        this(owner.getViewModelStore(), owner instanceof HasDefaultViewModelProviderFactory
                ? ((HasDefaultViewModelProviderFactory) owner).getDefaultViewModelProviderFactory()
                : NewInstanceFactory.getInstance());
    }

    public ViewModelProvider(@NonNull ViewModelStoreOwner owner, @NonNull Factory factory) {
        this(owner.getViewModelStore(), factory);
    }

    public ViewModelProvider(@NonNull ViewModelStore store, @NonNull Factory factory) {
        mFactory = factory;
        mViewModelStore = store;
    }

```
- 参数解析

    - ViewModelStoreOwner：是一个接口，我们熟悉的ComponentActivity和Fragment都实现了这个接口，所以我们在Activity或者Fragment中使用ViewModelProvider传入的this就带有ViewModelStoreOwner上下文的。
    - ViewModelStore：我们可以看出该对象实例是从owmer中的getViewModelStore()方法得到的
      
        - 其实ViewModelStore类主要是用来存储ViewModel对象的
        - 内部有一个HashMap集合用来存储ViewModel对象
        - Activity重建时，能拿到同一个ViewModel对象也跟又关系
        - 代码如下
        
    - Factory：是一个接口，用来创建ViewModel的（传递参数到ViewModel中有用）
###### ViewModelProvider #get()

- 上述的构造方法是创建ViewModel之前的准备工作，get才是获取到ViewModel真正方法

```java
    @NonNull
    @MainThread
    public <T extends ViewModel> T get(@NonNull Class<T> modelClass) {
        String canonicalName = modelClass.getCanonicalName();
        if (canonicalName == null) {
            throw new IllegalArgumentException("Local and anonymous classes can not be ViewModels");
        }
        //构造了一个key
        return get(DEFAULT_KEY + ":" + canonicalName, modelClass);
    }

  
    @SuppressWarnings("unchecked")
    @NonNull
    @MainThread
    public <T extends ViewModel> T get(@NonNull String key, @NonNull Class<T> modelClass) {
        ViewModel viewModel = mViewModelStore.get(key);

        if (modelClass.isInstance(viewModel)) {
            if (mFactory instanceof OnRequeryFactory) {
                ((OnRequeryFactory) mFactory).onRequery(viewModel);
            }
            return (T) viewModel;
        } else {
            //noinspection StatementWithEmptyBody
            if (viewModel != null) {
                // TODO: log a warning.
            }
        }
        if (mFactory instanceof KeyedFactory) {
            viewModel = ((KeyedFactory) (mFactory)).create(key, modelClass);
        } else {
            viewModel = (mFactory).create(modelClass);
        }
        mViewModelStore.put(key, viewModel);
        return (T) viewModel;
    }
```
- 代码解析

    - 调用get方法后，最终都会调用到第二个get方法，第一个get方法是传递一个key给了第二个get方法
    
    - 首先根据提供的key从ViewModelStore中获取一个ViewModel对象实例，
    
    - 如果这个获取到的ViewModel对象实例存在，那么就将其返回
    
    - 如果该ViewModel对象不存在，就通过工厂类Factory创建一个ViewModel对象，并将其存储到ViewModelStore中，将这个新创建的ViewModel对象返回。
    
    - 这里面存在三个Factory：Factory，KeyedFactory和OnrequeryFactory，其中keyedFactory和Factory相比就是create方法中多了一个key的参数，我们从ViewModelStore获取到ViewModel对象时，会判断当前mFactory是否是OnRequeryFactory类型的，是的话会回调琦onRequery方法
    
    - 那么OnRequeryFactory回调onRequery有什么用呢？其实ViewModel不仅可以因为配置改变可以恢复Activity数据，也能恢复因为系统资源紧张而回收掉的Activity数据，只不过后者需要依靠SaveStateHandler（这个后面在讲）

##### ViewModel的恢复
- 通过如上介绍我们知道一个ViewModel的获取方法了，但是我们知道在Activity重建的时候，依然拿到是同一个ViewModel对象，而ViewModel是从ViewModelStore中获取的，
- ViewModelStore是通过owner.getViewModelStore方法获取到的，我们不妨先看下 ComponentActivity的getViewModelStore()

###### ComponentActivity # getViewModelStore()
```java
@NonNull
    @Override
    public ViewModelStore getViewModelStore() {
        if (getApplication() == null) {
            throw new IllegalStateException("Your activity is not yet attached to the "
                    + "Application instance. You can't request ViewModel before onCreate call.");
        }
        if (mViewModelStore == null) {
            NonConfigurationInstances nc =
                    (NonConfigurationInstances) getLastNonConfigurationInstance();
            if (nc != null) {
                // Restore the ViewModelStore from NonConfigurationInstances
                mViewModelStore = nc.viewModelStore;
            }
            if (mViewModelStore == null) {
                mViewModelStore = new ViewModelStore();
            }
        }
        return mViewModelStore;
    }
```
- getViewModelStore()通过两种方法获取到ViewModelStore
  
    - 从NonConfigurationInstances中拿到
    - new一个出来
- NonConfigurationInstances这个类，就是一个Wrapper，用来包装一下因为不受配置更改影响的数据，包括我们非常熟悉的Fragment，比如说，一个Activity上面有一个Fragment，旋转了屏幕导致Activity重新创建，此时Activity跟之前的不是同一个对象，但是Fragment却是同一个。这就是通过NonConfigurationInstances实现的。
- 那么为什么NonConfigurationInstances为什么能够保证ViewModelStore是同一个的呢？
    - 其实我们可以猜一下，在Activity的销毁之前和重建的时候，一定做了什么骚操作了，
    - 我们知道一个Activity的创建，它的生命周期方法都是在ActivityThread中执行的，

###### ActivityThread #performDestroyActivity

- 代码如下

  ```java
  ActivityClientRecord performDestroyActivity(IBinder token, boolean finishing,
              int configChanges, boolean getNonConfigInstance, String reason) {
          
    					// .............代码省略
  
              performPauseActivityIfNeeded(r, "destroy");
  
              if (!r.stopped) {
                	//执行Activity的onStop()方法
                  callActivityOnStop(r, false /* saveState */, "destroy");
              }
              if (getNonConfigInstance) {
                  try {
                      /**
                       * retainNonConfigurationInstances()方法返回的是一个对象 就是我们之前看到的NonConfigurationInstances对象
                       * 返回的对象存储在ActivityClientrecord中
                       */
                      r.lastNonConfigurationInstances
                              = r.activity.retainNonConfigurationInstances();
                  } catch (Exception e) {
                      if (!mInstrumentation.onException(r.activity, e)) {
                          throw new RuntimeException(
                                  "Unable to retain activity "
                                  + r.intent.getComponent().toShortString()
                                  + ": " + e.toString(), e);
                      }
                  }
              }
              try {
                  r.activity.mCalled = false;
                  // 执行Activity的destroy()方法
                  mInstrumentation.callActivityOnDestroy(r.activity);
                  if (!r.activity.mCalled) {
                      throw new SuperNotCalledException(
                          "Activity " + safeToComponentShortString(r.intent) +
                          " did not call through to super.onDestroy()");
                  }
                  if (r.window != null) {
                      r.window.closeAllPanels();
                  }
              } 
    		
         	// .............代码省略
  
  
      }
  ```

- 我们可以看出在onStop()和onDestory()方法之前，我们会把NonConfigurationInstances对象存储在ActivityClientrecord中

###### ActivityThread # pefromLaunchActivity()

- 在perfromLaunchActivity中我们通过调用activity的attach方法将存储在ActivityClientrecord中的NonConfigurationInstances对象赋值给了新的Activity，也就是我们重建时的Activity

- 代码如下 perfromLaunchActivity # activity.attach()

  ```java
                   /**
                   * 此处调用了attach方法建立Activity与Context之间的联系，
                   * 并且创建了PhoneWindow对象
                   */
                  activity.attach(appContext, this, getInstrumentation(), r.token,
                          r.ident, app, r.intent, r.activityInfo, title, r.parent,
                          //将NonConfigurationInstance对象赋给新的Activity对象
                          r.embeddedID, r.lastNonConfigurationInstances, config,
                          r.referrer, r.voiceInteractor, window, r.configCallback,
                          r.assistToken);
  ```

- 这样对于新的Activity来说，同一个NonConfigurationInstance对象

- 同一个同一个NonConfigurationInstance对象中保存着同一个ViewModelStore对象

- 同一个ViewModelStore对象，存储着同一个key的value，这value就是ViewModel对象，

- 这样就保证了在因配置改变，ViewModel不变了。

###### 思考

- 以上我们都是以因配置项改变为前提的，如果我们是正常的Activity销毁，再次创建Activity也会获取相同的ViewModel对象吗？

- 其实我们在ComponentActivity的构造方法中发现了如下代码

  ```java
    getLifecycle().addObserver(new LifecycleEventObserver() {
              @Override
              public void onStateChanged(@NonNull LifecycleOwner source,
                      @NonNull Lifecycle.Event event) {
                //当我们收到Activity的onDestory()方法的执行，如果不是因为配置项而销毁的，就会将当前ViewModelStore中存储的ViewModel对象都给清除了
                  if (event == Lifecycle.Event.ON_DESTROY) {
                      if (!isChangingConfigurations()) {
                          getViewModelStore().clear();
                      }
                  }
              }
          });
  ```

- 所以正常销毁的话，是获取不到之前创建好的ViewModel对象了。



#### 总结一波

- ViewModel和onSaveInsatnceState()方法都可以因配置项改变而重建Activity提供数据，只不过ViewModel可以保存大量数据比如RecyclerView中的，但是onSaveInstanceState只能保存少量数据。
- 对于因配置项改变而重建的Activity来说，内部如果有ViewModel。那么该ViewModel对象的创建其实是获取重建前创建好的ViewModel，ViewModel对象一直未变。之所以说ViewModel的生命周期要比Activity要长，是有前提的，前提就是因配置项该拜年而导致Activity重建，Activity重建了但是ViewModel并没有重建，拿的是保存在ViewModelStore中缓存的。
  - 所以ViewModel中不要持有Activity，Activity虽然销毁了，但是ViewModel中持有了它的引用，会造成内存泄漏。
- ViewModel是用来管理UI组件的数据，Activity和Fragment中的UI组件。