#### Row

```kotlin
@Composable
fun RowSample() {
    Row(
        modifier = Modifier
            .size(200.dp)
            .background(Color.Green),
        horizontalArrangement = Arrangement.Center,
        verticalAlignment = Alignment.CenterVertically
    ) {
        Text(
            text = "Row item one ", color = Color.Red, modifier = Modifier.weight(1f), textAlign = TextAlign.Center
        )
        Text(
            text = "Row item two", color = Color.Yellow, modifier = Modifier.weight(1f), textAlign = TextAlign.Center
        )
    }
}
```

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230731114805956.png" alt="image-20230731114805956" style="zoom:200%;" />