#### 二分查找

###### 时间复杂度

O(logn)

###### 代码实现

```java
public class HalfSearch {
    public static int bSearch(int[] array, int length, int value) {
        return bSearchInternally(array, 0, length - 1, value);
    }

    private static int bSearchInternally(int[] array, int low, int high, int value) {
        if (low > high) return -1;
        int mid = low + ((high - low) >> 1);
        if (array[mid] == value) {
            return mid;
        } else if (array[mid] < value) {
            return bSearchInternally(array, mid + 1, high, value);
        } else {
            return bSearchInternally(array, low, mid - 1, value);
        }
    }
}

```

###### 二分查找的局限性

- 二分查找依赖的是顺序表结构，简单点说就是数组

  - 数组随机访问的时间复杂度是O(1)
  - 链表随机访问的时间复杂度是O(n)

- 二分查找针对的是有序数据

- 数据量太小不适合二分查找

  二分查找只有在数据量比较大的时候才会体现出它的优势

- 数据量太大也不适合二分查找

  主要原因是因为第一点，依赖顺序表结构，是需要一块连续的内存，如果是1GB的数据那么久需要连续申请1GB的内存；