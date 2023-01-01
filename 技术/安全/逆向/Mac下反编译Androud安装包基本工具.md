##  本篇文章主要从如下几点学习反编译工具的基本安装

- apk介绍

- 反编译apk需要的基本工具

- 常用ADB命令

#### apk介绍

- APK的全名称是Android application package直接翻译就是Android应用程序包。
- 我们平常安装的程序都是将apk文件下载到本地后在进行安装
- 我们下载下来的apk本身是ZIP文件格式的（还记得使用V2签名的时候是把签名信息在ZIP文件中格外开辟了一块空间进行存储）。

#### 反编译apk需要的基本工具

##### ApkTool

- 它是一个逆向的工具，有编译、反编译、签名等功能。

###### 下载ApkTool

- 可以从该连接https://ibotpeaches.github.io/Apktool/install/ 下载不同操作系统的工具

###### 安装ApkTool

> 根据文档提示进行安装就可以了

- 下载apktool.jar（第一下载的时候会带上版本名称，重命名一下就可以了）文件以及apktool文件

- 将下载好的文件方法 /usr/local/bin目录下

  - 可以在终端使用命令 open /usr/local/bin
  - 可以使用访达的快捷操作 command+shift+g 输入路径就可以了

- 为以上两个文件增加可执行权限

  - 终端切换到 /usr/local/bin路径下 执行如下两个命令即可

    ```java
    chmod +x apktool.jar
    chmod +x apktool
    ```

- 接着在终端输入***apktool***令,如果出现如下信息就说明ApkTool安装成功了。

###### 使用ApkTool

- 基本使用

  - 在终端 cd到你的apk文件目录下

  - 执行语句 apktool d  name.apk

  - 执行完语句后会在同级的目录下生成一个与apk文字一样的文件夹，该文件夹下就是反编译后的文件

    

  

##### dex2jar

- 将dex反编译成jar文件

###### 安装dex2jar

- 可以从https://sourceforge.net/projects/dex2jar/files/ 这里下载就行，下载后直接解压就可以了。

###### 使用dex2jar

- 同样刚才的test.apk文件，解压到当前文件夹下。

- 使用命令将dex文件转化成jar

  - 将该classes.dex和classes2.dex文件拷贝到dex2Jar解压的文件夹下。

  - 使用命令开始反编译

    ```java
    sh d2j-dex2jar.sh classes.dex
    sh d2j-dex2jar.sh classes2.dex
    ```

  - 通过如上操作，就能在文件夹下获取到jar文件了。

##### JD-GUI

- 查看反编译后的jar文件的可视化工具

###### 安装 JD-GUI

- 可以从http://jd.benow.ca/ 上下载。

###### 使用JD-GUI 查看反编译后的源码 也及时jar文件

- 双击JD-GUI程序，
- 将反编译好的jar拖进去就可以了。

#### 常用的ADB命令

- 查看手机中已经安装的所有apk文件包名
  - adb shell pm list packages
  - adb shell pm list packages -3 查看第三方包名
  - adb shell pm list packages -s 输出系统的包
- 根据包名，查看app的安装路径
  - adb shell pm path packageName
- 根据以上的安装路径导出apk
  - adb pull pathName 





