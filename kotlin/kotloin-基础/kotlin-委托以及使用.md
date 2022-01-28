#### Kotlin中的委托

#### 啥是委托

其实kotlin中的委托的理念就是委托模式也是叫做代理模式；在Kotlin中的委托中有两个对象 分别叫做委托对象和被委托对象，他俩呢都参与了同一个请求的处理，只不过这里被委托对象接受了这个请求但是不做处理把这个请求转而交给了委托对象来处理。

委托模式同代理模式，有两个对象 分别叫做被委托对象（具体的实施者），委托对象（接受请求），还有这两个对象的约束类，因为不管委托与被委托实质都有相同的功能，只不过为了委托对象与被委托对象之间的一对多的扩展性，就需要这个约束接口的出现。

下面我就用一张图表明一下这个混沌的描述哈哈！

![Kotlin中的委托模式.png](https://upload-images.jianshu.io/upload_images/4997216-3ef987d5aae2c04f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 现实中的委托

比如我们实现中，最近大家都在玩王者荣耀吧，玩王者荣耀我们通常不是匹配就是排位，这里我们就把约束接口中定义两个方法 macth()和upgrade()；有一天呢，我认识我现在的“大哥”，我大哥呢时不时也去匹配和排位，但是人家玩的好啊，有一天我就厚着我的老脸，就问我的大哥，大哥能不能帮我上上分啊，你看我都给你买了你最喜欢的皮肤，给你配置了最好的游戏手机，来吧，帮我上上分，大哥沉思了片刻，说行，上分没问题小意思；然后大哥就帮助我一步一步走向了人生巅峰哈哈哈！

其实在这个例子中 对应的上图中的角色即使 我们共同的游戏场景就是 匹配和排位 也就是我们的约束接口，而我呢就是委托对象，“大哥”呢就是被委托对象，对就是这样的！

这样吧我就用活生生的代码来看下这个委托模式到底是咋样的

###### 首先我们要定义好我的约束类 IPlayer

```kotlin
interface IPlayer {
    fun match()
    fun upgrade()
}
```

###### 接着定义我们的被委托类

```kotlin
class RealPlayer(var name:String):IPlayer {
    private  val TAG = "RealPlayer"
    override fun match() {
        Log.d(TAG, "$name 开始匹配！");
    }
    override fun upgrade() {
        Log.d(TAG, "$name 开始排位升级啦！ ");
    }
}
```

###### 然后定义委托类

```kotlin
class DelegatePlayer(var player:IPlayer):IPlayer by player 
```

对你没看错，委托类就是这样简单简约，我么你使用了关键字 by 而player就是我们的要委托的对象

这里我们传进了一个约束类型的被委托类，意思也就是一旦“大哥”那天有事不能帮助我上分了，我还能找二哥帮助我一下

###### 最后在我们的代码中跑一下

```kotlin
class DelegateActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_delegate)

        //创建一个被委托对象
        val realPlayer = RealPlayer("大哥")
        //委托对象使用这个被委托对象
        val delegatePlayer = DelegatePlayer(realPlayer)
        delegatePlayer.match()
        delegatePlayer.upgrade()
    }
}
```

执行结果

```xml
2020-12-12 12:19:45.850 9459-9459/com.chiatai.module_kotlin_appoint D/RealPlayer: 大哥 开始匹配！
2020-12-12 12:19:45.850 9459-9459/com.chiatai.module_kotlin_appoint D/RealPlayer: 大哥 开始排位升级啦！
```

其实在上看我们定义委托类的时候使用的委托的关键子by 这种委托在kotlin中也叫做类委托，那么处理类委托之外kotlin中的委托也可用作属性变量上看，下面我们就来看下kotlin中的属性委托

#### 属性委托

比如有这样的业务场景，我们从接口拿到一个商品的价格，显示到我们的UI界面上需要展示单位这时候我们就可以使用属性委托，让我们的委托类去帮助我们加上这个单位

###### 定义商品实体类 Product

```kotlin
class Product {

    // 这是我们使用by关键字将属性price委托给了 PriceDelegate委托类
    var price: String by PriceDelegate()
}
```

###### 定义被委托类

```kotlin
class PriceDelegate {
    private val TAG = "PriceDelegate"

    private var price: String = "0.0"

    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        Log.d(TAG, "设置的属性值为 ${price}: ")
        return "¥${price}"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        Log.d(TAG, "这是设置的值 $value")
        price = value
    }
}
```

当然了如果我们的属性是var修饰 那么可以提供getValue和setValue方法，如果是val修饰的话只能提供getValue()

其实所谓的属性委托即使将属性本身的get()和set()方法委托给了被委托类中getValue()和setValue()方法，这样既不用污染数据类本身，又能去满足我们的业务需求！

属性的委托的语法格式：val/var <属性名>:<数据类型>  by<关键字> <表达式>

上述getValue和setValue()方法中参数解释一波

thisRef：是属性的类型或者是它的超类型

property: 类型必须是KProperty<*>或者它的超类型

value：必须和属性的是同类型或者是它的子类型

###### 代码跑一下

```kotlin

var product = Product()
//这里就是调用到了被委托类中set的方法，设置一个值
product.price = "${20.0}"
// 这里就是调用了被委托类中的get方法 返回一个值
Log.d(TAG, product.price)

// 结果如下
2020-12-12 12:53:33.521 15955-15955/com.chiatai.module_kotlin_appoint D/PriceDelegate: 这是设置的值 20.0
2020-12-12 12:53:33.521 15955-15955/com.chiatai.module_kotlin_appoint D/PriceDelegate: 设置的属性值为 20.0: 
2020-12-12 12:53:33.521 15955-15955/com.chiatai.module_kotlin_appoint D/DelegateActivity: ¥20.0
```

感觉这样好难受啊，每次对属性的委托都要自己写getValue()和setValue()的方法，好麻烦，哈哈Google也觉得是，为此它为我们提供了相应的接口我们只需要关注逻辑就一了，我们走着，看看去！

#### ReadOnlyProperty 和ReadWriteProperty

哈哈，如果当前你的属性是val修饰的，那么你只需要实现ReadOnlyProperty就可以了，

如果是var修饰的，那么实现ReadWriteProperty就可以了，

```kotlin
class MyReadOnlyProperty:ReadOnlyProperty<Any,String> {
    override fun getValue(thisRef: Any, property: KProperty<*>): String {
       return ""
    }
}
```

```kotlin
class MyReadWriteProperty:ReadWriteProperty<Any,String> {
    override fun getValue(thisRef: Any, property: KProperty<*>): String {
        return ""
    }

    override fun setValue(thisRef: Any, property: KProperty<*>, value: String) {

    }
}
```

上述范性约束中，第一个参数是和thisRef一直，第二个是和value一直；

Kotlin中也为我们提供了对应的委托

延迟初始化：其中也使用了by关键字作为lazy属性的委托 ，并且该属性对应的值只会在第一访问的时候才会计算，之后都不会及进行计算了,表达式中最后一行代码的值作为该表达式的返回值

```kotlin
val name:String by lazy{
  Log.d(TAG,"仅仅在首次访问的时候才会执行")
  "dashingqi"
  "zhangqi"
	"dashingqi"
  "java"
}

 Log.d(TAG, "name =  $name")
 Log.d(TAG, "name =  $name")
```

```xml
2020-12-12 13:54:25.035 20887-20887/com.chiatai.module_kotlin_appoint D/DelegateActivity: 仅仅在首次访问的时候才会执行
2020-12-12 13:54:25.035 20887-20887/com.chiatai.module_kotlin_appoint D/DelegateActivity: name =  java
2020-12-12 13:54:25.035 20887-20887/com.chiatai.module_kotlin_appoint D/DelegateActivity: name =  java

```

#### 总结

委托类委托出去的是它的接口实现；委托属性，委托出去的是属性的getter,setter;

val test = by lazy{}

上述就是将text的getter委托给了lazy{}

好啦好啦，以上就是关于Kotlin中委托的相关介绍的，要去干饭啦！

