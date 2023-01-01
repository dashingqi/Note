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

一般情况下，只要算法中不逊在循环语句、递归语句，即使有成千上万行的代码，其时间复杂度也是O(1)

###### O(log^n)、O(n*log^2)

```java
 i=1;
 while (i <= n)  {
   i = i * 2;
 }
```

当有循环代码时，时间复杂度就是就是这段循环的代码

循环中做的事情就是每次乘以2

2^0、2^1、2^2、2^3 ...... 2^x = n

我们知道2^x = n

x = log2^n

那么这段代码就是 O(log2^n)

```java
 i=1;
 while (i <= n)  {
   i = i * 3;
 }
```

上面这段代码的时间复杂度是 O(log3^n)

**实际上不管是以2为底，还是3为底，还是10，可以把所有对数阶的时间复杂度记为 O(logn)**

###### O(m+n)、O(m*n)

```java
int cal(int m, int n) {
  int sum_1 = 0;
  int i = 1;
  for (; i < m; ++i) {
    sum_1 = sum_1 + i;
  }

  int sum_2 = 0;
  int j = 1;
  for (; j < n; ++j) {
    sum_2 = sum_2 + j;
  }

  return sum_1 + sum_2;
}
```

m和n是两个数据规模，无法判断谁的量级大，在表示时间复杂度的时候，就不能使用加分法则，省略掉其中一个

所以表示为 O(m+n)

加法规则：T1(m) + T2(n) = O(f(m) + g(n))

#### 空间复杂度

###### 时间复杂度

是渐进时间复杂度，表示算法的执行时间与数据规模之间的增长关系

###### 空间复杂度

是渐进空间复杂度，表示算法的存储空间与数据规模之间的增长关系

```java

void print(int n) {
  int i = 0;
  int[] a = new int[n];
  for (i; i <n; ++i) {
    a[i] = i * i;
  }

  for (i = n-1; i >= 0; --i) {
    print out a[i]
  }
}
```

空间复杂度是 O(n)

我们常见的空间复杂度是 O(1)、O(n)、O(n^2)

像O(logn)、O(nlogn)这样对数阶复杂度平时用不到；



