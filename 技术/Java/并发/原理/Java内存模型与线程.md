## Java内存模型与线程

#### 线程中的工作内存

- 每一个线程中里面，都会有一块内部的工作内存，这块工作内存保存了主内存共享数据的拷贝副本。
- 其实Java线程中并不存在所谓的工作内存，这个工作内存是对CPU寄存器和高速缓存的抽象描述。
- 线程是CPU的最小调度单位，线程的切换是要涉及到CPU的

#### CPU

- 线程中的字节码都是在CPU中执行的，我们Java程序中的数据都是放在主内存中，CPU是要操作主内存的

###### CPU可以直接操作主内存

###### 技术的发展，CPU通过高速缓存将数据刷到主内存中

#### 缓存一致性问题（可见性问题）

- CPU刷新数据到主存中不及时导致的

#### 指令重排（顺序性问题）

- 为了执行效率，进行了CPU指令优化，也就是打乱了代码的执行顺序

#### 内存模型（是一套规范，重要的是规范中的happens-befor原则）

- 内存模式解决了可见性和顺序性的问题，也即是CPU多级缓存、指令重排造成的内存访问问题。

#### happens-befor原则（先行发生原则）

- 同一个线程中，先执行的结果对后续代码来说是可见的
- 使用volative关键字定义的公共变量，线程A对其的写操作，对于线程B的读操作来说是可见的。
- 线程A中执行了线程B的 join方法，此时线程A被挂起那么在线程B中操作的公共变量对于线程A的后续来说是可见的。
- 使用synchronized关键字修饰的同步代码块中，线程A对代码块中的变量操作对于线程B来说是可见的。
- 具有传递性的，如果操作A happens-before 操作B，操作B happens-befor 操作C ，那么操作A一定 happens-before 操作C
- 线程的启动：线程的start方法是早于run方法执行的，如果在线程A执行过程中去开启线程B（通过start方法）在启动线程B之前，线程A操作的共享变量对于线程B来说是可见的。
- 锁定的规则：在单线程或者多线程环境中，如果一个锁是处于锁状态的，那么必须执行unlock操作，之后才能执行lock操作。
- 对象终结规则：一个对象初始化的完成发生在它的finalize()方法之前。

