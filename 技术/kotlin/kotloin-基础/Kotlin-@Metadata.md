#### @Metadata注解

```java
@Metadata(
   mv = {1, 5, 1},
   k = 1,
   d1 = {"\u0000\u0012\n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0000\n\u0002\u0010\b\n\u0002\b\u0005\b6\u0018\u00002\u00020\u0001B\u000f\b\u0004\u0012\u0006\u0010\u0002\u001a\u00020\u0003¢\u0006\u0002\u0010\u0004R\u001a\u0010\u0002\u001a\u00020\u0003X\u0086\u000e¢\u0006\u000e\n\u0000\u001a\u0004\b\u0005\u0010\u0006\"\u0004\b\u0007\u0010\u0004¨\u0006\b"},
   d2 = {"Lcom/dashingqi/dqkotlin/sealed/SealedColor;", "", "value", "", "(I)V", "getValue", "()I", "setValue", "app_debug"}
)
```



- 该注解会出现在Kotlin编译器生成的任何文件中，可以通过反射的方式获取@Metadata信息；

- @Metadata存储了Kotlin主要的语法信息例如扩展函数、typealias；这些信息都是由kotlinc编辑器以注解的形式存在Java的字节码中；元数据被丢弃掉，运行在JVM上会抛出异常。通过@Metadata注解提供的信息能确定源文件与Java字节码之间的对应关系；

- 虽然`@Metadata`注解本身并不会减少class文件大小，但Kotlin编译器会根据该注解生成对应的元数据文件（通常是一个名为"module-name.kotlin_module"的文件）。这个元数据文件包含了类的元数据信息，它会在运行时被使用。这种方式有助于减少class文件的冗余性，因为多个类可以共享相同的元数据信息，而不必每个class文件都重复存储这些信息。