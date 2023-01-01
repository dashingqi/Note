#### Gradle插件是什么

- 提供具体的构建功能
- 提高代码的复用性

#### 如何使用Gradle插件

##### 二进制插件（最常使用 -Android插件）

- 声明插件ID与版本号

  ```groovy
  // 对应的工程文件build.gradle中
  buildscript {
    dependencies {
      	classpath "id + versionCode"
    }
  }
  ```

- 应用插件

  ```groovy
  // 对应子工程的引用
  apply plugin: 'pluginName'
  ```

- 配置插件

  ```groovy
  android{
    compileSdkVersion 29
    ....
  }
  ```

  

##### 脚本插件

- 新建一个 gradle文件

  ```groovy
  other.gradle
  println "这是脚本文件"
  ```

- 在 app module的build.gradle文件中引入这个插件

  ```groovy
  apply from: project.rootProject.file("other.gradle")
  ```

#### Gradle插件开发流程

###### 建立插件工程

- 建立buildSrc子工程
- 建立插件运行入口

###### 实现插件内部逻辑

- 定义Extension
- 注册Extension
- 使用Extension
- 获取Extension

###### 发布与使用插件

- 发布到本地仓库

  ```groovy
  // 将 buildSrc工程 拷贝到 router-gradle-plugin中
  cp -rf buildSrc router-gradle-plugin
  
  // 使用gradle wrapper 命令调用任务发布插件
  ./gradlew :router-gradle-plugin:uploadArchives
  ```

- 在工程中引用插件

  