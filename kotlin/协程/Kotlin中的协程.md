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

在给出结论之前，我们先看下面这个场景

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
  GlobalScope.launch(Dispatchers.Main){
    
  }
}
```



