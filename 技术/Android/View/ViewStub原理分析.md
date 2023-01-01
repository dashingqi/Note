## 本文主要从如下几点来学习ViewStub

- ViewStub是啥
- ViewStub的属性解析
- ViewStub的代码实操
- ViewStub的原理解析
- ViewStub实际中一般常用的情景
- ViewStub的两个小问题

## ViewStub是啥

###### 在介绍ViewStub是啥之前，我们了解下为什么要用ViewStub

> 在我们日常开发中，有些布局或者控件一开始并不需要显示，是根据业务场景来控制显示状态的，我们通常的做法就是在xml文件设置不可见，然后通过setVisibility()方法来更新它的可见性能，但是这样做会对程序的性能有一定的影响，我们知道加载布局的时候有两个瓶颈 一个是将xml文件加载到内存中是IO操作，通过反射或者到View的对象，这两者都是耗时的操作。
>
> 基于上面的业务情况，出现了ViewStub的标签，它是按需加载View，能够很容易实现布局的懒加载来提升程序的性能。

- ViewStub 继承于 View

- 看下源码声明

  ```java
  /*
   * A ViewStub is an invisible, zero-sized View that can be used to lazily inflate
   * layout resources at runtime.
   * When a ViewStub is made visible, or when {@link #inflate()}  is invoked, the layout resource 
   * is inflated. The ViewStub then replaces itself in its parent with the inflated View or Views.
   * Therefore, the ViewStub exists in the view hierarchy until {@link #setVisibility(int)} or
   * {@link #inflate()} is invoked.
   */
  
  
  //ViewStub是一个不可见的，宽高为0的View，可用于在程序运行的时候延迟  加载布局资源的（用于实现布局资源的“懒加载”）
  
  //当使ViewStub可见或者调用inflate方法，可以使布局资源被加载！
  
  //ViewStub存在于视图的层级中直到setVisibility()方法或者inflate()方法被执行后，ViewStub相关的资源就会被加载并在控件层级结构中代替ViewStub，同时ViewStub会从控件中移除。
  
  ```

  

## ViewStub的属性解析

###### android:id

- ViewStub在布局文件中ID，用于在代码中访问
- View共有的

###### android:layout

- 在显示ViewStub时真正加载并且显示的布局文件
- ViewStub特有的

###### android:inflatedId

- 真正布局加载后的布局Id
- ViewStub特有的

## ViewStub的代码实操

###### Xml资源文件

- 代码如下

  ```java
  <ViewStub android:id="@+id/stub"               						android:inflatedId="@+id/subTree"              				             android:layout="@layout/mySubTree              android:layout_width="120dp"
  android:layout_height="40dp"/>
  ```

###### Java代码

- 代码如下

  ```java
  //1,通过id找到ViewStub，得到ViewStub对象
  ViewStub myViewStub = (ViewStub)findViewById(R.id.stub);
  if(myViewStub!=null){
    //2,通过inflate方法加载真正的布局View
    View myInflatedView = myViewStub.inflate();
    //3,找到相应的控件
   TextView myTextView =  myInflatedView.findViewById(R.id.my_text_view);
  }
  ```

  

## ViewStub的原理解析

###### ViewStub的构造方法

- 代码如下

  ```java
  public ViewStub(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
          super(context);
  
          final TypedArray a = context.obtainStyledAttributes(attrs,
                  R.styleable.ViewStub, defStyleAttr, defStyleRes);
          saveAttributeDataForStyleable(context, R.styleable.ViewStub, attrs, a, defStyleAttr,
                  defStyleRes);
  
          //获取在xml文件中定义的inflatedId属性
          mInflatedId = a.getResourceId(R.styleable.ViewStub_inflatedId, NO_ID);
          //获取到xml文件中定义的layout属性
          mLayoutResource = a.getResourceId(R.styleable.ViewStub_layout, 0);
          //获取xml文件中定义的id属性
          mID = a.getResourceId(R.styleable.ViewStub_id, NO_ID);
          a.recycle();
  
          //设置ViewStub直接不显示
          //也可以看出来，你在xml文件中如何控制它的显示属性，都是不显示的
          setVisibility(GONE);
          // 设置ViewStub不尽兴绘制
          setWillNotDraw(true);
      }
  ```

###### ViewStub的onMeasure和onDraw方法

- 代码如下

  ```java
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
          //设置宽和高都为0 也就是控件的大小为0
          setMeasuredDimension(0, 0);
      } 
  
  @Override
      public void draw(Canvas canvas) {
          //不进行任何绘制
      }
  ```

###### ViewStub的inflate方法

