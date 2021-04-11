- 路由表

- 注解

- 注解处理器    继承  AbstractProcessor （使用Google的 AutoService 注册注解处理器）

  - 生成代码

  - 根据注解，用来将Activity的class对象添加到路由表中（自动管理）

  - 三个方法需要重写

    - initI() 初始化Filer
    - getSupportedSourceVersion() 声明注解处理器支持的Java版本
    - getSupportedAnnotationTypes() 声明这个注解处理器要处理哪些注解

  - 核心方法 process

    ​	

写文件

​	JavaPoet