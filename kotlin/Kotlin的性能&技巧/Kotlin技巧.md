#### Kotlin技巧

#### 优雅处理字符串为空的情况

```kotlin
 // 当target为一个空字符串的时候 默认赋值为 str
 val name = target.ifEmpty { "str" }
```

#### 字符串转整形的优雅处理

```kotlin
val input = "123"
// 这个转换存在一定的问题，当input包含其他非数字字符会有问题 (java.lang.NumberFormatException)
val toInt = input.toInt()

// 优雅处理 字符串转数字的操作
val intResult = input.toIntOrNull() ?: 0
```

#### joinToString - 优雅将数组/集合 按照一定格式转换成字符串

```kotlin
// 集合
val data = listOf("java", "kotlin", "c++", "c")
.joinToString(
  separator = " | ",
  prefix = "{",
  postfix = "}"
)

// 数组
val arrayData = arrayOf("java", "kotlin", "c++", "c")
.joinToString(
  separator = " | ",
  prefix = "{",
  postfix = "}"
)

// 输出结果
2021-12-21 19:39:08.200 17302-17302/com.dashingqi.dqkotlin I/System.out: {java | kotlin | c++ | c}
2021-12-21 19:39:08.200 17302-17302/com.dashingqi.dqkotlin I/System.out: {java | kotlin | c++ | c}
```

#### 操作字符串的前后缀

```kotlin
val data2 = "**hi zq**"
// 移除前缀
println(data2.removePrefix("**"))
//移除后缀
println(data2.removeSuffix("**"))
//移除前缀和后缀
println(data2.removeSurrounding("**"))
// 返回第一次出现分割后的字符串
println(data2.substringAfter("**"))
//如果没有找到，返回原始字符串
println(data2.substringAfter("--"))
// 如果没有找到，返回默认字符串
println(data2.substringAfter("--", "DashingQi"))
```

#### Map集合的默认值

```kotlin
private fun withDefaultMethod() {
  val maps = mapOf<String, String>("0" to "Java").withDefault {
    println("it --> $it") // c++
    "DashingQi"
  }
  val currentC = maps.getValue("c++")
  println("currentC ---> $currentC")
}
```

使用withDefault()可以为集合指定一个默认值

当我们通过getValue()去获取key对应的value时 如果此时key不存在，正常是返回null，使用withDefault指定默认值时 就返回这个默认值了；

#### 使用require或者check函数作为条件检查

```kotlin
// require
private fun checkAge(age: Int) {
  require(age > 0) {
    "age must not be negative"
  }
}

private fun checkNotNullMethod(message: String?) {
  checkNotNull(message) {
    "name must not be null"
  }
}
```

#### 如何区分和使用 run、with、let、also、apply

| 函数    | 是否是扩展函数 | 函数参数（this, it） | 返回值（调用本身，最后一行） |
| ------- | -------------- | -------------------- | ---------------------------- |
| with    | 不是           | this                 | 最后一行                     |
| T.run   | 是             | this                 | 最后一行                     |
| T.let   | 是             | it                   | 最后一行                     |
| T.also  | 是             | it                   | 调用本身                     |
| T.apply | 是             | this                 | 调用本身                     |

