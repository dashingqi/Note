#### 变量和函数

##### 变量

- val
  - 对应Java中的非final变量
- var
  - 对应Java中的final变量
  - 解决Java中final关键字没有被合理使用问题
- 自动类型推导
  - val  a = 10 // 自动推导出是Int类型
  - 针对一个延迟赋值的变量，Kotlin无法自动完成类型推导，需要显示的指明变量的类型

##### 函数

- 代码如下

  ```kotlin
  fun largerNumber(a: Int, b: Int): Int {
      return max(a, b);
  }
  ```

  - fun关键字声明函数
  - 函数的参数列表中的参数 声明方式是： 参数名称:参数类型
  - 函数的返回值类型写在 （）之后 : 返回值类型

- 函数的写法简化

  - 当函数中只有一行代码时，Kotlin允许我们不必写函数体

    ```kotlin
    fun largerNumber1(a: Int, b: Int): Int= max(a, b)
    ```

  - 根据自动类型推导，函数的返回值类型可以不用写

    ```kotlin
    fun largerNumber1(a: Int, b: Int)= max(a, b)
    ```

#### 程序的逻辑控制

- 顺序语句
- 条件语句
- 循环语句

##### 条件语句

###### if条件语句

- Java中的if和Kotlin中的几乎没有什么使用上区别

- Kotlin中的if有一个额外的功能

  - 是可以有一个返回值的

  - 返回值就是每个条件中最后一行代码的返回值

    ```kotlin
    fun largerNumber(a:Int,b:Int) = if(a>b) a else b;
    //这种写法是最简的写法
    ```

    

###### when条件语句

- kotlin中的when语句相当于Java中的switch语句

- 说下Java中的switch语句

  - 只能传入整型或者短于整型的变量作为条件
  - JDK1.7之后，添加了字符串变量的支持
  - 判断每个case 都需要使用break；有时会忘记

- Kotlin中的when语句解决了上述Java中switch的痛点还有更为强大的特性

- 比如说输入一个人的名字，查找他的成绩(精确匹配)

  ```kotlin
  fun getScore(name:String)= when(name){
    "Tom" -> 67
    "Jim" -> 56
    "Jack" -> 89
    else -> 90
  }
  ```

  - 结构清晰
  - when和if一样都是可以有返回值的

- when的类型匹配 和 is 关键字配合使用

  -  is关键字相当于Java中的 instanceof关键字

    ```kotlin
    fun checkNumber(number:Number){
    	when(number){
        is Int -> println("is int")
        is Double -> println("is double")
        else -> println("no matcher")
      }
    }
    ```

    - Number是Kotlin中的内置的一个抽象类，Int，Double，等都是它的实现子类

- when不带参数的使用

  - 比如名字开头以Tom开始的 成绩为67

    ```kotlin
    fun getScore(name:String)= when{
      name.startWith("Tom") -> 67
      name == "Jim" -> 56
      name == "Jack" -> 89
      else -> 90
    }
    ```

    - Kotlin中判断字符串或者对象是否相等可以直接使用 == 关键字

##### 循环语句

> 其中while循环和Java中的一模一样，就不做过多的介绍了

###### for循环语句

- Java中的for-i在kotlin中被舍弃了

- Java中的for-each在kotlin中被增强了，被for-in取代

- 介绍下for-in

  - 介绍之前介绍一下区间的概念，在Java中是没有的

    - val range = 0 .. 10 // 相当于两端的闭区间 	[0,10] 包括0到10之间的数字
    - 区间的声明我们看到是使用 .. 

  - 使用for-in 遍历区间

    ```kotlin
    fun main(){
    	for(value in 0 .. 10){
        println("value = $value")
      }
    }
    //打印了0到10
    ```

- until关键字创建一个左闭右开的区间

  ```kotlin
  fun main(){
    for(value in 0 until 10){
      println("value == $value")
    }
  }
  //打印了 0到9
  ```

- step关键字来跳过其中元素

  - 正常我们的for-in都是默认加1 ，

  - step可以指定 加多少

    ```kotlin
    fun main(){
      for(value in 0 until 10 step 2){
        println("value == $value")
      }
    }
    //打印了 0 2 4 6 8
    ```

- 使用downTo关键字来进行逆序遍历

  - 正常我们都是从左往右遍历遍历打印（从小到大）

  - downTo可以指明从大到小去打印

    ```kotlin
    fun main(){
      for(value in 10 downTo 0 step 2){
        println("value == $value")
      }
    }		
    // 打印 	10 8 6 4 2 0
    //downTo关键字也是一个 双端闭区间的 [10,0]
    ```

#### 类与对象

###### 类

