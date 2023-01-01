#### 简介

- 上一遍文章中介绍了如何使用Lifecycle来感知Acitivity的生命周期的。
- 让我们来简单的回顾一下
  - 首先新建一个MyObsever 继承至 LifecycleObsever（如果是使用Java8 可以使用DefaultLifecycleObserver，可以不用写注解，直接重写对应的方法就可以了）
  - 在Activity（androidx下的 ComponentActivity）中通过 getLifecycle().addObserver(MyObserver()) 这行代码，将我们的观察者添加到Lifecycle中（确切说是LifcycleRegister中）
  - 通过如上的操作就能感知到Activity的生命周期了
- 其实对于我们程序员来说，日常最多的就是使用框架，使用API，来完成需求，针对我们项目中使用的开源或者官方提供封装好的框架，我们很有必要去了解一下它的原理，起码在和别人吹牛的时候，可以有东西吹啊，觉得你这个人不禁需求完成的不错，而且还有一颗强烈学习的心
- 针对Lifecycle的原理，我们可以从这行代码入手 getLifecycle().addObserver(MyObserver())，可以分解成如下
  - getLifecycle() ---> 获取到Lifecycle类型的对象
  - 将我们自定义好的观察者添加进去 addObserver()

#### 原理分析

###### getLifecycle()

- 点开我们的getLifecycle()方法我们可以看到，我们来到了ComponentActivity中

  ```java
   @NonNull
      @Override
      public Lifecycle getLifecycle() {
          return mLifecycleRegistry;
      }
  ```

  - 该方法返回的是一个LifyCycleRegistry对象
  - LifecycleRegister是继承至Lifecycle的

- 看下我们这个ComponentActivity（挑重要有关的说了）

  - 它继承至androidx.core.app.ComponentActivity
  - 并且实现了LifecycleOwner（生命周期拥有者）
    - 其实我们的getLifecycle()方法就是实现LifcycycleOwner接口内的方法

###### addObserver()方法

> 调用的addObserver()是属于LifyCycleRegistry中的，我们先来看下LifyCycleRegistry这个类

- 生命周期登记，它是Lifecycle的子类，起到添加观察者、响应生命周期事件、分发生命周期事件的作用

- 部分核心源码

  ```java
  private FastSafeIterableMap<LifecycleObserver, ObserverWithState> mObserverMap =
              new FastSafeIterableMap<>();
    
  private State mState;
     
  private final WeakReference<LifecycleOwner> mLifecycleOwner;
  
  // 顺便看下 Lifecycle的源码
  public abstract class Lifecycle {
  
     
     
      @MainThread
      public abstract void addObserver(@NonNull LifecycleObserver observer);
  
    
      @MainThread
      public abstract void removeObserver(@NonNull LifecycleObserver observer);
  
      @MainThread
      @NonNull
      public abstract State getCurrentState();
  
      @SuppressWarnings("WeakerAccess")
      public enum Event {
       
          ON_CREATE,
         
          ON_START,
         
          ON_RESUME,
         
          ON_PAUSE,
         
          ON_STOP,
       
          ON_DESTROY,
        
          ON_ANY
      }
  
    
      @SuppressWarnings("WeakerAccess")
      public enum State {
    
          DESTROYED,
  
          INITIALIZED,
  
         
          CREATED,
  
          STARTED,
  
         
          RESUMED;
  
          public boolean isAtLeast(@NonNull State state) {
              return compareTo(state) >= 0;
          }
      }
  }
  ```

  - FastSafeIterableMap是一个Map，用来保存观察者和它对应的状态
  - mState对应着当前的状态（Lifecycle）
  - LicecycleOwner表示生命周期的拥有者，我们的ComponentActivity实现了该接口
  - 可以看到Lifecycle中有两个枚举类，分别是事件和状态，还有添加和移除观察者的方法和赶会当前Lifecycle对应的状态

