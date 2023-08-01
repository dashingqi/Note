## ContentProvider工作原理

#### 描述

ContentProvider是一种内容共享型的组件，通过Binder向其他组件或者应用提供数据的访问接口；

有一个点就是ContentProvider的onCreate()方法要早于Application的onCreate()方法；

