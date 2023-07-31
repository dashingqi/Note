#### Box

```kotlin

@Composable
fun BoxSample() {
  Box() {

        Box(
            modifier = Modifier
                .size(200.dp)
                .background(Color.Red)
        )
        Text(
            text = "zhangqi", modifier = Modifier
                .padding(7.dp)
                .background(Color.Yellow)
                .align(Alignment.Center)
        )
        Text(
            text = "dashingqi", modifier = Modifier
                .padding(7.dp)
                .background(Color.Green), fontSize = 18F.sp
        )
    }
}
```

![image-20230731120242704](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230731120242704.png)

###### BoxWithConstraints

```kotlin
@Composable
fun BoxSample() {
BoxWithConstraints {
        if (maxWidth > maxHeight) {
            Box(
                modifier = Modifier
                    .size(200.dp)
                    .background(Color.Red)
            )
        } else {

            Box(
                modifier = Modifier
                    .size(100.dp)
                    .background(Color.Yellow)
                    .align(Alignment.Center)
            )
        }
    }
}
```

