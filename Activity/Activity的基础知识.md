## Activity的基础知识

#### Activity中展示Toast

#### 结束一个Activity

#### 使用显示Intent进行Activtiy的跳转

#### 使用隐式Intent进行Activyt的跳转

- 调用手机浏览器加载网页
- 调用手机的拨打电话界面

#### Activty间的数据传递

###### 使用Intent传递数据

###### 返回数据给上一个Activity

#### Activity的状态

- 运行
- 暂停
- 停止
- 销毁

#### Activity的生命周期

- ActivityA 启动 ActivityB 两个Actiity的生命周期的顺序

  ```java
  onPause(A) --> onCreate(B) --> onStart(B) --> onResume(B) --> onStop(A)
  ```

- 在ActivityB点击返回键返回到ActivityA中的生命周期顺序

  ```java
  onPause（B）--> onRestart(A) --> onStart(A) --> onResume(A) --> onStop(B) --> onDestroy(B)
  ```

- ActivityA启动一个DialogActivity生命周期顺序

  ```java
  onPause(A) --> (Dialog)onCreate() --> (Dialog)onStart() --> onResume()
  ```

- 在Dialog中按返回键返回到ActivityA中的生命周期顺序

  ```java
  onPause(Dialog) --> (A)onResume() --> (Dialog)onStop() --> (Dialog)onDestroy()
  ```

  

#### Activity的启动模式

- standard
- singleTop
- singleTask
- singleInstance