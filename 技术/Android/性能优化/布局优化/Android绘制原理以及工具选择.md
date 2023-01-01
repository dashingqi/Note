## 绘制原理

- CPU：负责计算显示内容
- GPU：负责棚格化
- 16ms发出VSync信号触发UI渲染
- 屏幕刷新频率：60HZ

## 工具

#### Systrace

- 关注Frames
- 正常：绿色

#### Layout Inspector

- Android自带工具（Tools）

#### Choreographer

###### 获取FPS

- 可在线上使用
- Choreographer.getInstance()

