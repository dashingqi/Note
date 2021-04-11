#### 优化

- 如果Item的高度是固定的，可以使用setHasFixedSize(true)来避免requesLayout
- 不要为每一个Item都设置监听事件，我们里面的Item都是经过复用的，大家应该公用一个，根据ID的不同进行不同的操作
- 布局优化：减少布局层级，简化了ItemView
- 数据优化：对于列表数据进行分页拉去，对于拉去的数据进行缓存便于二次快速加载。对于新增的数据或者删除的数据可以使用Diff来进行局部刷新而不是使用notifyDataSetChange()

