### Text

###### 简单属性

```kotlin
@Composable
fun TextSample() {
//    Text(text = "我终于要学会 Jetpack Compose 了")
    Text(
        // 文字内容
        text = stringResource(id = R.string.text),
        // 文字颜色
        color = Color.Red,
        // 文字大小
        fontSize = 14f.sp, //TextUnit(18f, TextUnitType.Sp)
        // 文字字体
        fontFamily = FontFamily.SansSerif,
        // 子间距
//        letterSpacing = 5.sp,
        // 装饰
        textDecoration = TextDecoration.combine(listOf(TextDecoration.LineThrough, TextDecoration.Underline)),
        // 设置斩首行数
        maxLines = 1,
        // 展示不全，文字展示规则
        overflow = TextOverflow.Ellipsis,
        // 文字对齐规则
        textAlign = TextAlign.Center,
//        style = 通用的样式可以用这个封装
    )

}
```

###### 稍微高级的用法

- 可点击的Text

```kotlin
val customBuildAnnotationString = buildAnnotatedString {
        append("我学会了")
  
        pushStringAnnotation("zq", "zhangqi")
        withStyle(
            style = SpanStyle(
                color = Color.Red, textDecoration = TextDecoration.Underline, fontSize = 18f.sp
            )
        ) {
            append(" compose")
        }
        pop()
				
  			append("hello World")
  
        pushStringAnnotation("dq", "dashignqi")
        withStyle(
            style = SpanStyle(
                color = Color.Red, textDecoration = TextDecoration.Underline, fontSize = 18f.sp
            )
        ) {
            append("嘿嘿哦呵呵呵呵呵")
        }
        pop()
    }


    // 点击方法
    ClickableText(text = customBuildAnnotationString, onClick = { offset ->
        customBuildAnnotationString.getStringAnnotations("zq", offset, offset).firstOrNull()?.let {
            Log.d("DQCompose", "你点击了我 ---> ${it.item}")
        }
        customBuildAnnotationString.getStringAnnotations("dq", offset, offset).firstOrNull()?.let {
            Log.d("DQCompose", "你点击了我 ---> ${it.item}")
        }

    })
```

- 被选中的Text

```kotlin
 SelectionContainer {
        Text(
            // 文字内容
            text = stringResource(id = R.string.text),
            // 文字颜色
            color = Color.Red,
            // 文字大小
            fontSize = 14f.sp, //TextUnit(18f, TextUnitType.Sp)
            // 文字字体
            fontFamily = FontFamily.SansSerif,
            // 子间距
//        letterSpacing = 5.sp,
            // 装饰
            textDecoration = TextDecoration.combine(listOf(TextDecoration.LineThrough, TextDecoration.Underline)),
            // 设置斩首行数
            maxLines = 1,
            // 展示不全，文字展示规则
            overflow = TextOverflow.Ellipsis,
            // 文字对齐规则
            textAlign = TextAlign.Center,
//        style = 通用的样式可以用这个封装
        )
    }
```

