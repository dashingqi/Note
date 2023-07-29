#### reified关键字（泛型实化）

###### 作用

- reified关键字是用于泛型函数，主要用于在泛型函数内部获取到泛型具体类型信息；
- reified关键字只能用于内联的泛型函数;(inline)
- 普通的泛型函数在编译时会进行泛型擦除操作，导致在泛型函数内部就不能获取到实际的泛型类型；

```kotlin
inline fun <reified T> checkType(value: Any) {
    if (value is T) {
        println("The value has type ${T::class.simpleName}")
    } else {
        println("The value does not have type ${T::class.simpleName}")
    }
}

fun main() {
    val str = "Hello"
    val num = 42
    checkType<String>(str)
    checkType<Int>(num)
}
```

```kotlin
inline fun <reified T> T?.notNull(notNullAction: (T) -> Unit, nullAction: () -> Unit = {}) {
    if (this != null) {
        notNullAction(this)
    } else {
        nullAction()
    }
}
```

