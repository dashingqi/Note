## 背景

- 知其然知其所以然
- 源码原理

## 加载源码的的跟踪

- setContentView
- LayoutInflater
- inflate
- getLayout(I/O)
- createViewFromTag(更具具体的Tag创建View)
- Factory
- createView（通过反射）

## 性能瓶颈

- 布局文件解析：IO过程
- 创建View对象：反射

## LayutInflater.Factory

- 创建View的一个Hook（挂钩）
- 定制创建View的过程：全局替换自定义TextView

## Factory/Factory2

- Factory2 extends Factory
- Factory2多了一个参数：parent

