- #### 简介

  - 在Android系统中，针对输入的事件由InputEvent来表示
    - 针对由键盘输入的事件封装成KeyEvent来进行传递
    - 针对View的点击和滑动的事件分装成MotionEvent来进行传递
  - 今天我们要分析的就是封装好的MotionEvent的事件传递

  ###### MotionEvent

  - 事件有分为如下几种
    - ACTION_DOWN:按下事件,一个事件序列中有一个ACTION_DOWN
    - ACTION_MOVE:滑动事件，一个事件序列中至少有一个ACTION_MOVE
    - ACTION_UP:抬起事件，一个事件序列中有一个ACTION_UP
    - ACTION_CANCEL:取消事件，一般是非认为因素影响的
    - 所以说一个事件序列中 包含一个ACTION_DOWN和一个ACTION_UP,中间有一个或者多个ACTION_MOVE

  - 在整个的事件分发中主要涉及到Activity、ViewGroup、View三个类
    - Activity作为用户层面事件的起源
    - ViewGroup和View作为
  - 事件传递得顺序是 Activity ---> ViewGroup ----> View

  整个事件分发过程中，涉及到的方法有 dispatchTouchEvent()、onInterceptTouchEvent()、onTouchEvent(),在Activity、ViewGroup、View中对于上述三个都有不同的实现，对应关系如下表格

  |       方法\控件       | Activity | ViewGroup | View |
  | :-------------------: | :------: | :-------: | :--: |
  |  dispatchTouchEvent   |    有    |    有     |  有  |
  | onInterceptTouchEvent |    无    |    有     |  无  |
  |     onTouchEvent      |    有    |    有     |  有  |

  ###### dispatchTouchEvent()

  - 该方法的意思就是起到事件分发的作用；
  - 当返回true就表示消费这个事件；
  - 返回false，就表示不分发这个事件，转而将这个事件传递给父View的onTouchEvent()；
  - 返回 super.dispatchTouchEvent(),如果当前控件是ViewGroup的话会将事件传递给自己的onInterceptTouchEvent()方法，如果当前控件是View的话会将事件传递给自己的onTouchEvent()。

  ###### onInterceptTouchEvent()

  - 该方法是起到事件拦截的作用，仅仅ViewGroup独有的，是在ViewGroup的dispatchTouchEvent()方法中调用的，Activity和View中没有此方法；
  - 当返回true的话，表示要拦截当前这个事件，并将这个事件传递给自己的onTouchEvent方法来处理
  - 放返回false的话，表示不拦截这个事件，如果当前ViewGroup没有子View能分发这个事件，就将这个事件传递给View的dispatchToucheEvent，否则的话将这个事件分发给子View的dispatchToucheEvent()方法
  - 当调用super.onInterceptTouchEvent(),等同于返回false，ViewGroup默认onInterceptTouchEvent()返回false

  ###### onTouchEvent()

  - 该方法是表示消费事件的处理方法
  - 当返回true的话就表示要消费这个事件
  - 当返回false的话，表示不消费这个事件，转而将这个事件传递给父View的onTouchEvent()方法。

  ###### requestDisallowInterceptTouchEvent

  - 除了以上三个作为事件分发的基础，该方法也能影响事件的分发，该方法主要是子View用来通知父View下一个事件该怎么处理
  - 如果设置为false，就告诉父View可以拦截这个事件，就不要传递给我了
  - 如果设置为true，就告诉父View不要拦截，要把事件传递给子View。

  #### 代码实操

  > 上面我们介绍了事件分发过程中涉及到的类以及方法，在下面我们通过继承ViewGroup和View，以及重写 dispatchTouchEvent(),onInterceptTouchEvent(),onTouchEvent()这三个方法打印一下事件，看下分发的流程。

  - 情况一：我们自定义 MyGroupViewOne,MyGroupViewTwo以及MyViewOne，同时重写了事件分发的方法，返回super.xxxx()

    ```kotlin
    class MyViewGroupOne :RelativeLayout {
        private  val TAG = "MyViewGroupOne"
    
        constructor(context:Context):super(context)
    
        constructor(context: Context,attributeSet: AttributeSet):super(context,attributeSet)
    
        constructor(context: Context,attributeSet: AttributeSet,defStyleAttr:Int):super(context,attributeSet,defStyleAttr)
    
        override fun dispatchTouchEvent(ev: MotionEvent?): Boolean {
            ev?.let {
              //这里的MotionEventutils是一个封装好的工具类，专门用来打印对应的事件的一个方法
               MotionEventUtils.printEvent(ev.action,"dispatchTouchEvent",TAG)
            }
            return super.dispatchTouchEvent(ev)
        }
    
        override fun onInterceptTouchEvent(ev: MotionEvent?): Boolean {
            ev?.let {
                MotionEventUtils.printEvent(ev.action,"onInterceptTouchEvent",TAG)
            }
            return super.onInterceptTouchEvent(ev)
        }
    
        override fun onTouchEvent(event: MotionEvent?): Boolean {
            event?.let {
                MotionEventUtils.printEvent(event.action,"onTouchEvent",TAG)
            }
            return super.onTouchEvent(event)
        }
    }
    ```

    ​	

    - [MotionEventUtils的代码](https://github.com/dashingqi/DQMaster/blob/master/DQCommonUtils/src/main/java/com/dashingqi/dqcommonutils/MotionEventUtils.kt)
    - 同样MyViewGroupTwo如同MyViewGroupOne，我们看下MyViewOne

  - MyViewOne的代码

    ```kotlin
    class MyViewOne : View {
    
        private val TAG = "MyViewOne"
    
        constructor(context: Context) : super(context)
    
        constructor(context: Context, attributeSet: AttributeSet) : super(context, attributeSet)
    
        constructor(context: Context, attributeSet: AttributeSet, defStyleAttr: Int) : super(
            context,
            attributeSet,
            defStyleAttr
        )
    
        override fun dispatchTouchEvent(event: MotionEvent?): Boolean {
            event?.let {
                MotionEventUtils.printEvent(event.action, "dispatchTouchEvent", TAG)
            }
            return super.dispatchTouchEvent(event)
        }
    
        override fun onTouchEvent(event: MotionEvent?): Boolean {
            event?.let {
                MotionEventUtils.printEvent(event.action, "onTouchEvent", TAG)
            }
            return super.onTouchEvent(event)
        }
    }
    ```

  - 同时我们重写MainActivity的dispatchTouchEvent()和onTouchEvent()方法

    ```kotlin
    override fun dispatchTouchEvent(ev: MotionEvent?): Boolean {
            ev?.let {
                MotionEventUtils.printEvent(ev.action, "dispatchTouchEvent", TAG)
            }
            return super.dispatchTouchEvent(ev)
        }
    
        override fun onTouchEvent(event: MotionEvent?): Boolean {
            event?.let {
                MotionEventUtils.printEvent(event.action, "onTouchEvent", TAG)
            }
            return super.onTouchEvent(event)
        }
    ```

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <com.dashingqi.module.event.viewgroup.MyViewGroupOne xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">
    
        <com.dashingqi.module.event.viewgroup.MyViewGroupTwo
            android:layout_width="400dp"
            android:layout_height="400dp"
            android:layout_centerInParent="true"
            android:background="@android:color/holo_red_light">
    
            <com.dashingqi.module.event.view.MyViewOne
                android:layout_width="200dp"
                android:layout_height="200dp"
                android:layout_centerInParent="true"
                android:background="@color/black" />
    
        </com.dashingqi.module.event.viewgroup.MyViewGroupTwo>
    
    </com.dashingqi.module.event.viewgroup.MyViewGroupOne>
    ```

  - 我们就是重写了方法，然后将对应的事件给打印出来了，我们看下打印的log

    ```java
    ======================= Down ==========================
    2020-12-06 10:14:48.328 14450-14450/com.dashingqi.module.event D/MainActivity: dispatchTouchEvent -----> action_down
    2020-12-06 10:14:48.330 14450-14450/com.dashingqi.module.event D/MyViewGroupOne: dispatchTouchEvent -----> action_down
    2020-12-06 10:14:48.330 14450-14450/com.dashingqi.module.event D/MyViewGroupOne: onInterceptTouchEvent -----> action_down
    2020-12-06 10:14:48.331 14450-14450/com.dashingqi.module.event D/MyViewGroupTwo: dispatchTouchEvent -----> action_down
    2020-12-06 10:14:48.331 14450-14450/com.dashingqi.module.event D/MyViewGroupTwo: onInterceptTouchEvent -----> action_down
    2020-12-06 10:14:48.331 14450-14450/com.dashingqi.module.event D/MyViewOne: dispatchTouchEvent -----> action_down
    2020-12-06 10:14:48.332 14450-14450/com.dashingqi.module.event D/MyViewOne: onTouchEvent -----> action_down
    2020-12-06 10:14:48.332 14450-14450/com.dashingqi.module.event D/MyViewGroupTwo: onTouchEvent -----> action_down
    2020-12-06 10:14:48.333 14450-14450/com.dashingqi.module.event D/MyViewGroupOne: onTouchEvent -----> action_down
    2020-12-06 10:14:48.335 14450-14450/com.dashingqi.module.event D/MainActivity: onTouchEvent -----> action_down
      
    ======================= Move和Up ==========================
    2020-12-06 10:47:46.204 18012-18012/com.dashingqi.module.event D/MainActivity: dispatchTouchEvent -----> action_move 
    2020-12-06 10:47:46.204 18012-18012/com.dashingqi.module.event D/MainActivity: onTouchEvent -----> action_move 
    2020-12-06 10:47:46.205 18012-18012/com.dashingqi.module.event D/MainActivity: dispatchTouchEvent ------> action_up
    2020-12-06 10:47:46.205 18012-18012/com.dashingqi.module.event D/MainActivity: onTouchEvent ------> action_up
    ```

    - 我们可以看到Down事件从Activity传递到MyViewOne，经过MyViewOne的onTouchEvent 层层传递到Activity的onTouchEvent()。在这过程中 ViewGroup没有消费也没有拦截，View也没有消费最终传递给了Activity的onTouchEvent()方法来处理了。
    - MOVE和UP事件只是在Activity中传递了一下，并没有传递到ViewGroup和View中，这里留一个悬念，留作在代码分析中解释一下。

  - Activity # onTouchEvent()

    ```java
    public boolean onTouchEvent(MotionEvent event) {
            if (mWindow.shouldCloseOnTouch(this, event)) {
                finish();
                return true;
            }
    
            return false;
        }
    ```

    - Activity的onTouchEvent()方法被调用，是在它之上的View都不能处理这个事件，就会回传给它，
    - 当处于窗口之外的触摸事件，View也是不能处理的，因为根本就不在坐标范围内嘛。
    - 可以看到,onTouchEvent中调用了 Window的 shouldCloserOnTouch

  - Window # shouldCloserOnTouch()

    ```java
    public boolean shouldCloseOnTouch(Context context, MotionEvent event) {
    				// 当事件是ACTION_UP并且不在窗口之内 isOutside 为true
    				// 当isOutside为true 是该方法返回true的其中一个条件
            final boolean isOutside =
                    event.getAction() == MotionEvent.ACTION_UP && isOutOfBounds(context, event)
                    || event.getAction() == MotionEvent.ACTION_OUTSIDE;
            if (mCloseOnTouchOutside && peekDecorView() != null && isOutside) {
                return true;
            }
            return false;
        }
    ```

  - 情况二：我们让MyViewOne主动消费一下Down的事件，让它的onTouchEvent返回true

    ```kotlin
     override fun onTouchEvent(event: MotionEvent?): Boolean {
            event?.let {
                MotionEventUtils.printEvent(event.action, "onTouchEvent", TAG)
            }
            return true
        }
    ```

    - 我们在MOVE事件到来的时候进行一下拦截，我们看下打印log

      ```java
      ======================= Down ==========================
      2020-12-06 10:52:12.587 19654-19654/com.dashingqi.module.event D/MainActivity: dispatchTouchEvent -----> action_down
      2020-12-06 10:52:12.589 19654-19654/com.dashingqi.module.event D/MyViewGroupOne: dispatchTouchEvent -----> action_down
      2020-12-06 10:52:12.589 19654-19654/com.dashingqi.module.event D/MyViewGroupOne: onInterceptTouchEvent -----> action_down
      2020-12-06 10:52:12.589 19654-19654/com.dashingqi.module.event D/MyViewGroupTwo: dispatchTouchEvent -----> action_down
      2020-12-06 10:52:12.589 19654-19654/com.dashingqi.module.event D/MyViewGroupTwo: onInterceptTouchEvent -----> action_down
      2020-12-06 10:52:12.590 19654-19654/com.dashingqi.module.event D/MyViewOne: dispatchTouchEvent -----> action_down
      2020-12-06 10:52:12.590 19654-19654/com.dashingqi.module.event D/MyViewOne: onTouchEvent -----> action_down
      
      ======================= Move和Up ==========================
      2020-12-06 10:52:12.648 19654-19654/com.dashingqi.module.event D/MainActivity: dispatchTouchEvent -----> action_move 
      2020-12-06 10:52:12.649 19654-19654/com.dashingqi.module.event D/MyViewGroupOne: dispatchTouchEvent -----> action_move 
      2020-12-06 10:52:12.649 19654-19654/com.dashingqi.module.event D/MyViewGroupOne: onInterceptTouchEvent -----> action_move 
      2020-12-06 10:52:12.649 19654-19654/com.dashingqi.module.event D/MyViewGroupTwo: dispatchTouchEvent -----> action_move 
      2020-12-06 10:52:12.649 19654-19654/com.dashingqi.module.event D/MyViewGroupTwo: onInterceptTouchEvent -----> action_move 
      2020-12-06 10:52:12.650 19654-19654/com.dashingqi.module.event D/MyViewOne: dispatchTouchEvent -----> action_move 
      2020-12-06 10:52:12.650 19654-19654/com.dashingqi.module.event D/MyViewOne: onTouchEvent -----> action_move 
      
      2020-12-06 10:52:12.840 19654-19654/com.dashingqi.module.event D/MainActivity: dispatchTouchEvent ------> action_up
      2020-12-06 10:52:12.841 19654-19654/com.dashingqi.module.event D/MyViewGroupOne: dispatchTouchEvent ------> action_up
      2020-12-06 10:52:12.841 19654-19654/com.dashingqi.module.event D/MyViewGroupOne: onInterceptTouchEvent ------> action_up
      2020-12-06 10:52:12.841 19654-19654/com.dashingqi.module.event D/MyViewGroupTwo: dispatchTouchEvent ------> action_up
      2020-12-06 10:52:12.841 19654-19654/com.dashingqi.module.event D/MyViewGroupTwo: onInterceptTouchEvent ------> action_up
      2020-12-06 10:52:12.841 19654-19654/com.dashingqi.module.event D/MyViewOne: dispatchTouchEvent ------> action_up
      2020-12-06 10:52:12.841 19654-19654/com.dashingqi.module.event D/MyViewOne: onTouchEvent ------> action_up
      ```

    - 可以看到事件Down传递到了MyViewOne中并且被消费了，同时MOVE和UP事件夜传递到了MyViewGroup和MyView中，并且被MyViewOne哥i消费调了，

    - 其实上述现象，可以将MyViewOne android:clickable="true" 设置为true 也没能看到上述现象

      ```
      override fun onTouchEvent(event: MotionEvent?): Boolean {
              event?.let {
                  MotionEventUtils.printEvent(event.action, "onTouchEvent", TAG)
              }
              return super.onTouchEvent(event)
          }
      ```

    - 这里也留个悬念，留作在源码分析的时候进行讲解

  - 情况三：我们改造一下MyViewGroupTwo的onInterceptTouchEvent(),同时保留MyViewOne的clickable = true

    ```java
    override fun onInterceptTouchEvent(ev: MotionEvent?): Boolean {
            ev?.let {
    
                MotionEventUtils.printEvent(ev.action, "onInterceptTouchEvent", TAG)
                when (ev.action) {
                    MotionEvent.ACTION_DOWN -> {
                        return false
                    }
    
                    MotionEvent.ACTION_MOVE -> {
                        //拦截事件
                        return true
                    }
    
                    MotionEvent.ACTION_UP -> {
                        return true
                    }
                    else ->
                        return false
                }
    
    
            }
    
            return false
        }
    ```

    

    ```java
    ======================= Down ==========================
    2020-12-06 11:00:53.614 20373-20373/com.dashingqi.module.event D/MainActivity: dispatchTouchEvent -----> action_down
    2020-12-06 11:00:53.615 20373-20373/com.dashingqi.module.event D/MyViewGroupOne: dispatchTouchEvent -----> action_down
    2020-12-06 11:00:53.616 20373-20373/com.dashingqi.module.event D/MyViewGroupOne: onInterceptTouchEvent -----> action_down
    2020-12-06 11:00:53.616 20373-20373/com.dashingqi.module.event D/MyViewGroupTwo: dispatchTouchEvent -----> action_down
    2020-12-06 11:00:53.616 20373-20373/com.dashingqi.module.event D/MyViewGroupTwo: onInterceptTouchEvent -----> action_down
    2020-12-06 11:00:53.616 20373-20373/com.dashingqi.module.event D/MyViewOne: dispatchTouchEvent -----> action_down
    2020-12-06 11:00:53.617 20373-20373/com.dashingqi.module.event D/MyViewOne: onTouchEvent -----> action_down
    
    
    ======================= Move 和 Up ==========================
    
    2020-12-06 11:00:53.690 20373-20373/com.dashingqi.module.event D/MainActivity: dispatchTouchEvent -----> action_move 
    2020-12-06 11:00:53.691 20373-20373/com.dashingqi.module.event D/MyViewGroupOne: dispatchTouchEvent -----> action_move 
    2020-12-06 11:00:53.691 20373-20373/com.dashingqi.module.event D/MyViewGroupOne: onInterceptTouchEvent -----> action_move 
    2020-12-06 11:00:53.691 20373-20373/com.dashingqi.module.event D/MyViewGroupTwo: dispatchTouchEvent -----> action_move 
    2020-12-06 11:00:53.691 20373-20373/com.dashingqi.module.event D/MyViewGroupTwo: onInterceptTouchEvent -----> action_move 
    2020-12-06 11:00:53.691 20373-20373/com.dashingqi.module.event D/MyViewOne: dispatchTouchEvent -----> action_cancel
    2020-12-06 11:00:53.691 20373-20373/com.dashingqi.module.event D/MyViewOne: onTouchEvent -----> action_cancel
      
    2020-12-06 11:00:53.714 20373-20373/com.dashingqi.module.event D/MainActivity: dispatchTouchEvent -----> action_move 
    2020-12-06 11:00:53.714 20373-20373/com.dashingqi.module.event D/MyViewGroupOne: dispatchTouchEvent -----> action_move 
    2020-12-06 11:00:53.714 20373-20373/com.dashingqi.module.event D/MyViewGroupOne: onInterceptTouchEvent -----> action_move 
    2020-12-06 11:00:53.714 20373-20373/com.dashingqi.module.event D/MyViewGroupTwo: dispatchTouchEvent -----> action_move 
    2020-12-06 11:00:53.714 20373-20373/com.dashingqi.module.event D/MyViewGroupTwo: onTouchEvent -----> action_move 
    2020-12-06 11:00:53.715 20373-20373/com.dashingqi.module.event D/MainActivity: onTouchEvent -----> action_move 
    
    2020-12-06 11:00:53.907 20373-20373/com.dashingqi.module.event D/MainActivity: dispatchTouchEvent ------> action_up
    2020-12-06 11:00:53.907 20373-20373/com.dashingqi.module.event D/MyViewGroupOne: dispatchTouchEvent ------> action_up
    2020-12-06 11:00:53.908 20373-20373/com.dashingqi.module.event D/MyViewGroupOne: onInterceptTouchEvent ------> action_up
    2020-12-06 11:00:53.908 20373-20373/com.dashingqi.module.event D/MyViewGroupTwo: dispatchTouchEvent ------> action_up
    2020-12-06 11:00:53.908 20373-20373/com.dashingqi.module.event D/MyViewGroupTwo: onTouchEvent ------> action_up
    2020-12-06 11:00:53.909 20373-20373/com.dashingqi.module.event D/MainActivity: onTouchEvent ------> action_up
    ```

    - 这里我们重点分析一下 第一个Move事件到来时，MyViewGroupTwo拦截了这个事件，并且MyViewOne 中dispatchTouchEvent和onTouchEvent中都收到了 cancel事件
    - 当第二个Move以及之后的Move事件到达的时候，MyViewGroupTwo同样拦截了，此时调用了自己的onTouchEvent，由于自己的onTouchEvent并没有消费这个事件，所以将事件向上传递，但是并没有回传给MyViewGroupOne的onTouchEvent而是Activity的onTouchEvent，其实这个地方我的理解就是，第一个Move事件 MyViewGroupOne没有拦截也就是不会将事件传递给自己的onTouchEvent方法处理，那么该序列中的其他Move事件到达时都不会传递给onTouchEVent方法处理，所以回传时就不会交给它来处理了。同样Up事件也是同理的。

  - 情况四：我们在MyViewGroupOne的onTouchEvent方法中使用requestDisallowInterceptTouchEvent(true)，同时保持情况三的条件

    ```java
    override fun onTouchEvent(event: MotionEvent?): Boolean {
            event?.let {
                MotionEventUtils.printEvent(event.action, "onTouchEvent", TAG)
                when(event.action){
                    MotionEvent.ACTION_DOWN ->{
                        parent.requestDisallowInterceptTouchEvent(true)
                    }
                }
            }
            return super.onTouchEvent(event)
        }
    ```

    - 在onTouchEvent() 的Down事件中 我们通知父View 当第一个Move事件到来时，不要拦截 要传递给我。

    ```java
    ======================= Down ==========================
    
    2020-12-06 11:23:33.468 21432-21432/com.dashingqi.module.event D/MainActivity: dispatchTouchEvent -----> action_down
    2020-12-06 11:23:33.468 21432-21432/com.dashingqi.module.event D/MyViewGroupOne: dispatchTouchEvent -----> action_down
    2020-12-06 11:23:33.468 21432-21432/com.dashingqi.module.event D/MyViewGroupOne: onInterceptTouchEvent -----> action_down
    2020-12-06 11:23:33.468 21432-21432/com.dashingqi.module.event D/MyViewGroupTwo: dispatchTouchEvent -----> action_down
    2020-12-06 11:23:33.468 21432-21432/com.dashingqi.module.event D/MyViewGroupTwo: onInterceptTouchEvent -----> action_down
    2020-12-06 11:23:33.468 21432-21432/com.dashingqi.module.event D/MyViewOne: dispatchTouchEvent -----> action_down
    2020-12-06 11:23:33.469 21432-21432/com.dashingqi.module.event D/MyViewOne: onTouchEvent -----> action_down
    
    ======================= Move 和 Up ==========================
    
    2020-12-06 11:23:33.517 21432-21432/com.dashingqi.module.event D/MainActivity: dispatchTouchEvent -----> action_move 
    2020-12-06 11:23:33.517 21432-21432/com.dashingqi.module.event D/MyViewGroupOne: dispatchTouchEvent -----> action_move 
    2020-12-06 11:23:33.518 21432-21432/com.dashingqi.module.event D/MyViewGroupTwo: dispatchTouchEvent -----> action_move 
    2020-12-06 11:23:33.518 21432-21432/com.dashingqi.module.event D/MyViewOne: dispatchTouchEvent -----> action_move 
    2020-12-06 11:23:33.518 21432-21432/com.dashingqi.module.event D/MyViewOne: onTouchEvent -----> action_move 
    
    
    2020-12-06 11:23:33.701 21432-21432/com.dashingqi.module.event D/MainActivity: dispatchTouchEvent ------> action_up
    2020-12-06 11:23:33.702 21432-21432/com.dashingqi.module.event D/MyViewGroupOne: dispatchTouchEvent ------> action_up
    2020-12-06 11:23:33.702 21432-21432/com.dashingqi.module.event D/MyViewGroupTwo: dispatchTouchEvent ------> action_up
    2020-12-06 11:23:33.702 21432-21432/com.dashingqi.module.event D/MyViewOne: dispatchTouchEvent ------> action_up
    2020-12-06 11:23:33.702 21432-21432/com.dashingqi.module.event D/MyViewOne: onTouchEvent ------> action_up
    ```

    - 通过日志我们看到，当MOVE和Up事件到达时候，MyViewGroup并没有执行onInterceptTouchEvent方法，直接跳过它了，这就是requestDisallowInterceptTouchEvent()的作用效果

  - 好接下来我们就通过源码开进一步揭开事件分发的神秘面纱，也能针对上述的情况做一下分析。

  #### 源码解析

  - Android的事件经过 系统进程的处理过程会传递到 应用进程的 Activity中 进而 向下传递，接下来我们就看下 Activity的 dispatchTouchEvent()

  ##### Activity # dispatchTouchEvent()

  ```java
  public boolean dispatchTouchEvent(MotionEvent ev) {
          if (ev.getAction() == MotionEvent.ACTION_DOWN) {
              onUserInteraction();
          }
          if (getWindow().superDispatchTouchEvent(ev)) {
              return true;
          }
          return onTouchEvent(ev);
      }
  ```

  

  - 当DOWN事件到来的时候，回调用到 onUserInteraction(),该方式是一个空实现，当你的手机按住home 会回调这个方法
  - 当 getWindow().superDispatchToucheEvent() 返回true ,该方法返回true，否则的话 就调用 onTouchEvent()方法来处理这个事件。

  ###### getWindow(),superDispatchTouchEvent()

  - getWindow()获取到的是一个 Window对象，Window是一个抽象类，A android中它的唯一实现子类是PhoneWindow()

  ###### PhoneWindow # superDispatchTouchEvent()

  ```java
  @Override
      public boolean superDispatchTouchEvent(MotionEvent event) {
          return mDecor.superDispatchTouchEvent(event);
      }
  ```

  - 其中 mDecor是DecorView类型的，DecorView是作为我们设置布局的父布局，继承FrameLayout

  ###### Decorview # superDispatchTouchEvent()

  ```java
  public boolean superDispatchTouchEvent(MotionEvent event) {
          return super.dispatchTouchEvent(event);
      }
  ```

  - 内部调用了父类的 dispatchTouchEvent(),FrameLayout作为DecorView的父类，内部没有实现dispatchTouchEvent(),而它的父类ViewGroup内部实现了 dispatchTouchEvent()
  - 这也是 Activity将事件传递到ViewGrouop的流程

  ###### ViewGroup # dispatchTouchEvent()

  - ViewGroup的dispatchTouchEvent()方法还是很长的，我们这里通过代码的功能拆分一下

  - 同时我们可以发现 该方法返回值是 handled 默认是 false，

  - 该方法最重要的逻辑都在 这个if条件成立的情况下

    ```java
     if (onFilterTouchEventForSecurity(ev)){.....}
    ```

  ###### onFilterTouchEventForSecurity()

  ```java
      public boolean onFilterTouchEventForSecurity(MotionEvent event) {
          //noinspection RedundantIfStatement
          if ((mViewFlags & FILTER_TOUCHES_WHEN_OBSCURED) != 0
                  && (event.getFlags() & MotionEvent.FLAG_WINDOW_IS_OBSCURED) != 0) {
              // Window is obscured, drop this touch.
              return false;
          }
          return true;
      }
  ```

  - 该方法主要是用来检查当前控件是否被遮蔽了，隐藏了，如果是的话就返回false，否则就返回true
  - 当返回false，dispatchTouchEvent()就返回false，就不做事件的分发了。

  ###### 然后我们走进了if语句中了，就下来第一个功能代码块如下

  ```java
    if (actionMasked == MotionEvent.ACTION_DOWN) {
        // Throw away all previous state when starting a new touch gesture.
        // The framework may have dropped the up or cancel event for the previous gesture
        // due to an app switch, ANR, or some other state change.
        cancelAndClearTouchTargets(ev);
        resetTouchState();
       }
  ```

  - 当前是Down事件的话，就执行 resetTouchState()

  ```java
   private void resetTouchState() {
          //清空链表
          clearTouchTargets();
          resetCancelNextUpFlag(this);
          //重置了 mGroupFlags 的状态
          mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
          mNestedScrollAxes = SCROLL_AXIS_NONE;
      }
      
   //内部执行了 clearTouchTaegets()
   
   private void clearTouchTargets() {
          TouchTarget target = mFirstTouchTarget;
          if (target != null) {
              do {
                  TouchTarget next = target.next;
                  target.recycle();
                  target = next;
              } while (target != null);
              //mFirstTouchTarget变量是指向了 链表的第一个节点
            	// mFirstTouchTarget 是TouchTarget类型的
              mFirstTouchTarget = null;
          }
      }
  ```

  - 当Down事件来的时候，会作为一个事件序列的开始，它会把事件链表给清空，同时将mFirstTouchTarget变量置为null
  - 并且会把mGroupFlags的状态重置，默认 mGroupFlags为0，mGroupFlags变量值会直接影响 内部onInterceptTouchEvent方法的执行

  ###### 谈下 TouchTarget

  ```java
   public static TouchTarget obtain(@NonNull View child, int pointerIdBits) {
     
     private static final int MAX_RECYCLED = 32;
          private static final Object sRecycleLock = new Object[0];
     			
          //用来记录最近一次被回收的TouchTarget，用于复用
          private static TouchTarget sRecycleBin;
     			// 被回收的次数
          private static int sRecycledCount;
  
          public static final int ALL_POINTER_IDS = -1; // all ones
  
          // 当前最近一次消费事件的子View
          @UnsupportedAppUsage
          public View child;
  
          // The combined bit mask of pointer ids for all pointers captured by the target.
          public int pointerIdBits;
  
          // The next target in the target list.
          public TouchTarget next;
     
              if (child == null) {
                  throw new IllegalArgumentException("child must be non-null");
              }
  
              final TouchTarget target;
              synchronized (sRecycleLock) {
                  if (sRecycleBin == null) {
                      target = new TouchTarget();
                  } else {
                      target = sRecycleBin;
                      sRecycleBin = target.next;
                       sRecycledCount--;
                      target.next = null;
                  }
              }
              target.child = child;
              target.pointerIdBits = pointerIdBits;
              return target;
          }
  
          public void recycle() {
              if (child == null) {
                  throw new IllegalStateException("already recycled once");
              }
  
              synchronized (sRecycleLock) {
                  if (sRecycledCount < MAX_RECYCLED) {
                      next = sRecycleBin;
                      sRecycleBin = this;
                      sRecycledCount += 1;
                  } else {
                      next = null;
                  }
                  child = null;
              }
          }
  ```

  - TouchTarget内部是一个链表的数据结构，有两个方法 obtain 和recycle
  - obtain()方法用于存储当前消费事件的View，并将它置于链表的头部
  - recycle 用于回收链表头部的View，将它置为null
  - 我们的mFirstTouchTarget就是指向了链表的头部，内部的child就是本次事件序列中事件的消费View

  ###### 接下来就是决定 onInterceptTouchEvent事件是否执行的代码块了

  ```java
   final boolean intercepted;
              if (actionMasked == MotionEvent.ACTION_DOWN
                      || mFirstTouchTarget != null) {
                	// 
                  final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                  if (!disallowIntercept) {
                      intercepted = onInterceptTouchEvent(ev);
                      ev.setAction(action); // restore action in case it was changed
                  } else {
                      intercepted = false;
                  }
              } else {
                  // There are no touch targets and this action is not an initial down
                  // so this view group continues to intercept touches.
                  intercepted = true;
              }
  ```

  - 当是Down事件或者 上一次的事件被子View消费了就会进入这个if条件语句中
  - mGroupFlags  和 FlAG_DISALLOW_INTERCEPT 进行与运算 ；当是DOWN事件的话 disallowIntercept 为false，就会执行onInterceptTouchEvent方法 否则的话 intercepted 赋值为false
  - 当既不是Down事件又没有子View消费事件的话就将 intercepted赋值为true

  ###### 看下 onInterceptTouchEvent()

  ```java
   public boolean onInterceptTouchEvent(MotionEvent ev) {
          if (ev.isFromSource(InputDevice.SOURCE_MOUSE)
                  && ev.getAction() == MotionEvent.ACTION_DOWN
                  && ev.isButtonPressed(MotionEvent.BUTTON_PRIMARY)
                  && isOnScrollbarThumb(ev.getX(), ev.getY())) {
              return true;
          }
          return false;
      }
  ```

  - 正常情况下 onInterceptTouchEvent就是返回true的
  - 在执行完上述代码块之后 如果不决定拦截的话就开始决定分发给哪个子View了

  ###### 遍历子View，寻找符合条件的子View进行事件的分发

  ```java
  if (!canceled && !intercepted) {
                  View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                          ? findChildWithAccessibilityFocus() : null;
  								//分析1
                  if (actionMasked == MotionEvent.ACTION_DOWN
                          || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                          || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                      final int actionIndex = ev.getActionIndex(); // always 0 for down
                      final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                              : TouchTarget.ALL_POINTER_IDS;
                    
                      removePointersFromTouchTargets(idBitsToAssign);
  
                      final int childrenCount = mChildrenCount;
                      if (newTouchTarget == null && childrenCount != 0) {
                          final float x =
                                  isMouseEvent ? ev.getXCursorPosition() : ev.getX(actionIndex);
                          final float y =
                                  isMouseEvent ? ev.getYCursorPosition() : ev.getY(actionIndex);
                          final ArrayList<View> preorderedList = buildTouchDispatchChildList();
                          final boolean customOrder = preorderedList == null
                                  && isChildrenDrawingOrderEnabled();
                          final View[] children = mChildren;
                        
                          // 分析2
                          for (int i = childrenCount - 1; i >= 0; i--) {
                            
                              final int childIndex = getAndVerifyPreorderedIndex(
                                      childrenCount, i, customOrder);
                              final View child = getAndVerifyPreorderedView(
                                      preorderedList, children, childIndex);
                            
                            	// 分析2.1
                              if (!child.canReceivePointerEvents()
                                      || !isTransformedTouchPointInView(x, y, child, null)) {
                                  continue;
                              }
                              newTouchTarget = getTouchTarget(child);
                              if (newTouchTarget != null) {
                                  newTouchTarget.pointerIdBits |= idBitsToAssign;
                                  break;
                              }
                              resetCancelNextUpFlag(child);
                            	//分析3
                              if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) { 
                                  // Child wants to receive touch within its bounds.
                                  mLastTouchDownTime = ev.getDownTime();
                                  if (preorderedList != null) {
                                      // childIndex points into presorted list, find original index
                                      for (int j = 0; j < childrenCount; j++) {
                                          if (children[childIndex] == mChildren[j]) {
                                              mLastTouchDownIndex = j;
                                              break;
                                          }
                                      }
                                  } else {
                                      mLastTouchDownIndex = childIndex;
                                  }
                                  mLastTouchDownX = ev.getX();
                                  mLastTouchDownY = ev.getY();
                                	//分析4
                                  newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                  alreadyDispatchedToNewTouchTarget = true;
                                  break;
                              }
                              ev.setTargetAccessibilityFocus(false);
                          }
                          if (preorderedList != null) preorderedList.clear();
                      }
                      if (newTouchTarget == null && mFirstTouchTarget != null) {
                          // Did not find a child to receive the event.
                          // Assign the pointer to the least recently added target.
                          newTouchTarget = mFirstTouchTarget;
                          while (newTouchTarget.next != null) {
                              newTouchTarget = newTouchTarget.next;
                          }
                          newTouchTarget.pointerIdBits |= idBitsToAssign;
                      }
                  }
              }
  ```

  - 能走到这个代码块说明 不会做拦截的操作。

  - 分析1:当是Down事件或者是MOVE事件的话 就进入到代码块中

  - 分析2:此时会遍历子View，然后去执行2.1的逻辑

    - 分析2.1 在这个地方回去验证当前的View是否符合分发的条件，符合分发的条件是
      - 当前的View没有做动画
      - 并且当事件的区域是在该子View的区域中

  - 分析3:此时会调用dispatchTransformedTouchEvent()方法，将符合条件的View传进去

    ```java
     private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
                View child, int desiredPointerIdBits) {
       
       					// 。。。。。。。省略之前的逻辑判断
            if (child == null) {
                //当child为null的时候，就调用ViewGroup的dispatchTouchEvent 也就是View的dispatchTouchEvent()
                handled = super.dispatchTouchEvent(transformedEvent);
            } else {
                final float offsetX = mScrollX - child.mLeft;
                final float offsetY = mScrollY - child.mTop;
                transformedEvent.offsetLocation(offsetX, offsetY);
                if (! child.hasIdentityMatrix()) {
                    transformedEvent.transform(child.getInverseMatrix());
                }
                //child不为空的时候 调用了View的dispatchTouchEvent()
                handled = child.dispatchTouchEvent(transformedEvent);
            }
            // Done.
            transformedEvent.recycle();
            return handled;
        }
    ```

    - 所以当符合条件的View 不管是否为null 都会将是将传递给View的dispatchTouchEvent()方法进行处理
    - 这也是事件从ViewGroup传递到View的dispatchTouchEvent

  - 在分析View的dispatchTouchEvent方法之前我们看下分析4 看下mFirstTouchTarget变量的处理

  - 分析4:此时调用了addTouchTarget()

    ```java
    private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
            final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
            target.next = mFirstTouchTarget;
            mFirstTouchTarget = target;
            return target;
        }
    ```

    - 能走到这个方法说明 子View的dispatchTouchEvent方法已经返回true说明要消费它了
    - 该方法内部调用了obtain()创建了一个TouchTarget的变量，它的child指向了消费该事件的View
    - 同时将mFirstTouchTarget指向了这个链表头部的TouchTarget

  - 还记得上文中情况4的吗，我们在子View中使用 requestDisallowInterceptTouchEvent(true)使得ViewGroup中的onInterceptTouchEvent()方法并没有执行，我们看下该方法

  ###### requestDisallowInterceptTouchEvent

  ```java
  public void requestDisallowInterceptTouchEvent(boolean disallowIntercept) {
  
          if (disallowIntercept == ((mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0)) {
              // We're already in this state, assume our ancestors are too
              return;
          }
  				
          if (disallowIntercept) {
              mGroupFlags |= FLAG_DISALLOW_INTERCEPT;
          } else {
              mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
          }
  
          // Pass it up to our parent
          if (mParent != null) {
              mParent.requestDisallowInterceptTouchEvent(disallowIntercept);
          }
      }
  ```

  - 该方法是就是子View用来告知父View下一个事件要不要拦截的作用
  - 当disallowIntercept = true的时候，说明告知父View不要拦截，此时mGroupFlags ==  FLAG_DISALLOW_INTERCEPT 
    - 此时上文中的 disallowIntercept == true 就不会去执行onInterceptTouchEvent
  - 当disallowIntercept = false的时候，告知父View下个事件不管你怎么处理 都不要传递给我吧 此时mGroup == 0
    - 此时上文中的disallowIntercept == false，会走自己的onInterceptTouchEvent方法。
  - 然后在从该ViewGroup 告知上一个ViewGroup 子View的态度

  #### View # dispatchTouchEvent()

  - View的dispatchTouchEvent()方法相对来说就比较简单，我们看下

  ```java
  public boolean dispatchTouchEvent(MotionEvent event) {
           //..... 省略了一些逻辑判断
          if (onFilterTouchEventForSecurity(event)) {
              if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                  result = true;
              }
              //noinspection SimplifiableIfStatement
              ListenerInfo li = mListenerInfo;
            	//分析1
              if (li != null && li.mOnTouchListener != null
                      && (mViewFlags & ENABLED_MASK) == ENABLED
                      && li.mOnTouchListener.onTouch(this, event)) {
                  result = true;
              }
  						
              //分析2
              if (!result && onTouchEvent(event)) {
                  result = true;
              }
          }
  
          if (!result && mInputEventConsistencyVerifier != null) {
              mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
          }
  
          if (actionMasked == MotionEvent.ACTION_UP ||
                  actionMasked == MotionEvent.ACTION_CANCEL ||
                  (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
              stopNestedScroll();
          }
  				//结果返回的是result值
          return result;
      }
  
  ```

  - 分析1:  当给控件设置了点击事件，并且当前控件是可用的 并且 给控件设置了 ToucheListener 并且onTouch返回true，那么 result为true。
  - 分析2:当result为false 并且 onTouchEvent()为true，那么就返回true，那么就消费掉这个事件了。
  - 可以看的出来，当给某一个控件设置了触摸事件 并且 onTouchEvent返回true，那么onTouchEvent方法就不会被执行了，发挥false 没啥事。

  ###### View # onTouchEvent()

  - 通过源码我们可以看到 View的onTouchEvent默认是返回false的。

    ```java
    public boolean onTouchEvent(MotionEvent event) {
      	//分析1
     	 final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
                    || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                    || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;
    	 //分析2
       if ((viewFlags & ENABLED_MASK) == DISABLED) {
           if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
             setPressed(false);
           }
           mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
           return clickable;
         }
      
      //分析3
      if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
                switch (action) {
                    case MotionEvent.ACTION_UP:
                        mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                        if ((viewFlags & TOOLTIP) == TOOLTIP) {
                            handleTooltipUp();
                        }
                        if (!clickable) {
                            removeTapCallback();
                            removeLongPressCallback();
                            mInContextButtonPress = false;
                            mHasPerformedLongPress = false;
                            mIgnoreNextUpEvent = false;
                            break;
                        }
                        boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                        if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                           
                            boolean focusTaken = false;
                            if (isFocusable() && isFocusableInTouchMode() && !isFocused()) {
                                focusTaken = requestFocus();
                            }
    
                            if (prepressed) {
                             
                                setPressed(true, x, y);
                            }
    
                            if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                                // This is a tap, so remove the longpress check
                                removeLongPressCallback();
    
                                // Only perform take click actions if we were in the pressed state
                                if (!focusTaken) {
                                    if (mPerformClick == null) {
                                        mPerformClick = new PerformClick();
                                    }
                                    if (!post(mPerformClick)) {
                                      	//分析4:
                                        performClickInternal();
                                    }
                                }
                            }
    
                            if (mUnsetPressedState == null) {
                                mUnsetPressedState = new UnsetPressedState();
                            }
    
                            if (prepressed) {
                                postDelayed(mUnsetPressedState,
                                        ViewConfiguration.getPressedStateDuration());
                            } else if (!post(mUnsetPressedState)) {
                                // If the post failed, unpress right now
                                mUnsetPressedState.run();
                            }
    
                            removeTapCallback();
                        }
                        mIgnoreNextUpEvent = false;
                        break;
    
                    case MotionEvent.ACTION_DOWN:
                        if (event.getSource() == InputDevice.SOURCE_TOUCHSCREEN) {
                            mPrivateFlags3 |= PFLAG3_FINGER_DOWN;
                        }
                        mHasPerformedLongPress = false;
    
                        if (!clickable) {
                            checkForLongClick(
                                    ViewConfiguration.getLongPressTimeout(),
                                    x,
                                    y,
                                    TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__LONG_PRESS);
                            break;
                        }
    
                        if (performButtonActionOnTouchDown(event)) {
                            break;
                        }
    
                        // Walk up the hierarchy to determine if we're inside a scrolling container.
                        boolean isInScrollingContainer = isInScrollingContainer();
    
                        if (isInScrollingContainer) {
                            mPrivateFlags |= PFLAG_PREPRESSED;
                            if (mPendingCheckForTap == null) {
                                mPendingCheckForTap = new CheckForTap();
                            }
                            mPendingCheckForTap.x = event.getX();
                            mPendingCheckForTap.y = event.getY();
                            postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());
                        } else {
                            // Not inside a scrolling container, so show the feedback right away
                            setPressed(true, x, y);
                            checkForLongClick(
                                    ViewConfiguration.getLongPressTimeout(),
                                    x,
                                    y,
                                    TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__LONG_PRESS);
                        }
                        break;
    
                    case MotionEvent.ACTION_CANCEL:
                        if (clickable) {
                            setPressed(false);
                        }
                        removeTapCallback();
                        removeLongPressCallback();
                        mInContextButtonPress = false;
                        mHasPerformedLongPress = false;
                        mIgnoreNextUpEvent = false;
                        mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                        break;
    
                    case MotionEvent.ACTION_MOVE:
                        if (clickable) {
                            drawableHotspotChanged(x, y);
                        }
    
                        final int motionClassification = event.getClassification();
                        final boolean ambiguousGesture =
                                motionClassification == MotionEvent.CLASSIFICATION_AMBIGUOUS_GESTURE;
                        int touchSlop = mTouchSlop;
                        if (ambiguousGesture && hasPendingLongPressCallback()) {
                            if (!pointInView(x, y, touchSlop)) {
                                // The default action here is to cancel long press. But instead, we
                                // just extend the timeout here, in case the classification
                                // stays ambiguous.
                                removeLongPressCallback();
                                long delay = (long) (ViewConfiguration.getLongPressTimeout()
                                        * mAmbiguousGestureMultiplier);
                                // Subtract the time already spent
                                delay -= event.getEventTime() - event.getDownTime();
                                checkForLongClick(
                                        delay,
                                        x,
                                        y,
                                        TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__LONG_PRESS);
                            }
                            touchSlop *= mAmbiguousGestureMultiplier;
                        }
    
                        // Be lenient about moving outside of buttons
                        if (!pointInView(x, y, touchSlop)) {
                            // Outside button
                            // Remove any future long press/tap checks
                            removeTapCallback();
                            removeLongPressCallback();
                            if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
                                setPressed(false);
                            }
                            mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
                        }
    
                        final boolean deepPress =
                                motionClassification == MotionEvent.CLASSIFICATION_DEEP_PRESS;
                        if (deepPress && hasPendingLongPressCallback()) {
                            // process the long click action immediately
                            removeLongPressCallback();
                            checkForLongClick(
                                    0 /* send immediately */,
                                    x,
                                    y,
                                    TOUCH_GESTURE_CLASSIFIED__CLASSIFICATION__DEEP_PRESS);
                        }
    
                        break;
                }
    
                return true;
            }
      
      return false
    }
    ```

  - 分析1:当控件是可点击的（ clickable = “true”或者设置了点击事件，比如Button默认即使clickable的，TextView就不是），或者长按点击事件是true  那么 clickable = true

  - 分析2: 当控件是不可用的话，直接将 clickable值返回，但是不做任何事件的响应。

  - 分析3:当控件是可点击的话，然后根据不同的事件，做出不同的处理，并且只要满足这个条件那么onTouchEvent就返回true，那么就表明消费了这个事件。

    - 这时候就可以解释给MyViewOne 设置成了 clickable =“true”属性之后 能接受到 MOVE和Up事件了，因为此时表明DOWN事件被MyViewOne给消费了，那么mFirstTouchTarget不为null，那么下一个事件下发的时候就会根据条件 分发到了当前能消费事件的View中。

  - 分析4: performClick()

    ```java
    public boolean performClick() {
            notifyAutofillManagerOnClick();
            final boolean result;
      			//分析5
            final ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnClickListener != null) {
                playSoundEffect(SoundEffectConstants.CLICK);
              	// 当给控件设置了点击事件并且当前事件是可用的，那么就会调用onClick() 并且将 result 设置为true
                li.mOnClickListener.onClick(this);
                result = true;
            } else {
                result = false;
            }
    
            sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
    
            notifyEnterOrExitForAutoFillIfNeeded(true);
    
            return result;
        }
    
    //设置点击事件
    public void setOnClickListener(@Nullable OnClickListener l) {
            if (!isClickable()) {
                setClickable(true);
            }
            getListenerInfo().mOnClickListener = l;
        }
    ListenerInfo getListenerInfo() {
            if (mListenerInfo != null) {
                return mListenerInfo;
            }
      			// 创建了ListenerInfo的对象并赋值给了 mListenerInfo
            mListenerInfo = new ListenerInfo();
            return mListenerInfo;
    }
    ```
  -   分析5: 这里将mListenerInfo 赋值给了 li ,mListenerInfo是在setOnclickListener()时候赋值的
      
      -  可以看出 要想调用到你设置的点击事件，首先当前控件是可点击的并且是可用的，然后如果设置了触摸事件，那么onTouch()不能返回true，然后才会调用到onTouchEvent()方法，继而在UP事件到来的时候会去回调onClick方法。

#### 总结

- 上面copy了这么多源代码下面就以一张图来表明一下事件分发的流程

![Andorid事件分发的原理图.png](https://upload-images.jianshu.io/upload_images/4997216-8a7b3e17f0122f51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 以上是一Down事件为基础做了一次事件分发的原理分析
  - 一个事件的分发可以看作递归的流程，事件从Activity进行下发，属于递的流程，如果当中有任何一个View将事件进行了消费那么就提前结束了递的流程，
  - 如果到头来没有任何View对这个事件有意思，就开始的归的流程依次调用onTouchEvent方法，如果能消费就消费，不能的话就归到Activity的onTouchEvent方法中，

###### Move和Up事件的流程解析

- 针对开头列出的几种情况，我们看下 Move和Up的事件递归流程

###### 情况1

![Android事件分发情况1.png](https://upload-images.jianshu.io/upload_images/4997216-9cacafb580c1c32f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 情况2

![Andorid事件分发情况2.png](https://upload-images.jianshu.io/upload_images/4997216-93a8d7b65376d41b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 情况3

![Android事件分发情况三.png](https://upload-images.jianshu.io/upload_images/4997216-170d2015d6af316b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 情况4

![Andorid事件分发情况4.png](https://upload-images.jianshu.io/upload_images/4997216-9830aa2da0fe1075.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 总结：
  - 在递的流程中，如果是ViewGroup拦截了Down事件，那么Move和Up只能到达这个ViewGroup，要不要归还要看自己的onTouchEvent能不能消费 能的话就不归了，不能就开启归的流程直到能消费这个事件为止。
  - 如果是View消费这个Down事件，同样Move和Up能达到View，在View中进行消费
  - 如果重Activity ---> 到ViewGroup ----> View 都没有消费Down事件，那么Down会走完整个递归的流程，而Move和Up直接从Activity的dispatchTouch 传递到onTouchEvent中进行处理了。
  - [本文中练习的代码](https://github.com/dashingqi/KotlinAndroid/blob/master/module-event/src/main/java/com/dashingqi/module/event/MainActivity.kt)