#### 新建文件的代码模版

```
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME}

#end
#parse("File Header.java")

import androidx.compose.runtime.Composable
import androidx.compose.ui.tooling.preview.Preview

#if (${Function_Name} == "" )
@Composable
fun ${NAME}() {
}

@Preview
@Composable
fun ${NAME}Preview() {
    ${NAME}()
}
#end

#if (${Function_Name} != "" )
@Composable
fun ${Function_Name}() {
}

@Preview
@Composable
fun ${Function_Name}Preview() {
    ${Function_Name}()
}
#end

```

#### Live Templates

Live Templates --> Android Compose



