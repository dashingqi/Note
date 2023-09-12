#### synchronized修饰类的class对象、静态方法、普通方法、代码块

###### synchronized修饰类

```java
ClassB{
	public void method(){
		synchronized(ClassB.class){
      	
    }
	}
}
//作用范围是括号括起来的部分
//作用的对象是当前的类的所有对象
```

###### synchronized修饰静态方法

```java
public synchronized static methodA(){
	 
}
//作用范围是修饰的这个方法
//作用对象：静态方法是属于类的类的成员，synchronized是用来同步时的，它的同时监视器是当前类，也就是作用域当前类的宿友对象。
```

###### synchronized修饰的普通方法

```java
public synchronized methodB(){
	
}
//作用范围是：当前这个方法
//作用对象是：普通方法是属于实例成员的，作用对象是当前调用方法的实例。
```

###### synchronized修饰的代码块

```java
synchronized {
	
}
//作用范围就是当前代码块中
//作用对象是：调用这个代码块的对象。
```

#### synchronized关键字的了解

- 该关键字是解决了多线程访问资源的同步性问题，使用该关键字可以保证资源在任意时刻只能有一个线程来访问。

###### 按照版本优化来说

**Java6之前**

- sychronized是属于重量级的锁，效率低下，因为同步监视器是依赖于底层操作系统的Mutex Lock锁来实现的。Java的线程是映射到操作系统的线程上的。
- 在挂起或者唤醒以一个线程，都需要操作系统来帮忙完成，是比较耗时间的。

**Java6之后**

- Java从JVM层面对synchronized进行了优化。
- 在JDK6的时候，引入了大量的优化，比如自旋锁，适应性自旋锁，消除锁，锁粗化，偏向锁，轻量级锁等技术来减少锁操作的开销。

#### synchronized 关键字底层原理

- synchronized关键字底层原理属于JVM层面

###### synchronized同步语块情况

- 使用 Java自带的 javac -c -s -v -l   demo.class查看编译后的文件
  - synchronized同步语块的实现 使用的 monitorenter和monitorexit指令
  - 其中monitorenter指令指向同步代码块的开始位置，monitorexit指令指向了同步语块的结束位置。
  - 当之心monitorenter指令的时候，就获取到monitor（monitor对象位于每个java对象的对象头中）的持有权，当计数器为0 就可以获取成功，获取后将计数器加1。当执行了monitorexit后计数器就设为0。

###### synchronized修饰方法的情况下

- 使用 Java自带的 javac -c -s -v -l   demo.class查看编译后的文件
  - synchronzied修饰的方法使用的是ACC_SYNCHRONIZED标识，指明了该方法是一个同步方法。

##### JDK1.6之后做的优化

- JDK1.6对锁的实现引入了大量的优化，如自旋锁，适应性自旋锁，偏向锁，轻量级锁，锁消除。
- 锁主要存在四种状态：无锁状态，偏向锁状态，轻量级锁状态，重量级锁状态，它们会随着竞争而逐渐升级。锁是可以升级的不可以降级的。来提高锁和释放锁的状态。

#### synchronized与ReentrantLock的区别

###### 两者都是可重入锁

###### synchronized依赖于JVM 而 ReentrantLock依赖于API层面的



#### AtomicInteger原理

- AtomicInteger类主要使用CAS+volatile关键字+native方法来保证原子性的
- 避免synchronized的高开销。