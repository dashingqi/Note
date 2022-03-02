## Context类型

- Context是维持Android程序中各个组件能够正常工作的一个核心功能类

#### Context的继承结构

###### 结构示意图

![Android中Context的继承结构.png](https://upload-images.jianshu.io/upload_images/4997216-e874ea8b11ca3a78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 直接子类的解释

- ContextWrapper：是上下功能的封装类，其中ContextWrapper有三个直接子类
  - Application
  - Service
  - ContextThemeWrapper：这是一个带主题的封装类，它的直接子类就是Activity
- ContextImpl：是上下文功能的实现类。



#### 我们熟悉的

- Application
- Activity
- Service

- 在我们Android项目中，Context可以分为Application，Activity，Service这三类，分别承担着不同的作用。

#### Context数量

###### 在一个进程中

- Context数量 = Activity数量+Service数量+1

#### Application Context的设计

###### getApplication

- 是用来获取Application实例；
- 这个方法只有在Activity和Service中才能调用。

###### getApplicationContext

- 获取到的也是Application本身的实例
- 这个方法可以在除了Activity、Service之外的其他地方使用，比如BraodcastReceiver
- 相比较getApplication，getApplicationContext的作用域会更广些。

