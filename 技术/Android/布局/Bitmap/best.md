#### Best

###### 检测图片是否为空白

```kotlin
fun throwIfBlank(bitmap: Bitmap, excludeColor: Int?) {
        var color: Int = Color.WHITE
        runCatching {
            val w = BLANK_WIDTH
            val h = BLANK_HEIGHT
            val scaleDown = Bitmap.createScaledBitmap(bitmap, w, h, true) ?: return
            val pixels = IntArray(w * h)
            scaleDown.getPixels(pixels, 0, w, 0, 0, w, h)
            // 有些手机上白屏不是#FFFFFF，比如存在#F7F7F7的情况，这里直接取第一个像素与其余像素对比
            color = pixels[0]
            val sameColor = !pixels.any { it != color }
            if (sameColor && (excludeColor == null || excludeColor != color)) {
                throw Exception(color)
            }
        }.onFailure {}
    }
```

###### 将Bitmap加工成带圆角的Bitmap

```kotlin
@Throws(OutOfMemoryError::class, IllegalArgumentException::class)
internal fun Bitmap.clipRoundRect(radius: Float): Bitmap {
    val bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888)
    val path = Path().apply {
        fillType = Path.FillType.EVEN_ODD
        addRoundRect(
            RectF(0f, 0f, width.toFloat(), height.toFloat()),
            radius,
            radius,
            Path.Direction.CW
        )
    }
    val canvas = Canvas(bitmap)
    canvas.clipPath(path)
    canvas.drawBitmap(this, Rect(0, 0, width, height), Rect(0, 0, width, height), null)
    return bitmap
}
```

