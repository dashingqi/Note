#### 密封接口 （Sealed Interface）

Sealed Interface和Sealed Classes一样都是用于表示受限制的类层次结构；

不同在于Sealed Classes被限制在单一的继承关系中，但是Sealed Interface更加灵活支持多实现；

```kotlin
sealed interface IArea {
    abstract fun area(): Double
}

sealed interface IColor

sealed interface IFigure

// 支持多实现 密封接口
class Round : IArea, IColor, IFigure {
    override fun area(): Double = 0.0
}
```



