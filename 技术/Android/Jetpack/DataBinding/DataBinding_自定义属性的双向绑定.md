#### 简介

在使用Data Binding的时候我们知道数据驱动UI的显示，这种单向的数据绑定也是我们使用它用的最多的地方，既然有单向的数据绑定应该会存在双向绑定？

不错Android官方确实为我们提供了相应的双向绑定的属性。

比如EditText和CheckBox中

```xml
<EditText
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          // 重点在于这个 =
          android:text="@={viewModel.etText}"
          app:layout_constraintLeft_toLeftOf="parent"
          app:layout_constraintTop_toTopOf="parent" />

<CheckBox
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:checked="@={viewModel.checkBoxStatus}"
          app:layout_constraintLeft_toLeftOf="parent"
          app:layout_constraintTop_toTopOf="parent" />
```

所谓的双向绑定就是数据能驱动UI的显示，UI的状态变换也能改变绑定的属性值；

针对上述的EditText和CheckBox，它的双向绑定是系统帮助我们处理好了，双向绑定的写法也很简单 “@={属性值}”

接着就我们就自定义一个双向绑定的属性，在实现前我们看下最终的效果图

![inverse.gif](https://upload-images.jianshu.io/upload_images/4997216-c120fbf1a52f7ec6.gif?imageMogr2/auto-orient/strip)

看了效果，你想用RecylerView来实现？可以，不过要是使用双向数据绑定可以很简单的。

#### 实现步骤

DataBinding为我们实现了双向数据绑定提供了@InverseBindingAdapter注解；

###### 步骤一

首先我们需要一个数据源用来设置控件上布局的数据，我们使用@BindingAdapter自定义一个属性 data用来接受数据源

```kotlin
 /**
 * 设置数据源
 */
 @JvmStatic
 @BindingAdapter(value = ["data"], requireAll = true)
 fun setData(inverseGroupView: InverseGroupView, data: List<String>?) {
   data?.let {
   inverseGroupView.setData(it)
   }
 }
```

###### 步骤二

然后我们需要定义双向绑定的属性 index 同样使用 @BindingAdapter

```kotlin
/**
* 设置角标
* 当数据发生边改的时候，会调用该方法设置数据 更新UI
*/
@JvmStatic
@BindingAdapter(value = ["index"], requireAll = true)
fun setIndex(inverseGroupView: InverseGroupView, index: Int) {
    inverseGroupView.selectIndex = index
    inverseGroupView.refreshSelectedIndex(index)
}
```

###### 步骤三

接着我们使用@InverseBindingAdapter注解，来将我们因UI状态改变而导致属性值改变同步给自定义属性index（也就是从View中读取到值）

在使用@InverseBindingAdapter注解使，内部有两个属性，attribute：对应着自定义属性，event：是一个事件名称改属性在下面我会详细说明一下

```kotlin
@JvmStatic
@InverseBindingAdapter(attribute = "index", event = "indexChange")
fun getIndex(inverseGroupView: InverseGroupView) = inverseGroupView.selectIndex
```

到这里总结一下哈：当我们的index值发生改变的情况会调用步骤二中方法，通知UI布局发生变换

当我们的UI布局状态发生改变的情况下我们可以调用步骤三方法通知给绑定的属性值，让它重新设置值。

但是有一个问题就是步骤三的方法不知道UI的状态发生变化的时机此时就需要我们步骤4的操作了

###### 步骤四

我们使用@BindingAdapter注解实现了一个View的状态值发生变化的事件通知，

注解中的value值要和@InverseBindingAdapter中的event中的值要保持一致，这样当View的状态值发生变化后会通知步骤三种的方法拿到值设置给绑定的属性值。

```kotlin
/**
* 双向数据绑定的
* InverseBindingListener 是一个监听器，用来处理属性改变时的通知
* 在这里我们给View设置了点击事件，当属性发生改变它会回调 onChange方法告诉DataBinding 去 @InverseBindingAdapter修饰的方法中取到值 然后设置给绑定的变量
*/
@JvmStatic
@BindingAdapter("indexChange")
fun setIndexChangeListener(
  inverseGroupView: InverseGroupView,
  changeListener: InverseBindingListener?
) {
      if (changeListener != null) {
        inverseGroupView.onSelectChangeListener = {
          changeListener.onChange()
        }
      } else {
        inverseGroupView.onSelectChangeListener = null
      }
}
```

到这里一个简单的自定义属性的双向绑定就完成了，这里我贴一下当时写的源码

```kotlin
/**
 * @author : zhangqi
 * @time : 12/7/20
 * desc : 使用DataBinding来自定义属性的双向绑定
 */
class InverseGroupView : LinearLayout {

    constructor(context: Context) : super(context)

    constructor(context: Context, attributeSet: AttributeSet) : super(context, attributeSet)

    constructor(context: Context, attributeSet: AttributeSet, defStyle: Int) : super(
        context,
        attributeSet,
        defStyle
    )
    /**
     * 当前选中的index
     */
    var selectIndex: Int = 0
    /**
     * tag点击事件的回调事件
     */
    var onSelectChangeListener: ((Int) -> Unit)? = null
    /**
     * 用于收集回收可复用的View
     */
    var recyclerView = ArrayList<View>()
    /**
     * 用于存储拿到的数据
     */
    var mData: List<Any>? = null
    /**
     * 设置数据
     */
    fun <T : Any> setData(data: List<T>) {
        updateViewData(data)
    }
    /**
     * 更新布局的数据
     * 创建布局，将数据设置到布局上
     * data:新的数据源
     */
    private fun <T : Any> updateViewData(data: List<T>) {
        mData = data
        // 每次执行到这个方法时，需要回收一下，移除一下，因为接下来是要重新绑定数据的，
        recyclerViewMethod()
        /**
         * 遍历循环数据源，将数据绑定帮控件上
         */
        data.forEachIndexed { index, any ->
            val tagView = getReuseView()
            val tvTagView = tagView.findViewById<TextView>(R.id.tag)
            tagView.isSelected = index == selectIndex
            tvTagView.text = any as String
            //设置一下点击事件
            tagView.setOnClickListener {
                //要更新下布局上按钮的状态
                refreshSelectedIndex(index)
            }
            // 将View添加到父布局中
            addView(tagView)
        }
    }
    /**
     * 刷新下选中的子View
     */
    private fun refreshSelectedIndex(clickIndex: Int) {
        selectIndex = clickIndex
        for (i in 0 until childCount) {
            getChildAt(i).isSelected = i == clickIndex
        }
        onSelectChangeListener?.invoke(clickIndex)
    }
    /**
     * 获取到布局View对象
     */
    private fun newView(): View {
        return LayoutInflater.from(context).inflate(R.layout.item_tag, null, false)
    }
    /**
     * 获取到布局文件
     * 回收池中有 就拿第一个，
     * 没有的话就重新常见一个View对象
     */
    private fun getReuseView(): View {
        return if (recyclerView.isNotEmpty() && recyclerView.size > 0) {
            recyclerView.removeAt(0)
        } else {
            newView()
        }
    }
    /**
     * 首先将目前布局上已经有的子View存储到复用池中
     * 然后将这些子View从布局上移除
     */
    private fun recyclerViewMethod() {
        for (i in 0 until childCount) {
            recyclerView.add(getChildAt(i))
        }
        removeAllViews()
    }
    companion object {
        /**
         * 设置数据源
         */
        @JvmStatic
        @BindingAdapter(value = ["data"], requireAll = true)
        fun setData(inverseGroupView: InverseGroupView, data: List<String>?) {
            data?.let {
                inverseGroupView.setData(it)
            }
        }
        /**
         * 设置角标
         * 当数据发生边改的时候，会调用该方法设置数据 更新UI
         */
        @JvmStatic
        @BindingAdapter(value = ["index"], requireAll = true)
        fun setIndex(inverseGroupView: InverseGroupView, index: Int) {
            if (inverseGroupView.selectIndex == index) return
            inverseGroupView.selectIndex = index
            inverseGroupView.refreshSelectedIndex(index)
        }
        /**
         * 获取到当前选中的角标
         * event：数据改变的事件
         *
         * 当View的状态发生改变的时候（包括数据的填充，bg的改变），会调用该方法来获取到值
         */
        @JvmStatic
        @InverseBindingAdapter(attribute = "index", event = "indexChange")
        fun getIndex(inverseGroupView: InverseGroupView) = inverseGroupView.selectIndex
        /**
         * 双向数据绑定的
         * InverseBindingListener 是一个监听器，用来处理属性改变时的通知
         * 在这里我们给View设置了点击事件，当属性发生改变它会回调 onChange方法告诉DataBinding 去 @InverseBindingAdapter修饰的方法中取到值 然后设置给绑定的变量
         */
        @JvmStatic
        @BindingAdapter("indexChange")
        fun setIndexChangeListener(
            inverseGroupView: InverseGroupView,
            changeListener: InverseBindingListener?
        ) {
            if (changeListener != null) {
                inverseGroupView.onSelectChangeListener = {
                    changeListener.onChange()
                }
            } else {
                inverseGroupView.onSelectChangeListener = null
            }
        }
    }

}
```

#### 注意点

由于当时我绑定的属性值使用的是LiveData，当我改变了View的状态值是会通知到@InverseBindingAdapter注解修饰的方法让它拿到值设置给绑定的属性值。

由于LiveData天生就有可观察性，当观察到数据源发生变化又会驱动UI状态值发生变化，这样UI发生变化又会被监听到 又去通知@InverseBindingAdapter注解修饰的方法让它拿到值设置给绑定的属性值。

这样就会陷入到无限的循环中，所以我当时的做法就是在绑定的属性值的 setter方法中做了新旧值的判断，如果值一致就不触发UI状态值的更新了

```kotlin
@JvmStatic
@BindingAdapter(value = ["index"], requireAll = true)
fun setIndex(inverseGroupView: InverseGroupView, index: Int) {
    if (inverseGroupView.selectIndex == index) return
    inverseGroupView.selectIndex = index
    inverseGroupView.refreshSelectedIndex(index)
}
```

[本文中完整的源码](https://github.com/dashingqi/KotlinAndroid/blob/master/module-databinding/src/main/java/com/chiatai/module/databinding/inverse/InverseGroupView.kt)

虽然DataBinding在报错的时候，错误查找起来不是很友好，但是作为AAC架构的基础，给我们带来很多方便之处，比如利用这种思想的开源库ItemBinding

就给我在日常开发中有很大的效率提高；这些好用的地方完全胜过它的一些小缺点。