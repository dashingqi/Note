#### ==

- 对于基本数据类型比较的是值是否相等；
- 对于对象的类型来说，调用的是对象的equals方法进行比较；（实际比较的是内存地址）

#### ===

- 基本数据类型比较的是值是否相等；

- 用于比较引用是都相同，也就是引用是否指向同一个对象；

```kotlin
private fun equalFun() {
        val a = 10
        val b = 10

        val x = "m"
        val y = "m"

        val m: Int? = 1 // 对于 Int数值会进行缓存， 缓存成一个对象 缓存的区间未 [-127,128) 
        val n: Int? = 1 // 当 m 和 n = 128时 就没做缓存 === 判定为 false

        println(a == b) // true
        println(a === b) // true

        println(m == n) // true 值相同
        println(m === n) // true

        println(x == y) // true 值相同
        println(x === y) // 引用相同，编译时对于字符串常量会有优化成一个对象；
    }
```

```kotlin
2023-08-22 20:39:23.080 10774-10774/com.dashingqi.dqkotlin I/System.out: true
2023-08-22 20:39:23.080 10774-10774/com.dashingqi.dqkotlin I/System.out: true
2023-08-22 20:39:23.080 10774-10774/com.dashingqi.dqkotlin I/System.out: true
2023-08-22 20:39:23.080 10774-10774/com.dashingqi.dqkotlin I/System.out: true
2023-08-22 20:39:23.080 10774-10774/com.dashingqi.dqkotlin I/System.out: true
2023-08-22 20:39:23.080 10774-10774/com.dashingqi.dqkotlin I/System.out: true
```

