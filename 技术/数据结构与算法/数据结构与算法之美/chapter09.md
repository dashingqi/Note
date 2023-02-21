### 队列：队列在线程池等有限资源池中的应用

#### 如何理解【队列】

先进者先出，这就是典型的队列

##### 队列的操作

入队：放一个数据到队列尾部；

出队： 从队列头部去一个元素；

<img src="https://static001.geekbang.org/resource/image/9e/3e/9eca53f9b557b1213c5d94b94e9dce3e.jpg?wh=1142*800"/>

#### 顺序队列和链式队列

用数组实现的队列叫做顺序队列

```java
public class ArrayQueue {

    private String[] items;
    private int n;
    // 对头下标
    private int head = 0;
    // 对尾下标
    private int tail = 0;

    public ArrayQueue(int capacity) {
        items = new String[capacity];
        n = capacity;
    }

    /**
     * 入队
     *
     * @return 是否成功
     */
    public boolean enqueue(String item) {
        // 如果队尾下标==n说明队列已经满了
        if (tail == n) {
            if (head == 0) {
                return false;
            }
            for (int i = head; i < tail; ++i) {
                items[i - head] = items[i];
            }

            // 更新tail和head
            tail -= head;
            head = 0;
        }
        items[tail] = item;
        ++tail;
        return true;
    }

    /**
     * 出队
     *
     * @return 元素
     */
    @Nullable
    public String dequeue() {
        // 说明此时队列已经空了
        if (head == tail) return null;
        String item = items[head];
        ++head;
        return item;
    }
}
```

<img src="https://static001.geekbang.org/resource/image/09/c7/094ba7722eeec46ead58b40c097353c7.jpg?wh=1142*639"/>

用链表实现的队列叫做链式队列

