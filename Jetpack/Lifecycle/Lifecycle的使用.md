#### 简介

- Lifecycle是一个类，用于存储有关组件（Activity、Fragment）的生命周期状态的信息，并且允许其他对象观察此状态

- Lifecycle主要使用两种枚举跟踪相关组件的生命周期状态

  - Event（事件）：从框架和Lifecycle类分派的生命周期事件。这些事件映射到Activity和Fragment中的回调事件

  - Statest（状态）：由Lifecycle对象跟踪的组件的当前状态

  - 用一张图来描述上述的Event和State

    ![Lifecycle](https://upload-images.jianshu.io/upload_images/4997216-e5b4865c28b3e8ab.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    - 当我们的状态处于CREATE时候，说明onCreate已经执行了，但是onStart并没有执行，当处于STARTED状态的时候，说明onStart方法已经执行了，但是onResume并没有执行
    - 注意：当onStop方法执行的时候，状态是处于CREATED的。

#### 使用

###### Lifecycle感知Activty的生命周期

- 新建观察者MyObserver

  ```kotlin
  class MyObserverDemo : LifecycleObserver {
  
  
      @OnLifecycleEvent(Lifecycle.Event.ON_START)
      fun performOnStart() {
          Log.d(Companion.TAG, "performOnStart ")
      }
  
      @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
      fun performOnStop() {
          Log.d(Companion.TAG, "performOnStop ")
      }
  
      companion object {
          private const val TAG = "MyObserverDemo"
      }
  
  }
  ```

  - 我们新建一个MyObserverDemo类，实现了LifecycleObserver接口，LifecycleObserver是属于Lifecycle工具包中的
  - 我们在方法上使用了@OnLifecycleEvent注解，并且传入了对应事件的类型，该事件类型分别有如下几种(前六种事件分别对应这个组件的生命周期)
    - ON_CREATE
    - ON_START
    - ON_RESUMR
    - ONPAUSE
    - ON_STOP
    - ON_DESTORY
    - ON_ANY：表示可以匹配组件任何的生命周期的回调
  - 上述的performOnStop()和performOnStart会在组件的onStart和onStop执行的时候被回调执行
  - 上述只是建立好了感知组件对应生命周期的观察者对象，还要在组件中去通知到观察者对象

- 通过LifeCycle去观察LifecycleOwner的生命周期

  - 在我们要去通知给对应的观察者对象的时候，我们需要写如下的代码

    ```java
    lifecycleOwner.lifecycle.addObserver(MyObserverDemo())
    ```

  - 在AndroidX中我们的AppCompatActivity或者Fragment都是实现了LifecycleOwner接口，所以我们可以在Activity或者Fragment通过上述的代码，来通知观察者

  - 代码如下

    ```kotlin
    class LifecycleActivity : AppCompatActivity() {
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_lifecycle)
            //重点是下面这个方法
            lifecycle.addObserver(MyObserverDemo())
        }
    }
    ```

- 运行结果如下

  ```java
  //程序开启时的log
  2020-06-21 17:18:49.016 30782-30782/com.dashingqi.lifecycledemo D/MyObserverDemo: performOnStart 
  //将程序退到后台的log
  2020-06-21 17:19:46.256 30782-30782/com.dashingqi.lifecycledemo D/MyObserverDemo: performOnStop 
  ```

###### 自定义LifecycleOwner

> 上述都是在AppCompatActivity前提下，去实现生命周期的感知，我们知道AppCompatActivity是实现了LifecycleOwner接口的，
>
> 如果这时候我们单纯时继承Activity想要感知生命周期该怎么办呢？
>
> 其实也是一样的，实现LifecycleOwner接口，也就是所谓的自定义LifecycleOwner

- 继承Activity，实现LifecycleOwner接口

  ```kotlin
  class ExtendsActivity : Activity(), LifecycleOwner {
      private var lifecycleRegistry: LifecycleRegistry? = null
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_extends)
          //第一步：新建LifecycleRegister
          lifecycleRegistry = LifecycleRegistry(this)
          //第二步：设置lifecycleRegister的各种状态
          lifecycleRegistry!!.currentState = Lifecycle.State.CREATED
          //第三步：通知给对应的观察者对象
          lifecycle.addObserver(MyObserverDemo(lifecycle))
  
      }
  
      override fun onStart() {
          super.onStart()
          lifecycleRegistry!!.currentState = Lifecycle.State.STARTED
      }
  
      override fun onStop() {
          super.onStop()
          lifecycleRegistry!!.currentState = Lifecycle.State.CREATED
      }
  
      /**
       * 实现了LifecycleOwner接口，实现了该方法，返回了lifecycleRegistry对象
       */
      override fun getLifecycle(): Lifecycle {
          return lifecycleRegistry!!
      }
  }
  ```

  - LifecycleRegister是继承了Lifecycle类的

###### 补充一点

- 除了使用LifecycleObserver之外，还可以使用DefaultLifecycleObserver,同样都是用来感知组件的生命周期方法的

- DefaultLifcycycleObser中为我们提供对应的生命周期方法，我们只需重写即可，

- 代码如下

  ```kotlin
  class Java8Observer : DefaultLifecycleObserver {
  
      override fun onCreate(owner: LifecycleOwner) {
  
      }
  
      override fun onResume(owner: LifecycleOwner) {
          Log.d(Companion.TAG, "performOnStart ")
      }
  
      override fun onStop(owner: LifecycleOwner) {
          Log.d(Companion.TAG, "performOnStop ")
      }
  
      companion object {
          const val TAG = "Java8Observer"
      }
  }
  ```

- 运行结果如下

  ```java
  2020-06-21 18:00:28.497 32403-32403/com.dashingqi.lifecycledemo D/Java8Observer: performOnStart
  ```

  

