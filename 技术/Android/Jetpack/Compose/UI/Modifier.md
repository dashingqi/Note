#### Modifier

```kotlin
@Composable
fun ModifierSample() {
    Text(
        text = "Modifier Sample Demo",
        style = TextStyle(background = Color.Yellow),
        modifier = Modifier
            .background(color = Color.Blue)
            .padding(8.dp)
            .clickable {
                Log.d(TAG, "点击到我了");
            },
    )
}
```

![image-20230730112000039](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230730112000039.png)