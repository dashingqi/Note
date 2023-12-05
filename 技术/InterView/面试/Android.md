#### Android InterView

### 内存

###### 内存泄漏-actiivty

- ImageView控件声明为static ---> 静态成员变量声明周期长
- Activity中注册监听事件未解注册（BroadcastReceiver、EventBus） --> onDestory() ---> 解注册
- 使用Handler造成的 ---> 静态class + WeakReference
- 传入三方SDK的Context --> 尽量使用context.getApplicationContext()  --> Toast
- FD泄漏 --> IO流/Cursor未关闭

###### 内存泄漏-检测-LeakCanary

- WeakReference + ReferenceQueue

- LeakCanary有Watcher相关类，用于监控相关引用的销毁的时机 --Activity --> onDestory(). regoisterActivitiyLifecycle()

- WeakReference持有当前要销毁的对象，并为其设置一个key，该key同时存储在一个set中，当GC完一轮后，会对比RefernceQueue中存在的引用key从Set中移除，经过几轮GC完后，好存在于set中的就说明是存在内存泄漏，就开始生成内存快照，发送上屏通知，列出泄漏的信息

- 对于LeakCanary检测内存泄漏的时机，是使用了IdleHandler的机制，向MessageQueue入队了一个空闲消息，当没有任务要执行时会执行空闲任务，去执行对应的内存检测以及手机内存信息；

###### 内存大头 --> Bitmap

- 计算内存占用的方式 720*1080 * 每个像素占用的内存大小   ARGB_8888 占用 4字节 RGB_565 2字节 inPreferredConfig

- 对于asset中文件和本地磁盘中图片文件生成的Bitmap对象大小与屏幕密度无直接关系

- 对于放置在Drawabel中的图片文件是与屏幕密度有关系 屏幕密度/drawable文件所对应的屏幕密度--> 缩放比 720*缩放比*1080 *缩放比* 每个像素占用的内存大小

- 自适应图片大小：采样率压缩 inSampleSize inJust

- 可复用Bitmap inMutable & inBitmap。复用的Bitmap大小要比将要分配的内存大就行；

- Bitmap的池化，最近最少使用的策略；设置的好池的大小，淘汰最近未使用的就行；

- 大图的加载使用BitmapRegionDecoder

- Android26 8.0之后将Bitmap的像素数据放置到native层中，bitmap对象放置到Java虚拟机的堆中，native层的像素数据可以随着java堆中的对象一起被回收掉；
  9.上述的补充：NativeAllocationRegistry Native层分配的内存与Java层内存简历映射

###### 内存监控-大图片

- Hook+大小阈值设定；

#### Glide-Bitmap的优化
- 自适应图片大小，避免加载过大的图片数据到内存中；
- 内存缓存：图片的加载优先使用内存缓存 接着磁盘缓存 网络请求；
- 磁盘缓存：无网络情况下也能搞定；
- Bitmap的复用机制：减少Bitmap对象重复创建以及回收
- Bitmap的配置：ARGB_8888 来平衡图片加载与内存占用
- 提供transform接口，让开发者可以对Bitmap做二次加工，比如裁剪、旋转
- 完整的生命周期管制：避免因生命周期而产生的内存泄漏问题
- 优先级加载：立即加载会绕过等待队列-直接请求网络获取图片数据；

### FPS的检测
- 线下可通过AS集成的工具 AS-Profiler里面的CPU模块中可以选择查看FPS帧率

- 线上可通过Choreographer类，它本身支持注册帧率变化的监听-postFrameCallback()方法；在页面可见时注册监听，在页面不可见时解注册；它的回调线程是与VSYNC垂直信号同步的，可以与主线程并行；

