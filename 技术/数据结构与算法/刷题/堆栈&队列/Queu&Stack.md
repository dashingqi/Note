

#### 堆栈刷题

###### 二叉树的深度优先遍历

```java
public static void deepOrderTraverseV1(TreeNode node) {
        if (node == null) {
            return;
        }

        Stack<TreeNode> stack = new Stack<>();
        stack.push(node);
        while (!stack.isEmpty()) {
            TreeNode treeNode = stack.pop();
            if (treeNode.rightChild != null) {
                stack.push(treeNode.rightChild);
            }
            if (treeNode.leftChild != null) {
                stack.push(treeNode.leftChild);
            }
            System.out.println(treeNode.data);
        }
    }
```

###### 使用两个栈实现队列

```java
/**
 * 使用两个栈实现队列
 */
static class DQQueue {

    /**
     * 栈 A
     */
    Stack<Integer> stackA = new Stack<Integer>();
    /**
     * 栈 B
     */
    Stack<Integer> stackB = new Stack<Integer>();

    public DQQueue() {

    }

    public void appendTail(int value) {
        stackA.push(value);
    }

    public int deleteHead() {
        if (stackB.isEmpty()) {
            if (stackA != null && !stackA.isEmpty()) {
                a2B();
            } else {
                return -1;
            }
        }
        return stackB.pop();
    }

    private void a2B() {
        while (!stackA.isEmpty()) {
            stackB.push(stackA.pop());
        }
    }
}
```



#### 队列刷题

###### 二叉树的层序遍历

```java
public static void levelOrderTraverseV2(TreeNode treeNode) {
    if (treeNode == null) {
        return;
    }
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(treeNode);
    while (!queue.isEmpty()) {
        TreeNode tempNode = queue.poll();
        System.out.println(treeNode.data);
        if (tempNode.leftChild != null) {
            queue.offer(tempNode.leftChild);
        }

        if (tempNode.rightChild != null) {
            queue.offer(tempNode.rightChild);
        }
    }
}
```

###### 使用一个队列实现栈

```java
static class DQStackV1 {
    Queue<Integer> mQueue;

    public DQStackV1() {
        mQueue = new LinkedList<>();
    }

    public void push(int x) {
      // 先取出当前队列的数量 这个很关键
      int n = mQueue.size();
      mQueue.offer(x);
      // 在做队列中item的轮换
      for (int i = 0; i < n; i++) {
          mQueue.offer(mQueue.poll());
      }
    }

    public int pop() {
        return mQueue.poll();
    }

    public int top() {
        return mQueue.peek();
    }

    public boolean empty() {
        return mQueue.isEmpty();
    }
}
```

###### 使用两个队列实现栈

```java
static class DQStack {
    Queue<Integer> queue1;
    Queue<Integer> queue2;

    public DQStack() {
        queue1 = new LinkedList<>();
        queue2 = new LinkedList<>();
    }

    public void push(int x) {
        queue2.offer(x);
        // 先清空 queue1，将值插入到queue2尾部
        while (!queue1.isEmpty()) {
            queue2.offer(queue1.poll());
        }
				// 进行交换
        Queue<Integer> tempQueue = queue1;
        queue1 = queue2;
        queue2 = tempQueue;
    }

    public int pop() {
        return queue1.poll();
    }

    public int top() {
        return queue1.peek();
    }

    public boolean empty() {
        return queue1.isEmpty();
    }
}
```

###### 有效括号

```java
public boolean isValidBracket(String s) {
    if (s == null && s.isEmpty()) {
        return false;
    }

    HashMap<Character, Character> maps = new HashMap<>();
    maps.put('(', ')');
    maps.put('[', ']');
    maps.put('{', '}');
    maps.put('?', '?');
    int len = s.length();
    LinkedList<Character> queue = new LinkedList<>();
  	// 添加的元素作为循环后判断标准
    queue.offer('?');
    for (int i = 0; i < len; i++) {
        char charValue = s.charAt(i);
        if (maps.containsKey(charValue)) {
            Character tempValue = maps.get(charValue);
          	// 添加到最后
            queue.addLast(tempValue);
        } else {
          	// 移除最后
            Character poll = queue.removeLast();
            if (poll != charValue) {
                return false;
            }
        }
    }
    return queue.size() == 1;
}
```



