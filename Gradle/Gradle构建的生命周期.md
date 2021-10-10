#### Gradle构建的生命周期

#### 监听Gradle的构建

```groovy
// 添加 构建过程的事件监听
gradle.addBuildListener(new BuildAdapter(){
    @Override
    void settingsEvaluated(Settings settings) {
        super.settingsEvaluated(settings)
        // 这个过程主要执行settings.gradle文件中的脚本
        println "初始化阶段完成"
    }

    @Override
    void projectsEvaluated(Gradle gradle) {
        super.projectsEvaluated(gradle)
        // 执行各个子Module下的build.gradle文件
        println "配置阶段完成"
    }

    @Override
    void buildFinished(BuildResult result) {
        super.buildFinished(result)
        println "执行阶段完成"
    }
})
```



#### 初始化阶段

收集所有参加构建的工程

#### 配置阶段

执行所有模块下build.gradle文件中的脚本 （主工程以及子工程）

把配置阶段生成的任务依赖图进行执行

#### 几个重要角色

###### 初始化阶段 - rootProject

###### 配置阶段 - project

###### 执行阶段 - task

- 执行任务

  ```groovy
  ./gradlew clean
  ```

- 创建任务

  ```groovy
  task testTask() {
      doLast {
          println "我是 test task 任务"
      }
  }
  ```

- 依赖任务

  ```groovy
  task testTask() {
      doLast {
          println "我是 test task 任务"
      }
  }
  
  task testTask1(){
      // 依赖testTask任务
      dependsOn testTask
      doLast {
          println "我是test task1 任务"
      }
  }
  ```

  







