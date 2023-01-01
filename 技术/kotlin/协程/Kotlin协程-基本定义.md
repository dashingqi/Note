#### Kotlin中的协程

在说协程之前，简单介绍下Kotlin

##### Kotlin

Kotlin是2011年起 由JetBrain公司开始开发；

到2017年GoogleIO大会上宣布Kotlin将称成为Android应用开发的第一语言；

更是在2019年IO大会上，宣布”Kotlin First“的口号，在Android应用开发中，Kotlin从此开始它的辉煌时刻了！

##### Kotlin中协程的定义

协程是在Kotlin1.3版本引入的；

Kotlin中的协程就是一套由Kotlin官方提供的线程框架API，就像Java的Executor和Android中的AsyncTask系列的API；

在Android的官方文档中也有提到：协程是我们在Android上进行异步编程的推荐解决方案；

那么既然有了Executor以及AsyncTask系列的API，为什么Android官方说它是异步编程的推荐解决方案呢？

##### Kotlin中协程好在哪里

###### 开始之前-闭包

在之前先说下【闭包】这个概念，因为在协程的使用过程中会涉及到这个概念；

我们在使用Kotlin的API的时候也会经常用到闭包；

【闭包】这个东西不是Kotlin中特有的特性，是从Java8开始就出现了；

我们就以Thread为例子讲述一下

```kotlin
// 创建一个Thread，内部传递一个Runnable的匿名内部类对象
// 在Java8之后，我们将只有单一方法的接口称为SAM（Single Abstract Method）接口
Thread(object:Runnable{
  override fun run(){
    // ....doWork....
  }
})
// 同样在Java8之后我们可以使用lambda简化 SAM接口的写法
Thread({
  // ....doWork....
})
// 在Kotlin中有这样一个语法糖：当函数最后一个参数Lambda表达式时，可以将Lambda写在括号外面，这就是Kotlin的闭包原则
Thread{
  // .... doWork ....
}
```

###### 基本的使用

```kotlin
// 方法一：这种方式开启的协程是线程阻塞的，不能在实际业务中使用；
runBlocking{
  
}
// 方法二：使用GlobalScope,直接调用launch开启协程
// 该方式开启的协程是不会阻塞线程的。但是它的生命周期是和app一致的。
GlobalScope.launch{
  
}
// 方法三：通过CoroutineScope创建一个协程对象
// 其中context是为CoroutineContext类型的参数
// 通常比较推荐使用这种方法，可以通过传递的context参数去管理和控制协程的生命周期；
// 这里的context可以理解为协程的上下文
// launch里面包裹的代码就是我们要开启的协程
val coroutineScope = CoroutineScope(context)
cortineScope.launch{
  // .... doWork ....
}
```

###### 好在哪里

在给出结论之前，我们先看下面这两个场景

场景一：我们从Server拿到一个学生信息：

```kotlin
private fun getData(){
    getStudentInfo(object :InfoCallback{
        override fun onSuccess(student:Student) { 
          // 获取到学生信息
        }
        override fun onFailure() {      
        }
    })
}
```

怎么样，产品是要迭代，需求是要变动的，这时出现了下面这个场景二

场景二：拿到学生信息之后，需要查询她/他是在哪个班级？

```kotlin
private fun getData(){
    getStudentInfo(object :InfoCallback{
        override fun onSuccess(student:Student) {
						//根据学生信息去查询班级信息
            getClassInfo(student:Student,object :InfoCallback{
        		override fun onSuccess(class:Class) {
            		//获取到班级信息
        			}
       			override fun onFailure() {
        			}
    			})
        }
        override fun onFailure() {  
        }
    })
}
```

那么我们来看下使用协程代码是怎么写的

