## 与HTTP相关的各种协议

#### TCP/IP

TCP/IP协议时一系列网络通信协议的统称，最核心的两个协议时TCP和IP，还有其他的【UDP】、【ICMP】、【ARP】等，共同构建了一个复杂有层次的协议栈

##### 协议栈

协议栈共分为四层：上层【应用层】最下层【链接层】，TCP和IP在这中间；

TCP属于【传输层】IP属于【网际层】；

##### IP协议

###### 基本

- 缩写【Internet Protocol】

###### 解决的问题/功能

- 解决寻址和路由的问题
- 如何在两点之间传递数据包

###### IP地址

- IP协议使用【IP地址】的概念来定位互联网上的每一台计算机；

- V4版和V6版本
  - V4 : 有2^32 约42亿可以分配的地址
  - V6 : 2^128 

##### TCP协议

###### 基本

- 缩写【Transmission Control Protocol】【传输控制协议】
- 它是位于IP协议之上

###### 解决问题/功能

- 基于IP协议提供【可靠的】【字节流】形式的通信
- 是HTTP协议得以实现的基础

###### 可靠

- 保证数据不丢失

###### 字节流

- 保证数据完整；

##### DNS

###### 基本

- 域名系统【Domain Name System】；
- 用有意义的名字作为IP地址的等价替代；

###### 解决问题/功能

- 域名解析
  - 使用TCP/IP协议来通信仍然要使用【IP】地址，需要把域名做下转换，【映射】到真实的IP上；

##### URI/URL

有了TCP/IP 和DNS后，我们还不能任意访问网络的资源；

我们需要告诉应答方，我们具体要访问那个资源

###### 基本

- URI:  统一资源标识符 (Uniform Resource Identifier)
- URL: 统一资源定位符 (Uniform Resource Lactor),我们俗称网址，URL是URI的一个字集

###### URI基本构成

http://nginx.org/en/download.html

- 协议名:【http】
- 主机名 ：【nginx.org】
- 路径：【/en/download.html】

##### HTTPS

HTTP是不安全的，容易被截获，串改；

HTTPS是能解决这个问题；

###### 基本

HTTPS是运行在SSL/TLS协议上的HTTP；

HTTPS = HTTP + SSL/TLS + TCP/IP

###### SSL/TLS

SSL 全称【Secure Socket Layer】,是由网景公司发明的，发展到【】

##### 代理



