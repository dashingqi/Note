> 自定义View是一个综合的技术体系，需要View的层次结构、View的事件分发机制、View的工作原理等技术细节。

#### 自定义View的分类

- 继承View重写onDraw()方法

- 继承ViewGroup派生特殊的Layout

- 继承特定的控件（TextView，EditText）

- 继承特定的ViewGroup（LinearLayout，FrameLayout）

##### 自定义View的注意事项

- 让View支持wrap_content

  > 这是因为继承View或者ViewGroup的控件如果不在onMeasure方法中针对wrap_content的情形做特殊处理，那么给自定义View设置wrap_content不会达到预期的效果。特殊处理见如下代码。

  ```java
  /**
  * 内部指定一个默认的宽和高。mWidth mHeight
  * 针对是wrap_content的时候，设置一个默认值。
  */
  protected void onMeasure(int widthMeasureSpec,int heightMeasureSpec){
    super.onMeasure(widthMeasureSpec,heightMeasureSpec);
    int heightMode = MeasureSpec.getMode(heightMeasureSpec);
    int heightSize = MeasureSpec.getSize(heightMeasureSpec);
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);
    int heightSize = MeasureSpec.getSize(wifthMeasureSpec);
    
    if(widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST){
      setMeasuredDimension(mWidth,mHeight);
    }else if(widthMode == MeasureSpec.AT_MOST){
      setMeasreDimension(mWidht,heightSize);
    }else if(heightMode == MeasureSpec.AT_MOST){
      setMeasureDimension(widthSize,mHeight);
    }
  }
  ```

  

- 如果可以，让你的View支持padding

  > 继承View的控件的，如果不在onDraw方法中处理padding，那么paddings属性不会起作用的。
  >
  > 继承ViewGroup控件的，需要在onMeasure和onLayout中考虑padding和子View的Margin造成的影响，不考虑的话会使padding和margin失效的。

- 尽量不要在View中使用Handler，没有必要

  > View的内部有一些列post方法，完成可以替代Handler的作用。如果内部明确要使用，可以使用Handler。

- View中有线程或者动画，需要及时停止

  > 当包含View的Activity退出或者当前的View被remove时，View的onDetachedFromWindow方法会被调用，当包含View的Activity启动时，View的onAttachedToWindow()方法会被调用。
  >
  > 当View退出的时候，线程和动画需要停止，不及时停止的话，可能会造成内存泄漏。

- View带有滑动嵌套式，需要处理好滑动冲突的问题

- 在View的onDraw()方法中不要大量创建临时对象（new 出来的对象），因为 onDraw()方法会频繁调用，这样就会创建很多临时对象，导致GC 频繁，内存抖动，影响 View 绘制；

- 自定义View提供可配置属性，提供可自定义属性，满足需求；

- 文档和注释要齐全；

##### 自定义View示例

###### 继承View重写onDraw()方法

- margin属性是由父容器所控制的，不需要在自定义View中做处理

- padding属性需要处理，如果不处理padding属性不起作用。处理的代码如下

  ```Java
  //处理就是把padding也考虑进去
    @Override
      protected void onDraw(Canvas canvas) {
          super.onDraw(canvas);
          int paddingBottom = getPaddingBottom();
          int paddingTop = getPaddingTop();
          int paddingLeft = getPaddingLeft();
          int paddingRight = getPaddingRight();
          int height = getHeight() - paddingTop - paddingBottom;
          int width = getWidth() - paddingLeft - paddingRight;
          int min = Math.min(height, width);
          canvas.drawCircle(paddingLeft+width / 2, paddingTop+height / 2, min / 2, mPaint);
      }
  ```

- wrap_content 不做处理，和match_parent的效果时一致的。处理的代码如下

  ```java
  @Override
      protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
          super.onMeasure(widthMeasureSpec, heightMeasureSpec);
          int widthMode = MeasureSpec.getMode(widthMeasureSpec);
          int widthSize = MeasureSpec.getSize(widthMeasureSpec);
  
          int heightMode = MeasureSpec.getMode(heightMeasureSpec);
          int heightSize = MeasureSpec.getSize(heightMeasureSpec);
  
          if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) {
              setMeasuredDimension(defaultValue, defaultValue);
          } else if (widthMode == MeasureSpec.AT_MOST) {
              setMeasuredDimension(defaultValue, heightSize);
          } else if (heightMode == MeasureSpec.AT_MOST) {
              setMeasuredDimension(widthSize, defaultValue);
          }
  
      }
  ```

  ###### 自定义属性

  - 在values文件夹下新建自定义属性文件夹 attrs.xml，在attrs.xml 文件中新建属性。

    ```java
    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <declare-styleable name="CircleView">
            <attr name="background_color" format="color" />
        </declare-styleable>
    </resources>
    ```

    

  - 在自定义文件中获取自定义属性的值

    ```java
    public CircleView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
            super(context, attrs, defStyleAttr);
            TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
            mBackgroundColor = typedArray.getColor(R.styleable.CircleView_background_color, Color.YELLOW);
            typedArray.recycle();
            init();
        }
    ```

  - 在布局文件中使用自定义属性

    ```java
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        tools:context=".MainActivity">
    
        <com.dashingqi.customondraw.CircleView
            android:layout_width="match_parent"
            android:layout_height="100dp"
            android:layout_margin="20dp"
            android:padding="40dp"
            app:background_color="@color/colorPrimary"
             />
    
    </LinearLayout>
    ```

    