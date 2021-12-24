## Kotlin性能

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