#### Kotlin协程-API解析

##### 协程作用域

每个协程都有自己的作用域范围，Kotlin中协程提供了多种协程作用域，让我们来看一下!

###### CoroutineScope

![image-20210608070017179](/Users/zhangqi/Library/Application Support/typora-user-images/image-20210608070017179.png)

- 创建一个通用的协程的作用域；
- 创建协程最好的方式是通过CoroutineScope和MainScope的工厂方法；
- 额外的上下文元素可以通过加号进行连接；

![image-20210608070528793](/Users/zhangqi/Library/Application Support/typora-user-images/image-20210608070528793.png)

- CoroutineScope是一个接口类；
- 内部包含一个协程的上下文变量。

###### GlobalScope

![image-20210607231639757](/Users/zhangqi/Library/Application Support/typora-user-images/image-20210607231639757.png)

- 是一个全局的协程作用域；
- 全局协程的生命周期是和应用的生命周期保持一致的；
- GlobalScope实现了CoroutineScope，自己实现全局的功能。

###### MainScope

![image-20210607232253040](/Users/zhangqi/Library/Application Support/typora-user-images/image-20210607232253040.png)

- 创建一个UI组件的协程作用域；
- 它是自带【SupervisorJob】和【Dispatchers.Main】的上下文元素；如果你想要上下文元素可以通过加号连接符（val scope = MainScope() + CoroutineName("MyActivity")）
- 它同样实现了CoroutineScope

##### 开启协程

在介绍开启协程之前先说下协程中的调度器；

###### 协程中的调度器

所谓调度器就是要明确将要开启的协程要运行在哪个线程上

- Dispatchers.Main：开启的协程度是运行在主线程上 ；
- Dispatchers.IO：开启的协程是运行在IO线程上；
- Dispatchers.Default

###### launch

![image-20210608081550131](/Users/zhangqi/Library/Application Support/typora-user-images/image-20210608081550131.png)

![image-20210608081600050](/Users/zhangqi/Library/Application Support/typora-user-images/image-20210608081600050.png)

- launch是CoroutineScope的扩展函数；扩展函数怎么样？可以在具有协程作用域中直接调用呗！
- 它返回一个Job对象，我们可以通过Job对象来操控协程【开启】和【取消】；同时launch函数启动的协程是不会阻塞当前线程；
- launch函数启动的协程会继承父协程所在的协程上下文【context】，如果其上下文context不包含任意协程调度器【dispatcher】，那么会默认使用【Dispatchers.Default】；
- 对于launch函数创建的协程内部抛出没有捕获的异常，那么【默认】情况下会导致父协程取消；

###### async

![image-20210608081453037](/Users/zhangqi/Library/Application Support/typora-user-images/image-20210608081453037.png)

![image-20210608081512061](/Users/zhangqi/Library/Application Support/typora-user-images/image-20210608081512061.png)

- async是CoroutineScope的扩展函数
- 它返回一个Deferred对象，该对象不仅可以取消协程更重要的是可以获取异步任务返回的结果；这是一个很有用的特性！异步任务的结果可以通过Deferred对象调用await()方法来获取到
- 对于async函数创建的协程，内部发生异常需要用户手动抛出默认是不会抛出，所以需要用户在await处进行捕获！

###### withContext()

###### runBlocking



