#### LeakCanary

### 官方文档

> https://square.github.io/leakcanary/fundamentals-how-leakcanary-works/

### 检测泄露原理 （Weakreference）

###### 如何检测内存泄漏

- WeakReference --> ObjectWatcher || ReferenceQueue
- 弱引用所引用的对象被GC，则该弱引用对象会被放入给定的ReferenceQueue中
- ReferenceQueue会记录被GC的WeakReference持有对象；

###### 堆转储

- 该过程是把当前的内存信息存储到.hprof文件中；
- 同时该过程会冻结应用程序一会；

###### .hprof文件分析

- 使用Shark来分析.hprof文件；
- 在文件中找到保留的对象；

##### 泄漏分类

- 应用程序泄漏
- 三方库泄漏

### 如何修复内存泄漏

##### 步骤

1. 找到内存泄露的痕迹
2. 缩小内存泄漏的范围
3. 找到具体内存泄漏的点
4. fix&修复内存泄漏

##### LeakCanary作用

- 包含上述的1、2步的操作

##### 图标的作用

- 以 `├─` 开头的行表示 Java 对象（类、对象数组或实例）；

- 以 `│ ↓` 开头的行表示对下一个 Java 对象的引用线；
- 以 `╰→` 开头的行表示泄漏对象。
- LeakCanary 使用 ~~~ 下划线突出显示所有怀疑导致此泄漏的引用。

