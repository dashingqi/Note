#### 区别

```kotlin
val str = ""
val strJavaClass = str.javaClass
val strKClassJava = str::class.java
val strKClass = str::class
```

###### str.javaClass

- 这是在获取对象 `str` 的 Java 类。在 Kotlin 中，当你对一个对象使用 `.javaClass`，它等同于 Java 中的 `str.getClass()`.
- 这返回一个 `Class<T>` 实例，其中 `T` 是对象的运行时类型。

###### str::class.java

- 这里，`str::class` 首先获取 Kotlin 的 `KClass` 表示，这是 Kotlin 的类表示。
- `.java` 是一个扩展属性，它将 `KClass` 实例转换为等效的 Java `Class` 实例。
- 这种方式常用于需要与 Java 库或 APIs 互操作时。

###### str::class

- 这是直接获取对象 `str` 的 Kotlin 类表示（`KClass`）。
- 它与 Java 中的 `getClass()` 不完全一样，因为它返回的是 `KClass` 而不是 `Class`。
- 这适用于完全在 Kotlin 中工作时。

#### KClass的出现

- kotlin 作为一门独立于 Java 但是又与之有紧密联系的语言，它自己有一套自己的特性和设计理念；因此它也需要一种机制来表达和操作 Kotlin 自己的类型信息，这就是 KClass出现的原因；
- Kotlin 语言的特性
  - 数据类（data class）
  - 协程
  - 扩展方法
  - 伴生对象
- kotlin 的发展不仅限于 JVM，还可以在 JS 或者 Native 平台；