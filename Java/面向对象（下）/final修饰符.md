## final修饰变量

- final关键字可以修饰类、变量、方法。

#### final修饰符

- final修饰的变量不可被改变，一旦获取了初始值，该final变量的值就不能被重新赋值。
- final在修饰成员变量和修饰局部变量时有一定的不同。

#### final成员变量

> 针对final修饰的成员变量一旦有了初始值，就不可以改变了。

- 类成员变量

  - 当类初始化的时候，系统会为类的变量分配内存，并分配默认值。
  - 类的成员变量可以在声明的时候进行赋值，也可以在静态代码块中进行赋值。

- 实例成员变量

  - 当创建实例的时候，会为实例的成员变量开辟内存空间，然后分配默认值。
  - 实例的成员变量可以在声明的时候指定默认值，也可以在构造器或者代码块中进行赋值。

  ###### 注意

  - 如果在代码块中进行了赋值，就不能在构造方法中进行赋值。不然是错误的代码。
  - 实例成员变量是不能在静态代码块中进行赋值，本身静态代码块是属于类的成员，不能去操作非静态成员。

- 针对成员变量，没有通过任何途径进行赋初始值，那么系统将会默认分配默认值 0、false，null。
- 然后这些变量就失去了存在的意义了。
- 因为就规定，final修饰的成员变量必须由我们手动进行赋值。

###### 与普通成员变量的对比

- final修饰的成员变量必须手动赋初始值，系统不会对final成员进行隐式初始化。

#### final局部变量

> 系统不会对局部变量进行初始化，局部变量必须由我们自己手动进行初始化。
>
> 所以局部变量可以在定义时赋值，也可以不指定默认值

###### 指定默认值

- 如果指定了默认值，在后面的代码中不能对该变量进行赋值操作

###### 没有指定默认值

- 如果没有指定默认值，可以在后 面的代码中对final的变量赋初始值，有且只有一次。

#### final修饰基本类型变量和引用类型变量的区别

###### 基本类型变量

- 如果对基本类型的变量赋初始值了，后面就不能对其进行修改。

###### 引用类型变量

- 对于引用类型变量来说，它仅仅保存的是引用，
- final修饰的话保证引用类型变量所引用的地址不变即可，也就是引用同一个对象。（也即是引用类型变量不能重新被赋值）
- 但是这个对象可以完全发生改变。

#### 可执行“宏替换”的final变量

###### 宏变量

- 针对类变量，实例变量还是局部变量，只要满足如下三个条件，就称之为宏变量
  - final修饰
  - 在定义final变量时指定了初始值
  - 该初始值可以在编译时就被确认下来。

###### 重要用途之一

- final修饰符的重要用途之一就是“宏变量”。当满足如上三个条件的时候，那么本质上就是一个宏变量。

- 编译器会把程序中所有用到该变量的地方直接替换成该变量的值。

  ```java
  public class FinalLocalTest{
    public static void main(String[] args){
      final int a = 9;
      System.out.println(a);
    }
  }
  // 上面这段代码，其实变量a根本不存在，当程序执行到打印语句的时候，实际是吧 a变量对应的值替换成了a
  ```

- 除了赋直接量的情况下，赋的表达式只是基本算术表达式或者字符串连接运算，没有访问普通变量，调用方法，同样会将这种fina变量当成宏变量处理。

  ```java
  public class Test {
  
      public static void main(String[] args) {
  
          final String name = "zhangqi";
          final String name1 = "zhang" + "qi";
        	// name2 因为涉及到String中方法的调用，所以name2就不是宏变量
          final String name2 = "zhang" + new String("qi");
  
          // true
          System.out.println(name1 == "zhangqi");
          // false
          System.out.println(name2 == "zhangqi");
      }
  
  }
  ```

  ###### 加深印象

  - Java中会使用常量池来管理曾经使用过的字符串变量

  - 比如执行 String a = "abc"后，会在常量池中缓存一个字符串的“abc”

  - 当程序再次出现 String b = "abc",此时会将b直接指向了常量池中的“abc”字符串

  - 因此 a == b true

    ```java
    public class Test2 {
    
        public static void main(String[] args) {
            //把字符串dashignqi存储到常量池中
            String sta = "dashingqi";
            // （在编译阶段就可以确认str1）str1 直接指向常量池中dashingqi的字符串
            String str1 = "dashing" + "qi";
            // true
            System.out.println(sta == str1);
    
             String a = "dashing";
             String b = "qi";
            // 由于a,b是两个普通的变量，所以不会进行宏替换
            String c = a + b;
            // false
            System.out.println(c == sta);
            
            // 当把把a,b 使用final修饰的时候，就可以执行宏替换
            // 也就是 String c = "dashing"+"qi";
        }
    }
    ```

