#### Kotlin-1.6

#### Exhaustive When

不要漏掉任何一个分支

```kotlin
sealed class Contact {
    data class PhoneCall(val number: String) : Contact()
    data class TextMessage(val number: String) : Contact()
}

fun sendMessage(contact: Contact, message: String) {

    when (message.isEmpty()) {
        true -> return
    }

    when (contact) {
        is Contact.PhoneCall -> {}
    }
}
```

在kotlin1.6之前，对于上述密封类来说，使用when表达式可以漏掉其他分支

在Kotlin1.6时会提示你，让你补充其他的情况

![image-20220108180954566](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20220108180954566.png)

在kotlin1.7时直接编译器报错

![image-20220108181306888](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20220108181306888.png)

让编译器激进点，哈哈哈哈

这样就能体验最新的检查

```groovy
freeCompilerArgs += '-progressive'
```

![image-20220108181427708](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20220108181427708.png)
