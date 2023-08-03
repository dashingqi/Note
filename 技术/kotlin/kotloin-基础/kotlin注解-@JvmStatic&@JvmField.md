#### 特有注解

##### @JvmStatic

> 该注解能用于 object修饰的单利类中或者正常类中的伴生对象的方法上

```kotlin
object KotlinJvmAnnotation {
    @JvmStatic
    fun getMethod(){

    }
}

class KotlinJvmAnnotationOne{
    
    companion object {
        @JvmStatic
        fun getMethod(){
            
        }
    }
}
```

- 未使用@JvmStatic注解时，Java处调用

```java
KotlinJvmAnnotation.INSTANCE.getMethod();
KotlinJvmAnnotationOne.Companion.getMethod();
```

- 使用该注解修饰后

```java
KotlinJvmAnnotation.getMethod();
KotlinJvmAnnotationOne.getMethod();
```

真正达到Java中静态方法调用的方式

###### 使用@JvmStatic之前的Java代码

```java
public final class KotlinJvmAnnotationOne {
   @NotNull
   public static final KotlinJvmAnnotationOne.Companion Companion = new KotlinJvmAnnotationOne.Companion((DefaultConstructorMarker)null);
   public static final class Companion {
      public final void getMethod() {
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}

public final class KotlinJvmAnnotation {
   @NotNull
   public static final KotlinJvmAnnotation INSTANCE;

   public final void getMethod() {
   }

   private KotlinJvmAnnotation() {
   }

   static {
      KotlinJvmAnnotation var0 = new KotlinJvmAnnotation();
      INSTANCE = var0;
   }
}
```



###### 使用@JvmStatic后的Java代码

```java
public final class KotlinJvmAnnotationOne {
   @NotNull
   public static final KotlinJvmAnnotationOne.Companion Companion = new KotlinJvmAnnotationOne.Companion((DefaultConstructorMarker)null);

   @JvmStatic
   public static final void getMethod() {
      Companion.getMethod();
   }
  
   public static final class Companion {
      @JvmStatic
      public final void getMethod() {
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}

public final class KotlinJvmAnnotation {
   @NotNull
   public static final KotlinJvmAnnotation INSTANCE;

   @JvmStatic
   public static final void getMethod() {
   }

   private KotlinJvmAnnotation() {
   }

   static {
      KotlinJvmAnnotation var0 = new KotlinJvmAnnotation();
      INSTANCE = var0;
   }
}

```

使用@JvmStatic注解修饰的方法，反编译成Java代码后，加入 static 关键字变成真正的静态方法；



#### JvmField

```kotlin
class KotlinJvmAnnotationOne{
    @JvmField
    var name = "dashingqi"
    companion object {
        fun getMethod(){

        }
    }
}
```

###### 未使用@JvmField时Java代码调用处

```java
KotlinJvmAnnotationOne kotlinJvmAnnotationOne = new KotlinJvmAnnotationOne();
String name = kotlinJvmAnnotationOne.getName();
kotlinJvmAnnotationOne.setName("");
```

```java
public final class KotlinJvmAnnotationOne {
   @NotNull
   private String name = "dashingqi";
   @NotNull
   public static final KotlinJvmAnnotationOne.Companion Companion = new KotlinJvmAnnotationOne.Companion((DefaultConstructorMarker)null);

   @NotNull
   public final String getName() {
      return this.name;
   }

   public final void setName(@NotNull String var1) {
      Intrinsics.checkNotNullParameter(var1, "<set-?>");
      this.name = var1;
   }
   public static final class Companion {
      public final void getMethod() {
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

会生成对应的 get和 set 方法来访问属性，属性为 private 级别

###### 使用@JvmField时 Java 处代码调用处

```java
KotlinJvmAnnotationOne kotlinJvmAnnotationOne = new KotlinJvmAnnotationOne();
String name = kotlinJvmAnnotationOne.name;
kotlinJvmAnnotationOne.name = "";
```

```java
public final class KotlinJvmAnnotationOne {
   @JvmField
   @NotNull
   public String name = "dashingqi";
   @NotNull
   public static final KotlinJvmAnnotationOne.Companion Companion = new KotlinJvmAnnotationOne.Companion((DefaultConstructorMarker)null);
   public static final class Companion {
      public final void getMethod() {
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

不会生成对应的Get和Set 方法来访问属性，属性访问级别为 public



