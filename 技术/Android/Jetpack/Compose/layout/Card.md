#### Card

```kotlin
@Composable
fun CardSample() {
    Card(
        modifier = Modifier.padding(10.dp),
        backgroundColor = Color.Green,
        contentColor = Color.Yellow,
        border = BorderStroke(1.dp, Color.Red),
        elevation = 10.dp,

    ) {
        Text(
            text = "this is card layout",
            modifier = Modifier.padding(12.dp)
        )

    }
}

```

![image-20230731115226929](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230731115226929.png)