#### LazyRow

```kotlin
@OptIn(ExperimentalFoundationApi::class)
@Composable
fun LazyRowSample() {
    val itemList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

    LazyRow() {
        stickyHeader {
            Text(
                text = "lazy row stick header", modifier = Modifier
                    .padding(8.dp)
                    .background(Color.Yellow)
            )
        }
        items(itemList) {
            Text(text = "it is Item --> $it", modifier = Modifier.padding(8.dp))
        }
    }
}

@Preview(
    showBackground = true
)
@Composable
fun LazyRowPreview() {
    LazyRowSample()
}
```

![image-20230802232237659](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230802232237659.png)