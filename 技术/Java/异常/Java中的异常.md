## 本文主要从如下几点学习Java中的异常

- 异常的分类
- 异常分类结构图
- 异常处理的方法

## 异常的分类

> Java库中本身内建了异常，这些类通常以Throwable为顶层类
>
> Throwable又派生成Error和Exception

#### Error（错误）

###### 定义

- Error类以及它的子类的实例，代表了JVM本身的错误。这个错误不能被程序员通过代码处理。

- 典型的就是 OutOfMemoryError

#### Exception

###### 定义

- Exception以及它的子类，代表程序运行时出现的各种不期望发生的。我们可以使用Java异常处理机制进行处理。

#### 非检查异常

- Error和RuntimeException以及它们的子类属于非检查异常；在编译的时候不会提示和发现任何错误，不要求在程序中处理这些异常。
- 对于这类异常通常都是我们写的代码有问题。
- 典型的 NullPointException、ArrayIndexOutOfBoundsException、ClassCastException。

#### 检查异常

- 除了Error和RunTimeException之外的其他异常。要求我们要提前对这类异常做预处理工作。
- 这类异常一般是由程序的运行环境导致的。
- 典型的就是：IOException、FileNotFoundException。

## 异常分类结构图

![Java中的异常分类.png](https://upload-images.jianshu.io/upload_images/4997216-834af8da915549a1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 异常处理方法

#### try ... catch...finally语句块

```java
try{
  // 包含着可能发生遗产的代码
  // 当try中的代码执行完毕，如果没有发生异常就会去执行finally中的代码（如果有的话）以及finally之后的代码
  // 当发生异常的时候，就会去匹配catch块
}catch(Exception exception){
  // catch块可以有多个，在Java7的时候可以在一个catch中声明多个异常
  // 如果try块中发生异常并且在catch中没有匹配到，那么先去执行finally块中的代码，然后再去caller中匹配异常处理器。
  // 如果try块中没有发生异常，catch块就直接被忽略。
  
}finally{
  // finally块通常是可选的不是必须的
  // 无论异常是否发生，以及异常是够被匹配处理，finally快都会被执行
  // finally块中通常做的是一些比如流的关闭。连接关闭的操作。
  //其实只有一种方法是可以让finally块不被处理的，就是使用System.exit()方法。
}
```

#### throws函数声明

```java
//1. 如果一个方法中的代码抛出检查异常，方法本身没有定义处理的代码，那么就得必须保证方法使用 throws关键字声明这些抛出的异常，否则编译不能发通过。
//2. 方法本身不知怎么处理这个异常，让调用者自己处理更好。

public void method() throws IOException {
  
}
```

#### throw异常抛出语句

```java
// 1. 我们可以通过throw语句手动显示的抛出一个异常
// 2. throw语句抛出的异常必须是在方法内的。

public void method(int i){
  if(i == 0){
    throw new IllegalArgumentException("i == 0")
  }
  
}
```

#### 一个不容易被理解的事实

- 在try块中即便有return、break、continue等改变执行流的语句，finally块也会被执行。