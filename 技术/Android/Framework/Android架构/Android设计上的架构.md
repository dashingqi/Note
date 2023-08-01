## MVC

#### M（Model）

- 模型层对应着业务模型，建立数据结构和相关的类。它主要负责网络的请求，数据库的处理以及IO操作等。

#### V（View）

- 视图层对应着Android中的xml文件以及java动态声明View的部分

#### C（Controller）

- 控制层对应着Android中的Activity，由Activity来承担。Activity本身就是用来加载布局展示Ui的。但是使用这种架构往往会把控制逻辑也要加入，承担过多。

## MVP

- 通过引入接口BaseView，让相应的Activity、Fragment实现BaseView，实现了视图层的独立，通过中间层实现了Model层与View的完全解耦。

## MVVM

- 随着我们业务的增长，UI在改变多的情况下，我们会建很多相关UI的接口，会造成View节藕很庞大。
- 而MVVM就解决了这个问题，通过双向绑定，实现数据和UI内容只要改其中一方，另一方都能够及时更新的一种设计理念。
- MVVM是一种思想，DataBinding是Google推出方便实现MVVM的工具
- 官方的架构组件：ViewModel、LiveData、DataBinding。

