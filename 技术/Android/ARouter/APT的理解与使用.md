#### 简介

- APT （Annotation Processing Tool）是一种处理注解的工具，对源代码文件进行检测找出其中的Annotation，使用Annotation进行额外的处理。
  - 它是一个Javac的一个工具，编译时注解处理器。
  - 获取注解以及生成代码都是在代码编译时完成的，相比较于反射在运行时处理注解大大提高了程序性能。
- Annotation处理器在处理Annotation时可以根据源文件中的Annotation生成额外的源文件和其他的文件。

#### 使用的地方

- ButterKnife
- EventBus
- Dagger2
- ARouter

#### AndroidStudio 搞一个APT项目

###### 需要两个模块

- Annotation模块
  - 用来存放自定义的注解
- Compiler模块
  - 需要依赖Annotation模块
- 项目中的模块和其他module
  - 需要依赖Annotation模块
  - 通过 annotationProcessor 依赖Compiler模块
- 注意：上述的两个模块 annotation和compiler需要建成Java Library，因为AndroidLibrary时基于OpenJDK的，它里面是不包含APT的相关代码的（AbstractProcessor）

