# Window的属性

- Window的属性是被定义在WindowManager的LayoutParams类中。
- Window的属性主要包含三种
  - Window的类型（分类）
  - Window的标志（Flag）
  - SoftInputMode （软键盘模式）

## Window（窗口）的分类

- 在Android中，Window分为三大类分别是应用窗口、子窗口、系统窗口
- 每个级别的窗口有对应不同Type的取值范围。

#### 应用窗口

- Activity就是典型的应用窗口
- 应用窗口的Type取值范围是 1～99

#### 子窗口

- PopupWindow是属于子窗口的
- 子窗口的Type取值范围是1000～1999

#### 系统窗口

- 我们常用的Toast、输入法窗口、音量调节窗口以及系统错误弹出的都是属于系统窗口的。
- 系统窗口的Type取值范围是2000～2999

#### 窗口显示的次序

- 手机屏幕可以虚拟的用X、Y、Z轴来表示，其中我们的窗口显示次序是按照Z轴方向排列的。
- 我们的虚拟Z轴，是垂直于屏幕，有屏幕内向外指向。
- 其中在Z轴上的次序称之为z-oder，我们窗口对应的Type取值越大我们的窗口离用户就越近。

## Windwo的标志

- Window的标志是用来控制Window的显示

- Window的标志可以通过如下三种方式进行设置

  - 通过addFlags

    ```kotlin
     //使用addFlags
     window.addFlags(WindowManager.LayoutParams.FLAGS_CHANGED)
    ```

    

  - 通过setFlags

    ```kotlin
    //使用setFlags
    window.setFlags(
         WindowManager.LayoutParams.FLAGS_CHANGED,
         WindowManager.LayoutParams.SOFT_INPUT_MASK_ADJUST
     )
    ```

    

  - 通过LayoutParams

    ```kotlin
    //使用LayoutParams指定Flag
    val layoutParams = WindowManager.LayoutParams()
    layoutParams.flags = WindowManager.LayoutParams.FLAGS_CHANGED
    val textView = TextView(this)
    val windowManager = getSystemService(Context.WINDOW_SERVICE) as WindowManager
    windowManager.addView(textView, layoutParams)
    ```

## 软键盘相关的模式

- 在我们日常的开发中，比如登录页面，当弹出软键盘的情况下可能会遮住底部的文案以及按钮，这时候我们可以通过设置软键盘的模式来解决这个问题

- 解决方式

  - 通过在Activity中设置属性

    ```xml
    <activity android:name=".MainActivity"
                android:windowSoftInputMode="adjustResize"
                >
                <intent-filter>
                    <action android:name="android.intent.action.MAIN" />
    
                    <category android:name="android.intent.category.LAUNCHER" />
                </intent-filter>
            </activity>
    ```

    

  - 通过代码设置窗口的软键盘属性

    ```kotlin
     window.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE)
    ```

    