- kotlin中使用class关键字声明一个类

  ```kotlin
  class Person(
      //只读属性，生成一个字段和一个简单的getter
      var name: String,
      //可写属性，生成一个字段、一个getter和一个setter
      var age: Int
  )
  ```

- Koltin中实例化一个类

  ```
  val person = Person();
  //与Java的区别在于 取消了使用new关键字
  ```

###### 对象

- 实例化Person类后赋值给了person变量
- 该person成为Person类的一个实例，也是一个对象。

##### 继承与构造函数

###### 继承

- 在Koltin中任何一个非抽象的类默认是不能被继承的，如果想要被继承，需要在类声明的时候使用open关键字

  ```kotlin
  open class Person{
  	var name = ""
    var age = 0
  }
  ```

- 当一个Studen类要继承Person类，我们使用冒号表明这个继承关系

  ```kotlin
  class Studen : Person(){
  	var sno = ""
  	var grade = 0
  }
  ```

###### 构造函数

- 主构造函数

  - 是最常用的构造函数

  - 每个类默认都会有一个不带参数的主构造函数，也可以显示给他指明参数

  - 它的特点是没有函数体

    ```kotlin
    class Studen(val son:String,val grade:Int) : Person(){
      //由于主构造函数没有函数体，我们想要在主构造函数中操作一些事情可以使用 init结构体
      init{
        println("son = $son")
      }
    }
    //实例化的时候必须传入参数
    val studen = Studen("123",56)
    ```

  - 我们知道Java中子类的构造函数必须要调用父类的构造函数，Kotlin也是如此

    - 所以为了优雅的实现这一点，在继承父类的时候，在父类的后面加上了()

  - 根据主构造函数的特点我们改造下Person

    ```kotlin
    open class Person(val name:String,val age : Int){
    }
    //此时子类继承的时候
    class Studen(val son:String,val grade:Int,name:String,age:Int) : Person(name,age){
      //由于主构造函数没有函数体，我们想要在主构造函数中操作一些事情可以使用 init结构体
      init{
        println("son = $son")
      }
    }
    ```

    - 在子类的主构造函数中声明了两个变量name和age
    - 这两个变量没有使用var和val来修饰，如果使用var和val修饰，那么该变量就属于当前类，会造成与父类同名字段就发生冲突了。

- 次构造函数

  - 次构造函数使用constructor关键字来定义的

  - Kotlin中规定，一个类既有主构造函数也有次构造函数时，次必须要调用主构造函数

    ```kotlin
    class Studen(val son:String,val grade:Int,name:String,age:Int) : Person(name,age){
      //由于主构造函数没有函数体，我们想要在主构造函数中操作一些事情可以使用 init结构体
      init{
        println("son = $son")
      }                constructor(name:String,age:Int):this("",0,name ,age)
    
      constructor():this("",0)
    }
    //当一个类中没有主构造函数，就得调用父类的的构造函数。使用super关键
    class Student:Person{
      constructor(name:String,age:Int):super(name,age)
    }
    ```

#### 接口

- Kotlin中接口类的声明

  ```kotlin
  interface Study{
  	fun readBooks()
  	fun doHomeWork()
  }
  ```

- 接口的实现 使用 ：

  ```kotlin
  class Student(name: String, age: Int) : Person(name, age), Study {
      override fun readBooks() {
          println("name is ${name}")
      }
  
      /**
       * 在接口Study中，doHomework函数有默认的实现，
       * 这里可以不用强制重写 doHomework方法
       */
      override fun doHomework() {
          println("age is ${age}")
      }
  }
  ```

- 接口中的函数进行默认实现

  ```kotlin
  interface Study {
      fun readBooks()
      //如果接口中一个函数拥有了函数体，这个函数中的内容就是它的默认实现。
      fun doHomework() {
          println("do homework default implementation")
      }
  }
  
  //在Student类中，可以不用实现doHomewok()方法
  class Student(name: String, age: Int) : Person(name, age), Study {
      override fun readBooks() {
          println("name is ${name}")
      }
  }
  ```

#### 数据类

- 数据类使用data关键字

- 会根据主构造函数中参数，来帮你声明toString()、equals()、hashCode()方法，减少了开发工作量

  ```kotlin
  data class CellPhone(val brand:String,val price:Double)
  ```

#### 单例类

- kotlin中的单例类 时将class关键字替换成object

  ```kotlin
  object Singleton{
  	fun method(){
  		println("called")
  	}
  }
  
  // 调用
  fun main(){
    SingleTon.method()
    //类似于Java中的静态方法调用，
    //Kotlin在背后帮我们自动创建老人一个SingleTon的实例类，并且保证了全局只会存放唯一一个SingleTon的实例
  }
  
  ```

#### Lambda

