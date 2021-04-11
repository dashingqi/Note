#### Linux的基础知识

##### 进程空间划分

- 一个进程空间分为 用户空间和内核空间，把进程内 用于与内核隔离起来

- 两者的区别

  - 用户空间的数据在进程间是不可共享的，归属于某一个进程的
  - 内核空间的数据时可共享，所有的进程共用一个内核空间。

- 用户空间与内核空间进行数据交互需要通过如下两个函数

  - copy_from_user:将用户空间的数据拷贝到内核空间。
  - copy_to_user:将内核空间的数据拷贝到用户空间。

- 示意图

  ![用户空间与内核空间进行数据交互.png](https://upload-images.jianshu.io/upload_images/4997216-5e0feecba1fa9fe2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 进程隔离与跨进程通信

###### 进程隔离

- 为了保证安全性和独立性，一个进程不能直接操作另外一个进程，Android是以Linux为内核，也满足这种机制，在Android中进程都是相互独立。

###### 跨进程通信

- 进程间需要通信，需要进行数据交互。

###### 跨进程通信的工作流程

- 发送方进程通过系统调用（copy_from_user）将数据拷贝到内核空间的缓冲区中。
- 内核服务程序唤醒了接受数据的进程，通过系统调用（copy_to_user）将数据发送到接受进程中。
- 这样就实现了进程间用户空间数据的交互。

###### 跨进程通信的缺点

- 需要进行两次的数据拷贝，效率比较低下。 用户空间 ---> 内核空间 ----> 用户空间
- 内核空间中接受数据的缓存需要接受方提供，但是接收方也不知道需要多大
  - 开辟尽量大的空间
  - 先调用API获取数据的大小，然后再去开辟空间
  - 第一种比较浪费空间，第二种比较耗时。
- 传统的跨进程通信需要拷贝2次数据，但是Binder机制只需要1次，主要是使用了内存映射。



#### Binder跨进程通信机制

##### 模型原理

- Binder是基于Client - Server模式

- 跨进程模型图

  ![Binder跨进程模型图.png](https://upload-images.jianshu.io/upload_images/4997216-b9b3a939382d7adb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### Binder跨进程通信的核心原理

***内存映射***

- 工作原理
  - Binder驱动创建一块接受缓存区在内核空间中。
  - 将接受进程的用户空间地址映射到Binder驱动创建的接受缓存区上，同时将内核缓冲区与Binder创建的接受缓存区进行映射，这样就实现了内核缓存区与接受进程用户空间的映射。
  - 这样到发送方通过系统调用，将数据发送内核缓冲区上的同时接受进程也收到数据了，这样就实现了跨进程通信。
- 优点
  - 仅仅进行一次数据拷贝，效率高；

###### 一次Bidner跨进程通信模型的详细步骤

- 注册服务
  - Server进程向Binder驱动发起注册
  - Binder驱动将这个注册请求转发给ServiceManager进程
  - ServiceManager进程间接管理Server进程。
- 获取服务
  - Client向Binder驱动发起获取服务的请求，
  - Binder驱动将该请求转发给ServiceManager进程
  - ServiceManager查找Client需要查找的Binder中注册的Server进程（具体的实现）
  - 通过Binder驱动将服务信息返回给Client。
- 使用服务
  - Binder为跨进程做准备，创建接受缓存区，实现内存映射
    - Binder驱动在内核空间创建一个接受缓存区
    - 实现映射：从ServiceManager中找到Server进程，实现内核缓存区和Service进程用户用户空间地址同时映射到Binder驱动创建的接受缓存区。
  - Client进程将请求参数发送到Server进程。
    - Client进程通过系统调用(copy_from_user())将数据发送到内核缓冲区中（当前线程被挂起）
    - Binder驱动通知Server进程
  - Server根据请求调用相应的方法
    - 收到Binder的通知后，从线程池中取出线程，调相应的方法
    - 将执行后的结果写入到自己的共享内存中
  - Server将目标方法的结果返回给Client进程
    - 将最终执行的结果写入到内核缓存区中
    - Binder通知Client进程，返回结果到了
    - Client通过系统调用（copy_to_user()）从内核缓冲区中拿到Server进程返回的数据。