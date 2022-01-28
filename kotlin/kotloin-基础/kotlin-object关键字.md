## Object关键字

#### Object可以定义

- 匿名内部类
- 单例模式
- 伴生对象

#### object:匿名内部类

java和kotlin相同的地方在于，它们的接口与抽象类，都不能直接创建实例。要想创建接口和抽象类的实例，我们必须通过匿名内部类的方式；

Kotlin中使用object不仅可以创建接口的匿名内部类，还可以创建抽象类的匿名内部类；

在kotlin中，我们使用object定义匿名内部类，还可以在继承一个抽象类时，来实现多个接口。

```kotlin
interface A {
    fun funA()
}

interface B {
    fun funB()
}

abstract class Man {
    abstract fun findMan()
}

fun main() {
    val item = object : Man(), A, B {
        override fun findMan() {
        }

        override fun funA() {

        }

        override fun funB() {

        }
    }
}
```

#### object:单例模式

```kotlin
object SingleInstance {
    
    fun funA(){
        
    }
}
```

反编译对应的Java代码

```java
public final class SingleInstance {
   @NotNull
   public static final SingleInstance INSTANCE;

   public final void funA() {
   }

   private SingleInstance() {
   }

   static {
      SingleInstance var0 = new SingleInstance();
      INSTANCE = var0;
   }
}
```

缺点

- 不支持懒加载
- 不支持传递参数构造单例

#### object:伴生对象

Kotlin中没有static关键字，不能直接定义静态变量和静态方法，kotlin提供的伴生对象就是解决这个问题的。

```kotlin
class CompanionMain {

    companion object {
        fun funA() {

        }
    }
}
```

反编译后的Java代码

```java
public final class CompanionMain {
   @NotNull
   public static final CompanionMain.Companion Companion = new CompanionMain.Companion((DefaultConstructorMarker)null);

   public static final class Companion {
      public final void funA() {
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

使用@JvmStatic之后

```kotlin
class CompanionMain {
    companion object {
        @JvmStatic
        fun funA() {

        }
    }
}
```

反编译后的Java代码

```java
public final class CompanionMain {
   @NotNull
   public static final CompanionMain.Companion Companion = new CompanionMain.Companion((DefaultConstructorMarker)null);

   @JvmStatic
   public static final void funA() {
      Companion.funA();
   }
   public static final class Companion {
      @JvmStatic
      public final void funA() {
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

#### 工厂模式

```kotlin
class FactoryClass private constructor(name: String) {
    companion object {
        @JvmStatic
        fun create(name: String): FactoryClass {
            return FactoryClass(name)
        }

    }
}
```

#### 伴生对象的 Double Check

```kotlin
class UserManager private constructor(name: String) {
    companion object {
        @Volatile
        private var INSTANCE: UserManager? = null
        fun getInstance(name: String): UserManager =
            INSTANCE ?: synchronized(this) {
                INSTANCE ?: UserManager(name)?.also {
                    INSTANCE = it
                }
            }
    }
}
```

#### 抽象类模版

```kotlin
abstract class BaseSingleton<in P, out T> {

    @Volatile
    private var instance: T? = null
    protected abstract fun creator(param: P): T
    fun getInstance(param: P): T =
        instance ?: synchronized(this) {
            instance ?: creator(param).also { instance = it }
        }
}
```

抽象类模版的使用

```kotlin
class UserManager private constructor(name: String) {
    companion object : BaseSingleton<String, UserManager>() {
        override fun creator(param: String): UserManager {
            return UserManager(param)
        }
    }
}
```

