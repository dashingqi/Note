#### Column

```kotlin
@Composable
fun ColumnSample() {
    Column(
        modifier = Modifier
            .size(200.dp)
            .background(Color.Green),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(
            text = " Column First one ", textAlign = TextAlign.Center,modifier = Modifier.padding(10.dp)
        )

        Text(
            text = " Column First two ",
            textAlign = TextAlign.Center,
            modifier = Modifier.padding(horizontal = 8.dp, vertical = 10.dp)
                .background(Color.Yellow),
            color = Color.Red
        )

    }

}
```

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230731114840274.png" alt="image-20230731114840274" style="zoom:200%;" />

