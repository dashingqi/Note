#### kotlin-扩展

##### 顶层扩展

扩展可用于函数和属性

函数扩展

```kotlin
/**
 * 顶层扩展 -> 函数扩展
 * @receiver String
 * @return Int
 */
fun String.lengthExt(): Int {
    return length
}
```

属性扩展

```kotlin
/**
 * 顶层扩展 -> 属性扩展
 */
val String?.isNullOrBlankExt: Boolean
    get() = this == null || this.isEmpty()
```

反编译后的对应的Java文件

```kotlin
@Metadata(
   mv = {1, 5, 1},
   k = 2,
   d1 = {"\u0000\u0014\n\u0000\n\u0002\u0010\u000b\n\u0002\u0010\u000e\n\u0002\b\u0002\n\u0002\u0010\b\n\u0000\u001a\n\u0010\u0004\u001a\u00020\u0005*\u00020\u0002\"\u0017\u0010\u0000\u001a\u00020\u0001*\u0004\u0018\u00010\u00028F¢\u0006\u0006\u001a\u0004\b\u0000\u0010\u0003¨\u0006\u0006"},
   d2 = {"isNullOrBlankExt", "", "", "(Ljava/lang/String;)Z", "lengthExt", "", "DQKotlin.app"}
)
public final class StringExtKt {
   public static final int lengthExt(@NotNull String $this$lengthExt) {
      Intrinsics.checkNotNullParameter($this$lengthExt, "$this$lengthExt");
      return $this$lengthExt.length();
   }

   public static final boolean isNullOrBlankExt(@Nullable String $this$isNullOrBlankExt) {
      boolean var10000;
      if ($this$isNullOrBlankExt != null) {
         CharSequence var1 = (CharSequence)$this$isNullOrBlankExt;
         boolean var2 = false;
         if (var1.length() != 0) {
            var10000 = false;
            return var10000;
         }
      }

      var10000 = true;
      return var10000;
   }
}
```

调用处的代码

```java
// 调用处
int lengthExt = StringExtKt.lengthExt(name);
boolean nullOrBlankExt = StringExtKt.isNullOrBlankExt(name);
```



顶层扩展的本质就是【静态】，和我们封装的工具类差不多；

##### 类内扩展

```kotlin
class Host(val hostName: String) {
    fun printHostName() {
        println("hostName is $hostName")
    }
}


class Test(val host: Host, val port: Int) {

    private fun printPort() {
        println("port is $port")
    }
		
  // 类内扩展方法
    fun Host.printConnectionString() {
        printHostName()
        println(":")
        printPort()
    }
	// 类内扩展属性
    val Host.isHomeEmpty: Boolean
        get() = hostName.isEmpty()
}
```

类内扩展原理

```kotlin
public final void printConnectionString(@NotNull Host $this$printConnectionString) {
  Intrinsics.checkNotNullParameter($this$printConnectionString, "$this$printConnectionString");
  $this$printConnectionString.printHostName();
  String var2 = ":";
  boolean var3 = false;
  System.out.println(var2);
  this.printPort();
}

public final boolean isHomeEmpty(@NotNull Host $this$isHomeEmpty) {
  Intrinsics.checkNotNullParameter($this$isHomeEmpty, "$this$isHomeEmpty");
  CharSequence var2 = (CharSequence)$this$isHomeEmpty.getHostName();
  boolean var3 = false;
  return var2.length() == 0;
}
```



#### 扩展是静态的

扩展是静态的,就是不支持多态

```kotlin
open class Shape

class Rectangle : Shape()

fun Rectangle.getName() = "Rectangle"

fun Shape.getName() = "shape"

fun printClassName(s: Shape) {
    println("name is ${s?.getName()}")
}

printClassName(Rectangle())

// shape
```

#### 带接受者的参数类型

```kotlin
// 正常的高阶函数
fun User.apply2(block: (user: User) -> Unit): User {
        block(this)
        return this
}

// 转化成带接受者的参数类型
fun User.apply(block: User.() -> Unit): User {
        block()
        return this
}
```

