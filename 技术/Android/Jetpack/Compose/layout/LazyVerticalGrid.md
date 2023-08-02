#### LazyverticalGrid

```kotlin
@OptIn(ExperimentalFoundationApi::class)
@Composable
fun LazyVerticalGridSample() {

    val itemList = listOf(0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    Column {


        LazyVerticalGrid(cells = GridCells.Fixed(3),content = {
            items(itemList) {
                Card {
                    Text(text = "Item is $it", modifier = Modifier.padding(8.dp), textAlign = TextAlign.Center)
                }
            }
        })

        LazyVerticalGrid(cells = GridCells.Adaptive(150.dp)) {
            items(itemList) {
                Card {
                    Text(text = "Item is $it", modifier = Modifier.padding(8.dp), textAlign = TextAlign.Center)
                }
            }
        }
    }
}

@Preview
@Composable
fun LazyVerticalGridSamplePreview() {
    LazyVerticalGridSample()
}
```

![image-20230802232335320](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230802232335320.png)