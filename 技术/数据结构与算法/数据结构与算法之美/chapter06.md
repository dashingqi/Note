### 如何实现LRU缓存淘汰算法

##### 链表结构

- 数组需要一块连续的内存空间来存储
- 链表并不需要一块连续的内存空间，它通过指针将一组零散的内存块串联起来使用；

<img src="https://static001.geekbang.org/resource/image/d5/cd/d5d5bee4be28326ba3c28373808a62cd.jpg?wh=1142*699"/>



###### 单链表

<img src="https://static001.geekbang.org/resource/image/b9/eb/b93e7ade9bb927baad1348d9a806ddeb.jpg?wh=1142*399"/>

**单链表的插入和删除的时间复杂度是O(1)**

<img src="https://static001.geekbang.org/resource/image/45/17/452e943788bdeea462d364389bd08a17.jpg?wh=1142*650"/>

**单链表随机访问的时间复杂度是O(n)**

因为链表不像数组那样支持【随机访问】，链表如果想要找到第几个节点时时需要从头开始一个一个遍历直到找到对应的结点；

链表访问结点时【顺序访问】

###### 循环链表

<img src="https://static001.geekbang.org/resource/image/86/55/86cb7dc331ea958b0a108b911f38d155.jpg?wh=1142*399"/>

**优点**

循环链表的优点是从链尾到链头比较方便；当要处理的数据具有环型结构特点时，就特别适合采用循环链表；

###### 双向链表

<img src="https://static001.geekbang.org/resource/image/cb/0b/cbc8ab20276e2f9312030c313a9ef70b.jpg?wh=1142*399"/>

双向链表需要额外的两个空间来存储【后继结点】和【前驱结点】

**双向链表的删除操作**

1. 删除结点中值等于某个给定值的结点
   1. 顺序查找给定的值，时间复杂度是O(n),删除操作的时间复杂度是O(1)；总的时间复杂度是O(n)
2. 删除给定指针指向的结点
   1. 已经找到要删除的结点，直接删除就行，时间复杂度是O(1),由于删除的结点知道其的前驱结点，但是换成单链表的话就得从头结点开始遍历，直到找到 p->next=q;所以这种情况下单链表删除时间复杂度是O(n);

###### 双向循环链表

<img src="https://static001.geekbang.org/resource/image/d1/91/d1665043b283ecdf79b157cfc9e5ed91.jpg?wh=1142*500"/>
