## 函数和变量

#### Hello World

- 代码如下

  ```kotlin
  fun main(args: Array<String>) {
      println("hello world")
  
  }
  ```

  - 关键字fun用来声明一个函数。
  - 参数的类型是卸载它的名称后面的。
  - 函数的定义可以不用放在类中，可以单独存在
  - println代替了System.out.println()方法。
  - 可以省略每行代码结尾的分号。

#### 函数

###### 带有返回值的函数

- 代码如下

  ```kotlin
  fun max(a: Int, b: Int): Int {
      return if (a > b) a else b;
  }
  ```

  - kotlin中if是有结果值的表达式。类似于三元运算符。

###### 表达式函数体

- 如上函数的函数体是一个表达式构成的，可以简化成如下代码

  ```kotlin
  //简化函数体是一个表达式的函数
  fun max1(a: Int, b: Int): Int = if (a > b) a else b;
  ```

###### 类型推导

- 每个变量和表达式都有类型，每个函数都有返回类型。

- 对与表达式体函数来说，编译器会分析作为函数体的表达式，把它的类型作为函数的返回类型。即使没有显示写出来。

- 针对上面的表达式函数体可以做进一步写法上的简化

  ```kotlin
  //类型推导
  fun max2(a: Int, b: Int) = if (a > b) a else b;
  ```

- 注意

  - 只有表达式体函数的返回类型可以省略。
  - 对于有返回值的代码块体函数，必须显式地写出返回值类型和return语句。

#### 变量

- Kotlin中以关键字开始，然后是变量名称，最后加上类型。（有时这个类型不加也可以）

  ```kotlin
  val question = "hahahahha";
  val answer = 42;
  
  // 显示指定变量的类型
  val answers: Int = 45;
  ```

- 和表达式体函数有异曲同工之处，如果你不指定变量的类型，编译器会分析初始化器表达式的值，把它的类型作为变量的类型。

- 如果没有指定初始值，就需要显示的指定类型

  - 如果不提供赋给这个变量值的信息，编译器就无法推断出它的类型

  ```kotlin
  val a: Int
  a = 43       
  ```

###### 可变变量和不可变量

- 声明变量的关键字有两个

  - val

    - 不可变引用：val声明的变量不能在初始化之后再次赋值

    - 对应的是Java的final变量

    - 定义了val变量的代码块执行期间，val变量只能进行唯一一次初始化。

    - 如果编译器确保只有唯一一条初始化语句会被执行，可以根据条件使用不同的值来初始化它。

    - 尽管val引用自身是不可变的，但是它指向的对象可以是变的

      ```kotlin
      //声明不可变的引用  val引用自身是不可变的，但是它指向的对象是可变的。
      val language = arrayListOf("Java")
      //改变引用执行的对象
      language.add("Kotlin")
      ```

  - var

    - 可变引用：这种变量的值可以被改变
    - 对应的是Java中普通（非fianl）变量

######  字符串模板

- Kotlin可以在字符串字面值中引用局部变量，只需要在变量名称前面加上$

  - 如上描述等价于Java中的字符串连接

- 当在字符串中使用$,需要进行转义

  ```kotlin
  fun main(args: Array<String>) {
      println("\$x")
  		//打印的是 $x
  }
  ```

- 引用复杂的表达式

  ```kotlin
  fun main(args: Array<String>) {
      //引用表达式
      println("${args[0]}")
  }
  ```

- 可以在双引号中直接嵌套双引号，只要它们处于某个表达式的范围内

  ```kotlin
  fun main(args: Array<String>) {
      println("Hello,${if (args.isNotEmpty()) args[0] else "zhangqi"}")
  }
  
  ```

  