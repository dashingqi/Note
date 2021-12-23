#### Kotlin技巧



###### 优雅处理字符串为空的情况

```kotlin
 // 当target为一个空字符串的时候 默认赋值为 str
 val name = target.ifEmpty { "str" }
```

###### 字符串转整形的优雅处理

```kotlin
val input = "123"
// 这个转换存在一定的问题，当input包含其他非数字字符会有问题 (java.lang.NumberFormatException)
val toInt = input.toInt()

// 优雅处理 字符串转数字的操作
val intResult = input.toIntOrNull() ?: 0
```

###### joinToString - 优雅将数组/集合 按照一定格式转换成字符串

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

###### 操作字符串的前后缀

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

###### equals方法 代替 toLowerCase()和toUpperCase方法

```kotlin
val newName = "zhangqi"
val oldName = "zhangQi"

// ========================================================
val result = newName.toLowerCase() == oldName.toLowerCase()
val result1 = newName.toUpperCase() == oldName.toUpperCase()
// ========================================================

newName.equals(oldName,true)
```

主要的原因

使用toLowerCase() 和 toUpperCase()方法回格外生成其他的对象

```java
String newName = "zhangqi";
String oldName = "zhangQi";
boolean var4 = false;
// 生成的对象
String var10000 = newName.toLowerCase();
Intrinsics.checkNotNullExpressionValue(var10000, "(this as java.lang.String).toLowerCase()");
var4 = false;
// 生成的对象
String var10001 = oldName.toLowerCase();
Intrinsics.checkNotNullExpressionValue(var10001, "(this as java.lang.String).toLowerCase()");
boolean result = Intrinsics.areEqual(var10000, var10001);
boolean var5 = false;
var10000 = newName.toUpperCase();
Intrinsics.checkNotNullExpressionValue(var10000, "(this as java.lang.String).toUpperCase()");
var5 = false;
var10001 = oldName.toUpperCase();
Intrinsics.checkNotNullExpressionValue(var10001, "(this as java.lang.String).toUpperCase()");
boolean result1 = Intrinsics.areEqual(var10000, var10001);
```

###### Kotlin遍历数组

- forEach

  ```kotlin
  val array = arrayOf(1, 2, 3)
  array.forEach {
    print("it is $it")
  }
  
  // 编译后的java代码
  
  // 创建额外的对象
  Integer[] var3 = array;
  int var4 = array.length;
  
  for(int var5 = 0; var5 < var4; ++var5) {
    // 装箱、拆箱的操作，有一定的损耗
    Object element$iv = var3[var5];
    int it = ((Number)element$iv).intValue();
    int var8 = false;
    String var9 = "it is " + it;
    boolean var10 = false;
    System.out.print(var9);
  }
  ```

  - 会生成额外的对象
  - 存在装箱与拆箱的操作，存在一定的损耗

- 使用indices

  ```kotlin
  // 使用 indices 遍历数组
  for (index in array.indices){
  
  }
  
  // 编译后的java代码
  int var1 = 0;
  for(int var11 = array.length; var1 < var11; ++var1) {
  }
  ```

  - 会生成临时的变量

- 使用区间表达式

  **.. 左闭右闭区间**

  ```kotlin
  for (index in 0..array.size) {
  
  }
  
  // 编译后的Java代码
  index = 0;
  var11 = array.length;
  if (index <= var11) {
    while(index != var11) {
      ++index;
    }
  }
  ```

  - 会生成临时的变量

  **downTo 实现降序**

  ```kotlin
  // 使用 downTo
  for (index in 0 downTo array.size) {
    println("downTo is $index")
  }
  
  // 编译后的Java代码
  index = array.length;
  
  for($i$f$forEach = false; index >= 0; --index) {
    var12 = "downTo is " + index;
    var13 = false;
    System.out.println(var12);
  }
  ```

  - 会生成临时的变量

  **until左闭右开**

  ```kotlin
  // 左闭右开
  for (index in 0 until array.size) {
    println("until index is $index")
  }
  
  // 编译后的Java代码
  for(var11 = array.length; index < var11; ++index) {
    var12 = "until index is " + index;
    var13 = false;
    System.out.println(var12);
  }
  ```

  - 会生成临时的变量

#### Lambda表达式作为函数的参数进行传递 （高阶函数）

```kotlin
fun request(type: Int, call: (code: Int) -> Unit) {

}
```

反编译对应的Java代码

```java
public final class LambdaDemoKt {
   public static final void request(int type, @NotNull Function1 call) {
      Intrinsics.checkNotNullParameter(call, "call");
   }
}

// 字节码
  L0
    ALOAD 1
    LDC "call"
    INVOKESTATIC kotlin/jvm/internal/Intrinsics.checkNotNullParameter (Ljava/lang/Object;Ljava/lang/String;)V
   L1
    LINENUMBER 9 L1
    ALOAD 1
    SIPUSH 200
    // 装箱
    INVOKESTATIC java/lang/Integer.valueOf (I)Ljava/lang/Integer;
    ILOAD 0
    INVOKESTATIC java/lang/Integer.valueOf (I)Ljava/lang/Integer;
    INVOKEINTERFACE kotlin/jvm/functions/Function2.invoke (Ljava/lang/Object;Ljava/lang/Object;)Ljava/lang/Object; (itf)
    POP
```

- 创建了一个Function1对象，增大了编译代码的体积
- 存在装箱和拆箱的损耗
