#### 字符串刷题

###### 回文字符串

```java
/**
 * 如果在将所有大写字符转换为小写字符、并移除所有非字母数字字符之后，短语正着读和反着读都一样。则可以认为该短语是一个 回文串 。
 * <p>
 * 字母和数字都属于字母数字字符。
 * 给你一个字符串 s，如果它是 回文串 ，返回 true ；否则，返回 false 。
 * <p>
 * 输入: s = "A man, a plan, a canal: Panama"
 * 输出：true
 * 解释："amanaplanacanalpanama" 是回文串。
 */
public boolean isPalindromeV1(String s) {
    if (s == null || s.isEmpty()) {
        return false;
    }
    int left = 0, right = s.length() - 1;
    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) {
            return false;
        }
        left++;
        right--;
    }

    return true;
}
```

###### 最长子串

```java
/**
 * 给定一个字符串 s ，请你找出其中不含有重复字符的 最长子串 的长度。
 * <p>
 * 输入: s = "abcabcbb"
 * 输出: 3
 * 解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
 */

public int lengthOfLongestSubstring(String s) {
    if (s.isEmpty()) {
        return 0;
    }
    int len = s.length();
    int maxLen = 0;
    HashMap<Character, Integer> maps = new HashMap<>();
    for (int start = 0, end = 0; end < len; end++) {
        char value = s.charAt(end);
        if (maps.containsKey(value)) {
            Integer index = maps.get(value);
            start = Math.max(start, index);
        }
        maxLen = Math.max(maxLen, end - start + 1);
        maps.put(value, end + 1);
    }
    return maxLen;
}
```