- 代码如下

  ```java
  public View inflate() {
          //获取ViewStub在布局文件中的父布局
          final ViewParent viewParent = getParent();
  
          if (viewParent != null && viewParent instanceof ViewGroup) {
              //mLayoutResource 就是属性 layout指定的真正要加载的布局
              if (mLayoutResource != 0) {
                  final ViewGroup parent = (ViewGroup) viewParent;
                  //把真正要显示的View布局文件渲染成View对象并且给返回
                  final View view = inflateViewNoAdd(parent);
                  //将ViewStub从布局文件结构中移除，并且把渲染好的View添加到ViewStub所处的位置。
                  replaceSelfWithView(view, parent);
  
                  mInflatedViewRef = new WeakReference<>(view);
                  if (mInflateListener != null) {
                      //保存当前View对象的弱引用，方便其他地方使用
                      mInflateListener.onInflate(this, view);
                  }
  
                  //返回创建的View对象
                  return view;
              } else {
                  //当我们没有为ViewStub指定layut属性时，会走这个case，抛出异常
                  throw new IllegalArgumentException("ViewStub must have a valid layoutResource");
              }
          } else {
              //第一个调用ViewStub的inflate方法后，会把ViewStub从布局文件结构中移除，就是没有了ViewGroup了
              // 当第二次调用ViewStub的inflate方法后，会走这个case，抛出异常。
              throw new IllegalStateException("ViewStub must have a non-null ViewGroup viewParent");
          }
      }
  
  private View inflateViewNoAdd(ViewGroup parent) {
          // 获取到布局渲染器
          final LayoutInflater factory;
          if (mInflater != null) {
              factory = mInflater;
          } else {
              factory = LayoutInflater.from(mContext);
          }
          // 把真正要显示的布局文件渲染成View对象
          final View view = factory.inflate(mLayoutResource, parent, false);
  
          //  mInflatedId 对应 android:inflatedId 如果指定了就为渲染好的View给设置进去
          if (mInflatedId != NO_ID) {
              view.setId(mInflatedId);
          }
          return view;
      }
  
  private void replaceSelfWithView(View view, ViewGroup parent) {
          // 获取ViewStub在父布局中所处在的位置
          final int index = parent.indexOfChild(this);
          // 将ViewStub从父布局中移除
          parent.removeViewInLayout(this);
          // 获取ViewStub的布局参数
          final ViewGroup.LayoutParams layoutParams = getLayoutParams();
          // 当设置了布局参数（例如 android:width="50dp",height="50dp"）
          if (layoutParams != null) {
              // 将渲染好的View连同ViewStub的布局参数添加到ViewStub所处的位置
              parent.addView(view, index, layoutParams);
          } else {
              //将渲染好的View添加到ViewStub所处的位置
              parent.addView(view, index);
          }
      }
  ```

- 在我看来inflate方法主要的就是inflateViewNoAdd和replaceSelfWithView方法

- inflateViewNoAdd：获取到布局渲染器将真正需要展示的布局文件渲染成View并且给返回。

- replaceSelfWithView：将ViewStub从布局文件结构中移除，同时把渲染好的View添加到ViewStub之前所处的位置

- 之后把渲染好的View的弱引用给存储起来。方便在setVisibility()方法中使用。

###### ViewStub的setVisibility()方法

- 代码如下

  ```java
  public void setVisibility(int visibility) {
          // 纵观全局，mInflatedViewRef只有在inflate方法中初始化了，
          // 当真正的布局文件被加载之后
          if (mInflatedViewRef != null) {
              // 获取到当前的View
              View view = mInflatedViewRef.get();
              if (view != null) {
                  //操纵当前View的可见行
                  view.setVisibility(visibility);
              } else {
                  throw new IllegalStateException("setVisibility called on un-referenced view");
              }
          } else {
              //没有调用inflate的话，会设置可见性
              super.setVisibility(visibility);
              //当 当前设置可见性为 VISIBLE或者INVISIBLE的时候，会调用inflate方法。
              if (visibility == VISIBLE || visibility == INVISIBLE) {
                  inflate();
              }
          }
      }
  ```

  



## ViewStub实际中一般常用情景

- 比如我们在无数据或者网络错粗的时候，需要单独显示一个布局，那么这个布局就可以用ViewStub。

## ViewStub的几个问题

###### 两次调用ViewStub的inflate方法会怎么样？

- 我们知道第一次调用inflate方法的时候，会将ViewStub从布局文件结构中移除。
- 当第二次调用的时候，ViewStub已经没有父控件了，当做错误检查的时候，会抛出异常。

###### 在xml文件中为ViewStub设置了 android:visibility="visible" 属性，ViewStub中真正要显示的View会显示嘛？

- 不会，我们在ViewStub的构造方法中，有看到它默认调用了setVisibility(GONE)
- 所以你在xml文件中如何操作可见行，都是没法显示的。