- LifyCycleRegistry # addObserver()

  ```java
  @Override
      public void 
        (@NonNull LifecycleObserver observer) {
        	//当前的状态是DESTROY的话，添加的Observer的初始状态就是DESTROY否则就是INITIALIZED
          State initialState = mState == DESTROYED ? DESTROYED : INITIALIZED;
          //创建一个ObserverWithState，它是LifecycleRegister类中的一个静态内部类，将添加的Observer与状态关联到一起，（说白就是用它来维持Observer与State的对应关系）
          ObserverWithState statefulObserver = new ObserverWithState(observer, initialState);
          // 以observer作为key，ObserverWithState作为value存储到Map中
        	//如果当前Observer对应的value不存在，就将observer与对应的value存储到map
        	//否则的话就将Observer对应的value获取到并且返回当前的value
        ObserverWithState previous = mObserverMap.putIfAbsent(observer, statefulObserver);
  
          if (previous != null) {
              return;
          }
          LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
          if (lifecycleOwner == null) {
              // it is null we should be destroyed. Fallback quickly
              return;
          }
  
          boolean isReentrance = mAddingObserverCounter != 0 || mHandlingEvent;
          State targetState = calculateTargetState(observer);
          mAddingObserverCounter++;
        	//注释1
          while ((statefulObserver.mState.compareTo(targetState) < 0
                  && mObserverMap.contains(observer))) {
              pushParentState(statefulObserver.mState);
              statefulObserver.dispatchEvent(lifecycleOwner, upEvent(statefulObserver.mState));
              popParentState();
              // mState / subling may have been changed recalculate
              targetState = calculateTargetState(observer);
          }
  
          if (!isReentrance) {
              // we do sync only on the top level.
              sync();
          }
          mAddingObserverCounter--;
      }
  ```

  - 该方法是添加LifecycleObserver观察者，并且可以将之前生命状态分发给当前添加的Observer的，例如我们在Activtity的onResume之后添加这个Observer，那么该Observer依然能收到ON_CREATE事件
  - 注释一：如果Observable的初始状态是INITIALIZED，当前的状态是RESUMED，那么需要将INITIALIZED到RESUMED之间的所有事件都分发给Observer
  - 到这里我们回过头来看下 ComponentActivity

- ComponentActivity # onCreate()

  ```java
   @Override
      protected void onCreate(@Nullable Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          mSavedStateRegistryController.performRestore(savedInstanceState);
        	//注释1
          ReportFragment.injectIfNeededIn(this);
          if (mContentLayoutId != 0) {
              setContentView(mContentLayoutId);
          }
      }
  ```

  - 注释1:我们调用了ReportFragment # injectIfNeededIn(this)

- ReportFragment # injectIfNeededIn()

  ```java
  public static void injectIfNeededIn(Activity activity) {
          // ProcessLifecycleOwner should always correctly work and some activities may not extend
          // FragmentActivity from support lib, so we use framework fragments for activities
          android.app.FragmentManager manager = activity.getFragmentManager();
          if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
              manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
              // Hopefully, we are the first to make a transaction.
              manager.executePendingTransactions();
          }
      }
  ```

  - 这里我们添加了一个Fragment到Activity中
  - 并且这个Activity是一个没有界面
  - 其实添加这个Fragment就是用来感知Activity的生命周期的

- ReportFragment的生命周期方法

  ```java
  @Override
      public void onActivityCreated(Bundle savedInstanceState) {
          super.onActivityCreated(savedInstanceState);
          dispatchCreate(mProcessListener);
          dispatch(Lifecycle.Event.ON_CREATE);
      }
  
      @Override
      public void onStart() {
          super.onStart();
          dispatchStart(mProcessListener);
          dispatch(Lifecycle.Event.ON_START);
      }
  
      @Override
      public void onResume() {
          super.onResume();
          dispatchResume(mProcessListener);
          dispatch(Lifecycle.Event.ON_RESUME);
      }
  
      @Override
      public void onPause() {
          super.onPause();
          dispatch(Lifecycle.Event.ON_PAUSE);
      }
  
      @Override
      public void onStop() {
          super.onStop();
          dispatch(Lifecycle.Event.ON_STOP);
      }
  
      @Override
      public void onDestroy() {
          super.onDestroy();
          dispatch(Lifecycle.Event.ON_DESTROY);
          // just want to be sure that we won't leak reference to an activity
          mProcessListener = null;
      }
  
  ```

  - 通过观察发现，都调用了dispatch()方法

