#### 简介

- LiveData是JetPack提供的一种响应式编程组件，它可以包含任何数据类型的数据（String，Int，Boolean等）。
- LiveData在数据发生变化的时候，会通知给观察者。
- 一般情况下，LiveData与ViewModel配合使用的，在这个前提下，LiveData都是在ViewModel中声明的。但是也是可以单独进行声明使用的。
  - LiveData是具有生命感知的，它的数据源一般都是ViewModel提供的，这样当数据源发生变化我们通过LiveData可以通知给观察者。
  - 当然也不能忽略生命周期感知这个提点，可以这样理解，当观察者处于Active状态的时候，LiveData发生变化观察者可以收到，但是观察者处于Paused或者Destoryed状态，那么观察者就不能收到LiveData的通知了
- 在MVVM中，我们可以使用LiveData帮助ViewModel向Activity或者Fragment进行通信
  - 因为ViewModel的生命周期都要比Activity要长的，所以我们不能让ViewModel持有Activity的引用，如果这时我们能在Activity中观察ViewModel中的LiveData，那么就不回因为ViewModel持有activity而造成的内存泄漏问题。
- LiveData是一个抽象类，其中我们常用的是它的实现子类 MutableLiveData，它是一个可变的LievData,主要有如下几个操作api
  - getValue():用于从LiveData中获取到包装的数据
  - setValue()：用于给LiveData设置数据
  - postValue()：同样也是给LiveData设置数据
- postValue与setValuede 区别
  - setValue()：只能作用于UI线程中，如果在工作线程中使用，会发生crash
  - postValue：可以用于工作线程中（非UI线程 ），当然在UI线程中也是可用的。

#### 使用

###### 基本使用

> 我们知道在MVVM中ViewModel是作为数据的持有者，而Activity或者Fragment是作为UI的载体（PhoneWindwo），在MVVM中我们一般是使用DataBinding，将XML文件自动编译成一个XXXDataBinding的文件，通过在Activity中，用这个XXDataBinding去绑定我们的数据持有者ViewModel，然后在xml引入这个ViewModel，并且去动态绑定我们ViewModel中的LiveData或者Observable。达到一个数据监听的作用。

- 场景设定：我们有一个ViewModel，内部有一个LiveData包装着String类型，同时在Activity中获取到这个ViewModel，注册一个观察者去监听ViewModel中的LiveData，当我们点击按钮设置一个数据源给LiveData，然后在观察者中去更新xml文件中的一个text。

- 代码实现如下

  ```kotlin
  class LiveDataMainActivity : AppCompatActivity() {
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_live_data_main)
          //获取到ViewModel（此种方式获取ViewModel要在Lifecycle2.2.0之后哟！）
          var viewModelDemo2 = ViewModelProvider(this)[ViewModelDemo2::class.java]
          //注册一个观察者，用于观察LiveData，当LiveData的数据源发生改变会通知到观察者中
          viewModelDemo2.liveData.observe(this, Observer {
              it?.let { value ->
                  tvText.text = value
              }
  
          })
  
          btnChange.setOnClickListener {
              viewModelDemo2.liveData.postValue("i am coming ")
  
          }
  
  
      }
  }
  
  class ViewModelDemo2 : ViewModel() {
  
      var liveData = MutableLiveData<String>()
  }
  ```

  

###### map

> 这里说的map实则是调用了Transformations#map()方法，通过该方法可以将我们的LiveData进行数据类型转换成新类型的LiveData,然后在Activity中注册新LiveDaya的观察者就能拿到数据源了，
>
> 说的可能比较抽象，以列子来说明吧

- 场景设定：有一个Person，包含姓名和性别，但是我们的UI控件上只用来显示姓名的，这时我们可以通过map这种方式来做处理

- 代码实现如下

  ```kotlin
  class MapLiveDataMainActivity : AppCompatActivity() {
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_map_live_data_main)
          var viewModel = ViewModelProvider(this)[MapLiveDataViewModel::class.java]
          var person = Person()
          person.name = "DashingQi"
          person.sex = "man"
  
          viewModel.transformLiveData.observe(this, Observer {
              it?.let { value ->
                  tvText.text = value
              }
  
          })
          setUser.setOnClickListener {
              viewModel.userLiveData.postValue(person)
          }
  
  
      }
  }
  
  class MapLiveDataViewModel : ViewModel() {
  
      var userLiveData = MutableLiveData<Person>()
  
      /**
       * 使用Transformations#map转换成一个新的LiveData
       * 第一个参数是我们要进行转化的LiveData。里面包装了我们需要的数据
       * 第二个参数是我们要进行转化的函数
       * 这样当userLiveData的数据源发生变化的时候，我们transformLiveData的观察者就能收到转化后的数据源了
       */
      var transformLiveData = Transformations.map(userLiveData) {
          it.name
      }
  }
  ```

  

