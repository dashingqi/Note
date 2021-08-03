## canvas # drawText

#### drawText函数的使用

##### canvas.drawText 解析

##### 四线格与FontMetrics

###### 四线格

###### FontMetrics

#### 常用函数

##### 字符串所占区域的高度和宽度

###### 高度

```kotlin
/**
 * includePadding: 
 * true ---> 表示获取字符串所占的最大区域高度
 * fasle ---> 表示获取字符串所占的实际高度
 */
fun getTextHeight(paint: TextPaint?, includePadding: Boolean): Float {
  var tempTextHeight = 0.0f
  paint?.let { textPaint ->
              tempTextHeight = if (includePadding) {
                textPaint.fontMetrics.bottom - textPaint.fontMetrics.top
              } else {
                textPaint.fontMetrics.descent - textPaint.fontMetrics.ascent
              }
             }
  return tempTextHeight
}
```

###### 宽度

```kotlin
/**
 * 获取到文本的宽度
 */
fun getTextWidth(content: String, textPaint: TextPaint): Float {

  return if (content.isEmpty()) {
    0.0f
  } else {
    //主要是这个方法 measureText()
    textPaint.measureText(content)
  }
}
```



##### 最小矩形

##### 定点写字



