## 注解

#### @Bindable

- 一般使用BaseObservable来刷新视图的时候使用
- 用来修饰数据Bean中的get或者is开头的方法。
- 在编译阶段会在BR类中生成对应的字段，与notifyPropertyChanged方法配合使用
- 当数据被修改的时候，会自动刷线视图显示的数据。而不需要setText这种操作

#### @BindingAdapter

- 用于标记方法，方法必须为公共静态方法

- 标记的第一个方法参数类型必须为View类型，否则报错

- 可以用来自定义View的任意属性

- 注解源码

  ```java
  @Target(ElementType.METHOD)
  public @interface BindingAdapter {
  
      String[] value();
      boolean requireAll() default true;
  }
  ```

  - 指定用来标记方法的
  - value属性是一个String数组，用来存放自定义属性
  - requireAll是一个布尔值，用来表示自定义的属性是否必须都要使用

