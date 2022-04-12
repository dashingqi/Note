## kotlin协程-Channel

为什么说Channel是“热”的

#### Channel就是管道

##### 怎么使用

```kotlin
fun main() = runBlocking{
  val channel = Channel<Int>()
  
  launch {
    (1..3).forEach{
      channel.send(it)
      println("Send $it")
    }
  }
  
  launch {
    for(i in channel){
      println("Receive $i")
    }
  }
}
```

上述代码在输出所有结果之后，并不会退出。主程序不会结束，整个程序还处于运行状态；

要解决上述问题，可以加上如下代码

```kotlin
fun main() = runBlocking{
  val channel = Channel<Int>()
  
  launch {
    (1..3).forEach{
      channel.send(it)
      println("Send $it")
    }
    
    channel.close() // 这个就是能保证主程序退出；
  }
  
  launch {
    for(i in channel){
      println("Receive $i")
    }
  }
}
```

channel 是一种协程资源，在用完channel之后，我们需要主动关闭它，不关闭会造成不必要的资源浪费。

##### Channel构造参数分析

```kotlin
public fun <E> Channel(
    capacity: Int = RENDEZVOUS,
    onBufferOverflow: BufferOverflow = BufferOverflow.SUSPEND,
    onUndeliveredElement: ((E) -> Unit)? = null
)
```

###### capacity（管道容量）

- UNLIMITED：代表了无限容量；
- CONFLATED：代表了容量为1，新的数据会替代旧的数据；
- BUFFERED：代表了具有一定的缓存容量，默认情况下是64

###### onBufferOverflow（指定了管道容量，当管道容量满时，Channel的应对策略是怎么样的）

- SUSPEND：当管道容量满后，如果发送方还要继续发送，我们就会挂起当前的send方法。
- DROP_OLDEST：丢弃最旧的那条数据，然后发送新的数据
- DROP_LATEST：丢弃最新的那条数据。注意此时是丢弃当前正准备发送的那条数据

###### onUndeliveredElement

就是一个回调，当Channel发送的数据，接受者没有接收到，就通过这个回调。发送方可以灵活处理未接收到的数据；

##### Channel关闭引发的问题

produce{}高阶函数创建的Channel能自动调用close()方法帮助我们关闭channel

```kotlin
fun main() = runBlocking {
  val channel = produce<Int>{
     (1..3).forEach{
       send(it)
       println("Send $it")
     }
  }
  
  launch {
    for(i in channel) {
      println("Receive $i")
    }
  }
}
```