- ReportFragment # dispatch()方法

  ```java
  private void dispatch(Lifecycle.Event event) {
          Activity activity = getActivity();
          if (activity instanceof LifecycleRegistryOwner) {
              ((LifecycleRegistryOwner) activity).getLifecycle().handleLifecycleEvent(event);
              return;
          }
  
          if (activity instanceof LifecycleOwner) {
              Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
              if (lifecycle instanceof LifecycleRegistry) {
                  ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
              }
          }
      }
  ```

  - 这里可以看出 有两个Owner 分别是LifecycleOwner和LifecycleRegistryOwner区别是：
    - LifecycleOwner # getLifecycle() ----> Lifecycle
    - LifecycleRegistryOwner # getLifecycle() ---> LifecycleRegister()
    - 而我们的LifecycleRegistryOwner是继承至LifecycleOwner的
  - 最终调用的都是 LifecycleRegistry #handleLifecycleEvent()的方法

- LifecycleRegistry # handleLifecycleEvent()

  ```java
   public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {
          State next = getStateAfter(event);
          moveToState(next);
      }
  		
  		/**
  		* 
  		*根据事件返回我们当前的状态
  		* 这个事件就是从ReportFragment中传递过来的
  		*/
     static State getStateAfter(Event event) {
          switch (event) {
              case ON_CREATE:
              case ON_STOP:
                  return CREATED;
              case ON_START:
              case ON_PAUSE:
                  return STARTED;
              case ON_RESUME:
                  return RESUMED;
              case ON_DESTROY:
                  return DESTROYED;
              case ON_ANY:
                  break;
          }
          throw new IllegalArgumentException("Unexpected event value " + event);
      }
  
  	/**
  	* 改变状态
  	*
  	*/
    private void moveToState(State next) {
          if (mState == next) {
              return;
          }
          mState = next;
          if (mHandlingEvent || mAddingObserverCounter != 0) {
              mNewEventOccurred = true;
              // we will figure out what to do on upper level.
              return;
          }
          mHandlingEvent = true;
          sync();
          mHandlingEvent = false;
      }
  ```

- LifecycleRegistry # sync()

  ```java
  private void sync() {
          LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
          if (lifecycleOwner == null) {
              throw new IllegalStateException("LifecycleOwner of this LifecycleRegistry is already"
                      + "garbage collected. It is too late to change lifecycle state.");
          }
          while (!isSynced()) {
              mNewEventOccurred = false;
              // no need to check eldest for nullability, because isSynced does it for us.
            	//注释1
              if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {
                  backwardPass(lifecycleOwner);
              }
              Entry<LifecycleObserver, ObserverWithState> newest = mObserverMap.newest();
             //注释2
              if (!mNewEventOccurred && newest != null
                      && mState.compareTo(newest.getValue().mState) > 0) {
                  forwardPass(lifecycleOwner);
              }
          }
          mNewEventOccurred = false;
      }
  ```

  - 注释1:如果当前的状态值小于 Observer状态值，需要将Observer的状态值减小到和当前状态值相等。
  - 注释2:如果当前当前的状态值大于Observer的状态值，需要将Observer的状态值增大到和当前状态值相等。

- LifecycleRegistry # backwardPass

  ```java
  private void backwardPass(LifecycleOwner lifecycleOwner) {
          Iterator<Entry<LifecycleObserver, ObserverWithState>> descendingIterator =
                  mObserverMap.descendingIterator();
          while (descendingIterator.hasNext() && !mNewEventOccurred) {
              Entry<LifecycleObserver, ObserverWithState> entry = descendingIterator.next();
              ObserverWithState observer = entry.getValue();
              while ((observer.mState.compareTo(mState) > 0 && !mNewEventOccurred
                      && mObserverMap.contains(entry.getKey()))) {
                  Event event = downEvent(observer.mState);
                  pushParentState(getStateAfter(event));
                  observer.dispatchEvent(lifecycleOwner, event);
                  popParentState();
              }
          }
      }
  ```

  - 对应着sync中注释1的代码

