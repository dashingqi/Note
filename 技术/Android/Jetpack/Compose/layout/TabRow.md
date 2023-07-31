#### TabRow 基础用法

```kotlin
Preview
@Composable
fun TabRowPreView() {
    TabRowSample()

}

@OptIn(ExperimentalMaterialApi::class)
@Composable
fun TabRowSample() {
    Column {

        var selectedIndex by remember {
            mutableStateOf(0)
        }

        TabRow(
            selectedTabIndex = selectedIndex,
            indicator = {},
            backgroundColor = Color(0xFFFFFFFF)
        ) {

            Tab(
                selected = selectedIndex == 0,
                onClick = { selectedIndex = 0 },
                selectedContentColor = Color.Red,
                unselectedContentColor = Color.Black
            ) {
                Text(text = "Tab0")

            }

            Tab(
                selected = selectedIndex == 1,
                onClick = { selectedIndex = 1 },
                icon = {
                    Icon(
                        imageVector = Icons.Default.Delete,
                        contentDescription = null
                    )
                },
                text = {
                    Text(text = "Tab1")
                }
            )


            // 这是一个不稳定的api
            LeadingIconTab(
                selected = selectedIndex == 2,
                onClick = { selectedIndex = 2 },
                icon = {
                    Icon(
                        imageVector = Icons.Default.AccountBox,
                        contentDescription = null
                    )
                },
                text = {
                    Text(text = "Tab3")
                })
        }

        Text(text = "current index is $selectedIndex")
    }

}
```

