#### 作用

- .class文件加载到JVM中才可以使用。
- 负责加载这些.class文件的就是类加载器（ClassLoader）

#### Java中的类何时被加载器加载

- 调用了构造器。(new)
- 调用了了类中的静态变量或者静态方法。

#### Java中的ClassLoader

###### JVM中自带3个类加载器

- 启动类加载器 BootstrapClassLoader
- 扩展类加载器 ExtClassLoader （JDK1.9之后改名为PlatformClassLoader）
- 系统类加载器 APPClassLoader

#### AppClassLoader（系统类加载器）

- 主要加载系统“java.class.path”配置下的类文件
- 是我们环境变量CLASS_PATH配置的路径
- 是面向用的类加载器
  - 自己编写的代码
  - 第三方的jar包，都是由它来加载的。

#### ExtClassLoader （扩展类加载器）

- 主要加载系统 "java.ext.dirs"配置下类文件

#### BootstrapClassLoader（启动类加载器）

- 不是用Java代码实现的，由C/C++语言编写的，属于虚拟机的一部分。
- 无法在Java代码中直接获取到它的引用。
- 加载系统属性“sun.boot.class.path”配置下类文件

#### 双亲委派模式（Parents Delegation Model）

- 当类加载器收到加载类或资源的请求时，通常都是先委托给父类加载器加载，当父类加载器找不到指定类或者资源时，自身才会执行实际的类加载过程。

  ```java
   protected Class<?> loadClass(String name, boolean resolve)
          throws ClassNotFoundException
      {
              // First, check if the class has already been loaded
              Class<?> c = findLoadedClass(name);
              if (c == null) {
                  try {
                      if (parent != null) {
                          c = parent.loadClass(name, false);
                      } else {
                          c = findBootstrapClassOrNull(name);
                      }
                  } catch (ClassNotFoundException e) {
                      // ClassNotFoundException thrown if class not found
                      // from the non-null parent class loader
                  }
  
                  if (c == null) {
                      // If still not found, then invoke findClass in order
                      // to find the class.
                      c = findClass(name);
                  }
              }
              return c;
      }
  ```

  - 如果当前类被加载过，就从缓存中拿到之前加载过的
  - 没有被加载过
    - 如果存在父类加载器，就使用父类加载器加载
    - 不存在父类加载器，就使用启动类加载器加载类
  - 如果，以上过程都没有能够加载类，那么就使用自身的类加载器去加载类

- Test test = new Test()

  - 首先使用系统类加载器APPClassLoader，将加载的任务委派给它的父类加载器（ExtClassLaoder）
  - 如果父类加载器为null，就将这个加载任务委派给启动类加载器（BootstractClassLaoder）
  - BootstractClassLoader在 jdk/lib目录下无法找到Test类，此时返回的Class为null，
  - 如果父类加载器和启动类加载器都没能够加载Test类，那么AppClassLoader回调用自身的findClass方法来加载Test类。

- 双亲委派 机制只是Java推荐机制，并不是强制的机制。

  - 我们可以通过继承 Java.lang.CLassLoader类，来实现自己的类加载器。
  - 如果我们想保持双亲委派机制，我们应该重写findClass(namg)方法
  - 如果我们想破坏双亲委派机制，我们应该重写loadClass(name)方法

#### 自定义ClassLoader

##### 步骤

- 自定义一个类继承抽象类ClassLoader
- 重写findCLass方法
- 在findClass中，调用defineClass方法将字节码转换成Class对象，并返回。

#### Android 中的ClassLoader

- 在Android中，ClassLoader的加载细节有所不同。
- Android虚拟机里是无法运行.class文件的，Android会将所有的.class文件转换成一个.dex文件，通过BaseDexClassLoader来加载这个.dex文件中。
- 一般我们使用BaseDexClassLoader两个子类：PathClassLoader和DexClassLoader来加载dex文件。

###### PathClassLoader

- 主要用来加载系统apk和被安装到手机中的apk内的dex文件。

- PathClassLoader类中就有两个构造方法，具体的实现都是在BaseDexClassLaoder中

  ```java
  // dexPath：dex文件路径，活着包含dex文件的jar包路径
  public PathClassLoader(String dexPath, ClassLoader parent) {
          super((String)null, (File)null, (String)null, (ClassLoader)null);
          throw new RuntimeException("Stub!");
      }
  
  // librarySearchPath: C/C++ native库的路径
      public PathClassLoader(String dexPath, String librarySearchPath, ClassLoader parent) {
          super((String)null, (File)null, (String)null, (ClassLoader)null);
          throw new RuntimeException("Stub!");
      }
  
  //打印结果如下
  classLoader = dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.dashingqi.classifybyjd-U-rbD7-hGiY-Xlo1oaKLsw==/base.apk"],nativeLibraryDirectories=[/data/app/com.dashingqi.classifybyjd-U-rbD7-hGiY-Xlo1oaKLsw==/lib/arm64, /system/lib64, /system/vendor/lib64]]]
  ```

- dexPath比较受限制，一般是已经安装应用的apk文件路径

- 当一个App被安装到手机后，apk里的class.dex中的class是通过PathClassLoader来加载的。

###### DexClassLoader

- 对比PathClassLoader只能加载已经安装应用中的dex文件，DexClassLoader没有这个限制，可以从SD卡中加载包含class.dex的 .jar 和 .apk 文件。

- DexClassLoader的源码中只有一个构造方法

  ```java
  public DexClassLoader(String dexPath, String optimizedDirectory, String librarySearchPath, ClassLoader parent) {
          super((String)null, (File)null, (String)null, (ClassLoader)null);
          throw new RuntimeException("Stub!");
      }
  ```

  - dexPath : 包含class.dex文件的 .jar活着.apk 文件路径，如果是多个路径的话用文件分割符分割（默认是" : "）
  - optimizedDirector : 用来缓存优化的dex文件的路径，也就是从apk或jar文件中提取出来的dex文件。

- 热修复和插件化都是基于DexClassLaoder

#### 总结

- ClassLoader是用来加载class文件，不管是jar还是dex中的class。
- Java中的ClassLoader是通过双亲委派机制来加载class文件。
- 可以自定义ClassLoader，一般覆盖findClass()，不建议重写loadClass()方法。
- Android中常用的两种ClassLoader分别为：PathClassLoader和DexClassLoader。