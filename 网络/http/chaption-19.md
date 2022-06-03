## 🐂HTTP中的Cookie机制🐂

HTTP本身是【无状态】，对于HTTP来说无状态这个特点是一把双刃剑

优点就是服务器不用存储状态。可以容易组成集群，缺点就是无法记录状态的事物操作；

后来出现了Cookie机制，由于HTTP本身的优点中就有可扩展，Cookie机制的加入这就给HTTP增加了【记忆功能】

#### 啥是Cookie

#### Cookie的工作过程

##### 头字段

响应头字段【Set-Cookie】和请求头字段【Cookie】

##### 工作过程

###### 创建Cookie

当浏览器第一个访问服务器时，服务器不知道这哥们是谁，就需要创建一个身份标识数据，格式时【key=value】，然后放进Set-Cookie字段里，随着响应报文一同发给浏览器；

###### 接受存储Cookie

浏览器收到响应报文时就查找Set-Cookie中给的身份，并且【存储起来】，等着下次请求时就自动把这个值放进Cookie字段里面发送给服务器；

###### 使用Cookie

当第二次请求时，请求报文中就携带了Cookie字段，服务器收到后就知道这哥们我认识，就拿出Cookie中的值，识别出了用户画像，提供个性化服务；

有时服务器会在响应头中添加多个Set-Cookie，浏览器这边就存储多对【key-value】，浏览器这边再次请求时就使用一个Cookie，对应多对【key-value】使用【;】进行分隔；

###### Cookie工作示意图

![HTTP中Cookie的工作机制](https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/202206031719357.png)

***需要明确***

cookie是由客户端存储的（浏览器），当我们换浏览器或者设备时（电脑）服务器就不认得我们啦；

#### Cookie的属性

所谓的Cookie属性就是用来保护Cookie防止外泄或者窃取；

##### Cookie的生存周期

Cookie的有效期，让它只能在一段时间内可以用；

Cookie的有效期可以使用【Expires】和【Max-Age】两个属性设置；

###### Expires

俗称过期时间，可以理解为截止日期

###### Max-Age

用的是相对时间，当客户端收到报文的时间点加上Max-Age，就可以得到失效的绝对时间；

客户端会 优先采用Max-Age的机制计算失效期；

##### Cookie的作用域

原理很简单，响应报文中Set-Cookie中，会携带【Domain】和【Path】的属性，

当我们浏览器请求时会从URI中提取host和path部分，对比Domain和Path的属性，如果匹配不成功，请求头字段中就不带Cookie了；

使用Cookie作用域可以为不同的域名和路径设置不同的Cookie；

##### Cookie的安全性

在日常的开发中。FE同学可以在JS脚本中用【document.cookie】来读写Cookie数据，这就带来隐患，可能会导致【跨站脚本】攻击窃取数据；

###### HTTPOnly

会告诉浏览器，该Cookie只能通过浏览器HTTP协议传输，禁止其他的方式访问；

那么JS引擎就会禁用documen.cookie的API

###### SameSite

可以防范【跨站请求伪造】攻击，有如下三种值可设置；

***SameSite=Strict***

严格规定Cookie不能随着跳转链接跨站发送

***SameSite=Lax***

稍微宽松一点，允许GET/HEAD等方法携带，但是禁止POST跨站携带发送

***SimeSite=None;Secure***

标识这个Cookie发送只能使用HTTPS协议加密传输，明文的HTTP协议会禁止发送；

最近在做Target31升级，就有SameSite属性变更的改造，由于业务方很多包含客户端、Server、FE这个改造成本很大，后面有机会写写这个改造过程；

