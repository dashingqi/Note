#### 链表题目

###### 链表反转

```java
/**
 * 链表反转
 * 给你单链表的头节点 head ，请你反转链表，并返回反转后的链表。
 * 输入：head = [1,2,3,4,5]
 * 输出：[5,4,3,2,1]
 */
public Node reverseListNode(Node head) {
    if (head == null) {
        return null;
    }
    Node preNode = null;
    Node curNode = head;
    while (curNode.next != null) {
        Node nextNode = curNode.next;
        curNode.next = preNode;
        preNode = curNode;
        curNode = nextNode;
    }
    return preNode;
}
```

###### 链表中有环

```java
/**
 * 给你一个链表的头节点 head ，判断链表中是否有环。
 * <p>
 * 如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，
 * 评测系统内部使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。注意：pos 不作为参数进行传递 。仅仅是为了标识链表的实际情况。
 * <p>
 * 如果链表中存在环 ，则返回 true 。 否则，返回 false 。
 */
public boolean hasCycle(Node head) {
    if (head == null) return false;
    Node slow = head;
    Node fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            return true;
        }
    }
    return true;
}
```

###### 合并两个有序链表

```java
/**
 * 将两个升序链表合并为一个新的 升序 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。
 * 输入：l1 = [1,2,4], l2 = [1,3,4]
 * 输出：[1,1,2,3,4,4]
 */
public Node mergeTwoListsV2(Node list1, Node list2) {
    if (list1 == null) {
        return list2;
    }
    if (list2 == null) {
        return list1;
    }

    if (list1.data < list2.data) {
        list1.next = mergeTwoListsV2(list1.next, list2);
        return list1;
    } else {
        list2.next = mergeTwoListsV2(list1, list2.next);
        return list2;
    }
}
```

###### LRU算法实现

```java
static class DQLruCacheV5 {

        private final int capacity;
        private final Node headNode;
        private final Node tailNode;
        private final Map<Integer, Node> cache;

        public DQLruCacheV5(int capacity) {
            this.capacity = capacity;
            cache = new ConcurrentHashMap<>(capacity);
            headNode = new Node(0, 0);
            tailNode = new Node(0, 0);
            headNode.next = tailNode;
            tailNode.pre = headNode;
        }

        public void put(int key, int value) {
            if (cache.containsKey(key)) {
                Node node = cache.get(key);
                node.value = value;
                moveToHead(node);
            } else {
                Node node = new Node(key, value);
                addToHead(node);
                if (cache.size() > capacity) {
                    removeTail();
                    cache.remove(key);
                }
            }
        }

        public int get(int key) {
            if (cache.containsKey(key)) {
                Node node = cache.get(key);
                moveToHead(node);
                return node.value;
            }

            return -1;
        }

        private void moveToHead(Node node) {
            // 移除节点
            removeNode(node);
            // 添加到头部
            addToHead(node);
        }

        private void removeNode(Node node) {
            node.pre.next = node.next;
            node.next.pre = node.pre;
        }

        private void addToHead(Node node) {
            Node tempNode = headNode.next;
            headNode.next = node;
            node.pre = headNode;
            tempNode.pre = node;
            node.next = tempNode;
        }

        private void removeTail() {
            Node node = tailNode.pre;
            removeNode(node);
        }
    }
```

