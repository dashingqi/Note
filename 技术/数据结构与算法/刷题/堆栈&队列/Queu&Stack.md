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



