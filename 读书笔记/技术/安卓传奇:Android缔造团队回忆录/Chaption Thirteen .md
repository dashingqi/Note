## 框架

框架是指包含了操作系统底层功能和应用程序用来访问这些功能的API的核心平台部分;

操作系统: 除内核以外的被系统其它部分依赖的底层功能;

Android的框架层提供如下功能

- 包管理器
- 电源管理
- 窗口管理
- 输入
- 活动管理器

#### Dianne(黛安娜) Hackborn 和Android框架

###### 简介

Dianne时最了解Android框架和整个平台的人;【她】编写了框架的大部分代码；

###### 经历

- Dianne出身计算机世家；父亲在惠普创立打印机部门，曾经有机会担任首席执行官；
- 在儿时，别的小朋友在用电脑玩游戏时，她就已经开始研究系统设计；
- 大学毕业时，在朗讯工作，在1999年年底加入Be，后来在PalmSource工作，最后加入了Android；

###### 怎么加入Android

- 在朗讯工作期间利用业务时间就在研究BeOS;

- 由兴趣转成工作的一部分，于1990年底就加入了旧金山湾区的Be公司；由于长期收入的问题最后倒闭了，卖给了Palm；
- Palm 拆分出PalmSource 并开发出Palm OS 6 ，它提供了一个强大的UI框架；之后被ACCESS收购；ACCESS收购之后并不看好当时的方向，改变了团队的操作系统策略；
- Google当时在做一个平台是开源的，并且不用担心资源的问题；之后Dianne就加入Google，并于2006年一月加入Android；

#### Activity

###### 简介

Activity这个概念是从团队早期（在PalmSource）的一个想法演化而来的，它是Android管理应用程序的一种方式。

###### 经历

- Activity一个重要元素是定义了可以被其他应用程序调用的入口，比如通知或者快捷方式，用户可以通过它们跳转到应用程序的某个位置。

- 移动应用与桌面应用有本质上的不同：用户每次只能访问一个应用，而且它们往往都很小，专注于一个特定任务；
- Activity没有main方法，操作系统会带调用它们（Activity）对事件作出响应；

- 在Android的早期，存在两种截然不同的操作系统愿景。一种是基于Activity，另一种是调用main()方法；

- 对于main方法的愿景工程师，主要考虑到Activity的生命周期过于复杂，有点难以控制；

###### 结论

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230205150653394.png" alt="image-20230205150653394" style="zoom:200%;" />

#### 资源

###### 简介

Dianne参与了资源系统的开发；

###### 经历

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230205153237057.png" alt="image-20230205153237057" style="zoom:200%;" />

- 移动设备屏幕存储
- 移动设备屏幕分辨率

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230205153326416.png" alt="image-20230205153326416" style="zoom:200%;" />

#### 窗口管理器

##### 软键盘

###### 简介

从输入法编辑器的设计上来看，Android系统的发展方向就不是为某一个设备单独定制化的，而是做一个平台一个通用的平台，提供灵活和可扩展的支持；

这个灵活不仅体现在系统框架的内部上，并且还为开发者提供了扩展的特性；开发者可以定制自己的输入法编辑器（Input Method Editor，IME）提供给用户进行使用；（AndroidIME的特性）

Android团队是一个很具有前瞻性的团队（当然从现在看Android系统的的发展确实是这样的）当时所处于的时代背景下，就能预计到设备和用户的生态系统可能会变得非常庞大和多样化；真是一帮【可怕（天才级别）】的人；

IMF（Input Method Framework）:是用来提供给开发开发第三方IME工具包；

###### ShapeWriter

在第三方开发的键盘中ShapeWriter是具有划时代意义的，是由当时在IBM的翟树民开发；

后来他加入了Android开发团队，并lead团队为Android系统的IME提供了手势输入；

#### 自上而下的Jeff Hamilton

###### 简介

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230205160507440.png" alt="image-20230205160507440" style="zoom:200%;" />

