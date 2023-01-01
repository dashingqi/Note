## RecyclerView的一些总结



#### 简介

> RecyclerView简称RV，是ListView和GridView的加强版。

#### 常规使用步骤

- 设置布局管理器（必须的）

  ```java
  RecyclerView.setLayoutManager()
  ```

  

- 设置适配器（必须的）

  ```java
  RecyclerView.setAdapter()
  ```

  

- 设置Item的装饰器（非必须的）

  ```java
  RecyclerView.setItemAnimator()
  ```

  

- 设置Item的动画（非必须的）

  ```java
  RecyclerView.addItemDecoration()
  ```

  

#### RecyclerView的绘制流程分析

> 按照 onMeasure() ---> onLayout() ---> onDraw()流程去分析

###### onMeasure()

- RecyclerView的测量过程是委托给LayoutManager来完成
- LayoutManager中的测量工作
  - 如果RecyclerView的宽和高都设置了match_parent或者具体数值，那么就调用LayoutManager中的onMeasure()方法来进行测量
  - 如果RecyclerView设置的是wrap_content,就会执行RecyclerView中的dispatchLayoutStep2()方法来进行测量
- 其中RecyclerView中的dispatchLayoutStep1()方法是和RecyclerView中的Item动画相关。

###### onLayout()

- RecyclerView的布局过程是委托给LayoutManager来完成后的
- onLayout方法中，核心方法是 dispatchLayout()
  - 在onMeasure方法中没有调用dispatchLayoutStep2()去测量子View，会在此方法中调用去测量。
- disPatchLayoutStep2()方法的核心就是调用LayoutManager的onLayoutChildren()方法去测量子View的大小，并确定子View的位置
  - 其中LayoutManager中的onLayoutChildren()方法是一个空方法，我们熟悉的LienarLayout、GridLayoutManager、StaggeredlayoutManager都对该方法进行了重写，有自己的实现方式。
- 看下LinearLayout的onLayoutChildren()方法
  - 核心是fill()方法
  - 在fill()方法，通过while循环来判断时候有过剩的空间来足够绘制一个完整的子View
  - 其中layoutChunk方法是子View测量布局的真正实现。
  - layoutChunk方法是一个非常核心的方法。该方法执行一次就将子View填充到RV中。
    - 其中会调用 layoutState.next()方法，获取到一个View（实际是从缓存中查找ViewHolder或者创刊一个新的ViewHolder，拿到ViewHolder绑定的View）

###### onDraw()

> 测量和布局的过程完毕，接下来就是绘制的操作

- 很简单，如果有添加ItemDecoration，那么就遍历取出，然后进行绘制操作

#### RecyclerView中的缓存复用原理Recycler

> RV中的缓存复用主要体现在ViewHolder的缓存和复用

- RV对ViewHolder的缓存和复用的核心代码都是在RV的内部类Recycler中完成的。

###### Recycelr的分级缓存

- 一级缓存（mAttachedScrap和mChangeScrap）
  - 一级缓存主要是用来缓存屏幕中的ViewHolder
  - 场景
    - 第一次我们调用setLayoutManger和setAdapter之后，Rv会进行一次layout并显示到屏幕中，此时的ViewHolder是经过onCreateViewHolder创建的。
    - 我们进行下拉刷新请求到数据的时候，会调用notifyXX的方法，此时会将屏幕中的可见的ViewHolder缓存到Scarp中，当缓存完之后，然后从Recycler的从缓存中根据postion获取相应的ViewHolder，之后将数据更新到相应的ViewHolder中。最后再将这些ViewHolder绘制到屏幕上。
- 二级缓存（mCachedViews）
  - 主要用来缓存屏幕之外的ViewHolder，缓存的容量为2个。（这个容量是可以通过setViewCacheSize方法来改变的）
  - 当缓存的容量已经满了，当有新的ViewHolder要进行缓存的话，就根据FIFO规则将旧的ViewHolder从缓存中移除。
  - 注意：刚被移除屏幕之外的ViewHolder通常会立即被使用到，所以被移除屏幕之外的ViewHolder不会被立即标记为无效ViewHolder，而是将它缓存在cache中 （也就是mCachedViews中），但是又不能将所有移除屏幕之外的ViewHolder都视为非无效的。所以它的缓存大小为2。
- 三级缓存（ViewCacheExtension）
  - 这是RV预留给我们开发人员的一个抽象类，我们可以继承这个抽象类，复写抽象方法，来自定义自己的缓存机制。
- 四级缓存（RecycledViewPool）
  - 也是用来缓存屏幕之外的ViewHodler，主要是用来缓存从二级缓存中淘汰的ViewHolder
    - 当ViewHolder缓存到RecycledViewPool中的时候，会将数据从ViewHodler中清空，当再次使用时候会调用onBindView重新设置数据
  - 多个RV可以共享一个RecycledViewPool。

#### RV从缓存中取出ViewHolder

- 在 onlayout过程中，layoutChunk方法通过layoutState.next方法拿到某一个子View，然后添加到RV中
- 在layoutState.next()方法中，通过getViewForPosition()方法获取到View
- getViewForPosition()方法调用了tryGetViewHolderforPositionByDealine()方法
- 在tryGetViewHolderforPositionByDealine()方法中会依次调用这4几缓存查找相应位置的ViewHolder
- 都缓存中都没有找到，就调用onCreateViewHolder()方法创建一个新的ViewHolder

