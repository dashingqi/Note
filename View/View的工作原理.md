

### ViewRoot和DecorView

#### ViewRoot

- ViewRoot对应的是ViewRootImpl类，是链接DecorView和WindowManager的纽带，View的三大流程全都是通过ViewRoot来完成的。
- 在ActivityThread中当Activity创建完毕后，会将DecorView添加到Window中，同时创建ViewRootImpl对象，并将ViewRootImpl对象和DecorView建立关联。
- ViewRootImpl和DecorView进行关联的代码
```java
root = new ViewRootImpl(view.getContext(),display);
root.setView(view,wparams,panelParentView);
```
- View的绘制是从ViewRoot的performTraversals()方法开始的。
- performTaversals方法会依次调用performMeasure()、performLayout()、performDraw()方法来完成顶层View的measure，layout和draw的过程。
#### DecorView

- DecorView是一个应用窗口根容器，本质上是一个FrameLayout

- 作为顶层View，一般情况下它内部都会包含一个竖直方向的LinearLayout，在这里LinearLayout里面有两部分，分别为title和content。
- Activity中通过setContentView所设置的布局就是被添加到id为content的FrameLyout中。
- 获取到content的布局
```java
//获取到content
ViewGroup content = finidViewById(R.android.id.content);
//获取到我们设置的View
View view = content.getChildAt(0);
```
- 通过源码我们知道DecorView就是一个FrameLayout （DecorView extends FrameLayout）

#### MeasureSpec

- 参与了View的Measure过程，“测量规格”。

- MeasureSpec代表一个32位的int，高2位是代表SpecMode（测试模式），后30位代表SpecSize（某种测量模式下的大小）。

- MeasureSpec提供了打包和解包的方法，将SpecMode和SpecSize打包成MeasureSpec；MeasureSpec通过解包将SpecMode和SpecSize分离出来。

  ##### SpecMode

  - UNSPECIFIED：父容器不对View有任何限制。

  - EXACTLY：父容器已经检测出View所需要的精确大小，这个时候View的大小就是SpecSize，对应于LayoutParams中的match_parent和具体的数值。

  - AT_MOST：父容器已经指定了一个大小即SpecSize，View的大小不能大于这个SpecSize，对应于LayoutParams中的wrap_content。

    

#### MeasureSpec和LayoutParams对应的关系

###### DecorView

- DecorView的MeasureSpec由窗口的尺寸和自身的LayoutParams来共同决定。

###### 普通View

- 由父容器的MeasureSpec和自身的LayoutParams来决定的。