## final方法

- final修饰的方法是不能被子类重写的。

- 在Object类中就有一个final方法：getClass（）；

  ```java
  public class MyFinalMethod {
      public final void methodA() {
          System.out.println("super");
      }
  }
  
  public class MuSuberClass extends MyFinalMethod {
    	// 当你试着重写父类中的 public final修饰的方法的时候，会编译报错的
      public final void methodA(){
  
      }
  }
  ```

###### 针对private修饰的方法

- 在父类中private修饰的方法子类是无法访问到的，所以子类无法重写该方法。
- 但是在子类中可以新建一个和父类private修饰的方法同名，同返回值，同形参的方法，仅仅是子类中的一个新的方法与父类没有任何关系。

## final类

- 被final修饰的类不能被继承，也就是不能有子类。
- 当子类通过继承，可以获取访问父类中某些数据，并且可重写某些方法
- 这样可能导致一些安全因素。
- 所以为了某个类不让继承，可以使用final去修饰。

## 不可变类

#### 定义

- 不可变类：当创建类的实例后，实例的变量是不可改变的。

- Java中提供了8个包装类和String类都是不可变类，当它们创建实例的时候，实例的变量是不可变的。

  ```java
  Double d = new Double(6.5);
  String str = new String("zhangqi");
  // 上述代码创建了Double对象和String对象，并且通过构造器传入了参数
  // 该参数在Double或者String类中，有相应的变量进行存储
  // 但是Double和String并没有提供修改该变量的方法
  // 这种的就叫做不可变类
  ```

#### 创建自定不可变类的规则

- 使用private和final修饰符来修饰该类的变量

- 提供代参数构造器，用于根据传入参数来初始化类的变量。

- 仅仅为该类的变量提供getter方法，不要提供setter方法。

- 如果有必要，重写Object的hashCode和equals方法。

  - equals方法以关键变量作为判断两个对象是够相等的标准。

  - 还应该保证equals方法判断相等的两个对象hashCode也想等。

    ```java
    public class ImmutableStringTest {
        public static void main(String[] args) {
            String str1 = new String("Hello");
            String str2 = new String("Hello");
            // false
            System.out.println(str1 == str2);
            // true
            System.out.println(str1.equals(str2));
            // 69609650====69609650 
            System.out.println(str1.hashCode() + "====" + str2.hashCode());
        }
    }
    ```

  ###### 自定义可变类

  - 代码如下，自定义一个不可变类Name

    ```java
    public class Name {
        //1, private final修饰
        private final String name;
        private final String sex;
    
        public Name() {
            this.name = "";
            this.sex = "";
        }
    
    
        //2。 提供传入参数的构造器，赋给变量
        public Name(String name, String sex) {
            this.name = name;
            this.sex = sex;
        }
    
        // 3。仅仅提供getter方法，不提供setter方法去修改变量
        public String getName() {
            return name;
        }
    
        public String getSex() {
            return sex;
        }
    
        //重写 equals和hashCode方法
    
        @Override
        public boolean equals(Object obj) {
            if (this == obj) {
                return true;
            }
            if (obj != null && obj.getClass() == Name.class) {
                Name name = (Name) obj;
                //当name和sex想等的时候，认为是想等的
                if (this.getName().equals(name.getName()) && this.getSex().equals(name.getSex())) {
                    return true;
                }
    
            }
            return false;
        }
    
        @Override
        public int hashCode() {
            return name.hashCode() + sex.hashCode() * 31;
        }
    }
    
    ```

  - 设置一个不可变类，需要注意引用类型变量，如果引用类型变量的类是可变的，就必须采取必要的措施来保护该变量所引用的对象不会被修改，这样才能创建真正的不可变类。

- 与不可变类对应是可变类

  - 我们日常使用中创建的几乎都是可变类。特别是JavaBean，提供了getter和setter方法
  - 与可变类相比，不可变类的实例在整个程序的生命周期内都是处于初始化状态，不能进行任何修改。

## 本文知识点总结

![final修饰符.png](https://upload-images.jianshu.io/upload_images/4997216-326d544e2d0b9a6e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

