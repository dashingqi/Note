## GC回收机制

- 垃圾回收（Garbage Collection 简写为 GC）
- 我们在运行时期创建的对象，处于动态，GC主要关注这部分对象的分配和回收

#### 什么是垃圾

> 所谓的垃圾就是内存中不再使用的对象

##### 怎么知道哪些对象是垃圾呢？

###### 可达性分析

- JVM中把内存对象之间的引用关系作为一张表，通过一组GC Root的对象为起点

- 把GC Root作为起点，开始向下搜索，搜索走过的路径称为引用链。

- 最后通过引用链来判断对象是否可达来决定哪些对象是垃圾，哪些不用回收

- 如图

  ![可达性分析.png](https://upload-images.jianshu.io/upload_images/4997216-f20b4a0e6f7f51a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 总结说
  - 对象A/B/C/D 这条引用链和 E/F这条引用链 都是直接或者间接与GC Root存在链接，所以这些对象不是垃圾对象
  - 而 H和I被G引用，但是G没有与GC Root存在引用关系，所以G/H/I没有达到可达，就是垃圾对象

#### GC Root对象

###### Java中作为GC Root对象

- Java虚拟机栈（局部变量表）中引用的对象
- 方法区中静态引用指向的对象
- 仍处于存活状态的线程对象
- Native方法中JNI引用的对象

###### 什么时候回收

- 在堆内存中分配内存时，由于剩余可用内存不足导致对象的分配失败，此时会触发一次GC
- System.gc();调用次方法可以用来请求一次GC操作。

#### 垃圾回收算法

##### 标记-清除算法

##### 标记-整理算法

##### 复制算法

##### 分代回收（只是一种策略，不同的年代区使用不同的回收算法）

- 所谓的分代回收就是JVM根据对象的存活的时间的不同，把堆内存分为 新生代和老年代。

  - 在HotSpot中除了新生代和老年代还有永久代。

- 新生代细分可以分为3部分

  - Eden
  - Survivor0（S0）
  - Survivor1（S1）
  - 这三部分按照8:1:1的比例来划分新生代
  - 绝大多数刚刚被创建的对象会存放在Eden区

- 堆内存的划分

  - 如图

    ![虚拟机中堆内存的划分.png](https://upload-images.jianshu.io/upload_images/4997216-acc91603d2e545cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 新生代

- 一般我们新创建的对象是放在新生代的Eden区的，当Eden区域第一次满的时候会进行一次垃圾回收的操作，将Eden区的垃圾对象进行回收，并且将存活的对象复制到S0区域。
- 当下一次Eden区域满的时候，此时会将Eden区域和S0区域的垃圾对象进行回收，同时把存乎的对象复制到S1区域，此时S0位空了。
- 如此反复在S0和S1区域之间切换复制几次（默认是15次），如果还有存活的对象，说明这些对象的生命力比较顽强呀，就将这些对象转移到老年代中。
- 新生代中每一次GC，都会需要进行对象的复制操作，所以新生代采用的是复制算法。

###### 老年代

- 一个对象在新生代存活的时间长的时候，就会将其放入到老年代中
- 一般老年代的内存大小要比新生代的要大
- 如果一个对象在分配内存的时候，此时新生代内存不足的话，会直接分配到老年代中。
- 老年代中的对象生命周期都是比较长的，复制的频率不太高，所以采用标记-整理的算法。

#### GC Log分析

##### 新生代和老年代打印的GC日志是有区别的

- 新生代 GC：这一区域的GC叫做Minor GC。

- 老年代GC：这一区域的GC叫做Major GC活着Full GC。

  - 当出现了一个Major GC，往往会出现至少一次Minor GC

- Java命令参数

  |       命令参数       |                    功能描述                     |
  | :------------------: | :---------------------------------------------: |
  |     -verbose:gc      |                显示GC的操作内容                 |
  | -Xms20M<br />-Xmx20M |  初始化堆大小为20M<br />设置堆最大分配内存20M   |
  |       -Xmn10M        |            设置新生代的内存大小为10M            |
  | -XX:+PrintGCDetails  |               打印GC的详细log日志               |
  | -XX:SurvivorRatio=8  | 新生代中Eden区域与Survivor区域的大小比值为8:1:1 |

- 在IDEA中 通过配置如上参数，运行如下代码

  - 代码如下

    ```java
    public class MinorGCTest {
    
        private static final int _1MB = 1024 * 1024;
    
        public static void main(String[] args) {
            testAllocation();
    
        }
    
        public static void testAllocation() {
            byte[] a1, a2, a3, a4;
            a1 = new byte[2 * _1MB];
            a2 = new byte[2 * _1MB];
            a3 = new byte[2 * _1MB];
            a4 = new byte[1 * _1MB];
        }
    }
    
    
    ```

    

  - 打印如下堆内存日志

    ```java
    Heap
     PSYoungGen      total 9216K, used 1188K [0x00000007bf600000, 0x00000007c0000000, 0x00000007c0000000)
      eden space 8192K, 14% used [0x00000007bf600000,0x00000007bf729140,0x00000007bfe00000)
      from space 1024K, 0% used [0x00000007bfe00000,0x00000007bfe00000,0x00000007bff00000)
      to   space 1024K, 0% used [0x00000007bff00000,0x00000007bff00000,0x00000007c0000000)
     ParOldGen       total 10240K, used 6527K [0x00000007bec00000, 0x00000007bf600000, 0x00000007bf600000)
      object space 10240K, 63% used [0x00000007bec00000,0x00000007bf25fcd8,0x00000007bf600000)
     Metaspace       used 2970K, capacity 4496K, committed 4864K, reserved 1056768K
      class space    used 328K, capacity 388K, committed 512K, reserved 1048576K
    ```

  - 字段解释

    |   字段    |       含义       |
    | :-------: | :--------------: |
    | PSYongGen |      新生代      |
    |   eden    | 新生代中的eden区 |
    |   from    | 新生代中的S0区域 |
    |    to     | 新生代中的S1区域 |
    | ParOldGen |      老年代      |

#### Java中的四大引用

#### 引用描述

| 引用类型 |                          GC回收机制                          |                          实例                           |
| :------: | :----------------------------------------------------------: | :-----------------------------------------------------: |
|  强引用  |             对象具有强引用，垃圾回收器不会回收它             |              Obejct object = new Object()               |
|  软引用  |           在内存实在不足的时候，会对软引用进行回收           | SoftReference<Object> softObject = new SoftReference(); |
|  弱引用  | 在进行一次GC操作的时候，如果垃圾回收器扫描到了弱引用，就会将其回收 | WeakReference<Object> weakObject = new WeakReference(); |
|  虚引用  |                                                              |                         是个谜                          |

