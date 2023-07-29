#### Operator关键字

> operator对特定的函数进行运算符的重载；
>
> 通过这个关键字可以为自定类会哦这数据类型重载一些运算符；
>
> 提高代码的可读性和表达能力

```kotlin
data class Point(val x: Int, val y: Int) {
    // 重载二元运算符 +
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }

    // 重载二元运算符 -
    operator fun minus(other: Point): Point {
        return Point(x - other.x, y - other.y)
    }
}

fun operatorMain() {
    val p1 = Point(2, 3)
    val p2 = Point(1, 5)

    // 使用重载的 + 运算符
    val p3 = p1 + p2
    println("p3: $p3") // Output: p3: Point(x=3, y=8)

    val p4 = p3 - p1
    println("p4: $p4")
}
```

