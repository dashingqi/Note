- ViewModelProvider.Factory：工厂类。是用来创建ViewModel的
- mViewModelStore：就是用来存储ViewModel对象的，我们屏幕旋转能恢复数据跟它有关系
- ViewModelStoreOwner：是一个接口，它的实现类 我们熟悉的就是 androidX下的 ComponentActivity和Fragment



- get方法

#### onSaveInstanceState和ViewModel的区别

###### 保存数据的场景

- onSaveInstanceState
  - 配置项改变
  - 资源紧张的时候
  - 保存的数据有限，适合保存及少量的数据（使用过Bundle）
- ViewModel
  - 配置项改变的时候
  - 可以保存大量的数据，比如RecyclerView的item的数据