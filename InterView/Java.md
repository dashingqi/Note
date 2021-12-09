#### Java中的char是由两个字节组成，如何存储UTF-8

```java
/**
 * 1. Java中的char是由两个字节组成，如何存储UTF-8
 * char中的字节存储的是Unicode码点
 * Unicode是字符集 字符集包括 Unicode和ASCII 可以理解字符集是一种 一一对应的一种规则；
 * 字符到整数的映射是通过码点
 * UTF-16 最小单位是2个字节 UTF-8的最小单位是1个字节
 * Java中的char存储的是UTF-16 （有点浪费）
 * char就联想到字符串
 * Java9之前使用的是char数组进行存储char ，一个char使用两个字节存储 编码格式是UTF-16，浪费
 * Java9中对于字符串进行了优化，使用了byte数组，对于拉丁字符 使用Latin-1进行编码（一个字节就能存储下），对于中文以及其他字符超过Latin-1
 * 使用UTF-16
 * Java9中没有修复 字符数!=字符长度的bug 比如表情 一个表情的长度是2
 */
public class BaseMain {

    public static void main(String[] args) {
        char c = '张';
        // char 里面存储的是什么，打印出来的是 0x5e、0x86 是Unicode的码点
        // Unicode是字符集，字符集包括 Unicode和ASCII

        // UTF-8 最小单位是1个字节
        // UTF-16最小单位是2个字节
        // UTF-8 、 UTF-16 是编码格式

        // Java中的char存储的是 UTF-16 编码后

        System.out.println(Integer.toHexString(c));

        byte[] bytes = "张".getBytes(StandardCharsets.UTF_16);
        for (byte b : bytes) {
            System.out.println("b == " + Integer.toHexString(b));
        }
        // b == fffffffe
        // b == ffffffff
        // b == 5f
        // b == 20
        // 其中 fe 和 ff 是字节系列标志 不代表真正的内容
        System.out.println("UTF_16 byte length " + bytes.length);
        // Java9 中对与拉丁字符做了存储空间上的优化，9之前使用 char[] 数组存储，9以及之后使用 byte[] 默认是Latin-1编码也就是使用一个字节
        // 当字符串出现中文以及其他超出Latin-1 编码范围的字符的时候，这时候就使用UTF-16进行编码，默认是占2个字节的。
        System.out.println("\uD83D\uDE0A".length());
    }
}
```

#### Java中String有多长

```java
/**
 * Java String 有多长
 * 字面量
 * 存储在栈中，具体是方法区的常量池中
 * .java ---> .class 文件的时候
 *  String 是以如下结构进行存储
 *  CONSTANT_Utf8_info {
 *        u1 tag;
 *        u2 length;   // 0 ~ 65535
 *        u1 bytes[length];
 *    }
 *  UTF-8 编码
 *  length： 长度 u2：两个字节
 *  u1:使用byte[]进行存储，长度为 两个字节的长度也就是 65535个字节 (1111111111111111)
 *
 *  当使用 String a = "65535个拉丁字母" 会提示 超长
 *  当声明为65534个，是可以编译通过的
 *  在编译器的 Gen.java 文件中我们可以发现 对于字面量字节个数的检查，能编译通过是要小于65535
 *  这是编译器的一个Bug
 *  还要取决与 栈内存的空间大小
 *
 *  byte [] bytes = loadFile();
 *  String content = new String(bytes)
 *
 *  虚拟机指令 newarray 数组最长 是 Integer.MAX_VALUE
 *  堆内存空间的大小
 *
 *
 *
 */
public class StringMain {

    private static final String cd = "aaaaaa";
    public static void main(String[] args) {
        // 字面量  该种形式是存储在 栈内存 方法区种
        // 65535：编译器源码中有做判断 必须小于65535个字节
        // （javac里面的bug，拉丁字符最多是65534，非拉丁 最多65535）
        String longSting = "aaaaaaa";

        // 是存储在堆内存中的
        byte[] bytes = new byte[1024 * 1024];
        String s = new String(bytes);
    }
}
```

#### Java匿名内部类有哪些限制

###### 匿名内部类的概念和用法

没有人类认知意义上的名字

```java
public class InnerClass {

    public void run(){
       // 匿名内部类，字面上意思就是没有名字的内部类；
       // 其实从虚拟机角度来看是有名字的.只不过这个名字是由Java虚拟机定义的
       // 名字就是 base.InnerClass$1
       // $1 表示是该类中的第一个匿名内部类
        Foo foo = new Foo() {
            @Override
            public int bar() {
                return 0;
            }
        };

        // 匿名内部类,
        // 名字就是  base.InnerClass$2
        Runnable runnable = new Runnable() {

            @Override
            public void run() {

            }
        };

        // 方法体内部定义一个类 继承 Foo， 实现Runnable接口
        class Test extends Foo implements Runnable {
            @Override
            public void run() {

            }
        }

    }
}
```

###### 语言规范以及Kotlin的横向对比

匿名内部类的继承结构；只能继承一个父类或者实现一个接口