```kotlin
private fun getData(){
  // 创建一个CoroutineScope对象
  val coroutineScope = CoroutineScope(Dispatchers.Main)
  // 调用launch方法 开启一个协程，其中Dispatchers.Mian就制定我们的协程是跑在主线程中的
  cortineScope.launch(Dispatchers.Main){
    // async 是开启了一个新的协程，它返回的是Coroutine是实现了Deferred接口的
    val student = async { getStudent() }
    val class   = async { getClass(student.await()) }
    // 拿到班级信息，通过调用deferred.await()可以获取到方法的返回值
    val class1  = class.await()
    tv.text = class1.name
  }
}

/**
 * 获取学生信息
 */
private suspend fun getStudent(): Student {
    withContext(Dispatchers.IO){
      delay(100)
      return "zhangqi"
    } 
}

/**
 *获取班级信息
 */
private suspend fun getClass(): Class {
    withContext(Dispatchers.IO){
      delay(100)
      return "一班" 
    }
}
```

怎么样 是不是 没有对比就没有伤害！！！

我们可能使用回调这种方式使用习惯了，没觉得有什么不妥的，但是和Kotlin协程实现方式相比

这种回调嵌套回调，是不是有点代码可读性差，维护性也不好；

所以协程好在哪里：

使我们的代码可读性和可维护性变强了；

最重要的是：用同步代码的方式写出异步的代码，这也是Kotlin协程相比较于之前多线程框架的核心竞争力；

那么协程凭啥就能这么写呢，我们看上面的代码

1. 使用了 async{} 和await
2. 我们在我们定义的函数中使用了suspend关键字 ，被suspend修饰方法的我们称之为挂起函数

接着我们介绍下挂起函数

##### 挂起函数

所谓的挂起就是能自动切线程并且能自动切回来的动作

自定义挂起函数 要使用 suspend 关键字

```kotlin
// 挂起函数只能由协程或者其他挂起函数进行调度
suspend fun method(){
  withContext(Dispatchers.IO){
    
  }
}
```

当我们使用launch或者async启动协程后，在协程中调用到挂起函数的时候，会被【suspend】也即就是挂起

当我们的挂起函数执行完毕之后，协程会帮助我们把线程切回来

就是当我们的协程运行在主线程上时，当调用到suspend函数，根据Dispatchers（调度器）制定线程的不同，就可能会发生线程切换，当我们的代码执行完毕后，协程会自动切回到原线程上；

【切回】这个动作在Kotlin中叫做 resume 也就是恢复，这个是协程的特性；

所以挂起函数要么在其他挂起函数中调用，要么在协程中调用；为啥呢？因为它要切过去还要切回来呢！

其实suspend关键字并不是起到挂起的作用，它单纯只是一个【提醒】的作用，告诉调用者我这个函数将会一个耗时的函数，请在协程中调用我;

###### 什么时候需要挂起函数

执行耗时的任务

###### 怎么写

```kotlin
suspend fun method(){
  // withContext 也是一个挂起函数 内部是由suspend关键字修饰的
	withContext(Dispatchers.IO){
    //在IO线程中，执行耗时的任务
  }	
}
```

在Kotlin中这个挂起是非阻塞式的？

##### 非阻塞挂起

针对非阻塞挂起 我们可以拆分开 【非阻塞】【挂起】

【非阻塞】

非阻塞是相对于阻塞而言的；

对于线程的阻塞可以拿我们生活中一些事来说：

你开车在路上，不巧车坏了，你停在这里堵主后面的司机师傅了（你在执行耗时任务），

等到车修好了，可以出发了（耗时任务结束了）；

或者你可以把车换到其他没有车路上，等车被修好（开启了其他线程，执行你这个任务）

其实非阻塞是针对挂起这个动作特点的一个描述；本身挂起就是能自动切线程并切回来的操作，就是不阻塞UI线程的；相比较于之前的线程框架 这个能自动切换回来；

在我们代码中的体现就是 我们看似以同步的方式在写代码，但是依靠协程却完成了异步任务；

##### 总结一波

协程就是切线程；

挂起就是能自动切回来的切线程

挂起的非阻塞就是看似以阻塞的方式完成了非阻塞的任务
