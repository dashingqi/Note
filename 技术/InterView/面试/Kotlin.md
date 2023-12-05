#### Kotlin
###### 优势
- 安全性更好，在语言层面上避免空指针的问题
- 更多好用的关键字以及高级语法糖（扩展、高阶函数）
- 带有的协程，简化了多线程编程；
- 能有Java互存互调；
###### 协程
- Kotlin中的协程区别于Go语言等其他协程，它是一个假协程，是语言层面用于简化多线程编程的线程框架；
- 解决了多线程循环切套的问题，以同时代码的格式，完成异步操作；
- 得益于挂起函数-suspend，状态极+回调
- 挂起函数只能在挂起函数或者协程作用域中进行调用；
- withContent()、referied+await（B依赖A的结果）、lifecycleCoroutine(Activity)、viewModelCoroutinr(ViewModel)
###### let、apply、run、with
- let 作用域默认以it为当前闭包的对象，开发者可根据上下文去自定义该闭包对象，返回值是作用域最后一行；
- apply 作用域以this为当前闭包对象，返回值是当前闭包对象；
- run 与 apply 相似，但是返回值是函数最后一行
###### by lazy 、lateinie
- lazy{}只能用于val类型上、lateinie只能用在var类型上；
- lazy{}是最后一行代码作为返回值，并且初始化后就不能更改，lateinie 可以进行多次初始化；

###### const、val

- 当一个变量被val修饰后当表当前变量初始化后就不能被修改了，类似于Java中的final关键字；
- const用于声明编译时常量的关键字，该关键字通常与val关键字搭配使用，用于顶层类声明中的属性；
  - 属性的类型必须是原生类型（Int，String）;

###### kotlin-synchronized

```kotlin
@Synchronized
fun useSynchronizedMethod(){

    synchronized(lock){

    }
}
```

 