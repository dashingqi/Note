#### 简介

- 之前的文章有介绍过LiveData的基本使用，这篇文章主要来讲解一下LiveData的工作原理
- 在我们使用LiveData的时候，会在组件中注册一个观察者来监听我们LiveData数据源的变化，调用了Observer
- 当我们去改变LiveData的数据源的时候我们在UI线程中一般使用的是setValue(),而在非UI线程中我们必须使用postValue()
- 那么综上所述，我们就从如下三个点去看下LiveData的工作流程
  - LiveData # observer()方法
  - setValue()
  - postValue()

#### 源码解析

###### obsever()

```java
 @MainThread
    public void observe(@NonNull LifecycleOwner owner, @NonNull Observer<? super T> observer) {
      	//必须运行在主线程中
        assertMainThread("observe");
      	//如果当前组件的状态是DESTROYED就不做任何操作（其实就是不添加这个观察者）
        if (owner.getLifecycle().getCurrentState() == DESTROYED) {
            // ignore
            return;
        }
      	// 将owner和observer传入到LifecycleBoundObserver构建了一个LifecycleBoundObserver对象
      	// LifecycleBoundObserver extends ObserverWrapper implements LifecycleEventObserver
				// LifecycleEventObserver extends LifecycleObserver
        LifecycleBoundObserver wrapper = new LifecycleBoundObserver(owner, observer);
      	// 注释1
        ObserverWrapper existing = mObservers.putIfAbsent(observer, wrapper);
        if (existing != null && !existing.isAttachedTo(owner)) {
            throw new IllegalArgumentException("Cannot add the same observer"
                    + " with different lifecycles");
        }
        if (existing != null) {
            return;
        }
      	//注释2
        owner.getLifecycle().addObserver(wrapper);
    }
```

- 注释1处：mObservers ----> SafeIterableMap<Observer<? super T>, ObserverWrapper> mObservers =
              new SafeIterableMap<>();
  - mObservers的putIfAbsent()和map的put方法还不一样，如果根据key获取的value不为null，就将当前的value返回，如果为null，就将当前的键值对存储起来，然后返回一个null
- 注释2处：如果注释1返回的值为null ，就会走到注释2处，添加观察者来观察生命周期拥有者的生命周期也就是我们的Activity或者Fragment的，在Lifecycle分析中我们知道 组件的生命周期发生变化的时候会回调Observer#onStateChanged()方法，看下LifecycleBoundObserver的onStateChanged

```java
@Override
        public void onStateChanged(@NonNull LifecycleOwner source,
                @NonNull Lifecycle.Event event) {
                // 当组件的生命周期状态变成DESTROYED
            if (mOwner.getLifecycle().getCurrentState() == DESTROYED) {
            		// 就将当前添加的观察者移除，这也是为什么处于不可见状态的组件中的观察者收不到 LiveData数据源变换的回调
                removeObserver(mObserver);
                return;
            }
            activeStateChanged(shouldBeActive());
        }
```



###### setValue()

```java
@MainThread
    protected void setValue(T value) {
      	//检查是否运行在祝线程中
        assertMainThread("setValue");
      	//版本自增1
        mVersion++;
      	//将当前的value赋值给mData
        mData = value;	
      	// 调用了dispatchingValue的方法 传入了null值
        dispatchingValue(null);
    }


 void dispatchingValue(@Nullable ObserverWrapper initiator) {
				// 当前正在处于分发数据中   			
        if (mDispatchingValue) {
            mDispatchInvalidated = true;
          	//继续之前的分发数据
            return;
        }
        mDispatchingValue = true;
        do {
            mDispatchInvalidated = false;
          	//注释1
            if (initiator != null) {
                considerNotify(initiator);
                initiator = null;
            } else {
              	//注释2
                for (Iterator<Map.Entry<Observer<? super T>, ObserverWrapper>> iterator =
                        mObservers.iteratorWithAdditions(); iterator.hasNext(); ) {
                    considerNotify(iterator.next().getValue());
                    if (mDispatchInvalidated) {
                        break;
                    }
                }
            }
        } while (mDispatchInvalidated);
        mDispatchingValue = false;
    }
```

- 在注释1和注释2中，都会调用到considerNotify()方法，区别在于
  - 注释1: initiator不为null，并且只调用一次considerNotify()
  - 注释2：initiator为null，会遍历mObservers中存储的Observer，然后调用considerNotify(拿到对应的值，传给considerNotify，这个值对应着ObserverWrapper)

```java
private void considerNotify(ObserverWrapper observer) {
  			// 如果当前的组件不是处于活跃状态，就停止运行
        if (!observer.mActive) {
            return;
        }
        // Check latest state b4 dispatch. Maybe it changed state but we didn't get the event yet.
        //
        // we still first check observer.active to keep it as the entrance for events. So even if
        // the observer moved to an active state, if we've not received that event, we better not
        // notify for a more predictable notification order.
        if (!observer.shouldBeActive()) {
            observer.activeStateChanged(false);
            return;
        }
        if (observer.mLastVersion >= mVersion) {
            return;
        }
        observer.mLastVersion = mVersion;
  			// 注释1
        observer.mObserver.onChanged((T) mData);
    }
```

- 在注释1处：如果当前的组件处于活跃状态，之前的条件都满足了，就会拿到添加的观察者，并且回调它的onChanged方法。onChanged方法就是我们监听到数据源发生改变做的处理逻辑了。

###### postValue()

```java
 protected void postValue(T value) {
        boolean postTask;
        synchronized (mDataLock) {
            postTask = mPendingData == NOT_SET;
            mPendingData = value;
        }
        if (!postTask) {
            return;
        }
        //从工作线程切换到UI线程，同样通过创建一个Handler关联到祝线程的Looper，调用了post方法，提交一个Runnable，切换到祝线程去执行run方法中的逻辑
        ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
    }
    
   private final Runnable mPostValueRunnable = new Runnable() {
        @SuppressWarnings("unchecked")
        @Override
        public void run() {
            Object newValue;
            synchronized (mDataLock) {
                newValue = mPendingData;
                mPendingData = NOT_SET;
            }
            // postValue()最终还是调用了setValue()方法
            setValue((T) newValue);
        }
    };
```

#### 总结

- LiveData内部是依靠Lifecycle来感知组件的生命周期，开避免内部泄漏的。
- 当观察到组件处于不可见状态的时候，会移除当前观察者的监听
- 当处于活跃状态的时候，当LiveData的数据源发生变化的会监听到数据变化拿到数据，去更新UI
- 对于set和post这两种更新数据源，其实最终post还是调用了set，只不过在中间用到了Handler关联到了主线程中的Looer，调用了post切换到主线程中。