#### Gradle DSL

##### Android 工程中的DSL

```groovy
android {
  // ....
}

dependencies {
  // ...
}
```

##### 闭包

Gradle中的DSL是基于Groovy中的闭包实现的

```groovy
def method = {println("Hello Grovvy")}
// 闭包执行
method()

// 带有参数的闭包
def method2 = {it -> println("method2 it is $it")}
method2("dashinbgqi")

def method3 = {name -> println("method2 it name $name")}
method3("dashinbgqi")

def method4 = {name1,name2 -> println("method2 it name1 $name1 name2 $name2")}
method4("dashinbgqi","heiheha")
```

###### 实现自定义DSL

