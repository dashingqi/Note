#### ListItem

```kotlin
@OptIn(ExperimentalMaterialApi::class)
@Composable
fun ListItemSample() {
    val listItem = listOf("Item 0", "Item 1", "Item 2", "Item3")
    Column(modifier = Modifier.background(Color.White)) {
        listItem.forEach { item ->
            ListItem(icon = {
                Box(contentAlignment = Alignment.Center) {
                    Icon(imageVector = Icons.Default.Home, contentDescription = null)
                }

            }, text = {
                Text(text = item)
            }, secondaryText = {
                Text(text = "Second Text")
            })
        }
    }
}
```

![image-20230731115105925](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230731115105925.png)