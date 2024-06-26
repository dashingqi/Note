#### 数组刷题

###### 合并两个数组

```java
/**
* 给你两个按 非递减顺序 排列的整数数组 nums1 和 nums2，另有两个整数 m 和 n ，分别表示 nums1 和 nums2 中的元素数目。
* <p>
* 请你 合并 nums2 到 nums1 中，使合并后的数组同样按 非递减顺序 排列。
* <p>
* 注意：最终，合并后数组不应由函数返回，而是存储在数组 nums1 中。为了应对这种情况，nums1 的初始长度为 m + n
* ，其中前 m 个元素表示应合并的元素，后 n 个元素为 0 ，应忽略。nums2 的长度为 n 。
* <p>
* <p>
* <p>
* 示例 1：
* <p>
* 输入：nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3
* 输出：[1,2,2,3,5,6]
* 解释：需要合并 [1,2,3] 和 [2,5,6] 。
* 合并结果是 [1,2,2,3,5,6] ，其中斜体加粗标注的为 nums1 中的元素。
*/
public void merge(int[] nums1, int m, int[] nums2, int n) {
  int i = m - 1;
  int j = n - 1;
  int k = m + n - 1;
  while (i >= 0 && j >= 0) {
      if (nums1[i] > nums2[j]) {
          nums1[k--] = nums1[i--];
      } else {
          nums1[k--] = nums2[j--];
      }
  }

  while (j >= 0) {
      nums1[k--] = nums1[j--];
  }
  System.out.println(nums1);
}
```

###### 有序数组，返回无重复数组

```java
/**
* 输入：nums = [1,1,2]
* 输出：2, nums = [1,2,_]
* 解释：函数应该返回新的长度 2 ，并且原数组 nums 的前两个元素被修改为 1, 2 。不需要考虑数组中超出新长度后面的元素。
* 示例 2：
* <p>
* 输入：nums = [0,0,1,1,1,2,2,3,3,4]
* 输出：5, nums = [0,1,2,3,4]
* 解释：函数应该返回新的长度 5 ， 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4 。不需要考虑数组中超出新长度后面的元素。
*/

public int removeDuplicatesV1(int[] nums) {
  if (nums == null || nums.length == 0) {
      return 0;
  }
  int len = nums.length;
  int fast = 1, slow = 1;
  while (fast < len) {
      if (nums[fast] != nums[fast - 1]) {
          nums[slow] = nums[fast];
          ++slow;
      }
      ++fast;
  }

  return slow;
}
```

###### 反转字符数组

```java
/**
 * 编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 s 的形式给出。
 * <p>
 * 不要给另外的数组分配额外的空间，你必须原地修改输入数组、使用 O(1) 的额外空间解决这一问题。
 * 输入：s = ["h","e","l","l","o"]
 * 输出：["o","l","l","e","h"]
 */
public void reverseString(char[] s) {
    if (s == null || s.length == 0) {
        return;
    }
    int left = 0, right = s.length - 1;
    while (left < right) {
        char tempValue = s[right];
        s[right] = s[left];
        s[left] = tempValue;
        left++;
        right--;
    }
}
```

###### 中心角标 - (左侧之和和右侧之和相等)

```java
public int pivotIndex(int[] array) {
    int total = 0;
    int sum = 0;
    for (int intValue : array) {
        total += intValue;
    }
    for (int i = 0; i < array.length; i++) {
        total -= array[i];
        if (sum == total) {
            return i;
        }
        sum += array[i];
    }

    return -1;
}
```

###### 两数之和

```java
/**
     * 给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。
     * <p>
     * 你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。
     * 输入：nums = [2,7,11,15], target = 9
     * 输出：[0,1]
     * 解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
     */
public int[] twoSum(int[] nums, int target) {
    if (nums == null || nums.length == 0) {
        return new int[0];
    }
    HashMap<Integer, Integer> maps = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int value = nums[i];
        int realKey = target - value;
        if (maps.containsKey(realKey)) {
            Integer realIndex = maps.get(realKey);
            return new int[]{realIndex, i};
        } else {
            maps.put(value, i);
        }
    }
    return new int[0];
}
```

###### 使用数组实现队列

```java
static class DQQueueV1 {
        int[] arr;
        int capacity;
        int head = 0;
        int tail = 0;

        public DQQueueV1(int capacity) {
            this.capacity = capacity;
            arr = new int[capacity];
        }

        /**
         * 入队
         *
         * @param value
         */
        public void enqueue(int value) {
            // 当前队列已满
            if (tail == capacity) {
                return;
            }
            arr[tail] = value;
            tail++;
        }

        /**
         * 出队
         *
         * @return
         */
        public int dequeue() {
            // 判断当前队列空为空
            if (head == tail) return -1;
            int tempValue = arr[head];
            head++;
            return tempValue;
        }

        /**
         * 大小
         *
         * @return
         */
        public int size() {
            return tail - head;
        }
    }
```

