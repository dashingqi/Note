#### Java的char是两个字节，如何存储UTF-8字符



1个char是两个字节

char是两个字节存储数据 --> 存储的事UTF-16

UTF8字符是通过1-3个字节存储数据

### 字符集

常见的字符集有如下两种

###### ASCII

![ASCII字符集表](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/14/172139e153703b76~tplv-t2oaga2asx-watermark.awebp)

#### Unicode

它将世界上所有的符号都纳入其中，成功实现了每个数字代表唯一的至少在某种语言中使用的符号；

Unicode兼容ASCII，即0 - 127 的意义依然不变。

##### 码点

码点就相当于ASCII中ASCII值，它既是Unicode字符集中唯一表示某个字符的标识，在Unicode中称为码点。

##### Unicode编码

Unicode字符集衍生出的编码方案有三种：UTF-32、UTF-16和UTF-8；

###### UTF-32

属于定长编码，永远用4字节存储码点

###### UTF-16

属于变长存储，使用2或者4字节来存储码点

###### UTF-8

属于变长存储，根据不同的情况使用1-4字节来存储码点；



