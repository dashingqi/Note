> 如果系统存在多个耗时的任务需要去执行，并且每个耗时任务都会开启个新线程去执行耗时任务，这样会导致创建多个线程和销毁线程，从而影响到性能了。这时使用HandlerThread

## 原理

- HandlerThread是继承至Thread，内部有自己的Looper对象，可以进行loop循环。
- 首先需要调用 HanderThread的start()方法来创建Looper对象
- 使用HandlerThread的looper对象构造一个工作Handler，这样当用工作的Handler发出消息的时候，消息都会进入到子线程去执行
- 当拿到数据之后，需要从子线程中切换到Ui线程中（runOnUiThread/或者创建一个与主线程关联的Handler，调用post）
- HandlerThread的run()方法是一个无限循环，所以在OnDestroy()中需要调用HandlerThread的quitSafely来终止线程的执行。

## 优缺点

###### 优点

- 是异步不会堵塞的，提升效率。
- 避免创建多个线程与销毁多个，提升效率。

###### 缺点

- 不能同时同步执行多个任务，要等待进行处理，效率低。
- HandlerThread是一个串队列，背后只有一个线程。

