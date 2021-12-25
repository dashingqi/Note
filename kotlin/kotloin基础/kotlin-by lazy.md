#### by lazy

##### lazy的延迟模式

###### mode = LazyThreadSafetyMode.SYNCHRONIZED

lazy默认的模式，可以省掉，该模式的意思是：如果有多个线程访问，只有一条线程可以去初始化lazy对象；

###### mode = LazyThreadSafetyMode.NONE

只能在单线程下使用，不能在多线程下使用，不会有锁的限制；

也就是不会有任何线程安全的保证以及相关的开销；

###### mode = LazyThreadSafetyMode.PUBLICATION

对于没有被初始化的lazy对象，可以被不同的线程调用；如果lazy对象初始化完成，其他的线程使用的是初始化完成的值；

