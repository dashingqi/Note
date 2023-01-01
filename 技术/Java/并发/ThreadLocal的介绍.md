## ThreadLocal的介绍

#### 简介

ThreadLocal是一个线程内部的数据存储类，同一个ThreadLocal在不同的线程中存储的数据，只能在当前线程中获取到，其他线程是获取不到该线程下存储的数据，起到了线程隔离的作用。
在Android中我们通常很少使用这个东西，不过在Android消息机制中，关于存储Looper的时候我们就用到了ThreadLocal这个类。因为在Android中消息机制的能运转起来需要Looper的支撑，而每个线程中只能有一个Looper以及对应的消息队列。

#### 实操一下

我们在主线程中初始化一个ThradLocal类，然后分别开启两个子线程分别做存储的操作，然后在主线程中也做存储操作，同时提供一个点击事件获取当前主线程下存储值。

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {

        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_thread)

        val threadLocal = ThreadLocal<String>()
        threadLocal.set("8")

        thread {
            threadLocal.set("5")
            Log.d(TAG,Thread.currentThread().name)
            var thread1Get = threadLocal.get()
            Log.d(TAG, "thread1Get == $thread1Get")
        }

        thread {
            threadLocal.set("6")
            Log.d(TAG,Thread.currentThread().name)
            var thread2Get = threadLocal.get()
            Log.d(TAG, "thread2Get == $thread2Get")
        }

        btnMainThread.setOnClickListener {
            var mainGet = threadLocal.get()
            Log.d(TAG, "mainGet == $mainGet")
        }
    }
```

运行结果如下

```xml
2020-12-26 22:43:06.070 11233-11310/com.chiatai.module_java D/ThreadActivity: thread2PreGet == null
2020-12-26 22:43:06.070 11233-11310/com.chiatai.module_java D/ThreadActivity: Thread-3
2020-12-26 22:43:06.070 11233-11310/com.chiatai.module_java D/ThreadActivity: thread2Get == 6
2020-12-26 22:43:06.071 11233-11309/com.chiatai.module_java D/ThreadActivity: thread1PreGet == null
2020-12-26 22:43:06.071 11233-11309/com.chiatai.module_java D/ThreadActivity: Thread-2
2020-12-26 22:43:06.071 11233-11309/com.chiatai.module_java D/ThreadActivity: thread1Get == 5
2020-12-26 22:43:14.716 11233-11233/com.chiatai.module_java D/ThreadActivity: mainGet == 8

```

通过日志我们可以看到，同一个ThreadLcoal实例在不同的线程下分别存储了不同的数据，线程之间互不干扰，各自存储各自获取，那么我们的ThreadLocal是怎么实现这种线程间数据的隔离呢？

#### ThreadLocal的工作原理

```java
public class ThreadLocal<T> {....}
```

可以看出ThreadLoca是一个泛型类可以接受任何类型的数据

###### set

上述中我们使用了set方法来进行数据的存储我们看下ThreadLocal的set

```java
 public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }
```

set方法本身可接受任何类型的数据

首先获取到了调用set方法所处的线程

接着通过getMap()方法获取到了一个ThreadLocalMap,这里的ThreadLocalMap是一个自定义的类似于HashMap的哈希映射，它仅仅使用于用来存储线程局部的值。

当我们获取到的ThreadLocalMap不为空的时候会调用map的set方法将threadLocal实例和value以key-value的形式存储起来

当 该线程下没有创建好的ThreadLcoalMap的时候 我们创建一个ThreadLocalMap并且将ThradLcoal实例与value以key-value的形式存储起来

以上就是ThreadLocal的set方法工作原理，其实总结说下就是 在我们的操作线程下 我们会维护一个ThradLocalMap的哈希映射用来存储 对应的ThradLocal和value，每一个线程中都有一个ThradLcoalMap，仅仅用于维护线程的局部变量。

看完了set方法我们我们看下get的取值操作

###### get

```java
 public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```

首先也是获取到当前操作的线程，然后获取到当前线程下的ThradLocalMap对象

当我们获取到的map不为空的时候 我们根据ThreadLcoal实例获取到对应的value，当value不为null的时候就强转成我们需要的类型数据并将之返回回去

当map为空的时候调用了setInitialValue()方法

```java
private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }
```

首先initialValue返回了一个null，那么setInitialValue也就返回了一个null，说明当线程中还没有创建ThreadLocalMap的时候调用了get方法就返回null

接着我们看下其他代码干嘛了。一看，嗯很好原来当map存在的时候就把null存储了一下？？ 没有存储的话就在当前线程中创建一个map然后存储

我们的理解就是既然在当前线程中调用了get方法 之前没有做过存储的操作就返回一个null，同时为该线程创建一个ThreadLocalMap，这样之后不管是get还是set就不需要走创建ThreadLocalMap的操作了。

下面我们就用一张图来表明一下ThraedLocal的工作流程

![ThreadLocal工作原理的示意图.png](https://upload-images.jianshu.io/upload_images/4997216-6c8fd53600c36072.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)