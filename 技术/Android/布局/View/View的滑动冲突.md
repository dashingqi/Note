### 常见滑动冲突的场景

##### 外部滑动和内部滑动方向不一致

- 具体描述

​          ViewPager和Fragment组合使用，ViewPager可以左右滑动切换页面，Fragment中使用ListView可以上下滑动，这时就出现滑动冲突；当时ViewPager内部解决这个问题了，把ViewPager换成ScrollView就会出现滑动冲突。

##### 外部滑动和内部滑动方向一致

- 具体描述

  这种情况是外部和内部都可以滑动，并且滑动的方向是一致，同时可以左右滑动或者上下滑动，这时系统不知道处理谁的事件，这时就发生滑动冲突了。要么只有一层在滑动，要么两层都在滑动显得很卡顿。

##### 以上两种情况的嵌套

- 具体描述

  这种是以上两种情况的嵌套

### 滑动冲突的解决方式

##### 外部拦截法

- 外部拦截法是通过父容器拦截处理的，通过重写父容器的onInterceptTouchEvent()方法，达到拦截条件就去拦截，否则将事件传递给子View这样就解决事件的拦截了。

```java
package com.dashingqi.viewgroupinterceptevent;

import android.content.Context;
import android.util.AttributeSet;
import android.view.MotionEvent;
import android.view.ViewGroup;

/**
 * @author Dashingqi
 * @description:
 * @date :2020-02-14 11:36
 */
public class MyGroup extends ViewGroup {
    private float mLastX = 0.0f;
    private float mLastY = 0.0f;

    public MyGroup(Context context) {
        super(context);
    }

    public MyGroup(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MyGroup(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {

    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean interceptTouchEvent = false;
        float x = ev.getX();
        float y = ev.getY();
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN: {
                //如果ACTION_DOWN事件被拦截那么之后的ACTION_MOVE和ACTION_UP都会被拦截。交给父容器来处理
                interceptTouchEvent = false;
            }
            break;
            case MotionEvent.ACTION_MOVE: {
                if (x > 0) {
                    interceptTouchEvent = true;

                } else {
                    interceptTouchEvent = false;
                }
            }
            break;
            case MotionEvent.ACTION_UP: {
                interceptTouchEvent = false;

            }
            break;
        }
        mLastX = x;
        mLastY = y;
        return interceptTouchEvent;

    }
}

```

#####内部拦截法

- 内部拦截是父容器不做事件拦截处理，所有事件都传递给子元素进行处理，如果事件子元素需要就消费这个事件，如果子元素不需要这个事件那么交给父容器进行处理。

- 需要配合requestDisallowInterceptTouchEvent()方法进行使用。

  ```java
  package com.dashingqi.viewgroupinterceptevent;
  
  import android.content.Context;
  import android.util.AttributeSet;
  import android.view.MotionEvent;
  import android.view.View;
  
  import androidx.annotation.Nullable;
  
  /**
   * @author Dashingqi
   * @description:
   * @date :2020-02-14 15:02
   */
  public class MyView extends View {
      private float mLastX = 0.0f;
      private float mLastY = 0.0f;
  
      public MyView(Context context) {
          super(context);
      }
  
      public MyView(Context context, @Nullable AttributeSet attrs) {
          super(context, attrs);
      }
  
      public MyView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
          super(context, attrs, defStyleAttr);
      }
  
      @Override
      public boolean dispatchTouchEvent(MotionEvent event) {
          float x = event.getX();
          float y = event.getY();
          switch (event.getAction()) {
              case MotionEvent.ACTION_DOWN: {
                  getParent().requestDisallowInterceptTouchEvent(true);
  
              }
              break;
              case MotionEvent.ACTION_MOVE: {
                  float delayX = x - mLastY;
                  float delayY = y - mLastY;
  
                  //父容器需要此类点击事件
                  if (delayX > 0) {
                      getParent().requestDisallowInterceptTouchEvent(false);
                  }
              }
              break;
              case MotionEvent.ACTION_UP: {
  
              }
              break;
          }
  
          mLastX = x;
          mLastY = y;
          return super.dispatchTouchEvent(event);
      }
  }
  
  //父容器的 onInterceptTouchEvent() 修改如下
  @Override
      public boolean onInterceptTouchEvent(MotionEvent ev) {
          //内部拦截处理如下
          if (ev.getAction() == MotionEvent.ACTION_DOWN) {
              return false;
          } else {
              return true;
          }
  
      }
  ```

  