### Touch事件的分发
#### 事件的流转
- 一个触摸事件的开始是屏幕经由手机的驱动层传递到FrameWork层的InputManagerService
- FrameWork中由wms将事件经由ViewRootImpl传递到窗口 Window
- DecorView将事件分发到页面中的控件。
#### 几个概念
###### ViewGroup和View
- ViewGroup是一组View的集合，在事件分发过程中它起到承上启下的作用，决定事件的分发以及拦截；
- View是一个具体控件，它通常是事件消费的源头，我们可重写 onToucheEvent()方法，注册触摸监听重写它的onTouch方法，onTouch > onToucheEvent() > onClick
###### 事件 MotionEvent
- 一个事件是由 一个down 多个move 以及一个up事件组成；
###### 事件分发的方法模型
- dispatchTouchEvent()-代表事件分发-也是一个大的递归函数
- onInterceptToucheEent():ViewGroup中特有的，用于决定当前事件是否要做拦截;
- onTouchEvent() 事件的消费处；
#### 事件的流程
- 当一个Down事件开始流转时，首先会调用到dispatchTouchEvent()方法，如果是down事件直接调用onInterceptTouchEvent()来判断当前事件是否要做拦截，如果不做拦截就将当前事件分发给子View，同时调用子View的diaptchTouchEvent()方法，继而调用onTouchEvent(),如果消费了事件那么同时会将ViewGroup中的mFirstTouchTarget赋值给当前子View；
- 当我们的move事件开始流转时，校验mFirstTouchTagret是否为null，不为null，说明之前的Down事件是由当前View消费了，那么后续的move以及up事件都交由该View进行消费；
- cancel事件的流转：当子View收到cancel事件时，之前的父View的不拦截事件，交由子View消费，此时又做拦截并且mFirstTouchTagret不为null，此时会发送一个cancel事件；

#### LayoutInflter.inflate(int resource ,View parent,boolean attachToRoot)方法入参
###### parent != null && attachToRoot = true
- 父布局不为空，此时resource声明的布局参数生效正常测量布局，此时resource被添加到parent中；此时返回的View为parentView
###### parent != null && attachToRoot = false
- 父布局不为空，此时resource声明的布局参数生效正常测量布局，此时resource没有被添加到parent中，此时inflate返回的View为resource；
###### parent == null && attachToRoot = false
- 父布局为空，此时resource声明的布局参数不生效，默认为wrap_content,此时resource没有被添加到parent中，此时inflate返回的View为resource；

#### 包体积优化
##### 代码
- 使用Proguard工具对我们的源代码进行混淆，将代码类文件换成简约的名字文件，并且具有一定的安全性；
- 对于三库的使用：避免拥有相同功能引入不同三方库的场景；三方库功能居多，业务场景使用有限，代码进行改造；
- 无用代码的删除- AOP思想实现Activity/Fragment的使用情况，其他类构造方法；
- 功能组件统一UI样式，工具类的封装；
###### 跨Dex调用优化，
- 65536问题我们进行多Dex解决这个问题，就会出现跨Dex调用问题，跨Dex调用就会在Dex中存储methodId，这个会影响dex中class对象数，就会出现更多的Dex；
- 使用ReDex「FaceBook」的InterDexPass进行优化，将存在跨Dex调用的情况，分配在同一个Dex中；
###### 插件化工程

- 将独立的功能以插件的形式进行后下发；

##### 资源
- 大文件资源进行后下发形式，避免内置占用包体积；
- 使用Lint Remove UnuseResource 将冗余的资源进行删除；
- 内置图片资源进行压缩处理，UI同学提供的资源图片未进行压缩，
- 资源的混淆，将资源名称进行混淆，换成简单名称，减小resources.arsc、metadata 签名⽂件以及 ZIP ⽂件⼤⼩的效果，极限压缩使用 7-Zip
- 重复资源的优化-ByteX
- 图片的格式：WebP > png > jpg,
- shape drawable 文件的复用 保持相同风格；
##### so文件
- 保留Armeabi 架构的so文件，去除别的架构so文件；
##### 长期方案
###### 利用CI（流水线）
- 设置阈值：一个需求的提交代码增量不能超过 100KB，超过需要给出合理解释以及豁免
- apk-check，现有Apk提及与线上APK包体积进行比较 ，设置阈值 超过1MB 进行发邮件通知，

#### ANR
##### ANR种类
- Activity#onCreate() 或者Input 事件超过5S
- BroadcastReceiver ，应用处于前台10S，后台60S
- ContentProvider在publish超过10S
- Service 前台服务 20S，后台200s
##### 引起原因
- 主线程有耗时操作 （IO操作、复杂布局）
- 被Binder对端block住（进程）
##### ANR排查
- 拿到 bugReport文件，搜索关键字 ANR in 找到对应进程pid与发生时间，通过pid和发生向下搜索 main关键字找到对应log信息；
##### ANR的监控
- 低版本使用FileObserver监控 data/anr/trace.txt文件的变化；
- 高版本使用ANR-WatchDog 监控主线程
