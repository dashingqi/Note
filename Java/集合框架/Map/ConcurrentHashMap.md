## ConcurrentHashMap

> 相比较于HashTable和Collections.synchronizedMap
>
> ConcurrentHashMap在读写的性能上都有所提升
>
> ConcurrentHashMap使用的是分段锁技术

#### Segment

- Segment本身相当于一个HashMap对象。
- 同时Segment包含一个HashEntry数组，数组中每一个HashEntry是一个键值对

- 在ConcurrentHashMap中，这些Segment保存在Segment数组中

#### 支持

- 支持不同Segment的put操作
- 支持Segment的一读一写
- 同一个Segment的并发写入需要加锁，因此另一个将会被阻塞
- 也就是ConcurrentHashMap中的Segment各自持有一把锁。
  - 保证线程安全时降低了锁的粒度，让并发操作效率更高。

#### Get方法

- 为输入的key做Hash运算，得到hash值
- 通过hash值，定位到Segment对象
- 再次通过hash值，定位为Segment中数组的位置

#### Put方法

- 为输入的key做Hash运算，得到hash值
- 用得到的hash值，找到对应的Segment对象
- 获取到可重入锁
- 再次通过hash值，找HashEntry数组中的位置
- 插入或者覆盖当前的HashEntry对象。
- 释放锁