#### LazyColumn

```kotlin
@OptIn(ExperimentalMaterialApi::class, ExperimentalFoundationApi::class)
@Composable
fun LazyColumnSample() {

    val data = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 10, 11, 12, 14, 14, 13, 15, 16)
    // 滑动状态
    val lazyState = rememberLazyListState()

    // 协程
    val coroutineScope = rememberCoroutineScope()

    LazyColumn(state = lazyState) {
      // 粘性头部
        stickyHeader {
            Text(
                text = "Stick Header",
                modifier = Modifier.fillMaxSize(),
                textAlign = TextAlign.Center,
                color = Color.Blue
            )
        }
      // LazyColumn #items
        items(data) {
            ListItem(icon = {
                Icon(imageVector = Icons.Default.Email, contentDescription = null)
            }, text = {
                Text(text = "Item is $it")
            }, secondaryText = {
                Text(text = "second title")
            }, modifier = Modifier.clickable {
                coroutineScope.launch {
                    lazyState.animateScrollToItem(data.size - 1)
                }
            })

            //可以坚挺条目的生命周期
            DisposableEffect(Unit) {
                Log.d("======", "effect = $it ")
                onDispose {
                    Log.d("======", "onDispose = $it")
                }
            }
        }
    }
}
```

![image-20230731114932041](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230731114932041.png)