##### 集合创建与遍历

###### List

- 代码如下

  ```kotlin
  val list = ArrayList<String>();
  //创建不可变集合
  val list = listOf<String>("apple", "banana", "orange")
  //创建可变集合
  val list = mutableListOf<String>("apple", "banana", "orange")
  
  //使用for-in循环遍历
  for(fruit in list){
    println("fruit = $fruit")
  }
  ```

###### Set

- Set和List一致  提供了setOf()和mutableSetOf()两个方法

##### Map

- 代码如下

  ```kotlin
   val map = HashMap<String, Int>()
      map.put("apple", 1)
      map.put("orange", 2)
      map.put("banana", 3)
      //Kotlin中不建议使用put和get方法来对Map进行添加和读取的操作，而是推荐使用一种类似于数组下标的语法
      map["heihie"] = 5
      //从map中读取一条数据
      val number = map["apple"]
      //上述代码经过优化可以写成如下
      val map1 = HashMap<String, Int>()
      map1["apple"] = 1
      map1["orange"] = 2
      map1["banana"] = 3
  
      // 上述还不是最简便的写法 还是提供了mapOf()和mutableMapOf()方法来继续简化Map的用法
      //使用mapOf
      val map2 = mapOf<String, Int>("apple" to 1, "orange" to 2, "banana" to 3)
      //我们使用for-in来遍历去除map2中的数据
      for ((fruit, number) in map2) {
          println("fruit is ${fruit},number is ${number}")
      }
  ```

#### 集合的函数式API

##### maxBy()

- 它是一个普通的函数而已，接收了一个时Lambda类型的参数

- 求集合中最长的字段

  ```kotlin
  val list = listOf<String>("apple", "banana", "orange")
  val lambda = {first:String -> fruit.lenght}
  val maxLengFruit = list.maxBy(lambda);
  //开启简化写法的构造流程
  val maxLengFruit = list.maxBy({first:String -> fruit.lenght});
  //Kotlin规定，当Lambda参数是函数的最后一个参数的时候，可以将Lambda表达式转移到函数括号之外
  val maxLengFruit = list.maxBy(){first:String -> fruit.lenght};
  //如果Lambda表达式是函数的唯一参数的时候嘛，可以把函数的的括号省略了
  val maxLengFruit = list.maxBy{first:String -> fruit.lenght};
  //kotlin拥有自动类型推导的功能，可以省略类型的声明
  val maxLengFruit = list.maxBy{first -> fruit.lenght};
  //当Lambda表达式参数列表中只有一个参数的时候，不必声明参数名，而是用it关键字来代替
  val maxLengFruit = list.maxBy{it.lenght};
  ```

##### map函数

- 将集合中的每个元素映射成一个另外的值，这个映射规则在lambda中声明，最终生成一个新的集合

  ```kotlin
  val newList = fruitList.map { it.toUpperCase() }
      for (fruit in newList) {
          println("fruit is ${fruit}")
      }
  ```

##### filter函数

- 是用来过滤集合中的数据，可以单独使用，也可以配合map函数使用

  ```kotlin
  val newLists = fruitList.filter { it.length >= 5 }
          .map { it.toUpperCase() }
  
      for (fruit in newLists) {
          println(fruit)
      }
  ```

###### any和all函数

- 代码如下

  ```kotlin
      //any和all函数
      //其中any函数是用来判断集合是否至少存在一个元素满足指定条件。
      // all函数是用来判断集合中是否所有元素都满足指定条件
  
      //是否存在长度大于5的
      val anyResult = fruitList.any { it.length >= 5 }
      //是否所有的长度都大于5
      val allResult = fruitList.all { it.length >= 5 }
      println("anyResult is ${anyResult}, allResult is ${allResult}")
  ```

#### Java函数式API的使用

#### 空指针检查

###### 可空类型系统

- Kotlin将空指针异常的检查提前到了编译时期

- 所谓的可空类型系统就是在类名的后面加上一个问号 ？

  ```kotlin
  fun main(){
    //可以传递null 如果不加？ 传递null就会提醒错误
  	doStudy(null);
  }
  fun doStudy(study:Study?){
    //处理下空指针异常
    if(study!=null){
    	study.readBooks();
    	study.doHomeWork();
    }
  }
  ```

######判空辅助工具

- ?.
- ?:
- let

#### 字符串内嵌表达式

- Kotlin允许我们在字符串中嵌入${}这种语法结构的表达式

  ```kotlin
  "hello ${obj.name},nice to meet you "
  ```

- 当表达式中仅有一个变量的时候，可以将两边的大括号省略

  ```kotlin
  "hello $name. nice to meet you"
  ```

  

#### 函数的参数默认值



###### 