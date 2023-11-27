#### 树刷题

###### 前序遍历

```java
/**
 * 二叉树前序遍历
 * 根节点 左节点 右节点
 *
 * @param node 二叉树节点
 */
public static void preOrderTraverse(TreeNode node) {
    if (node == null) {
        return;
    }
    System.out.println(node.data);
    preOrderTraverse(node.leftChild);
    preOrderTraverse(node.rightChild);
}
```

###### 中序遍历

```java
/**
 * 二叉树中序遍历 （左节点、根节点、右节点）
 *
 * @param node 二叉树节点
 */
public static void inOrderTraverse(TreeNode node) {
    if (node == null) {
        return;
    }

    inOrderTraverse(node.leftChild);
    System.out.println(node.data);
    inOrderTraverse(node.rightChild);

}
```

###### 后序遍历

```java
/**
 * 后序遍历 (左子树、右子树、根节点)
 *
 * @param node 二叉树节点
 */
public static void postOrderTraverse(TreeNode node) {
    if (node == null) {
        return;
    }
    postOrderTraverse(node.leftChild);
    postOrderTraverse(node.rightChild);
    System.out.println(node.data);
}
```

###### 层序遍历

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

###### 深度遍历

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

