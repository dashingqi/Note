#### Spacer

```kotlin
@Composable
fun SpacerSample() {
    Column(
        modifier = Modifier
            .size(200.dp)
            .background(Color.Green)
    ) {
        Text(text = "Column Item One One ...", color = Color.Red)
      	// 间隔
        Spacer(modifier = Modifier.height(20.dp))
        // 分割线
        Divider(startIndent = 3.dp, thickness = 12.dp, color = Color.Magenta)
        Text(text = "Column item two teo   ....", color = Color.Yellow)
    }
}
```

![image-20230731120651097](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230731120651097.png)