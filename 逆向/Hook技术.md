## Hook技术

> Hook翻译过来就是钩子的意思。
>
> 一套完整的流程就有开始和结束的，从开始到结束一步一步的事件流程的。
>
> 而我们的钩子 就是能够钩在事件上，然后处理一些我们自己特定的事件

- Hook能使自身的代码融入到被钩住的程序进程中，成为目标进程的一个部分。

#### Android中的Hook机制

- 要root权限，直接Hook系统，可以干掉所有APP
- 免root权限，只能Hook自身，对系统其他App无能为力。

#### Hook方案

###### Xposed

- 通过替换/system/bin/app_process程序控制Zygote进程，使得app_process在启动过程中会加载 XposedBridge.jar这个Jar包。从而完成对Zygote进程及其创建的Dalvik虚拟机的劫持。

###### Cydia Substrate

###### Legend

