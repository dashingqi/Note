## 背景介绍

- 布局文件读取过程：IO过程
- 创建View慢：反射（比new慢3倍）

## 解决

#### 根本

#### 侧面缓解

- AsyncLayoutInflater（异步Inflate）
- workThread加载布局
- 回调主线程
- 节约主线程时间

###### AsyncLayoutInflater的使用

- 导包

- 代码实操

  ```
  
  ```

- 不能设置LayoutInflater.Factory(自定义解决)
- 注意View的加载是在子线程中操作的，不能绑定于主线程的操作