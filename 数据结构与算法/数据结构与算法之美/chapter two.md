## 复杂度分析：如何分析、统计算法的执行效率和资源消耗

衡量算法代码的执行效率 - 时间、空间复杂度

#### 大O复杂度表示法

算法的执行效率，粗略地讲，就是算法代码执行的时间；

所有代码执行的时间T(n)与每行代码执行的次数f(n)成正比

Tn = O*(f(n))

#### 时间复杂度分析

三个比较使用的方法

###### 只关注循环执行次数最多的一段代码

大O复杂度表示方法是一种变化趋势，我们通常会忽略掉公式中的【常量】、【低阶】、【系数】，只需要记录最大阶的量级就可以了；

在分析一个算法、一段代码的时间复杂度、仅仅关注【循环次数】最多的那一段代码；

```java

 int cal(int n) {
   int sum = 0;
   int i = 1;
   for (; i <= n; ++i) {
     sum = sum + i;
   }
   return sum;
 }
```

T(n) = O(n)

###### 加法法则：总复杂度等于量级最大的那段代码的复杂度

```java

int cal(int n) {
   int sum_1 = 0;
   int p = 1;
   for (; p < 100; ++p) {
     sum_1 = sum_1 + p;
   }

   int sum_2 = 0;
   int q = 1;
   for (; q < n; ++q) {
     sum_2 = sum_2 + q;
   }
 
   int sum_3 = 0;
   int i = 1;
   int j = 1;
   for (; i <= n; ++i) {
     j = 1; 
     for (; j <= n; ++j) {
       sum_3 = sum_3 +  i * j;
     }
   }
 
   return sum_1 + sum_2 + sum_3;
 }
```

该段代码的时间复杂度为 T(n) = O(n^2)

###### 乘法法则：嵌套代码的复杂度等于潜逃内外代码复杂度的乘积

*乘法法则可以堪称是嵌套循环*

```java

int cal(int n) {
   int ret = 0; 
   int i = 1;
   for (; i < n; ++i) {
     ret = ret + f(i);
   } 
 } 
 
 int f(int n) {
  int sum = 0;
  int i = 1;
  for (; i < n; ++i) {
    sum = sum + i;
  } 
  return sum;
 }
```

T(n) = T1(n) * T2(n) = O(n*n) = O(n^2)

##### 几种常见时间复杂度实例分析

###### O(1)

```java
 int i = 8;
 int j = 6;
 int sum = i + j;
```



###### O(log^n)、O(n*log^2)





