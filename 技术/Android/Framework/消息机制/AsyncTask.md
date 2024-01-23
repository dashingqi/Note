> AsyncTask是一个轻量级的异步执行类，可以在线程池中执行任务，然乎把任务的进度和结果发送到UI线程中，去更新UI。

## 问题

###### 生命周期

- AsyncTask的生命周期不会随着Activity销毁而销毁，它会一直执行doInBackground()方法到完，然后如果cancel(boolean)被调用，会执行onCanceled();否则去执行onPostExecute().
- 在Activity销毁之前如果我们没有取消掉AsyncTask，可能会导致应用Crash的，因为它要更新的View不存在了。

###### 内存泄漏

- 如果AsyncTask被声明为一个非静态的内部类，默认会持有Activity的引用，在Activity销毁的时候，AsyncTask后台线程还在执行，那么就会导致Activity无法被回收，引发内存泄漏

###### 并行还是串行

- 在Android1.6之前是串行的，Android1.6到Android3.0这之前是并发执行，考虑到并发执行带来的错误，Android3.0之后改成了串行。

## 原理

- AsyncTask中有两个线程池（SerialExecutor、THREAD_POOL_EXECUTOR）和一个Handler（InternalHandler）,其中SerialExecutor是用于任务的排队，线程池THREAD_POOL_EXECUTOR是用于任务的执行，而InterlHandler用于将环境从工作线程切换到UI线程上。
- InternalHandler的对象引用snvim-treesitter[comment]: Error during download, please verify your internet connectionHandler是一个静态对象，为了让工作环境从工作线程切换到主线程中，sHandler必须要在主线程中创建，由于是静态的所以在类加载的就已经进行初始化了，这就变相要求AsyncTask需要在主线程中创建！
