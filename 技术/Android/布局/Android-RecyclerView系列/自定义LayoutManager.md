#### 简介

- 实现一个LinearlayoutManager

#### 实现

###### 继承Recycler.LayoutManager

- 需要实现的方法

  - generateDefaultLayoutParams() : 用于布局子Item
  - onLayoutChildren():摆放子View的位置，在这里我们要控制要实现一屏幕的ItemView就可以了（要是全部显示的话，打开Activity都会变得非常慢了）
  - canScrollVertically()：开启能够垂直滑动
  - scrollVerticalBy()：在滑动中，一般在该方法中处理一些事物

- 实现一个能够上下滑动

  

- 关于滑动到底部和顶部还能继续滑动的处理

- 关于回收复用ViewHolder的处理

