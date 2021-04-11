#### 两者结合Java来做对比

#### !!.

- 代码如下

  ```java
  a!!.foo()
  //java中
  if(a!=null){
    a.foo()
  }else{
    throw new KotlinNullPointException()
  }
  ```

#### ?.

- 代码如下

  ```java
  a?.foo()
  //java中
  if(a!=null){
    a.foo()
  }
  ```


####  ?:

- 代码如下

  ```kotlin
  var result:String = str ?: ""
  //java中
  String result = str== null ? "" : str;
  ```

  