- 先天父类

  ```java
  class InnerClass$1 extends Foo {
      InnerClass$1(InnerClass var1) {
          this.this$0 = var1;
      }
  
      public int bar() {
          return 0;
      }
  }
  ```

- 实现接口

  ```java
  class InnerClass$2 implements Runnable {
      InnerClass$2(InnerClass var1) {
          this.this$0 = var1;
      }
  
      public void run() {
      }
  }
  ```

- 方法体内部可以实现也就是 同时继承一个类实现一个接口 在方法体内部

  ```java
  // 方法体内部定义一个类 继承 Foo， 实现Runnable接口
  class Test extends Foo implements Runnable {
    @Override
    public void run() {
  
    }
  }
  
  // 对应的字节码文件
  
  class InnerClass$1Test extends Foo implements Runnable {
      InnerClass$1Test(InnerClass var1) {
          this.this$0 = var1;
      }
  
      public void run() {
      }
  }
  ```

- Kotlin可以实现

  ```kotlin
  class InnerClassKotlin {
      fun test() {
          // Kotlin 是可以实现
          // 匿名内部类继承Foo，同时实现Runnable接口
          val runnable = object : Foo(), Runnable {
              override fun run() {
              }
          }
      }
  }
  ```

  

###### 内存泄漏的切入点

匿名内部类的构造方法

- 匿名内部类的构造方法是谁定义的 --- 虚拟机定义

- 构造方法参数列表

- Java中Lambda表达式：SAM

  ```java
  // SAM
  // 必需是接口，同时这个接口内部只有一个方法
  Runnable able = () -> {
  
  };
  ```

#### 怎么理解Java的方法分派

###### 问题分析点

- 多态、虚方法表的认识
- 对编译和运行是的理解和认识
- 对Java语言规范和运行机制的深入认识

- 就是确定调用谁的那个方法
- Java中的方法重载
- Java中的复写

###### Java方法分派

```java
/**
 * Java方法中的分派
 * 调用谁的那个方法
 *
 * 静态分派 -- 方法重载 （编译期确定）
 * 动态分派 -- 方法复写 （运行时确定，子类复写父类的方法）
 *
 * 程序输出什么 几乎Java中的所有方法都是虚方法；程序输出什么取决于运行时的实际类型--> SubClass 输出结果就是"Hello Sub"
 * 怎么调用的：取决于编译时期声明的类型
 *
 * 虚方法：能被子类复写的方法
 * private、public static 修饰的方法是不能被复写的
 *
 */
public class MethodQuestion {

    public static void main(String[] args) {
        SuperClass subClass = new SubClass();
        println(subClass);


        // 字节码的形态
        // SubClass var1 = new SubClass();
        // println((SuperClass)var1);

    }

    private static void println(SuperClass superClass) {
        System.out.println("Hello " + superClass.getName());
    }

    private static void println(SubClass subClass) {
        System.out.println("Hello " + subClass.getName());
    }
}
```

#### Java泛型的实现机制是怎么样的

###### 类型擦除有哪些优点

- 减少运行时内存的占用
- 兼容老版本的代码运行

###### 类型擦除有哪些问题

- 基本类型无法作为泛型实参
  - 需要基本类型的包装类型
  - 装箱和拆箱的开销
  - Google-Android 提供的SparseArray能解决

- 泛型类型无法用作方法重载

  ```java
  private void method1(List<String> str){
  
  }
  private void method1(List<Integer> str){
  
  }
  
  // 编译报错 编译时，类型擦除 List<Object> str
  ```
  
- 类型强转的运行时开销

  ```java
  // Java 1.5之前
  List strList = new ArrayList();
  strList.add("aaa");
  String value = (String)strList.get(0);
  
  // Java 1.5之后
  List<String> strList = new ArrayList<>();
  strList.add("aaa");
  String value = strList.get(0);
  
  // 编译后的字节码
  ArrayList var2 = new ArrayList();
  var2.add("aaa");
  // 进行了强转
  String var3 = (String)var2.get(0);
  
  ```
  
- 泛型类型无法作为真实类型使用

  ```java
  // 泛型类型无法当做真实类型使用
  static <T> void getMethod(T t) {
    T instance = new T(); // 报错
    T[] ts = new T[0]; // 报错
    Class<T> tClass = T.class; // 报错
  
    // 泛型擦除 T -> Object 通过
    List<T> list = new ArrayList<T>();
  
    // T -> Object 报错
    if (list instanceof List<String>) {
  
    }
  }
  ```

- 静态方法无法引用类的泛型参数

  ```java
  // 报错：类的泛型参数 在创建类的实例的时候才会确定，
  // 静态方法属于类本身，调用的时候无法确定泛型参数
  public class Method<T> {
      // 静态方法无法引用类的泛型参数
      static T get(T t){
          
      }
  }
  
  // ============解决办法==========
  
  static <R> R get(R a,R b){
    return a;
  }
  ```

- 泛型的方法签名

  - Gson
  - Retrofit

#### Activity的onActivityResult()这么麻烦，为什么不设计成回调？

- onActivityResult()基本使用
- 是否想过使用回调替换过onActivityResult
- 回调替换onActivityResult存在什么问题

###### onActivityResult()为什么麻烦





