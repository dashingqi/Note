### 为什么插入排序要比冒泡排序更受欢迎

#### 如何分析一个“排序算法”

###### 最好情况、最坏情况、平均情况时间复杂度

###### 时间复杂度的系数、常数、低阶

###### 比较次数和交换（或移动）次数

#### 排序算法的内存消耗

###### 原地排序

特指空间复杂度是O(1)的排序算法；

#### 排序算法的稳定性

###### 稳定性

如果待排序的序列中存在值相等的元素，经过排序之后，相等元素之间原有的先后顺序不变；

#### 冒泡排序

###### 代码实现

```java
public class BubbleSort {

    /**
     * 冒泡排序
     * @param a 要排序的数组
     */
    public static void bubbleSort(int[] a) {
        int length = a.length;
        if (length <= 1) return;
        for (int i = 0; i < length; ++i) {
            boolean flag = false;
            for (int j = 0; j < length - i - 1; ++j) {
                if (a[j] > a[j + 1]) {
                    int temp = a[j];
                    a[j] = a[j + 1];
                    a[j + 1] = temp;
                    flag = true;
                }
            }
            if (!flag) break;
        }
    }
}

```

###### 冒泡排序是原地排序算法吗？

是一个原地排序算法；

###### 冒泡排序是稳定的排序算法吗？

是排序算法

###### 冒泡排序的时间复杂度是多少？

需要看

#### 插入排序

