#### synchronized

##### 作用域

- 实例方法
  - 修饰的实例方法，锁对象是当前类的实例。
  - 在不同线程中只有用同一个实例对象调用这个方法会产生排斥。
- 静态方法
  - 修饰的静态方法，锁是当前类的Class对象。
  - 在不同线程中，使用不同实例对象调用这个方法也会产生排斥。
  - Javap 查看编译后的字节码
    - 方法的flags属性会被标记为ACC_SYNCHRONIZED 标志。
    - 当虚拟机访问一个被ACC_SYNCHRONIZED标记的方法，会在方法的开头和结尾处加上monitorrnter和monitorexit指令
    - 这两个指令可以理解为一把锁，在锁中保存着两个比较重要的属性指针和计数器
      - 计数器：默认为0 当执行monitorenter指令时（说明这把锁被线程持有了），会加1 当执行monitorexit会减1
      - 指针指向持有这把锁的线程
- 代码块
  - 修饰代码块，可以指定锁对象。

#### ReentrantLock

- ReentrantLoc的使用与synchronized有点不同，加锁和解锁都需要手动写代码来完成
  - 使用lock来加锁
  - 使用unlock来解锁
    - 都是在finally代码块中调用unlock方法
    - 在程序发生异常的情况下，synchronized是能自动释放锁的
    - ReentrantLock不会自动释放锁，所以unlock方法最好写在finally代码块中。

###### 公平锁的实现

> 所谓公平锁就是通过同步队列来实现多个线程按照申请锁的顺序来获取锁。

- 实现如下

  ```java
  public class FairLockTest implements Runnable {
  
      private int num = 0;
      private static ReentrantLock fairLock = new ReentrantLock(true);
  
      @Override
      public void run() {
          while (num < 20) {
              fairLock.lock();
              try {
  
                  num++;
                  System.out.println(Thread.currentThread().getName() + " num = " + num);
              } finally {
                  fairLock.unlock();
              }
          }
  
      }
  
      public static void main(String[] args) {
          FairLockTest task = new FairLockTest();
          Thread t1 = new Thread(task);
          Thread t2 = new Thread(task);
          Thread t3 = new Thread(task);
          t1.start();
          t2.start();
          t3.start();
  
      }
  }
  ```

###### 读写锁（ReentrantReadWriteLock）

> 需要在读操作时获取读锁，写操作时获取写锁就可以
>
> 当写锁获取到时，后续的读写锁都会被阻塞，写锁释放之后，所有的操作继续执行

- 代码实操

  ```java
  public class ReadWriteLockTest {
      private static ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
      private static String number = "";
  
  
      public static void main(String[] args) {
          Thread t1 = new Thread(new Reader(), "读线程1");
          Thread t2 = new Thread(new Writer(), "写线程0");
          Thread t3 = new Thread(new Writer(), "写线程1");
          t1.start();
          t2.start();
          t3.start();
  
      }
  
      /**
       * 读操作
       */
      static class Reader implements Runnable {
          @Override
          public void run() {
              for (int i = 0; i < 20; i++) {
                  try {
                      lock.readLock().lock();
                      System.out.println(Thread.currentThread().getName() + "  正在读取  " + number);
                  } finally {
                      lock.readLock().unlock();
                  }
  
              }
  
          }
      }
  
      /**
       * 写操作
       */
      static class Writer implements Runnable {
          @Override
          public void run() {
              for (int i = 0; i < 20; i++) {
                  try {
                      lock.writeLock().lock();
                      System.out.println(Thread.currentThread().getName() + "    正在写入 ------>   " + i);
                      number = number.concat("" + i);
                  } finally {
                      lock.writeLock().unlock();
                  }
  
              }
  
          }
      }
  
  }
  
  ```

  

