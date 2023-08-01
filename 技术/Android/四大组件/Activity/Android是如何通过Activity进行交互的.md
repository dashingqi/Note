- ![Android_Banner.jpg](https://upload-images.jianshu.io/upload_images/4997216-e887f9f9dcfb554a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  ###### taskAffinity会默认使Activity在新的栈中分配吗？

  > 每一个Activity都有一个Affinity属性，如果不在清单文件中指定，默认为当前应用的包名。

  - 我们创建两个Activity（ActivityA和ActivityB，其中A能跳转到B中），将ActivityB指定一个taskAffinity属性，区别于默认的，使用命令 adb shell dumpsys activity activities 查看 ,两个Activity是处于同一个TaskRecord（任务栈）中的（9.0之后使用adb shell dumpsys activity containers 查看简略的栈信息）
  - 我们如果给ActivityB指定lauchMode为singleInstance，使用命令观察可得 ,两个Activity是处于不同的任务栈中的
    - 也就是taskAffinity不能导致Activity被创建在新的任务栈中的，需要配合singleTask或者singleInstance
      - 这里说明下，使用singleInstance肯定能处于不同任务栈中
      - 使用singleTask 需要使用taskAffinity设定不同的属性，才能使得启动的Activity处于不同的任务栈中的。

  ###### taskAffinity+allowTaskReparenting

  - allowTaskReparenting：赋予Activity在各个Task中间转移的特性。
  - 一个在后台任务栈中的ActivityA，当有其他任务进入前台，并且taskAffinity和A的相同，那么会将A添加到当前的任务栈中。
  - 例子：两个项目 First和Second
    - Firs项目中有FirstA、B、C三个任务，依次跳转到FirstC，给FirstC taskAffinity设置为 acticvity.test 并且allowTaskReparenting设置为true,FirstA和B为默认的
    - Second项目中有SecondA一个任务，它的taskAffinity也设置为activity.test
    - 此时启动应用First，跳转到FirstC，此时推出到后台，然后启动应用Second，会发现看见的是FirstC页面，按返回键才能看到SecondA的页面。
    - 上述现象使用命令查看（adb shell dumpsys activity activities）会发现FirstA和B处于一个任务栈中，而FirstC和SecondA处于一个任务栈中，并且FirstC处于栈顶。

  ###### 通过Binder传递数据的限制

  - Intent传递数据过大（Android系统对使用Binder传数据进行了限制，通常情况为1M）
  - 解决办法
    - 对传递的数据中非必须字段使用transient关键字修饰
      - 目的是对非必须要使用数据，避免将其序列化
    - 将对象转化为JSON字符串，减少数据体积。比如使用Gson.toJson
      - 大多时候，将类转化成JSON字符串之后，还是会超出Binder限制，此时可以考虑使用EventBus或者本次持久化来实现数据的共享。

  ###### process造成多个Application

  - 通常我们会自定义一个Application，在onCreate中做些初始化的操作，比如推送的初始化，ARouter路由的使用等。
  - 其实Activity可以在不同进程中使用，而创建一个新的进程就会重新创建一个Application，这样就会造成Application的onCreate重新执行一次。那么上述我们要做的初始化操作就会重新初始化一次。
  - 解决办法
    - Application的onCreate方法判断进程的名字，只有在符合要求的情况下，去执行初始化的操作。
    - 抽象出一个与Application生命周期同步的类，并根据不同的进程创建相应的Application实例。

  ###### 后台启动Activity失效

  - 为了用户体验，在Android10（29）开始，后台运行的任务执行完毕并不能直接调用startActivity来启动新界面，而是用过NotificationManager来发送Notification到状态栏，让用户自己去决定，既保证的体验也通知到了用户。

  ###### App中的进程模式
  - 前台进程
  - 可见进程
  - 服务进程
  - 后台进程
  - 空进程
  ###### ActivityA  已经启动，此时启动ActivityB，这时的生命周期
  > onPause(A) ---> onCreate(B) ----> onStart(B) -----> onResume(B) ---> onStop(A)
  ###### 为什么会执行上述的流程
  - 首先对于一个Activity来说，它就是一个应用窗口，当一个Activity执行到onStart()方法时，所处的应用的进程属于可见进程，所谓的可见进程就是当前的窗口还没有获取到焦点。执行到onResume()方法时应用所处的进程属于前台进程，此时当前窗口是能获取到焦点的。
  - Android系统中，由于屏幕的限制，只能有一个窗口是能获取到焦点的，那么ActivityA想要启动ActivityB，就必须要将窗口焦点拥有权交出去，让给ActivityB，这是先执行onPause()方法的原因
  - 为什么后执行onStop()呢？因为为了让新的Activity尽快的启动并显示在屏幕上，所以ActivityB启动完之后在执行ActivityA的onStop了。
  ###### onPause()方法内不要做耗时的操作
  - 在启动新Activity之前会先执行旧Activity的onPause()方法，如果在onPause()中做过多的耗时操作，会影响新Activity的启动，影响用户的体验，以为很卡顿！

  #### 任务和返回栈
  > 在Android中存在 ActivityStack、TaskRecord、ActivityRecord这几个关键词
  - 任务：对应着TaskRecord
  - 返回栈：对应着ActivityStack
  - ActivityRecord：用于描述Activity的基本信息，对应着一个界面。
  ###### 三者之间的关系
  - ActivityStack 包含多个TaskRecord
  - TaskRecord中存储着多个ActivityRecord
  ![](https://images.xiaozhuanlan.com/photo/2019/538ab2817daa3d825e06c1cf161ef709.png)

  #### 为什么分别存在任务和返回栈
  - 它们存在主要是为了维护 “页面跳转的连贯性”体验

  #### 为何存在启动模式的设计
  - 启动模式是用于定义Activity与任务的关联方式
  #### 为何存在多种启动模式的设计
  - 为每一个Activity都创建一个新的实例，开销比较大，所以有了singleTop和singleTask这两种复用模式。
  - 考虑到多个App可能需要共享同一个实例，又设计出了singleInstance的复用模式。
  #### 如何设置启动模式
  ###### 在清单文件
  ###### 在代码中，通过setFlag的方式。
  -  常用的FLAG有如下几种方式
      - FLAG_ACTIVITY_SINGLE_TOP:该启动方式对应着singleTop
      - FLAG_ACTIVITY_SINGLE_TASK:和singleTask差不多。
      - FLAG_ACTIVITY_CLEAR_TOP：和singleTask差不多，当activity已经处于任务栈中，当该activity再次入栈的时候那么该activity已经其上的activity都会出栈，并且该activity会被重新创建和入栈的。
      - FLAG_ACTIVITY_NO_HISTORY,被指定的activity在跳转到其他activity后，将从任务中移除。
      - FLAG_ACTIVITY_EXCLUDE_FROM_RECENTERS,被指定的Activity不出现在“最近应用”列表中。
  -  关于FLAG_ACTIVITY_SINGLE_TASK和FLAG_ACTIVITY_SINGLE_TOP 
      -  即使两者同时使用，也是达不到singleTask的模式的。
      -  singleTask模式是不会重新创建新的activity实例的，singleTask会走onNewIntent()方法的。

  #### 关于启动模式的几点总结

  - standard和singleTop模式的组件处于哪个任务中，和启动该组件的在同一个任务中。
  - singleTop确实是栈顶复用的模式。
  - singleTask确实是栈中复用的模式，并且是清空在其之上的Activity。

  #### 一个APP中到底有多少个ActivityStack

  ###### Android9.0之前

  - 一个App中只有一个ActivityStack

  ###### Android9.0之后

  - 一个App中每创建一个TaskRecord就会创建一个对应的ActivityStack