###### switchMap

> switchMap同样是Transformations中的，该方法同样有两个参数 参数一：LiveData 参数二：函数转换体
>
> 与map不同，switchMap的函数转化中必须要返回一个LiveData对象，
>
> 参数一：我们可以作为一个条件决定函数转化中返回什么类型的LiveData，也可以将其包装的数据作为新的LiveData在转化函数中返回

- 场景设定：同样有一个Person，是一个外国人，有firstName和lastName，不过我们有一个条件就是，UI控件只能显示lastName或者firstName

- 代码实现如下

  ```kotlin
  class SwitMapMainActivity : AppCompatActivity() {
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_swit_map_main)
          var switchMapViewModel = ViewModelProvider(this)[SwitchMapViewModel::class.java]
          switchMapViewModel.transformationsLiveData.observe(this, Observer {
              it.let { value ->
                  nameText.text = value
              }
  
          })
  
          setUser.setOnClickListener {
              var person = Person()
              person.firstName = "zhang"
              person.lastName = "qi"
              //当切换 true或者false观察控件显示是否发生变化了呢
              person.condition = false
              switchMapViewModel.conditionLiveData.postValue(person)
          }
      }
  }
  class SwitchMapViewModel:ViewModel() {
  
      var conditionLiveData = MutableLiveData<Person>()
  
      val transformationsLiveData = Transformations.switchMap(conditionLiveData){
           if (it.condition){
              MutableLiveData(it.firstName)
          }else{
             MutableLiveData(it.lastName)
         }
      }
  }
  class Person {
      var name = ""
      var sex = ""
      var firstName = ""
      var lastName = ""
      var condition = false
  }
  ```



###### 合并多个LiveData的数据源

> 该操作使用的是MediatorLiveData，它继承MutableLiveData，在原有功能的基础上，新增了合并多个LiveData的数据源的功能，实则就是一个组件监听多个LiveData，通过addSource方法

- 场景设定：我们有一个控件，和两个LiveData(L1,L2),当L1的数据源发生变化，我们控件显示L1的，当L2数据源发生变化实现L2的，可能我们的做法就是在ViewModel中声明两个LiveData，然后在Activity中分别注册这两个LiveData的观察者，当数据源发生改变去更新UI，其实我们可以使用MediatorLiveData去简化这个操作，不用在Activity中注册两个LiveData的观察者

- 代码实现

  ```kotlin
  class MediatorLiveDataMainActivity : AppCompatActivity() {
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_mediator_live_data_main)
          var mediatorLiveDataViewModel =
              ViewModelProvider(this)[MediatorLiveDataViewModel::class.java]
          mediatorLiveDataViewModel.mediatorLiveData.observe(this, Observer {
              text.text = it
          })
  
          setLiveData1.setOnClickListener {
              mediatorLiveDataViewModel.liveData1.postValue("liveData1")
          }
  
          setLiveData2.setOnClickListener {
              mediatorLiveDataViewModel.liveData2.postValue("liveData2")
          }
      }
  }
  class MediatorLiveDataViewModel : ViewModel() {
      var liveData1 = MutableLiveData<String>()
      var liveData2 = MutableLiveData<String>()
  
      var mediatorLiveData = MediatorLiveData<String>()
  
  
      init {
          mediatorLiveData.addSource(liveData1) {
              Log.d("perform livedata1", it)
              mediatorLiveData.postValue(it)
          }
  
          mediatorLiveData.addSource(liveData2) {
              Log.d("perform livedata2", it)
              mediatorLiveData.postValue(it)
          }
      }
  }
  ```

  

#### 总结

- 主要介绍了一下LiveData的几种使用场景
- LiveData作为ViewModel和Activity之间的桥梁，具有生命感知，并且不会存在内存泄漏，其实靠的是Lifecycled组件
- LiveData在内部使用了Lifecycles组件，来感知生命周期的变化，当Activity销毁的时候也会释放掉引用，防止内存泄漏。
- LiveData的数据源发生变化只会通知到处于Active状态的Activity中的观察者，当Activty有不可见变为可见的状态，那么此时观察者回收到通知，实则为了减少性能损耗嘛
  - 当Activity处于不可见状态是，LiveData发生了多次改变，当变成可见状态时只能收到最新的变化通知

