## Class对象在执行引擎中的初始化过程

- ClassLoader主要的作用就是将编译好的.class文件加载到JVM中。
- 一个class文件被加载到内存中的需要经过如下三步
  - 装载
  - 链接
    - 验证
    - 准备
    - 解析
  - 初始化

#### 装载

> 指的是Java虚拟机查找.class文件并生成字节流，根据字节流创建Java.lang.Class对象的过程

###### 装载的过程主要完成如下事情

- ClassLoader通过一个类的全限定名（包名+类名）来查找.class文件，将生成二进制字节流。
- 把得到的.class文件的二进制字节流解析成JVM内部特定的数据结构，并存储在方法区中。
- 在内存中创建一个Java.lang.Class类型的对象。

###### 加载时机

- 隐式装载：在程序运行过程中，当碰到通过new关键字生成类的对象时，系统会隐式调用ClassLoader区加载对应的class文件到内存中
- 显示装载：写代码的时候，主动调用Class.formName("name")时，会进行class的装载操作。

#### 链接

> 链接的过程分为如下三个过程

###### 验证

> 主要为了保证.class文件的字节流包含的信息符合当前虚拟机的需要，会做如下几个的校验

- 文件格式校验
- 元数据校验
- 字节码校验
- 符号引用校验

###### 准备

- 为类中的静态变量分配内存，并为其设置“0值”

- 有一种情况比较特殊 就是静态常量

  ```java
  public static final int value = 100
  ```

  - 在准备阶段为静态变量value分配内存空间，并且设置为100（不是静态常量都是在准备阶段分配内存，在初始化阶段赋值）

- Java中基本类型的默认“0值”

  - 基本类型（short,int,long,float,double,boolean,char,byte,boolean）默认值为0
  - 引用类型默认值是null

###### 解析

- 这个过程主要就是把常量池中的符号引用转换为直接引用（内存地址）

#### 初始化

> 这是class加载的最后一步，这一阶段是执行类构造器方法的过程，并真正初始化类变量

- 比如

  ```java
  public static int value = 100;
  ```

  - 在链接的准备阶段为静态变量value分配内存
  - 在现在的初始化阶段value变量赋值为100

###### 初始化时机

- 有如下几种情况会触发class的初始化
  - 虚拟机启动时，初始化包含main方法的主类
  - 遇到new关键字创建类的对象时，如果该类没有初始化就开始初始化操作
  - 当访问类中的静态方法或者静态变量时，该类没有被初始化就进行初始化操作
  - 子类在初始化的时候，会先去检查父类是否被初始化话，没有的话就先进行父类的初始化操作
  - 使用反射API进行调用时，如果类没有进行过初始化操作就先进行初始化操作。
  - 第一次调用java.lang.invoke.MethodHandle实例时，需要初始化MethodHandle指向方法所在的类。

###### 初始化类变量

- 在初始化阶段，仅仅会初始化与类相关的静态变量和静态语句（static修饰的），没有被static修饰的属于实例成员，在创建类的实例的时候才会被初始化。

- 主动引用

- 被动引用

  - 典型案例就是在子类中调用父类的静态变量

    ```java
    public class Parent {
        public static int value = 1;
    
        static {
            System.out.println("parent is init");
        }
    }
    
    public class Child extends Parent{
    
        static {
            System.out.println("child is init");
        }
    }
    
    public class ClassInitMain {
        public static void main(String[] args) {
            //ClassInitTest.value = 3;
    
            //调用了父类的静态变量
            Child.value = 5;
        }
    }
    
    //运行结果
    parent is init
    ```

    - 通过运行结果我们能分析出，Child类并没有被初始化
    - 对于静态字段来说，只有直接定义这个字段的类在调用该字段时，才会被初始化。
    - 上述操作，Child没有被初始化但是进行了装载和链接的操作。

###### class初始化和对象的创建顺序

> 当我我们使用new关键字来创建类的实例时，类中的静态成员，构造器和非静态成员的执行顺序是怎样的呢？

- 模拟代码如下

  ```java
  public class InitOrder {
  
      public static void main(String[] args) {
          Parent p = new Child();
          System.out.println("-------------------");
          p = new Child();
  
      }
  
      static class Parent {
          static {
              System.out.println("parent static init");
          }
  
          {
              System.out.println("parent no-static-init");
          }
  
          public Parent() {
              System.out.println("parent constructor");
          }
      }
  
      static class Child extends Parent {
          static {
              System.out.println("child static init");
          }
  
          {
              System.out.println("child no-static init");
          }
  
          public Child() {
              System.out.println("child constructor");
          }
      }
  }
  ```

- 结果如下

  ```java
  parent static init
  child static init
  parent no-static-init
  parent constructor
  child no-static init
  child constructor
  -------------------
  parent no-static-init
  parent constructor
  child no-static init
  child constructor
  ```

- 通过以上的结果可以得出如下结论
  - 静态变量/代码块 > 普通代码块 > 构造器
    - 父类的静态变量和静态代码块
    - 子类的静态变量和静态代码块
    - 父类的普通代码块
    - 父类的构造器
    - 子类的普通代码块
    - 子类的构造器

