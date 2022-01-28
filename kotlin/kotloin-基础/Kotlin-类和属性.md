## 类和属性

- 在Kotlin中，public是默认的可见性，所以你能省略它。

###### 属性

- Java中，字段和它的访问器（getter和setter）的组合常常被叫做属性。

- 在Kotlin中，属性是头等的语言特性，完全替代了字段和访问器方法

  ```kotlin
  class Person(
      //只读属性，生成一个字段和一个简单的getter
      val name: String,
      //可写属性，生成一个字段、一个getter和一个setter
      var age: Int
  )
  ```

  ```kotlin
  //使用Person
  
  //调用构造方法不需要关键字new
  val person = Person("zhangqi", 23);
  //直接访问属性，但是调用的是getter
  println(person.name);
  println(person.age)
  
  ```

###### 自定义访问器

