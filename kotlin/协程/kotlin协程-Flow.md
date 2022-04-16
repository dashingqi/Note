## Kotlin协程-Flow

#### 使用

###### 基本使用

**flow{}创建Flow**

```kotlin
flow<Int> {
        emit(1)
        emit(2)
        emit(3)
        emit(4)
        emit(5)
    }.filter { it > 2 }
        .map { it * 2 }
        .take(2)
        .collect {
            logX("Result is $it")
        }
```

**flowof创建Flow**

```kotlin
flowOf(1, 2, 3, 4, 5, 6, 7, 8, 9)
        .filter { it > 5 }
        .map { it * 3 }
        .take(3)
        .collect {
            logX("Result is $it")
        }
```

###### Flow转List

```kotlin
flow {
        emit(1)
    }.toList()
        .filter { it > 0 }
        .map { it * 4 }
        .take(2)
        .forEach {
            logX("Result is $it")
        }
```

###### List转Flow

```kotlin
listOf(1, 2, 3, 4, 5)
        .asFlow()
        .filter { it > 2 }
        .map { it * 2 }
        .take(2)
        .collect {
            logX("Result is $it")
        }
```

