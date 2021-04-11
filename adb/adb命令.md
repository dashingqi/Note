## ADB命令大集合

#### 导出手机中已经安装的apk

- 查看手机中已经安装的所有apk文件包名
  - adb shell pm list packages
  - adb shell pm list packages -3 查看第三方包名
  - adb shell pm list packages -s 输出系统的包
- 根据包名，查看app的安装路径
  - adb shell pm path packageName
- 根据以上的安装路径找到apk
  - adb pull pathNamelist li

