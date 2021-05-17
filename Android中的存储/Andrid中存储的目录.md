## Android中存储目录

对于一个APP来说，存储是分为应用内部存储和应用外存储。



#### 内部存储

##### 描述

应用内部存储是一块比较特殊的区域，对于一个APP来说，系统会在 /data/data/PackagName/XXX创建一个文件夹，这个文件夹就是对应于APP的内部存储；

当我们把文件存储当内部存储中的时候，该文件只能被我们的应用被访问到。其他应用是没有权限访问这个文件的。

当我们应用被卸载的话，内部存储中存储的文件就会从手机中删除掉；

对于内部存储的区域用于自己是访问不到的，当我们的手机被Root了，才能访问到这个区域；

所以内部存储是和应用绑定的，并且是安全的。

##### 访问

Android官方系统有提供相应的API让应用本身去访问到该区域

```kotlin
var filesPath = filesDir.absolutePath
var cachePath = cacheDir.absolutePath
```

其实访问应用的内存储的API是需要Context

###### context.getFilesDir()

该API打印的路径为

/data/user/0/PackageName/files

通常来说通过该API获取的内部存储路径为/data/data/PackageName/file，但是在列入我当前使用的测试机（小米10）上为上述表现；getCacheDir同理；

###### context.getCacheDir()

该API打印的路径为

/data/user/0/PackageName/cache

该目录下存储的文件会不太稳定，就是当设备的内存不足的时候会优先被删除；



#### 外部存储

在早期的手机，是能插入外置的SD卡，这时插入的SD卡是作为外部存储，手机自带的存储称之为内部存储，当随着行业的发展现在手机自带的存储变得越来越大了，随之插入的外置SD卡也被取消了。

所以这里称外部存储是手机自带存储+插入的SD卡；

对于一个App来说，外部存储也是分为 应用的私有目录和公共目录的，其中私有目录是存放当前应用的数据，儿公共目录是存放共享文件目录的。

##### 私有目录

私有目录下的文件其他应用是可以访问到的，当应用被卸载了，私有目录下的文件也随之被删除。

对于该目录Android官方也提供了对应的API来访问

```kotlin
var cacheDir = externalCacheDir?.absolutePath
var filesDir = getExternalFilesDir(null)?.absolutePath
```

###### context.getExternalCacheDir()

该APi获取到的路径地址为：/storage/emulated/0/Android/data/PackageName/cache

###### context.getExternalFilesDir(String type)

该APi获取到的路径地址为：/storage/emulated/0/Android/data/PackageName/files

##### 公共目录

公共目录下的文件是自由访问的，当应用被卸载之后存在在公共目录下的文件依然存在！

提供的API

```kotlin
var path = getExternalStorageDirectory().path
//对应的路径地址为：/storage/emulated/0
```









