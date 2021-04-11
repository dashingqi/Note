#### 原理

当一个Activity执行完onDestroy()之后，将它放入WeakReference中，然后将这个WeakReference类型的Activity与ReferenceQueue关联。这时再从ReferenceQueue中查看是否有没有该对象，如果没有，执行GC，再次查看还是没有的话就判断发生内存泄漏的，

最后使用HAHA这个开源库去分析dump之后的heap内存。



当弱引用被回收后，就会被加入到我们关联它的ReferenceQueue中，我们只要观测这个ReferenceQueue就能知道引用是否被回收了。

###### 问题1

这里并没有使用System.gc()方法进行回收，因为system.gc()并不会每次都执行。而是从AOSP中拷贝一段GC回收的代码，从而相比System.gc()更能够保证垃圾回收的工作。 拷贝的GC代码，一定会触发GC吗？

ASOP的GC回收肯定要比普通的System.gc方法回收率要高，没有哪一份GC代码一定能保证触发GC,因此要做一些兜底方案或容错处理，leakcanary也并不能保证检测出所有的内存泄露

 IdleHandler，在主线程的消息队列为空的时候，也即 CPU 空闲的时候去执行耗时的 GC 检测逻辑无疑能减少界面卡顿，带来更好的用户体验。



#### LeakCanary1.0与2.0之间的区别

###### 2.0

- 采用了Kotlin语言编写
- Heap分析工具使用了Shark，替换之前的HaHa，官方说内存上减少了10倍
- 实现了自动注册 采用ContentProvider（AppWatchInstaller），在应用启动的时候会由ActivityThread创建并初始化。

- heap分析模块作为一个独立的模块。
- 分析hprof文件：开启了一个前台服务，使用Shark分析工具去分析

