## 数组：为什么很多编程语言中数组都从0开始编号

#### 数组

数组是一种线性表数据结构。它用一组连续的内存空间，来存储一组具有相同类型的数据；

###### 线性表

线性表是数据排成像一条线一样的结构，每个线性表上的数据最多只有前和后两个方向；

数据、链表、栈、队列都是典型的线性表的数据结构

<img src="https://static001.geekbang.org/resource/image/b6/77/b6b71ec46935130dff5c4b62cf273477.jpg?wh=1142*833" alt="image-20230130223526117" style="zoom:200%;" />

###### 非线性表

典型：二叉树、图

<img src="https://static001.geekbang.org/resource/image/6e/69/6ebf42641b5f98f912d36f6bf86f6569.jpg?wh=1142*727"/>

###### 【连续的内存空间】和【相同类型的数据】

杀手锏特性：【随机访问】

这两个特性让数组的操作变得非常低效，比如想要在数组中删除、插入一个数据，为了保证连续行需要做大量的工作；

##### 数组如何实现下标随机访问数组元素

计算机会给每个内存单元分配一个内存地址，计算机通过地址来访问内存中的数据；

寻址公式

```java
a[i]_address = base_address + i * data_type_size

// data_type_size 为数组中存储数据的字节大小
// base_address 为内存首地址
// i 为当前角标
```

数组支持随机访问，根据下标随机访问的时间复杂度为O(1)

###### 低效的【插入】和【删除】

数组为了保持内存数据的连续性，会导致插入、删除这两个操作比较低效；

**插入**

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230130230726802.png" alt="image-20230130230726802" style="zoom:200%;" />

**删除**

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230130231004740.png" alt="image-20230130231004740" style="zoom:200%;" />

###### 数组更合适

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230130231652552.png" alt="image-20230130231652552" style="zoom:200%;" />

###### 为什么数组从零开始

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230130232138806.png" alt="image-20230130232138806" style="zoom:200%;" />

**历史原因**

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230130232236404.png" alt="image-20230130232236404" style="zoom:200%;" />
