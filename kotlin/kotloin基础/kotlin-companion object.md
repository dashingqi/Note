#### Companion object 伴生对象

```kotlin
class CompanionObjectDemo {
    companion object {
      	// 相当于类的私有的常量
        private const val TAG2 = "dsdsds"
      	// 相当于类的公共常量
        const  val TAG = "CompanionObjectDemo"
      	// 类的私有静态变量 提供get和set方法进行访问在Companion对象中
        var currentTag = "dsdkjfkd"
      	// 类的私有静态变量，提供get方法进行访问在Companion对象中
        val tempTag = "fdjkldfjkld"
    }
}
```

反编译后的Java代码

```java
public final class CompanionObjectDemo {
  private static final String TAG2 = "dsdsds";
  @NotNull
  public static final String TAG = "CompanionObjectDemo";
  @NotNull
  private static String currentTag = "dsdkjfkd";
  @NotNull
  private static final String tempTag = "fdjkldfjkld";
  @NotNull
  public static final CompanionObjectDemo.Companion Companion = new CompanionObjectDemo.Companion((DefaultConstructorMarker)null);

  public static final class Companion {
    @NotNull
    public final String getCurrentTag() {
      return CompanionObjectDemo.currentTag;
    }

    public final void setCurrentTag(@NotNull String var1) {
      Intrinsics.checkNotNullParameter(var1, "<set-?>");
      CompanionObjectDemo.currentTag = var1;
    }

    @NotNull
    public final String getTempTag() {
      return CompanionObjectDemo.tempTag;
    }

    private Companion() {
    }

    // $FF: synthetic method
    public Companion(DefaultConstructorMarker $constructor_marker) {
      this();
    }
  }
}
```

