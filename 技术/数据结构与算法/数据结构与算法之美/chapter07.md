### 写出正确的链表代码



###### 写出反转单链表

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



