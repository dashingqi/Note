> 在我很长一段时间内，以为布局的显示都是Activity来帮我们搞定的，直到我点击setContentView()的源码中，我才恍然大悟

#### Activity的setContentView()方法

###### 源码以及分析过程

- 源码

  ```java
    public void setContentView(@LayoutRes int layoutResID) {
      		//分析------> 1
          getWindow().setContentView(layoutResID);
          initWindowDecorActionBar();
      }
   public Window getWindow() {
          return mWindow;
      }
  ```

  - 分析1：我们看到Activity源码中是调用了Window的setContentView
    - Window，其实在我们分析事件分发的时候，就知道它是一个抽象类，系统源码内唯一的实现类就是PhoneWindow，那么我们看下PhoneWindow的setContentView()方法

  - 但是你有没有想过，这个mWindow实例在什么时候赋值呢？
    - 其实在我们创建Activty的时候，在attach()方法中初始化了PhoneWindow类，获得到它的实例（这不是本文的重点，后面会分析下startActivity的流程，里面会详细介绍下）

  ```java
  public void setContentView(int layoutResID) {
    			//分析 -----> 1
          if (mContentParent == null) {
              installDecor();
          } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            	//当没有5.0的转场动画，就将添加的布局清空
              mContentParent.removeAllViews();
          }
          if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            	//当有5.0的转场动画
              final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                      getContext());
              transitionTo(newScene);
          } else {
            // 分析 -------> 2
              mLayoutInflater.inflate(layoutResID, mContentParent);
          }
          mContentParent.requestApplyInsets();
          final Callback cb = getCallback();
          if (cb != null && !isDestroyed()) {
              cb.onContentChanged();
          }
          mContentParentExplicitlySet = true;
      }
  
    private void installDecor() {
          mForceDecorInstall = false;
      		//初始化DecorView
          if (mDecor == null) {
              mDecor = generateDecor(-1);
              
            ....... 省略代码
          } else {
              mDecor.setWindow(this);
          }
      		//初始化mContentParent
          if (mContentParent == null) {
              mContentParent = generateLayout(mDecor);
          }
    }
  ```

  - 分析1：当mContentParent为null，我们会调用installDecor()方法，去初始化一下De````````````````````````````````````````````````546 V3 ````````````````````````````````````````````C7orView````````````````````````````````````````````和mContentParent
  - 分析2：在这里我们将要实现的布局添加到mContentParent中
  - 总结来说就是，PhoneWindow中会有一个DecorView（是一个FrameLayout），这个DecorView中会有一个mContentParent（是一个ViewGroup），我们在Activity里通过setContentView()设置的布局都会添加到这个mContentParent中。
  - 上述代码我们用一张图来表示就是如下
    - ![布局显示-Activity到PhoneWindow.png](https://upload-images.jianshu.io/upload_images/4997216-2a5025d82d8b1ac8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 到这里我们把要显示的布局添加到PhoneWindow中DecorView的mContentView中了，但是此时DecorView并没有和Activity建立任何联系，那么DecorView是如何被显示到屏幕上的呢
  - 在上述问题的疑问上，我们都知道在Activity的onCreate方法中，我们只是初始化了需要显示的内容，onResume()方法的执行才标志着我们能于界面进行交互了，就在这个阶段才是将DecorView绘制到屏幕上的。

#### DecorView添加到WindowManager中

###### 源码以及分析过程

- 在ActivityThread中的handleResumeActivity()中（创建的Activity的生命周期方法执行都是在ActivityThread中进行的），会调用WindowManager的addView方法将DecorView添加到WMS中

  ```java
  public void handleResumeActivity(IBinder token, boolean finalStateRequest, boolean isForward,
              String reason) {
          
          	......... 代码省略
              
          if (r.window == null && !a.mFinished && willBeVisible) {
              r.window = r.activity.getWindow();
              View decor = r.window.getDecorView();
              decor.setVisibility(View.INVISIBLE);
            	// 获取到WindowManger
              ViewManager wm = a.getWindowManager();
              WindowManager.LayoutParams l = r.window.getAttributes();
              a.mDecor = decor;
              l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
              l.softInputMode |= forwardBit;
              if (r.mPreserveWindow) {
                  a.mWindowAdded = true;
                  r.mPreserveWindow = false;
                  ViewRootImpl impl = decor.getViewRootImpl();
                  if (impl != null) {
                      impl.notifyChildRebuilt();
                  }
              }
              if (a.mVisibleFromClient) {
                  if (!a.mWindowAdded) {
                      a.mWindowAdded = true;
                    	// 将DecorView添加到WidnowManger中，
                      wm.addView(decor, l);
                  } else {
                      a.onWindowAttributesChanged(l);
                  }
              }
  ```

- 我们知道WindowManager是一个接口类，它的唯一实现类是WindowMangerImpl，上述的addView就是调用到了WindowManagerImpl中的addView

  ```java
    @Override
      public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
          applyDefaultToken(params);
        	// 调用到了WindowManagerGlobal中的addView()
          mGlobal.addView(view, params, mContext.getDisplay(), mParentWindow);
      }
  
  // 调用到了WindowManagerGlobal的addView，WindowManagerGlobal是一个单利类，每一个进程中只有一个它的实例对象。
  public void addView(View view, ViewGroup.LayoutParams params,
              Display display, Window parentWindow) {
          	
    					........ 省略部分代码
  						
                //这里创建了ViewRootImpl
              root = new ViewRootImpl(view.getContext(), display);
  
              view.setLayoutParams(wparams);
  
              mViews.add(view);
              mRoots.add(root);
              mParams.add(wparams);
  
              // do this last because it fires off messages to start doing things
              try {
                // 调用了ViewRootImpl的setView()方法
                  root.setView(view, wparams, panelParentView);
              } catch (RuntimeException e) {
                  // BadTokenException or InvalidDisplayException, clean up.
                  if (index >= 0) {
                      removeViewLocked(index, true);
                  }
                  throw e;
              }
          }
      }
  ```

  - 其实setView方法主要实现了两件事件
    - DecorView被渲染到了屏幕上了（最后从Window ------> WMS）
    - DecorView可以接受屏幕触摸事件

#### 将DecorView添加到ViewRootImpl中

###### 源码以及分析

- ViewRootImpl的setView()方法中的核心代码分析

  ```java
   public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
         					// 分析 ------> 1
                  requestLayout();
                  if ((mWindowAttributes.inputFeatures
                          & WindowManager.LayoutParams.INPUT_FEATURE_NO_INPUT_CHANNEL) == 0) {
                      mInputChannel = new InputChannel();
                  }
                  mForceDecorViewVisibility = (mWindowAttributes.privateFlags
                          & PRIVATE_FLAG_FORCE_DECOR_VIEW_VISIBILITY) != 0;
                  try {
                      mOrigWindowType = mWindowAttributes.type;
                      mAttachInfo.mRecomputeGlobalAttributes = true;
                      collectViewAttributes();
                    	// 分析 ------> 2
                      res = mWindowSession.addToDisplay(mWindow, mSeq, mWindowAttributes,
                              getHostVisibility(), mDisplay.getDisplayId(), mTmpFrame,
                              mAttachInfo.mContentInsets, mAttachInfo.mStableInsets,
                              mAttachInfo.mOutsets, mAttachInfo.mDisplayCutout, mInputChannel,
                              mTempInsets);
                      setFrame(mTmpFrame);
                  } catch (RemoteException e) {
                      mAdded = false;
                      mView = null;
                      mAttachInfo.mRootView = null;
                      mInputChannel = null;
                      mFallbackEventHandler.setView(null);
                      unscheduleTraversals();
                      setAccessibilityFocus(null, null);
                      throw new RuntimeException("Adding window failed", e);
                  } finally {
                      if (restore) {
                          attrs.restore();
                      }
                  }
      }
  
   mWindowSession = WindowManagerGlobal.getWindowSession();
  ```

  - 分析1：requestLayout()方法是刷新布局的操作，调用这个方法后，ViewRootImpl所关联的View会进行一次测量、布局、以及绘制也就是measure --> layout --> draw 的过程，为了确保在View显示到屏幕之前完成了View的测量和绘制操作。
  - 分析2：此处会调用到mWindowSession的addTiDisplay将View添加到WindowManagerService中
    - 其中mWindowSession是WindowManagerGlobal中的单利对象

- sWindowSession实际上是IWindowSession类型的，是一个Binder类型，真正的实现类是在系统进程中的，也就用于跨进程通信的。ViewRootImpl中是通过调用WindowManagerGlobal的getWindowSession()方法来获取到实例的，

  ```java
    public static IWindowSession getWindowSession() {
          synchronized (WindowManagerGlobal.class) {
              if (sWindowSession == null) {
                  try {
                      InputMethodManager.ensureDefaultInstanceForDefaultDisplayIfNecessary();
                      IWindowManager windowManager = getWindowManagerService();
                    	// 分析 ----> 1
                      sWindowSession = windowManager.openSession(
                              new IWindowSessionCallback.Stub() {
                                  @Override
                                  public void onAnimatorScaleChanged(float scale) {
                                      ValueAnimator.setDurationScale(scale);
                                  }
                              });
                  } catch (RemoteException e) {
                      throw e.rethrowFromSystemServer();
                  }
              }
              return sWindowSession;
          }
      }
  ```

  - 可以看到mWindowSession是一个单利的对象，进程中存在唯一一个对象。
  - 分析1：调用该方法是用AIDL从System进程中获取Session对象，那么上述的addToDisplay()方法的调用就是一次跨进程通信，将View发送到系统进程中，也就是从Window转向了WindowManagerService中了。剩下的工作就交给了WMS来完成了。

#### 总结

- Activity的参与度比较低，大部分View的添加操作都是由PhoneWindow中完成的。
- 我们的Activity相当于给开发人提供的管理类，通过它可以简单的实现Window于View的操作逻辑。
- 所以总的说就是，Activity是给开发人员的管理类，Window是进行View的添加操作将View添加到DecorView中，View是携带我们要显示的数据，真正背后是WMS在运作--->也就是将数据显示到屏幕上。
- 而将View作为窗口添加到WMS中是由WindowManger来完成的。
- 注意的几个小点
  - 一个Activity对应一个PhoneWindow对象，PhoneWindow中有一个DecorView，我们会讲layout添加到这个DecorView中。
  - 一个PhoneWindow对应一个ViewRootImpl对象
  - ViewRootImpl的setView方法主要完成两件事情
    - View的渲染（requestLayout()）
    - 接受触摸事件