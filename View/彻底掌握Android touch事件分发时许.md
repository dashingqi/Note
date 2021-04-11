#### 关于Android的事件分发有如下几点可以深入了解

-  touch事件是如何从驱动层传递给Framework层的InputManagerService
- WMS是如何通过ViewRoolmple将事件传递到目标窗口的
- touch事件到达DecorView后，是如何一步步传递到内部的子View中的。

#### 2个概念

###### ViewGroup

- 是一组View的组合，其内部可能包含多个子View。
- 它的内部事件分发的重心是处理当前Group和子View之间的逻辑关系
  - 当前Group是否需要拦截touch事件
  - 将touch事件分发给子View
  - 如何将touch事件分发给子View

###### View

- 就是单纯一个控件，内部也没有子View。
- 它内部事件分发重点在于当前View如何处理touch事件，根据相应的手势进行一些效果展示
  - 是否存在TouchListener
  - 是否自己接受处理touch事件

###### 事件分发的核心方法 dispatchTouchEvent()

- 决定是否要拦截当前事件

  - 判断当前ViewGroup是否需要拦截该touch事件，如果拦截了该touch事件（一般都是down事件），那么该touch事件的其他状态的都不会传递给子View了。（或者以CANCEL的方式通知子View）

- 将事件分发给子View

  - 如果ViewGroup没有对touch事件进行拦截，那么事件分发给子View进行处理，那么会把mFirstTouchTarget指向捕获Touch事件的View。

- 通过mFirstTouchTarget来决定事件该怎么分发

  - mFirstTouchTarget为null，说明当前事件没有被子View捕获过，会直接调用dispatchTransformedTouchEvent方法，传入的child为null，该方法内部会去调用super.dispatchTouchEvent(),实际上最终会调用自身的onTouchEvent方法

  - mFirstTouchTarget不为null，说明子View捕获过这个touch事件，那么这个touch事件就会发送给mFirstTouchTarget指向的子View去处理

###### 为什么Down事件特殊

- 所有touch事件都是从DOWN事件开始的，这是DOWN事件比较特殊的原因之一。
- DOWN事件的处理结果会直接影响后续MOVE、UP事件的逻辑
- 后续的MOVE、UP等事件的分发交给谁，取决于它们的启始事件DOWN是由谁捕获的。

###### 容易被遗漏的CANCEL事件

- 当父视图的onInterceptTouchEvent先返回false，然后子View的dispatchTouchEvent中返回true。
- 在接下来的MOVE事件，父视图的onInterceptTouchEvent又返回true，intercepted被重置为true
- 此时子控件就会收到一个ACTION_CANCEL的touch事件，并且事件的主导权就会重新回到ViewGroup中。

