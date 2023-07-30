#### Image

```kotlin
@Composable
fun ImageSample() {
    Image(
        painter = painterResource(id = R.drawable.jsy),
        contentDescription = null,
        modifier = Modifier.size(200.dp),
        // 裁剪方式
        contentScale = ContentScale.Crop,
        // 滤镜
        colorFilter = ColorFilter.tint(Color.Yellow, blendMode = BlendMode.Color)
    )
}
```

![image-20230730112308612](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230730112308612.png)