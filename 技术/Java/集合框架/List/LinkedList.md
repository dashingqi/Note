## LikedList原理解析

#### 基本总结

- 是继承至AbstractSequentialList，实现了List、Deque、Cloneable、Serializable接口
- 底层的数据结构是一个双端链表
  - 前置节点、该节点、后置节点
  - 双端链表由node组成，第一个节点的前置节点为null，最后一个节点的后置节点为null
- 支持高效率的插入和删除的操作

#### LinkedList最核心最基本的方法

###### linkFirst()

- 往链表的头部插入节点

###### linkLast()

- 往链表的尾部插入节点

###### linkBefore()

- 往目标节点的前插入节点
