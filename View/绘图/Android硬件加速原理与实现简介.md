> 好文章 https://tech.meituan.com/2017/01/19/hardware-accelerate.html

#### 了解硬件加速对App开发的意义

实现一个圆形按钮

###### 使用PNG图片

- 解码PNG图片生成Bitmap，传到底层，由GPU渲染
- 图片解码消耗CPU运算资源，Bitmap占用内存大，绘制慢。

###### 使用XML文件

- 直接将Shape信息传递到底层，由GPU渲染
- 消耗CPU资源少，占用内存小，绘制快。

#### 页面渲染的背景知识

- 页面渲染的时候，被绘制的元素最终要转换成矩阵像素点（即多维数组形式，类似于Android中的Bitmap），才能被显示器显示
- 页面由各种基本元素组成，例如圆形、圆角矩形、线段

#### CPU与GPU的对比

###### CPU

- 中央处理器，是计算机设备核心器件，用于执行程序代码，我们对此都很熟悉
- 主要擅长各种复杂的逻辑运算

###### GPU

- 主要用于处理图形运算，通常所说“显卡”的核心部件就是GPU
- 主要擅长大量数学运算而设计的。
- 硬件加速的主要原理就是通过底层软件代码，将CPU不擅长的图形计算转换成GPU专用指令，由GPU完成。

###### 禁用GPU硬件加速的方法

- Application层级上：为整个应用程序关闭硬件加速

  ```xml
  <application android:hardwareAccelerated="false"/>
  ```

  

- Activity层级上

  ```xml
  <activity android:hardwareAccelerated="false"/>
  ```

  

- Window层级上：在Window层级上不支持关闭硬件加速，支持开启硬件加速

  ```java
  getWindow().setFlags(WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED,
                       WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED
                      );
  ```

  

- View层级上:在View层级上不支持开启硬件加速，支持关闭硬件加速

  ```java
  setLayerType(View.LAYER_TYPE_SOFTWARE,null);
  
  // 获取使用 android:layerType="software"来关闭硬件加速
  <LinearLayout android:layerType="software">
  ```

  

#### Android绘制流程

- 从`ViewRootImpl.performTraversals`到`PhoneWindow.DecroView.drawChild`是每次遍历View树的固定流程，首先根据标志位判断是否需要重新布局并执行布局；然后进行Canvas的创建等操作开始绘制。
  - 如果硬件加速不支持或者被关闭，则使用软件绘制，生成的Canvas即`Canvas.class`的对象；
  - 如果支持硬件加速，则生成的是`DisplayListCanvas.class`的对象；
  - 两者的`isHardwareAccelerated()`方法返回的值分别为false、true，View根据这个值判断是否使用硬件加速。
- View中的`draw(canvas,parent,drawingTime)` - `draw(canvas)` - `onDraw` - `dispachDraw` - `drawChild`这条递归路径（下文简称**Draw路径**），调用了`Canvas.drawXxx()`方法，在软件渲染时用于实际绘制；在硬件加速时，用于构建DisplayList。
- View中的`updateDisplayListIfDirty` - `dispatchGetDisplayList` - `recreateChildDisplayList`这条递归路径（下文简称**DisplayList路径**），仅在硬件加速时会经过，用于在遍历View树绘制的过程中更新DisplayList属性，并快速跳过不需要重建DisplayList的View。

