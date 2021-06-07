#### Kotlin协程-API解析

##### 协程作用域

每个协程都有自己的作用域范围，Kotlin中协程提供了多种协程作用域，让我们来看一下!

###### CoroutineScope



###### GlobalScope

![image-20210607231639757](/Users/zhangqi/Library/Application Support/typora-user-images/image-20210607231639757.png)

###### MainScope

![image-20210607232253040](/Users/zhangqi/Library/Application Support/typora-user-images/image-20210607232253040.png)

##### 开启协程

###### 协程中的调度器

- Dispatchers.Main
- Dispatchers.IO
- Dispatchers.Default

###### launch

###### async

###### withContext()

