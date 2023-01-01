## 数据双向绑定

#### 数据刷新视图 

> 如果数据发生变化，视图也跟着变化那么可以使用如下两种方法

###### BaseObservable

- 提供的刷新UI方法
  - notifyPropertyChanged():刷新属于它的UI
  - notifyChange()：刷新所有UI

- 局限性
  - 要求子类继承BaseObservable
    - 我们的实体类已经继承了其他父类，就不能在继承BaseObservable了。
  - 继承之后必须要使用@Bindable注解和notifyPropertyChanged()方法
    - 我们的字段如果很多的话，我们在每个get方法都加上@Bindable注解在set方法内部加入notifyPropertyChanged()方法。

###### ObservableField

- 针对基本数据类型提供了专门的包装类
  - ObservableBoolean
  - ObservableByte
  - ObservableChar
  - ObservableDouble
  - ObservableFloat
  - ObservableInt
  - ObservableLong
  - ObservableShor
- 针对集合提供了专门包装类
  - ObservableArrayList
    - 可以用于当数据发生变化的时候，通知到取刷新适配器（在没有使用ItemBinding的时候）
  - ObservableMap
- 针对实现了Parcelable接口的对象提供的包装类
  - ObservableParcelable
- 针对其他类型提供的包装类
  - ObservableField。

#### 视图刷新数据

###### @=的使用

> 同样视图数据的更新，通知到本地的数据，本地的数据更新了，要通知到视图上。
>
> 比如EditText输入内容更新TextView的显示

