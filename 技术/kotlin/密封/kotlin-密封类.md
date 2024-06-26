#### Kotlin-密封类

#### 密封类

- Sealed Classes用于表示受限制的类层次结构；
- 从某种意义上说，Sealed Classes是枚举类的扩展；
- 枚举的不同之处在于，枚举中常量仅仅作为单个实例存在，而Sealed Classes的子类可以表示不同状态的实例；

#### Sealed Classes用于表示受限制的类层次结构

Sealed Clases用于表示层次结构

子类可以是任意的类，数据类、Kotlin对象、普通类、也可以是另外一个Sealed CLasses

**Sealed Classes受限制**

###### Kotlin 1.0

密封子类必须是在密封类的内部类

```kotlin
sealed class SealedClass {
    class Java(ver: String) : SealedClass()
    class JavaScript(ver: String) : SealedClass()
}
```

###### Kotlin 1.1

取消了子类必须在密封类内部定义的约束，

为了保证编译的同步仍然需要在同一文件中

```kotlin
sealed class SealedClass
class Java(ver: String) : SealedClass()
class JavaScript(ver: String) : SealedClass()
```

###### Kotlin 1.5

条件再次放宽

允许子类可以定义在不同的文件中

只要保证子类和父类在同一个module并且是同一个包名下就可以

```kotlin
// SealedClass.kt
sealed class SealedClass

// Compiled.kt
class Java(ver: String) : SealedClass()
class JavaScript(ver: String) : SealedClass()
```

#### 相对于密封类，枚举和抽象类的局限性

###### 枚举每个类型只允许有一个实例

```java
public enum Person {
  MAN(1),
  WOMEN(2);
  private int age;

  private Person(int age){
    this.age = age;
  }

  public int getAge() {
    return age;
  }

  public void setAge(int age) {
    this.age = age;
  }
}

private fun enumMethod() {
  val women = Person.WOMEN
  val man = Person.WOMEN
  println("enum boolean  ${women == man}")
}
```

输出结果

```xml
I/System.out: enum boolean  true
```

可以看到无论声明 Person.WOMEN 多个对象，他们的实例都是同一个；这既是枚举的局限也是它的优点；（相对于密封类）

###### 所有枚举常量使用相同类型的值

```kotlin
public enum Person {
    MAN(1),
    WOMEN(2);
    private int age;
    private String name;

    private Person(int age) {
        this.age = age;
    }

    private Person(int age, String name) {
        this.age = age;
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

如果你要这么做

![image-2021](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/202112281132104.png)



#### Sealed Classes是什么

![image-20220102114824361](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20220102114824361.png)

sealed classes 是一个抽象类，它本身不能被实例化。只能用它的子类实例化对象；

sealed classes的构造方法私有化；


#### Java枚举

###### 编译之前

```java
public enum Person {
//    CHILDRNE("efdf"),
    MAN(1),
    WOMEN(2);
    private int age;
    private String name;

    private Person(int age) {
        this.age = age;
    }

    private Person(int age, String name) {
        this.age = age;
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

###### 反编译Person.class文件之后

```java
/*
 * Decompiled with CFR 0.152.
 */
package com.dashingqi.dqkotlin.sealed;

public final class Person
extends Enum<Person> {
    public static final /* enum */ Person MAN = new Person("MAN", 0, 1);
    public static final /* enum */ Person WOMEN = new Person("WOMEN", 1, 2);
    private int age;
    private String name;
    private static final /* synthetic */ Person[] $VALUES;

    public static Person[] values() {
        return (Person[])$VALUES.clone();
    }

    public static Person valueOf(String string) {
        return Enum.valueOf(Person.class, string);
    }

    private Person(String string, int n, int n2) {
        super(string, n);
        this.age = n2;
    }

    private Person(String string, int n, int n2, String string2) {
        super(string, n);
        this.age = n2;
        this.name = string2;
    }

    public int getAge() {
        return this.age;
    }

    public void setAge(int n) {
        this.age = n;
    }

    static {
        $VALUES = new Person[]{MAN, WOMEN};
    }
}

```

#### 使用密封类优化标记类

**标记类**

```kotlin
class Figure(
    private val shape: Shape,
    private val radius: Double = 0.0,
    private val length: Double = 0.0,
    private val width: Double = 0.0
) {

    enum class Shape {
        RECTANGLE, CIRCLE
    }

    fun area(): Double = when (shape) {
        Shape.RECTANGLE -> length * width
        Shape.CIRCLE -> 3.14 * radius * radius
        else -> throw AssertionError(shape)
    }

    companion object {
        fun createCircle(radius: Double) =
            Figure(
                shape = Shape.CIRCLE,
                radius = radius
            )


        fun createRectangle(length: Double, width: Double = 0.0) =
            Figure(
                shape = Shape.RECTANGLE,
                length = length,
                width = width
            )


    }
}

// 使用

val area = Figure.createCircle(5.4).area()
println("area == $area")
```

**密封类**

利用密封类的层级结构特性

```kotlin
sealed class FigureSealed {
    abstract fun area(): Double
}

/**
 * 长方形
 * @property length Double 长度
 * @property width Double 宽度
 * @constructor
 */
class Rectangle(private val length: Double, private val width: Double) : FigureSealed() {
    override fun area(): Double = length * width
}

/**
 * 圆形
 * @property radius Double 直径
 * @constructor
 */
class Circle(private val radius: Double) : FigureSealed() {
    override fun area(): Double = 3.14 * radius * radius

}

// 使用
val circleArea = Circle(5.0).area()
println("circleArea is $circleArea")
```

如果我们要新增图形的话，就新增一个类就可以不必要改动原有代码结构

```kotlin
/**
 * 正方形
 * @property width Double 宽度
 * @constructor
 */
class Square(private val width: Double) : FigureSealed() {
    override fun area(): Double = width * width

}
```

配合when语句使用

```kotlin
fun FigureSealed.valida() = when (this) {
    is Circle -> {

    }

    is Rectangle -> {

    }
    is Square -> {
        
    }
}
```
