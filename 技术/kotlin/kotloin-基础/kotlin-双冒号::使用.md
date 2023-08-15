#### 双冒号 :: 使用

kotlin中双冒号的使用表示把一个方法当作一个参数，传递到另外一个方法中进行使用；

不过需要注意的是，作为参数的函数，该函数的参数类型和返回值类型一定要和规定的一致

```kotlin

private fun getResult(str: String, str2: String): String = "result is {$str , $str2}"

private fun lock(str: String, str2: String, method: (str: String, str2: String) -> String): String {
  return method.invoke(str, str2)
}

// 使用

 println("result is ${lock("dashingqi", "zhangqi", ::getResult)}")
```



#### 类对象

```kotlin
fun main(){
  val a = A::class.java
  val b = A::class
}

class A{}
```

