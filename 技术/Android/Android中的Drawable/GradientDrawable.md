## GradientDrawable

#### 优势

- 可以快速实现View的形状（圆形，方形，线性）
- 可以快速实现View的圆角，渐变，阴影等效果
- 可以替代图片来实现View的一些效果
- 可以减少内存的使用以及APK包的大小
- 方便维护

#### 静态使用

- 在AndroidStudio中res文件下的drawable文件夹下自定义shape文件

#### 动态使用

- 创建GradientDrawable的对象，编辑相关属性，实现效果

- 代码实操

  ```kotlin
  fun setBackgroundDrawable(view: View, color: Int, radius: Float) {
          var drawable = GradientDrawable()
          drawable.setColor(color)
          drawable.cornerRadius = DensityUtil.dip2px(view.context, radius)
          view.background = drawable
      }
  ```

#### 从静态方式转化成动态，修改属性

- 新建shape文件，设置background属性，在代码中获取到background转化成GradientDrawable

- 代码实操

  ```java
  GradientDrawable drawable =(GradientDrawable)view.getBackground();
  drawable.setColor(fillColor); // 设置填充色   
  drawable.gd.setStroke(strokeWidth, strokeColor); // 设置边框宽度和颜色
  gd.setColors(colors);    // 设置渐变颜色数组
  ```

#### 注意

- shape定义Drawable都是属于GradientDrawable的。