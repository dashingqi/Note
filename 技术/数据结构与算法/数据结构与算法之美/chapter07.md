### 写出正确的链表代码

#### 理解指针或引用的含义

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230207220725734.png" alt="image-20230207220725734" style="zoom:200%;" />

#### 警惕指针丢失和内存泄漏

<img src="https://static001.geekbang.org/resource/image/05/6e/05a4a3b57502968930d517c934347c6e.jpg?wh=1142*513"/>

```kotlin
p.next -> x
x.next -> p.next
```

上述做法是错误的，原因是因为p.next已经指向了x说明此时p.next存储了x的内存地址，此时将x.next执向了p.next也就是是x，岂不是乱套了嘛？

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/%E6%8C%87%E9%92%88%E4%B8%A2%E5%A4%B1.png" alt="指针丢失" style="zoom:200%;" />

正确的写法

```kotlin
x.next -> p.next
p.next -> x
```

#### 利用哨兵简化实现难度

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230207223927890.png" alt="image-20230207223927890" style="zoom:200%;" />

#### 重点留意边界条件处理

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230207224607613.png" alt="image-20230207224607613" style="zoom:200%;" />

#### 举例画图，辅助思考

#### 多写多练，没有捷径

- 单链表转换

  ```java
  /**
    * 反转链表
    *
    * @param head
    * @return 反转后的链表
    */
  public ListNode reverseList(ListNode head) {
    ListNode cur = null;
    ListNode pre = head;
    while (pre != null) {
      ListNode tempNode = pre.next;
      pre.next = cur;
      cur = pre;
      pre = tempNode;
    }
    return cur;
  }
  ```

  <img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/202302062028066.png" alt="反转列表" style="zoom:200%;" />

- 链表中环的检测

  <img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/%E7%8E%AF%E5%BD%A2%E9%93%BE%E8%A1%A8.png" alt="环形链表" style="zoom:200%;" />

  ```kotlin
   /**
     * 环形链表 通过快慢指针
     * @param head ListNode?
     * @return Boolean
     */
  internal fun hasCycle(head: ListNode?): Boolean {
      head ?: return false
      head.next ?: return false
      var fast = head
      var slow = head
      while (fast?.next != null) {
          fast = fast.next?.next
          slow = slow?.next
          if (fast == slow) {
              return true
          }
      }
      return false
  }
  ```

- 两个有序的链表合并；

- 删除链表倒数第n个结点；

- 求链表的中间结点；

  ```kotlin
  /**
   * 求出链表的中间结点
   * @param head ListNode? 头结点
   * @return ListNode? 中间结点
   */
  fun middleNode(head: ListNode?): ListNode? {
      var fastNode = head
      var slowNode = head
      while (fastNode?.next != null) {
          fastNode = fastNode.next?.next
          slowNode = slowNode?.next
      }
  
      return slowNode
  }
  ```

  
