#### state

```kotlin
@Composable
fun StateSample() {
//    val count = remember {
//        mutableStateOf(1)
//    }
    var count by remember {
        mutableStateOf(1)
    }
    Text(text = "今天叫了${count}个小姐姐", modifier = Modifier
        .padding(10f.dp)
        .clickable {
            count++
        })
}

@Composable
fun StateWithLiveDataSample(myViewModel: MyViewModel) {
    val uiState = myViewModel.uiState
    Button(onClick = {
        myViewModel.changeUiState()
    }) {
        Text(text = uiState)
    }

}

@Preview
@Composable
fun StateSamplePreview() {
    StateSample()
}


class MyViewModel : ViewModel() {

    private val _myData = MutableLiveData<String>()
    val myData: LiveData<String> = _myData

    var uiState by mutableStateOf("zhangqi")
        private set

    fun changeUiState() {
        uiState = "dashingqi"
    }

    fun updateData(newData: String) {
        _myData.value = newData
    }
}

```

