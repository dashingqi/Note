## 描述

- ANR （Application Not Responding）
- 在Android系统上，如果你的应用程序在一段时间响应不够灵敏，系统就会向用户显示一个对话框，这个对话框称作应用无响应的对话框。

## 引起点

- Service在特定时间内无法完成处理，这个特定时间是20秒
- BroadcastReceive在10秒内无法完成相应处理
- UI线程中存在耗时操作（I/O操作、耗时的计算）
- UI线程中存在错误的操作，比如调用了Thread.sleep()、Thread.await()

## 解决

- 将耗时的操作、复杂的计算都放到子线程中去操作，尽量不要将耗时的操作放到UI线程上，然后通过 Handler、runonUiThread、AsyncTask等从子线程切换到主线程上然乎去更新UI。