### 栈：如何实现浏览器的前进和后退功能呢

#### 如何理解栈

先进后出数据结构

<img src="https://static001.geekbang.org/resource/image/3e/0b/3e20cca032c25168d3cc605fa7a53a0b.jpg?wh=1142*713"/>

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230213232425175.png" alt="image-20230213232425175" style="zoom:200%;" />

#### 如何实现一个“栈”

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230213232551897.png" alt="image-20230213232551897" style="zoom:200%;" />

###### 基于数组实现的栈

```java
public class ArrayStack {

    /**
     * 栈中元素的个数
     */
    private int stackCount;

    /**
     * 栈的大小
     */
    private final int stackSize;

    /**
     * 数组
     */
    private final String[] arrayStack;

    public ArrayStack(int stackSize) {
        this.arrayStack = new String[stackSize];
        this.stackSize = stackSize;
        // 一开始初始化时，栈中元素个数为0
        stackCount = 0;
    }

    /**
     * 入栈
     *
     * @param item 元素
     * @return true/false
     */
    public boolean push(String item) {
        // 如果当前栈满了就不需要入栈，直接返回false
        if (stackCount == stackSize) return false;
        arrayStack[stackCount] = item;
        ++stackCount;
        return true;
    }

    /**
     * 出栈
     *
     * @return 栈中元素
     */
    public String pop() {
        // 当栈中没有存储的元素时就返回null
        if (stackCount == 0) return null;
        String tempItem = arrayStack[stackCount - 1];
        --stackCount;
        return tempItem;
    }

}
```

##### 时间复杂度和空间复杂度

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230219171259036.png" alt="image-20230219171259036" style="zoom:200%;" />

#### 支持动态扩容的顺序栈

<img src="https://static001.geekbang.org/resource/image/b1/da/b193adf5db4356d8ab35a1d32142b3da.jpg?wh=1142*824"/>

#### 栈的应用

###### 栈在表达式求值中的应用

- 表达式：3+5*8-6

<img src="https://static001.geekbang.org/resource/image/bc/00/bc77c8d33375750f1700eb7778551600.jpg?wh=1142*790"/>

###### 栈实现浏览器的前进和后退

- 顺序查看 a,b,c页面

  <img src="https://static001.geekbang.org/resource/image/4b/3d/4b579a76ea7ebfc5abae2ad6ae6a3c3d.jpg?wh=1142*399"/>

- 从c页面返回到a页面

  <img src="https://static001.geekbang.org/resource/image/b5/1b/b5e496e2e28fe08f0388958a0e12861b.jpg?wh=1142*399"/>

- 又想查看b页面，此时点击前进按钮

  <img src="https://static001.geekbang.org/resource/image/ea/bc/ea804125bea25d25ba467a51fb98c4bc.jpg?wh=1142*399"/>

- 从b页面跳转到d页面,此时c页面无法在通过前进进入了就需要清理出去	<img src="https://static001.geekbang.org/resource/image/a3/2e/a3c926fe3050d9a741f394f20430692e.jpg?wh=1142*399"/>
