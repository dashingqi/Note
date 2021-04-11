#### 参考文章
```
//这篇文章分析的不错
https://mp.weixin.qq.com/s/bOl9BRS9YXGuqYxZIWvSDg

//这篇文章很经典
https://www.jianshu.com/p/38015afcdb58

https://www.jianshu.com/p/e99b5e8bd67b


```

#### 事件分发的顺序
- Activity ---> ViewGroup ---> View
> Activity作为事件开始的分发者，如果Activity把事件给拦截了，那么屏幕都不会响应事件这不是我们想要的效果。

> View作为事件的接受着要么进行事件的消费，要么把事件回传给父容器，没有有事件拦截功能的必要。

###### Activity的事件到达ViewGroup
- Activity中（dispatchTouchEvent()---> getWindow().superDispatchTouchEvent()）
- 其中 getWindow()获取到的是Window对象，Window是抽象类，调用的是抽象方法，PhoneWindow是Window的实现类。
- PhoneWindow中（superDispatchTouchEvent(){mDector.superDispatchTouchEvent()}其中 mDector是DectorView类在PhoneWindow中的对象，）
- DectorView是继承FrameLayout，FrameLayout继承ViewGroup，也就是说ViewGroup是DectorView的间接父类，这样事件就传递到了ViewGroup的dispatchTouchEvent()方法中了，这样事件就从Activity传递到了ViewGroup中了。
#### ViewGroup的事件分发
```
if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
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

            // If intercepted, start normal event dispatch. Also if there is already
            // a view that is handling the gesture, do normal event dispatch.
            if (intercepted || mFirstTouchTarget != null) {
                ev.setTargetAccessibilityFocus(false);
            }
```
#### View的事件分发
```
if (onFilterTouchEventForSecurity(event)) {
            if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
                result = true;
            }
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }

            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }

```
- 按照事件顺序 mOnTouchListener() > onTouchEvent() > mLongClickListener()> mClickListener()
#### 按照View来分析事件
###### View
- dispatchTouchEvent()
```
return true
// 表示消费了这个事件
return false
// 将事件回传到父View的onTouchEvent进行处理，
调用父类的同名方法
//将事件传递给自己的onTouchEvent进行处理
```
- onTouchEvent()
```
return true，或者同名方法
//表示消费该事件
return false
//将事件回传给父View的onTouchEvent进行处理
```

###### ViewGroup
- dispatchTouchEvent()
```
return true
//消费
return false
//事件回传到父View的onTouchEvent()来处理
```
- onInterceptTouchEvent()
  - 只有ViewGroup有这个方法，在dispatchTouchEvent()方法中使用
```
return true
//表示要拦截这个事件，并且调用自己的onTouchEvent
return false
//不拦截这个事件，继续传递事件
super.onInterceptTouchEvent()
//默认返回false，不做事件拦截

```
- onTouchEvent()
```
return true
//自己消费事件了。
return false 或者是调用同名方法
//不消费事件，将事件回传到父View的onTouchEvent()进行处理。
```

#### View的事件传递机制
- 触摸事件的传递流程是从dispatchTouchEvent开始的，如果不进行人工干预，那么事件将会依照View树的嵌套层次从外到内进行传递，把事件最终传递给了最内层的View的onTouchEvent方法进行处理。
- 如果事件在传递过程中进行了人工的干预，那么事件分发函数返回true表明要自己消费这个事件，如果返回了false那么会调用父View的onTouchEvent方法进行事件的处理，如果返回了父类的同名方法那么会调用自己的onTouchEvent方法进行事件的处理。
- 如果事件在传递的过程中，事件的处理函数返回true表示要消费处理这个事件，返回false表示要回传到父View的同名方法进行处理。返回调用父类的同名方法表明要处理消费这个事件。
- 事件的触发先触发onTouch如果onTouch返回true那么onClick将不会被执行，事件在onTouch中进行处理了，返回false那么事件将会继续传递，调用到onClick方法。

#### ViewGroup的事件传递机制
- 触摸事件是从Activity传递到ViewGroup的dispatchTouchEvent中的，调用了onInterceptTouchEvent，返回true表示要拦截这个事件不会继续传递给子View了，返回false或者调用了父View的同名方法，那么事件将会继续传递给子View。
- 事件在传递的过程中，如果进行了人工的干预，事件的分发函数返回true事件将会被消费，返回false那么会把事件回传给父View的onTouchEvent方法中进行处理，调用了同名方法那么事件将继续传递。
- 事件的传递过程中，如果进行了人工的干预，事件的处理函数返回true，表示事件要自己处理，返回false那么将事件回传到父View的oTouchEvent中进行处理，同理调用了同名方法也是回传父View的oTouchEvent中进行处理。