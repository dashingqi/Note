> 好文参考： https://www.bzblg.com/article/326.html

## AppGlideModule

- 可以不必重写AppGlideModule中的任何一个方法，子类中完全可以不用写任何东西，只需要继承AppGlideModule，并且添加@GlideModule注解
- AppGlideModule的实现必须使用@GlideModule注解标记。
  - 如果注解不存在，那么这个自定义的module将不会被Glide发现。

#### 通过AppGlideModule可以自定义配置

- 自定义内存缓存
- 自定义Bitmap池
- 自定义磁盘缓存
- 设置默认配置