- 针对普通View来说，View的measure过程是从ViewGroup中传递过来，代码如下

  ```java
  /**
  *看的出来View的MeasureSpec由父容器的MeasureSpec和自身的LayoutParams决定，也和Padding和Margin有关。
  */
  protected void measureChildWithMargins(View child,
              int parentWidthMeasureSpec, int widthUsed,
              int parentHeightMeasureSpec, int heightUsed) {
          final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
  
          final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                  mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                          + widthUsed, lp.width);
          final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                  mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                          + heightUsed, lp.height);
  
          child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
      }
  
  public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
          int specMode = MeasureSpec.getMode(spec);
          int specSize = MeasureSpec.getSize(spec);
  
          int size = Math.max(0, specSize - padding);
  
          int resultSize = 0;
          int resultMode = 0;
  
          switch (specMode) {
          // Parent has imposed an exact size on us
          case MeasureSpec.EXACTLY:
              if (childDimension >= 0) {
                  resultSize = childDimension;
                  resultMode = MeasureSpec.EXACTLY;
              } else if (childDimension == LayoutParams.MATCH_PARENT) {
                  // Child wants to be our size. So be it.
                  resultSize = size;
                  resultMode = MeasureSpec.EXACTLY;
              } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                  // Child wants to determine its own size. It can't be
                  // bigger than us.
                  resultSize = size;
                  resultMode = MeasureSpec.AT_MOST;
              }
              break;
  
          // Parent has imposed a maximum size on us
          case MeasureSpec.AT_MOST:
              if (childDimension >= 0) {
                  // Child wants a specific size... so be it
                  resultSize = childDimension;
                  resultMode = MeasureSpec.EXACTLY;
              } else if (childDimension == LayoutParams.MATCH_PARENT) {
                  // Child wants to be our size, but our size is not fixed.
                  // Constrain child to not be bigger than us.
                  resultSize = size;
                  resultMode = MeasureSpec.AT_MOST;
              } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                  // Child wants to determine its own size. It can't be
                  // bigger than us.
                  resultSize = size;
                  resultMode = MeasureSpec.AT_MOST;
              }
              break;
  
          // Parent asked to see how big we want to be
          case MeasureSpec.UNSPECIFIED:
              if (childDimension >= 0) {
                  // Child wants a specific size... let him have it
                  resultSize = childDimension;
                  resultMode = MeasureSpec.EXACTLY;
              } else if (childDimension == LayoutParams.MATCH_PARENT) {
                  // Child wants to be our size... find out how big it should
                  // be
                  resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                  resultMode = MeasureSpec.UNSPECIFIED;
              } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                  // Child wants to determine its own size.... find out how
                  // big it should be
                  resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                  resultMode = MeasureSpec.UNSPECIFIED;
              }
              break;
          }
          //noinspection ResourceType
          return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
      }
  
  ```

  ##### 代码整合成表格形式

  - ![View的MeasureSpec计算规则](https://upload-images.jianshu.io/upload_images/4997216-488c680ff93e5c3f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### View的工作流程

#### Measure

- 一个原始的View通过measure过程就可以测量完毕，如果是一个ViewGroup那么需要自身的measure还需要遍历子View，调用子View的measure。

  ##### View的Measure

  - View的measure是由View中的measure方法开始的，measure方法是final类型，所以子类不能重写该方法。

  - 在measure方法中调用了onMeasure方法，看onMeasure方法的实现即可。

    ```java
     protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
       //setMeasuredDimension()方法会设置View测量后的宽和高,View的最终宽和高是在layout过程确定的
            setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                    getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
        }
    
    //看getDefaultSize()
     public static int getDefaultSize(int size, int measureSpec) {
            int result = size;
            int specMode = MeasureSpec.getMode(measureSpec);
            int specSize = MeasureSpec.getSize(measureSpec);
    
            switch (specMode) {
            case MeasureSpec.UNSPECIFIED:
                result = size;
                break;
            case MeasureSpec.AT_MOST:
            case MeasureSpec.EXACTLY:
                result = specSize;
                break;
            }
            return result;
        }
       // 如果View没有设置背景，那么height就是mMinHeight的值，mMinHeight对应的是      android:mMinHeight属性的值，如果没有指定默认值就是0，
      // 如果设置了背景了就是取max(mMinHeight, mBackground.getMinimumHeight())两者之间的最大值。
      protected int getSuggestedMinimumHeight() {
              return (mBackground == null) ? mMinHeight : max(mMinHeight, mBackground.getMinimumHeight());
      }
    
      //getSuggestedMinimumWidth()同理getSuggestedMinimumHeight()
    
      /**
    
       * Returns the minimum height suggested by this Drawable. If a View uses this
       * Drawable as a background, it is suggested that the View use at least this
       * value for its height. (There will be some scenarios where this will not be
       * possible.) This value should INCLUDE any padding.
    
       * @return The minimum height suggested by this Drawable. If this Drawable
       * doesn't have a suggested minimum height, 0 is returned.
         返回Drawable的原始的高，如果Drawable没有原始高度，默认值就是0
                */
               public int getMinimumHeight() {
         final int intrinsicHeight = getIntrinsicHeight();
         return intrinsicHeight > 0 ? intrinsicHeight : 0;
               }
           //getMinimunWidth()同理与getMinimunHeight()。
    ```

    ```
  
    
  
  ##### ViewGroup的Measure
  
  - ViewGroup除了完成自己的measure过程后，还需要循环遍历子View，调用子View的meassure过程。
  
  - ViewGroup是一个抽象类因此没有重写View的onMeasure，提供了一个measureChildren()方法。
  
  - ViewGroup没有定义自己的测量过程。本身是一个抽象类，并没有实现onMeasure方法，onMeasure需要子View的自己去实现。
  
    
  
    ```java
    protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
            final int size = mChildrenCount;
            final View[] children = mChildren;
            for (int i = 0; i < size; ++i) {
                final View child = children[i];
                //如果子View是处于GONE就不进行测量了。
                if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                    measureChild(child, widthMeasureSpec, heightMeasureSpec);
                }
            }
        }
    
     /**
         * Ask one of the children of this view to measure itself, taking into
         * account both the MeasureSpec requirements for this view and its padding.
         * The heavy lifting is done in getChildMeasureSpec.
         *
         * @param child The child to measure
         * @param parentWidthMeasureSpec The width requirements for this view
         * @param parentHeightMeasureSpec The height requirements for this view
         */
        protected void measureChild(View child, int parentWidthMeasureSpec,
                int parentHeightMeasureSpec) {
            final LayoutParams lp = child.getLayoutParams();
    
          //子View的MeasureSpec受父容器的MeasureSpec和自己的LayoutParams影响
            final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                    mPaddingLeft + mPaddingRight, lp.width);
            final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                    mPaddingTop + mPaddingBottom, lp.height);
    				//开始子View的measure过程
            child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
        }
    ```




  ##### View获取到宽高

  - View的绘制与Activity的生命周期不是同步的，当生命周期执行完毕，View的绘制还没有执行完毕，这时在onCreate()、onStart()、onPause获取到的. 和height都是0。

  - Activity/View.onWindowFocusChanged()方法中获取

    ```Java
    //当Window获取焦点和失去焦点的时候，该方法会被调用的，触发该方法的时候， View已经测试完毕，可以获取View的宽和高
    /**
         * 该方法会在window获取到焦点和失去焦点的时候调用，可能会频繁的调用，
         *
         * @param hasFocus
         */
        @Override
        public void onWindowFocusChanged(boolean hasFocus) {
            super.onWindowFocusChanged(hasFocus);
          	//当获取到焦点的时候去执行下面的代码
            if (hasFocus) {
                int width = mBtnAsyncTask.getWidth();
                int height = mBtnAsyncTask.getHeight();
                Log.d("measure", "MeasureWidth = " + width + " measureHeight = " + height);
            }
        }
    
    ```

    

  - View.post(Runnable)

    ​	

    ```Java
    //通过post方法将一个runnable投放到消息队列的的尾部，等待looper取到消息调用到这个runnable的时候，View已经初始化好了
    //典型的模版代码如下
    @Override
        protected void onStart() {
            super.onStart();
            mBtnToastWorkThread.post(new Runnable() {
                @Override
                public void run() {
                    int width = mBtnToastWorkThread.getWidth();
                    int height = mBtnToastWorkThread.getHeight();
                    Log.d("post", "MeasureWidth = " + width + "MeasureHeight = " + height);
    
                }
            });
        }
    ```

  - ViewTreeObserver

    ```Java
    //使用ViewTreeObserver的众多回调接口可以实现获取到View的宽和高
    //当View树的状态发生改变或者View树内部的View可见性发生改变的时候，此时OnGlobalLayout接口中的onGlobalLayout()方法将会被回调。此时是获取到View的宽和高的好时机。
    @Override
        protected void onStart() {
            super.onStart();
            ViewTreeObserver viewTreeObserver = mBtnIntentService.getViewTreeObserver();
            viewTreeObserver.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
                @Override
                public void onGlobalLayout() {
                    mBtnIntentService.getViewTreeObserver().removeOnGlobalLayoutListener(this);
                    int width = mBtnIntentService.getWidth();
                    int height = mBtnIntentService.getHeight();
                    Log.d("ViewTreeObserver = ", "height = " + height + " width = " + width);
                }
            });
        }
    ```

    

  - View.measure(int widthMeasureSpec,int heightMeasureSpec)

  

#### Layout

- layout()方法确认了View本身的位置，onLayout()方法确认了所有子元素的位置。

  ##### layout()方法的流程

  - 首先通过setFrame()方法确定了View的四个顶点的位置。
  - 确认了View的顶点也就是确认了View在父布局中的位子。
  - 接着调用了onLayout()方法，这个方法是父容器确认了子元素的位置。
  - onLayout()在View和ViewGroup中都没有具体的实现，不同View有不同的布局实现。

  ##### LinearLayout_layoutVertcial

  - LinearLayout通过layout方法调用了onLayout方法中的layoutVertical
  - 在layoutVertical()方法中调用了setChildFrame来确认子view的位置。
  - setChildFrame方法中调用了子View的layout()方法，子View中layout方法中调用了onLayout()方法来确认了子 view的顶点的位置。

  ##### View的测量后宽和高与最总得到的宽和高相等吗？

  	> 该问题就是 getMeasuredHeight()和getMeasuredWidth() 与 getHeight()和getWidth() 得到得值相等吗？

  - 正常情况下测量后得到的值域最终得到的值是相等的
  - 前者是发生在measure过程中，后者发生在layout过程中。
  - 赋值的时机是不同的。
  - 某些情况下measure过程中得到的值域layout过程得到的值是不同的

  ```Java
  public void layout(int l,int t,int r,int b){
  	super.layout(l,t+100,r,b+20)
  }
  //如上的操作，就会导致 layout过程中得到的值与measure规程得到的值是不同的。
  ```

  

#### Draw

>  View的draw作用就是将View绘制到屏幕上，遵循如下几步操作

- 绘制背景 background.draw(canvas)。
- 绘制自己（onDraw()）
- 绘制children（dispatchDraw()）
- 绘制装饰（onDrawScrollBars()）