- LifecycleRegistry # forwardPass

  ```java
  private void forwardPass(LifecycleOwner lifecycleOwner) {
          Iterator<Entry<LifecycleObserver, ObserverWithState>> ascendingIterator =
                  mObserverMap.iteratorWithAdditions();
          while (ascendingIterator.hasNext() && !mNewEventOccurred) {
              Entry<LifecycleObserver, ObserverWithState> entry = ascendingIterator.next();
              ObserverWithState observer = entry.getValue();
              while ((observer.mState.compareTo(mState) < 0 && !mNewEventOccurred
                      && mObserverMap.contains(entry.getKey()))) {
                  pushParentState(observer.mState);
                  observer.dispatchEvent(lifecycleOwner, upEvent(observer.mState));
                  popParentState();
              }
          }
      }
  ```

  - 对应着sync注释2的代码

- 在上述的两段代码中。（forwardPass和backwarPass）在调整State时，都调用了 observer # dispatchEvent()

  - Observer ---> ObserverWithState

- LifecycleRegistry # ObserverWithState # dispatchEvent()

  ```java
   void dispatchEvent(LifecycleOwner owner, Event event) {
              State newState = getStateAfter(event);
              mState = min(mState, newState);
              mLifecycleObserver.onStateChanged(owner, event);
              mState = newState;
          }
  ```

  - 在此方法中我们调用了 LifecycleEventObserver的onStateChange方法
  - 那么这个LifecycleEventObserver时怎么来的呢？
  - 在上文中 我们在LifecycleRegistry # addObserver()方法中我们构建了一个ObserverWithState(),我们看下ObserverWithState()的构造方法

- ObserverWithState()

  ```java
  
  ObserverWithState(LifecycleObserver observer, State initialState) {
              mLifecycleObserver = Lifecycling.lifecycleEventObserver(observer);
              mState = initialState;
   }
  ```

  - 可以看到我们的mLifecycleObserver对象时通过 Lifecycling.lifecycleEventObserver(observer);获取到的

- Lifecycling # lifecycleEventObserver()

  ```java
  @NonNull
      static LifecycleEventObserver lifecycleEventObserver(Object object) {
        	// 我们传入的object是我们自定义的MyObservable
        	// 这里面 LifecycleEventObserver extends LifecycleObserver
        	// FullLifecycleObserver extends LifecycleObserver
        	// 对于instanceof关键字 左面是类的引用，右面是一个接口或者一个类 的类型
        	// 返回true的情况，左边的引用类型是右边类型的，或者是是其子类，或者是实现类的类型
       		// 针对这里的情况，好像都不是
        	// isLifecycleEventObserver ==  false
        	// isFullLifecycleObserver == false
          boolean isLifecycleEventObserver = object instanceof LifecycleEventObserver;
          boolean isFullLifecycleObserver = object instanceof FullLifecycleObserver;
          if (isLifecycleEventObserver && isFullLifecycleObserver) {
              return new FullLifecycleObserverAdapter((FullLifecycleObserver) object,
                      (LifecycleEventObserver) object);
          }
          if (isFullLifecycleObserver) {
              return new FullLifecycleObserverAdapter((FullLifecycleObserver) object, null);
          }
  
          if (isLifecycleEventObserver) {
              return (LifecycleEventObserver) object;
          }
  
          final Class<?> klass = object.getClass();
          int type = getObserverConstructorType(klass);
          if (type == GENERATED_CALLBACK) {
              List<Constructor<? extends GeneratedAdapter>> constructors =
                      sClassToAdapters.get(klass);
              if (constructors.size() == 1) {
                  GeneratedAdapter generatedAdapter = createGeneratedAdapter(
                          constructors.get(0), object);
                  return new SingleGeneratedAdapterObserver(generatedAdapter);
              }
              GeneratedAdapter[] adapters = new GeneratedAdapter[constructors.size()];
              for (int i = 0; i < constructors.size(); i++) {
                  adapters[i] = createGeneratedAdapter(constructors.get(i), object);
              }
              return new CompositeGeneratedAdaptersObserver(adapters);
          }
          return new ReflectiveGenericLifecycleObserver(object);
      }
  ```

  - 针对上述的分析，我们可以知道，lifecycleEventObserver()方法真正我们需要的是如下代码

  ```java
  final Class<?> klass = object.getClass();
  				// 获取到的type=1
          int type = getObserverConstructorType(klass);
  				// GENERATED_CALLBACK == 2
          if (type == GENERATED_CALLBACK) {
              List<Constructor<? extends GeneratedAdapter>> constructors =
                      sClassToAdapters.get(klass);
              if (constructors.size() == 1) {
                  GeneratedAdapter generatedAdapter = createGeneratedAdapter(
                          constructors.get(0), object);
                  return new SingleGeneratedAdapterObserver(generatedAdapter);
              }
              GeneratedAdapter[] adapters = new GeneratedAdapter[constructors.size()];
              for (int i = 0; i < constructors.size(); i++) {
                  adapters[i] = createGeneratedAdapter(constructors.get(i), object);
              }
              return new CompositeGeneratedAdaptersObserver(adapters);
          }
          return new ReflectiveGenericLifecycleObserver(object);
  ```

  - 经过上述分析，可以看出返回的是 ReflectiveGenericLifecycleObserver()

