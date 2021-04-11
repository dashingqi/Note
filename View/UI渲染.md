## UI渲染

#### 屏幕

- LCD：液晶显示器
- OLED：有机发光二极管
- OLED相比较于LCD，屏幕更软，适合用作于市面上那些挖空屏啊，曲面屏以及折叠屏。

#### CPU和GPU

- 除了能看得见的屏幕，UI的渲染还涉及到两个硬件，CPU和GPU。

- UI绘制到屏幕之前，都会经过棚格化的操作，本身棚格化是一个比较耗时的操作，GPU可以帮助我们加快棚格化的操作。

- GPU称之为图形处理器，主要用于图形的运算。

- CPU用于软件的绘制而GPU用于硬件的绘制

  ![图片](https://static001.geekbang.org/resource/image/1c/8d/1c94e50372ff29ef68690da92c6b468d.png)

- 其中软件绘制最终时使用到了Skia库，Skia库是一个框架，该框架最主要的功能是将高质量的2D图像呈现到底端设备上。

###### OpenGL与Vulkan

- OpenGL
  - OpenGL是一个跨平台的图形 API，它为 2D/3D 图形处理硬件指定了标准软件接口
- Vulkan
  - Vulkan 是用于高性能 3D 图形的低开销、跨平台 API。相比 OpenGL ES，Vulkan 在改善功耗、多核优化提升绘图调用上有着非常明显的优势。

#### 绘图中各个组件的作用

- 画笔：Skia库 或者 OpenGL ES
- 画纸：Surface； 在Android中Window是View的容器，每一个Window都会关联一个Surface，View上的各个元素的渲染和绘制都是在Surface上的。WindowManager用于管理Window，它会将surafce发送给SurfaceFlinger。
- 画板：Graphic Buffer（图形缓冲区）；用于程序图形的绘制；在Android4.1之前使用的是双缓冲机制；Android4.1之后使用的是三缓冲机制了。
- 显示：SurfaceFlinger；收到WindowManager发送过来的Surface，会通过 Hardware Composer （硬件合成器）将图像合成显示到屏幕上 。

#### 绘制流程

###### 软件绘制

- 在Android3.0之前的时候没有开启硬件加速，使用的是软件绘制

  
  
  ![软件绘制](https://static001.geekbang.org/resource/image/8f/97/8f85be65392fd7b575393e5665f49a97.png)
  
  - surface: 每个Window都会对应一个surafce，WindowManager管理着Window，会将对应的Surface传递给SurfaceFlinger
  - canvas：通过Surface的lockCanvas()方法获取到，在软件绘制中相当于与Skia框架的底层接口
  - Graphic Buffer：Skia将绘制的2D图形数据通过棚格化交给Graphic Buffer，这个Graphic Buffer是由SurfaceFlinger内部 BuggerQueue中维护的
  - SurfaceFlinger：从Graphic Buffer中拿到数据通过硬件合成将合成数据的数据显示到Display（屏幕上）

###### 硬件加速绘制

- 在Android3.0的时候开始支持硬件加速，到Android4.0的时候默认开启了硬件加速

  ![硬件加速绘制](https://static001.geekbang.org/resource/image/79/e8/79c315275abac0823971e5d6b9657be8.png)

  - 与上述软件绘制的主要区别在于，硬件加速绘制使用了OpenGl ES 通过GPU将surface中的数据通过棚格化交给了Graphic Buffer。
  - 在硬件加速绘制的过程中，出现了一个DisplayList这么一个概念，每个View中都会有DisplayList，如果要进行重新绘制的操作，会将DisplayList置为Dirty.
  - 当需要重新绘制的时候，仅仅重新绘制View中的DisplayList就可以。

#### Project Butter (黄油计划)

- Google在Android4.1的时候启用了黄油计划。
- 黄油计划中有两个重要的元素：VSYNC 和 Triple Buffering

###### VSYNC 信号

- ![](https://static001.geekbang.org/resource/image/06/bd/06753998a26642edd3481f85fc93c8bd.png)

- Vsync信号的开始处进行绘制，CPU计算数据，同时GPU开始进行图像的合成，当所有图像合成完毕显示到屏幕上之后，在进行下一个Vsync信号的发出。

###### 三缓冲机制

> 三缓冲机制也是为了解决Android4.1之前的双缓冲机制的不足而出现的，在说三缓冲之前先说下双缓冲机制。

- 双缓冲机制
  - 每个surface对应的BufferQueue内部都会有两个 Graphic Buffer，一个用于绘制，一个用于显示。首先会把数据绘制到离屏缓冲区中，当需要显示的时候通过Swap Buffer 将数据拷贝到 Front Graphic Buffer中。