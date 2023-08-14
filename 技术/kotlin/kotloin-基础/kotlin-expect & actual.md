#### expect

> expect关键字与actual关键字搭配使用，实现了期望/实际的模式；

###### 不同平台的期望

```kotlin
expect class ClassA

expect fun methodA()
```

#### actual

> actual关键字用于提供期望声明的实际实现。实现Kotlin的多平台支持；

###### 不同平台的实际

```kotlin
// androidMain
actual class ClassA{
  
}

actual fun methodA(){
  // android独有实现
}

// iosMain

actual class ClassA{}

actual fun methodA(){
  // ios独有实现
}
```

#### @file:JvmName("")



