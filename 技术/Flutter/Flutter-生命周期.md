## 生命周期

#### 简介

Flutter中的Widget的生命周期是通过State来体现；

#### State生命周期

![img](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/bba88ebb44b7fdd6735f3ddb41106784.png)

###### 创建

State的初始化会依次执行：构造方法 -> initState -> didiChangeDependencies -> build --> 完成页面的渲染

- 构造方法：State生命周期的起点；
- initState：在State对象被插入视图树时候调用；这个函数在State生命周期中只会被调用一次；
- didChangeDependencies： 专门用来处理State对象依赖关系变化，在initState()调用结束后，被Flutter调用；
- build：作用是构建视图；

###### 更新

Widget状态更新，主要由3个方法触发：setState、didChangeDependencies、didUpdateWidget

- setState: 当状态数据发生变化时，开发者调用这个方法通知Flutter；数据有变，请使用最新数据重建UI
- didChangeDepencies：State对象的依赖关系发生变化，Flutter会回调该方法，随后触发组建构建；
  - 系统语言Locale
  - 应用主题改变时
  - 上述举例的情况系统会通知State执行didChangeDependencies
- didUpdateWidget:当Widget的配置发生变化时
  - 父Widget触发重建
  - 热重载
  - 上述两种情况，系统会调用这个函数

###### 销毁

组件被移除或者页面被销毁的时候，系统会调用deactivate和dispose这两个方法

- deactivate:当组件可见状态发生变化时，deactivate函数会被调用；
- dispose：当State被永久从视图树中移除时，Flutter会调用dispose函数；

![img](https://static001.geekbang.org/resource/image/72/d8/72e066a4981e0e2381b1dab6e61307d8.png?wh=1518*592)

#### APP生命周期

WidgetsBindingObserver

```dart

abstract class WidgetsBindingObserver {
  //页面pop
  Future<bool> didPopRoute() => Future<bool>.value(false);
  //页面push
  Future<bool> didPushRoute(String route) => Future<bool>.value(false);
  //系统窗口相关改变回调，如旋转
  void didChangeMetrics() { }
  //文本缩放系数变化
  void didChangeTextScaleFactor() { }
  //系统亮度变化
  void didChangePlatformBrightness() { }
  //本地化语言变化
  void didChangeLocales(List<Locale> locale) { }
  //App生命周期变化
  void didChangeAppLifecycleState(AppLifecycleState state) { }
  //内存警告回调
  void didHaveMemoryPressure() { }
  //Accessibility相关特性回调
  void didChangeAccessibilityFeatures() {}
}
```

##### 生命周期回调

###### didChangeAppLifecycleState回调函数

AppLifecycleState枚举类：Flutter对App生命周期状态的封装

- resumed：可见，并能响应用户的输入
- inactive：处在不活动状态，无法处理用户响应
- paused：不可见并不能响应用户的输入，但是在后台继续活动中；

应用压后台：AppLifecycleState.inactive --> AppLifecycleState.paused

应用从后台切换到前台 : AppLifecycleState.paused --> AppLifecycleState.resumed

##### 帧绘制回调

我们需要在组件渲染之后做一些与显示相关的操作；

Andriod中是view.post()插入消息队列；

###### 单次Frame绘制回调

在当前Frame绘制完成后进行回调，并且只会回调一次，如果要再次监听需要再设置一次；

```dart
WidgetsBinding.instance.addPostFrameCallback((_){
    print("单次Frame绘制回调");//只回调一次
  });
```

###### 实时Frame绘制回调

会在每次绘制Frame结束后进行回调，可以用做FPS监测；

```dart

WidgetsBinding.instance.addPersistentFrameCallback((_){
  print("实时Frame绘制回调");//每帧都回调
});
```

