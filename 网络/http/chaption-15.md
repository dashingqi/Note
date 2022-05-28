## HTTP的Body

#### 数据类型与编码

对于HTTP（TCP/IP）协议中，传输的数据基本上都是【header+body】；

对于TCP/IP来说，它是属于传输层协议只管传输数据到对方，不关心数据是什么；

但是对于HTTP来说，它是属于应用层协议，保证数据传输到大对方只是完成了一半，它还要告知上层应用传递的body数据是什么类型的；

##### 类型与编码

###### MIME type

【MIME type】就是用于标记body数据类型的

早在HTTP协议诞生之前，就已经有了针对上述问题的解决方案，就是【MIME】

HTTP中常用的MIME type 有如下几类

- text：文本格式的可读数据
  - text/html:超文本文档
  - text/plain:纯文本
  - text/css:样式表
- iamge：图像文件
  - image/gif
  - image/jpeg
  - image/png
- audio/video：音频和视频数据
  - audio/mpeg
  - Video/mp4
- application：数据格式不固定，可能是文本也可能是二进制；
  - application/json
  - application/javascript
  - application/pdf
  - application/octet-stream 不透明的二进制数据；

###### Encoding type

在数据传输时，为了节约带宽，有时候还会压缩数据，为了让Client能认出数据，需要Encoding type，用于告知数据是用什么编码格式的；

常用的Encoding type有如下几类

- gzip:GNU zip压缩格式，互联网上最流行的压缩；
- deflate：zlib压缩格式，流行层序仅次于gzip
- br:一种专门为HTTP优化的新压缩算法

##### 头字段中类型与编码的体现

有了上述MIME type 和Encoding type，client或者server就能识别出body数据的类型了；

在hTTP协议中在头字段中 定定义了两个Accept请求头字段和两个Content实体头字段，用于Client和Server进行【内容协商】；

下图就是MIME type 和 Encoding type 在头字段中的体现

Accept和Accept-Encoding用于请求头中，分别对应MIME type 和Encoding type

Content-Type和Content-Encoding用于响应头中，分别对应MIME type和Encoding type

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/202205262059852.png" alt="数据类型和编码在头字段中的表现" style="zoom:150%;" />

#### 语言类型与编码

##### 语言类型与字符集

###### 语言类型

语言类型就是人类使用的自然语言【英语】、【汉语】；

在明确区分的时候需要使用【type-subtype】形式

en:任意英语、en-US:表示美式英语、en-GB:英式英语

zh-CN 我们的汉语

###### 字符集

早期各个国家都有自己的字符编码处理文字

英语：ASCII

汉语：GBK

后来出现的Unicode和UTF-8把世界上所有的语言容纳到一种编码方案中；

UTF-8也成为互联网上的标准字符集；

##### 语言类型和字符集在头字段中的体现

语言类型：HTTP协议中在请求头中使用Accpet-Language；在响应头中使用Content-Language

字符集：HTTP协议中在请求头中使用Accept-Charset；在响应头中没有Content-Charset；是在Content-Type字段的数据类型后面用【charset=xxx】来表示

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/202205262059900.png" alt="语言类型和字符集在头字段中的体现" style="zoom:150%;" />

#### 内容协商的质量值

在HTTP协议中，我们上述讲解的 【数据类型与编码】、【语言类型与编码】

在头字段中一个key都对应多个value；那么当指定多个value的时候我们怎么区分这个优先级呢？

这个时候就需要【内容协商的质量值】【q】-->【quality favtor】

【q】有一个对应的值区间【1 0.01】

最大值是1，最小值是0.01，默认值是1

比如下面的（在HTTP中 【;】的意义是要小于【,】的）

```java
Accept : text/html, application/json;q=0.9,*/*;q=0.8
```

解释一下：

客户端最希望使用的HTML文件，权重是1；

如果你不能满足HTML，我接下来希望的是json格式的数据，权重是0.9

最后才是任意类型数据类型，权重是0.8



#### 害

想一想在北京都3年多了；

最近一直都在坚持跑步，5公里的成绩也在慢慢提高从5分配到现在轻松的440～430,在到最好420；经过了很长的一段时间

我跑步的原因很简单，就是很享受跑步过程中的挣扎（就看提高配速的情况下自己能在坚持多长时间）以及跑后那种兴奋感；

前天晚上跑完站在附件商场楼前，看了好久，那个地方算是附近很繁华的地方，晚上灯光也很漂亮；

如果再过几年不在北京了，想一想这些繁华这些硬件资源你都带不走，你能带走的只有你健康的身体、你的经历经验；你走了你就走了，这个城市你如果再不来可能就跟你没多大关系了；

大家五湖四海汇聚在北京，相识真是的天大的缘分，能走到一起的真的要好好珍惜；

有时候真的有不可抗拒的外在因素，地域、认知、价值观；

大话西游中的经典台词【曾经有一份真挚的爱情摆在我面前我没有珍惜】对于好多好多太多人来说，这一别可真就是一辈子了；



