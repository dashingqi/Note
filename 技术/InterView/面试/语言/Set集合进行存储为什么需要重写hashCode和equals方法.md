#### 为什么需要重写hashCode和equals方法

###### 测试代码

- main方法

  ```kotlin
  fun main() {
      val student1 = Student("zhangqi", 16, "1")
      val student2 = Student("zhangqi", 16, "1")
      val student3 = Student("zhanglu", 20, "3")
      val studentList = mutableListOf<Student>()
      studentList.apply {
          add(student1)
          add(student2)
          add(student3)
      }
  
      val studentSet = HashSet<Student>()
      for (student in studentList) {
          studentSet.add(student)
      }
      println("studentList Size = ${studentSet.size}")
  }
  ```

- Student类

  ```kotlin
  /**
   * data class 数据类本身重写了equals和hashCode方法
   */
   class Student(val name: String, val age: Int, val classID: String) {
      override fun equals(other: Any?): Boolean {
          if (this === other) return true
          if (javaClass != other?.javaClass) return false
  
          other as Student
  
          if (name != other.name) return false
          if (age != other.age) return false
          if (classID != other.classID) return false
  
          return true
      }
  
      // hashCode当添加对象的数量比较多的情况下，具有一定hash冲突碰撞的概率，需要重写equals方法进行补充；
      override fun hashCode(): Int {
          var result = name.hashCode()
          result = 31 * result + age
          result = 31 * result + classID.hashCode()
          return result
      }
  }
  ```

- 测试结果

  - 当Student类重写hashCode和equals方法

    <img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230130075127742.png" alt="image-20230130075127742" style="zoom:150%;" />

  - 当Student类不重写hashCode和equals方法

    <img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230130075332511.png" alt="image-20230130075332511" style="zoom:150%;" />

#### 分析一下

HashSet的add()方法

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230130075650856.png" alt="image-20230130075650856" style="zoom:150%;" />

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230130075848425.png" alt="image-20230130075848425" style="zoom: 200%;" />

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230130080015660.png" alt="image-20230130080015660" style="zoom:200%;" />

![image-20230130080411446](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230130080411446.png)