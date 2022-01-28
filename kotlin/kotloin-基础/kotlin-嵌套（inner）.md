## Kotlin-嵌套

看一段代码

```kotlin
class A {

    val name: String = "A"
    fun method() = 2
    
    class B {

    }
}
```

如果你在B中访问A中的属性和方法，会报错

![image-20220121231551778](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20220121231551778.png)

看下代码反编译后成Java的代码

```java
public final class A {
   @NotNull
   private final String name = "A";

   @NotNull
   public final String getName() {
      return this.name;
   }

   public final int method() {
      return 2;
   }
  
   public static final class B {
   }
}
```

可以看见反编译后的Java代码，class B是静态的，A属于普通类，静态类只能访问类的成员，不能访问对象的成员，就报错了；



如果想正常访问外部类的成员，可以使用inner

![image-20220121231523971](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20220121231523971.png)

看下反编译后的Java代码 (非静态内部类默认持有外部类的引用)

```java
public final class A {
   @NotNull
   private final String name = "A";

   @NotNull
   public final String getName() {
      return this.name;
   }

   public final int method() {
      return 2;
   }

   public final class B {
      @NotNull
      private final String bName = A.this.getName();
      private final int bMethod = A.this.method();

      @NotNull
      public final String getBName() {
         return this.bName;
      }

      public final int getBMethod() {
         return this.bMethod;
      }
   }
}

```

