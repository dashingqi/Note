#### LinkedBlockingQueue

> 它是实现`BlockingQueue`接口的线程安全的队列,使用的是锁分离实现，支持并发`读写`，但不支持`读读`和`写写`；

###### 底层数据结构

- Node

```kotlin
static class Node<E> {
        E item;
        Node<E> next;
        Node(E x) { item = x; }
    }
```

```java
		/** 容量*/
    private final int capacity;
		
    /** 链表数*/
    private final AtomicInteger count = new AtomicInteger();

 		/** 头节点*/
    transient Node<E> head;

   	/**尾节点*/
    private transient Node<E> last;

  	/** 读锁*/
    private final ReentrantLock takeLock = new ReentrantLock();

    /** 队列为空时，存储出队操作线程，出队等待队列*/
    private final Condition notEmpty = takeLock.newCondition();

    /**写锁 */
    private final ReentrantLock putLock = new ReentrantLock();

    /** 写操作，存储入队操作线程，入队等待队列*/
    private final Condition notFull = putLock.newCondition();
```

##### 入队

###### put 

- 调用方法上述方面将数据入队时，如果当前队列已经满的情况，会将当前线程存放到等待队列中（notFull）；
- 如果当前队列未满时，会调用`enqueue`方法将数据存放到内部链表中；
- 当入队成功后，首先判断当前队列是否满了，未满的情况下，需要调用notEmpty.single()来唤醒此时等待出队队列；并且

###### offer(v) & offer(v,t,u)

- 当队列已满时，此时调用offer(v)方法会直接返回false；
- 当队列已满时，如果调用 offer(v,t,u)会出发超时等待策略，允许线程等待设定时间，在设定的时间内数据还是没有入队那么就会直接返回 false；
- 上述如果添加成功就会返回 true；

##### 出队

###### take

- 当队列为空时，会将当前线程放置到 notEmpty 等待队列中；
- 如果当前队列不为空时，尝试获取到锁，调用dequeue方法，取出队列中的元素；
- 当出队成功后，在判断队列中是否为空，不为空，会唤醒notEmpty等待队列,来进行出队；
- 最后判断队列是否为满，不满的情况会唤醒notFull等待队列中的线程，来进行入队；

###### poll & poll(t,unit)

- 当队列为空时，调用该方法会直接返回 null；否则首先会尝试获取到锁，然后获取到数据；
- 当队列为空时，同时它支持超时等待策略，也就是设置的有效时间内如果获取到数据就返回否则就返回 null；

###### peek()

- 支持获取队列中第一个元素，获取到队列中的第一个元素，并不会从队列中移除；

###### ------------------------------------------------------------------------------------------------

###### 读写

- 读写锁是分离的（内部各自存在一把读锁与写锁，可重入锁），读是操作队尾，写是操作队首；互不影响；

###### 容量限制

- 它是一个可设置容量大小的队列，默认容量大小为 Integer.MAX_VALUE;可以理解为是一个有界或者无界的队列；有界使用者可以设置固定容量，无界可以理解为默认的容量`Integer.MAX_VALUE`
- 容量过大，当生产者速度快于消费者速度时，会产生内存拥挤，造成内存过大；

###### 链表存储

- 内部默认使用的是链表进行存储，所以在删除以及插入上有一定的优势；

###### 阻塞操作

- `LinkedBlockingQueue` 支持阻塞的插入和删除操作。当队列已满时，插入操作将被阻塞，直到队列有足够的空间。同样，当队列为空时，删除操作将被阻塞，直到队列中有元素可供删除。

###### 使用场景

- 任务调度与执行：Java 提供的线程池中有用到默认构造参数的LinkedBlockingQueue
- 生产者-消费者：`LinkedBlockingQueue`可作为生产者消费者场景的数据通道，实现数据交换同时具备线程安全；