- ReflectiveGenericLifecycleObserver

  ```java
  class ReflectiveGenericLifecycleObserver implements LifecycleEventObserver {
      private final Object mWrapped;
      private final CallbackInfo mInfo;
  
      ReflectiveGenericLifecycleObserver(Object wrapped) {
          mWrapped = wrapped;
          mInfo = ClassesInfoCache.sInstance.getInfo(mWrapped.getClass());
      }
  
      @Override
      public void onStateChanged(@NonNull LifecycleOwner source, @NonNull Event event) {
          mInfo.invokeCallbacks(source, event, mWrapped);
      }
  }
  
   @SuppressWarnings("ConstantConditions")
          void invokeCallbacks(LifecycleOwner source, Lifecycle.Event event, Object target) {
              invokeMethodsForEvent(mEventToHandlers.get(event), source, event, target);
              invokeMethodsForEvent(mEventToHandlers.get(Lifecycle.Event.ON_ANY), source, event,
                      target);
          }
  
          private static void invokeMethodsForEvent(List<MethodReference> handlers,
                  LifecycleOwner source, Lifecycle.Event event, Object mWrapped) {
              if (handlers != null) {
                  for (int i = handlers.size() - 1; i >= 0; i--) {
                      handlers.get(i).invokeCallback(source, event, mWrapped);
                  }
              }
          }
      }
  
   void invokeCallback(LifecycleOwner source, Lifecycle.Event event, Object target) {
              //noinspection TryWithIdenticalCatches
              try {
                  switch (mCallType) {
                      case CALL_TYPE_NO_ARG:
                          mMethod.invoke(target);
                          break;
                      case CALL_TYPE_PROVIDER:
                          mMethod.invoke(target, source);
                          break;
                      case CALL_TYPE_PROVIDER_WITH_EVENT:
                          mMethod.invoke(target, source, event);
                          break;
                  }
              } catch (InvocationTargetException e) {
                  throw new RuntimeException("Failed to call observer method", e.getCause());
              } catch (IllegalAccessException e) {
                  throw new RuntimeException(e);
              }
          }
  ```

  - 根据调用事件线走下去，调用了ReflectiveGenericLifecycleObserver的onStateChanged()
  - 继而从invokeCallbacks() ---> invokeMethodsForEvent() ---> invokeCallback()
  - 当生命周期发生改变的时候，最终通过反射（ReflectiveGenericLifecycleObserver存储了我们在 Observer 里注解的方法）调用了我们在观察者中利用注解声明的方法。

###### 总结

- Lifecycle将Activity的生命周期函数对应成Event，生命周期改变，会将Event传递给LifecycleRegistry，LifecycleRegistry中会修正State的值，并且触发事件的分发，通过反射通知到LifecycleObsever中接受事件的方法。