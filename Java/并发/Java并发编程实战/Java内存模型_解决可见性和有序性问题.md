## 	Java内存模型

- 我们知道导致可见性的问题时缓存，导致有序性问题是编译优化。
- 解决可见性、有序性最直接可能想到的是禁用缓存和编译优化，虽然问题解决了，但是程序的性能就不行了。
- 合理的方案就是按需禁用缓存以及编译优化。

#### 内存模型

- Java内存模型规范了JVM如何提供按需禁用缓存和编译优化的方法
- 具体来说，这些方法包括 volatile、synchronized、final三个关键字，以及六项Happens-Before规则。

#### volatile

- volatile在古老的C语言中也有，它最原始的意义就是禁用CPU缓存

- 声明一个volatile变量

  ```java
  volatile int a = 0;
  // 该语义对于这个变量的读写不准使用CPU缓存，必须从内存中读写。
  ```

- 如下代码

  ```Java
  class VolatileExample {
    int x = 0;
    volatile boolean v = false;
    public void writer() {
      x = 42;
      v = true;
    }
    public void reader() {
      if (v == true) {
        // 这里x会是多少呢？
      }
    }
  }
  ```

  - 当线程A执行writer方法，按照volatile的作用，此时内存中的v为true，那么线程B执行reader方法时，此时的x是多少呢？
    - 在低于1.5版本中是 0或者是42（CPU的缓存导致的可见性问题）
    - 但是在1.5以上的时候就是42
  - 在JDk1.5的时候对volatile语义进行了增强。实则就是下面我们要说的Happens-Before规则。

## Happens-Before规则

- 真正要表达的是：前面一个操作的结果对后续操作是可见的。

#### 程序的顺序性规则

- 前面的操作对于后续的任意操作都是可见的，

#### volatile变量规则

- 对于一个volatile变量的写操作，于后续对于这个volatile变量的读操作是可见的。

#### 传递性

- 如果 a 对于 b可见 ，b对于c可见 那么 a对于c可见

###### 实例分析

```java
class VolatileExample {
  int x = 0;
  volatile boolean v = false;
  public void writer() {
    x = 42;
    v = true;
  }
  public void reader() {
    if (v == true) {
      // 这里x会是多少呢？
    }
  }
}
```

- x =42 的操作 对于变量V的写操作是可见 （规则一）
- v = true 的写操作对于 v ==  true的读操作的是可见（规则2）
- 再根据传递性规则 那么 在线程B中 变量x = 42 对于 v= true的读操作是可见的，（规则3）
- 如上就是JDK1.5中对于volatile语意的增强。

#### 管程中锁的规则

###### 管程

- 是一种通用的同步原语，在Java中指的是synchronized，synchronized是Java里对管程的具体实现

- 管程中的锁对于我们来说是隐式的

  ```java
  synchronized (this) { //此处自动加锁
    // x是共享变量,初始值=10
    if (this.x < 12) {
      this.x = 12; 
    }  
  } //此处自动解锁
  ```

  - 假设变量x = 0 ，那么线程A进入到同步代码块中，此时过后应该x = 12，执行完毕后释放掉锁资源，线程B进入此时对于线程B来说X = 1

#### 线程start()规则

- 指的是主线程A启动子线程B后，子线程B能够看到主线程在启动子线程B前的操作

- 代码实例

  ```java
  Thread B = new Thread(()->{
    // 主线程调用B.start()之前
    // 所有对共享变量的修改，此处皆可见
    // 此例中，var==77
  });
  // 此处对共享变量var修改
  var = 77;
  // 主线程启动子线程
  B.start();
  ```

####  线程join规则

- 子线程在主线程中调用了 join方法，主线程等到子线程执行完毕才能继续往下执行，在子线程完成后主线程能够看到子线程的操作。也就是在线程操作的共享变量 在之后的主线程中也是可见的。

- 代码实例

  ```java
  Thread B = new Thread(()->{
    // 此处对共享变量var修改
    var = 66;
  });
  // 例如此处对共享变量修改，
  // 则这个修改结果对线程B可见
  // 主线程启动子线程
  B.start();
  B.join()
  // 子线程所有对共享变量的修改
  // 在主线程调用B.join()之后皆可见
  // 此例中，var==66
  ```

## final关键字

- final修饰变量是，就是告诉编译器：这个变量是不可变的，可以使劲的优化。
- 在JDk1.5以后Java内存模型对final类型变量的重拍进行了约束，只要我们提供的构造函数没有逸出，就不会出问题。

