#### Button

```kotlin
@Composable                                                                                    
fun ButtonSample() {                                                                           
//    Button(                                                                                  
//        onClick = { Log.d(TAG, "你点击我了"); }, colors = ButtonDefaults.buttonColors(            
//            backgroundColor = Color.Blue, contentColor = Color.Yellow                         
//        ), border = BorderStroke(Dp(2f), Color.Gray), enabled = true                         
//    ) {                                                                                      
//        Text(text = "这是一个按钮")                                                                
//    }                                                                                        
    // 简单的文本按钮                                                                                 
//    TextButton(onClick = {}) {                                                               
//        Text(text = "this is text button")                                                   
//                                                                                             
//    }                                                                                        
                                                                                               
    //带有边框的Button                                                                              
    OutlinedButton(onClick = { /*TODO*/ }) {                                                    
        Text(text = "我是中国人")                                                                   
    }                                                                                          
}                                                                                              
                                                                                                                                                                                   
```

![image-20230730112444172](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/image-20230730112444172.png)

