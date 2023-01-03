###### Gradle是什么

Gradle 是一个构建工具

###### 系统全局安装Gradle

- 安装JDK
- 下载Gradle的安装包
- 配置Gradle的环境变量

```groovy
export PATH=$PATH:gradlePath
```

###### 校验Gradle

```groovy
gradle -v
```

###### Gradle Wrapper

- 生成Gradle Wrapper
  - Android Studio 创建的工程默认创建了Gradle Wrapper
  - gradle wrapper（生成wrapper）

##### Gradle的执行

使用gradle wrapper 

###### 清理编译产物

```groovy
./gradlew clean -q
```

###### 产看所有子工程

```groovy
./gradlew projects
```

###### 查看所有任务

```groovy
./gradlew tasks
```

##### Gradle的升级

###### 工程中升级Gradle

```groovy
./gradlew wrapper --gradle-version 7.0
```
