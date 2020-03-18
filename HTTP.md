# HTTP 的安全问题		

​		目前大多数网站和app的接口都是采用http协议，但是http协议很容易就通过抓包工具监听到内容，甚至可以篡改内容，为了保证数据不被别人看到和修改，可以通过以下几个方面避免。

​		重要的数据，要加密，比如用户名密码，我们需要加密，这样即使被抓包监听，他们也不知道原始数据是什么（如果简单的md5，是可以暴力破解），所以加密方法越复杂越安全，根据需要，常见的是 md5(不可逆)，aes（可逆），自由组合吧,你还可以加一些特殊字符啊，没有做不到只有想不到， 举例：username = aes(username), pwd = MD5(pwd + username);。。。。。

​		非重要数据，要签名，签名的目的是为了防止篡改，比如http://www.xxx.com/getnews?id=1，获取id为1的新闻，如果不签名那么通过id=2,就可以获取2的内容等等。怎样签名呢？通常使用sign，比如原链接请求的时候加一个sign参数，sign=md5(id=1)，服务器接受到请求，验证sign是否等于md5(id=1)，如果等于说明正常请求。这会有个弊端，假如规则被发现，那么就会被伪造，所以适当复杂一些，还是能够提高安全性的。

​		登录态怎么做，http是无状态的，也就是服务器没法自己判断两个请求是否有联系，那么登录之后，以后的接口怎么判定是否登录呢，简单的做法，在数据库中存一个token字段（名字随意），当用户调用登陆接口成功的时候，就将该字段设一个值，（比如aes(过期时间)），同时返回给前端，以后每次前端请求带上该值，服务器首先校验是否过期，其次校验是否正确，不通过就让其登陆。（redis 做这个很方便哦，key有过期时间）

 

aes:https://blog.csdn.net/qq_28205153/article/details/55798628





# HTTP 与 HTTPS 的区别

1. **加密：**http协议对传输的数据不进行加密；https协议对传输的数据使用SSL安全协议进行加密，https加密需要CA签发的证书。 
2. **端口：**http协议使用TCP的80端口；https协议使用TCP的443端口 
3. 网络分层模型：http可以明确是位于应用层；https是在http的基础上加上了SSL安全协议，而SSL是运输层协议，所以https是应用层和传输层的结合（我不同意网上将https粗暴地归为运输层的说法）





# HTTPS密钥交换

HTTPS使用SSL安全协议来保障安全性。具体体现在密钥和证书验证上。 

+ **密钥：**

1. 服务端生成一对公钥和私钥，将公钥和证书发送给客户端； 
2. 客户端验证证书通过后生成一个对称加密的密钥，并使用服务器生成的公钥加密，发送给服务器； 
3. 服务器使用私钥解密获得对称加密密钥。 
4. 客户端和服务器相互发送消息认可对称加密密钥，至此加密通道建立。 
5. 开始数据传输，在检验数据完整性的基础上，使用对称加密密钥进行加密解密。


+ **证书验证：**
     一般来说浏览器都内置了权威CA的根证书，客户端使用根证书的密钥对服务器发来的证书进行解密验证，若域名、有效期、签发机关等验证项符合则通过，否则认定证书无效，断开连接。





# HTTP协议简介

​		HTTP（超文本传输协议）是应用层上的一种客户端/服务端模型的通信协议,它由请求和响应构成，且是无状态的。（暂不介绍HTTP2）

+ 协议

  协议规定了通信双方必须遵循的数据传输格式，这样通信双方按照约定的格式才能准确的通信。

+ 无状态

  无状态是指两次连接通信之间是没有任何关系的，每次都是一个新的连接，服务端不会记录前后的请求信息。

+ 客户端/服务端模型
  <img src="file:////private/var/folders/nq/c0jgjyfj3ld3xjykp1z9dhlw0000gn/T/com.kingsoft.wpsoffice.mac/wps-arvin/ksohtml/wpsjJYM7Q.png" alt="img" style="zoom:50%;" />



<img src="file:////private/var/folders/nq/c0jgjyfj3ld3xjykp1z9dhlw0000gn/T/com.kingsoft.wpsoffice.mac/wps-arvin/ksohtml/wpsSaqP8m.png" alt="img" style="zoom:50%;" />
***\*协议内容\****

### 请求（Request）

客户端发送一个HTTP请求到服务端的格式：

+ 请求行
+ 请求头
+ 请求体

![img](https://www.runoob.com/wp-content/uploads/2013/11/2012072810301161.png)

<img src="https://www.runoob.com/wp-content/uploads/2013/11/httpmessage.jpg" alt="img" style="zoom:67%;" />



### 响应（Response）

服务端响应客户端格式：

+ 状态行
+ 响应头
+ 响应体
  <img src="file:////private/var/folders/nq/c0jgjyfj3ld3xjykp1z9dhlw0000gn/T/com.kingsoft.wpsoffice.mac/wps-arvin/ksohtml/wps9hyXHs.png" alt="img" style="zoom: 50%;" />







# 状态码

HTTP状态码由三个十进制数字组成，第一个十进制数字定义了状态码的类型，后两个数字没有分类的作用。HTTP状态码共分为5种类型：
<img src="file:////private/var/folders/nq/c0jgjyfj3ld3xjykp1z9dhlw0000gn/T/com.kingsoft.wpsoffice.mac/wps-arvin/ksohtml/wpsYRbdi3.png" alt="img" style="zoom:67%;" />

| 状态码 |         状态码英文名称          | 中文描述                                                     |
| :----: | :-----------------------------: | :----------------------------------------------------------- |
|  100   |            Continue             | 继续。[客户端](http://www.dreamdu.com/webbuild/client_vs_server/)应继续其请求 |
|  101   |       Switching Protocols       | 切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议 |
|        |                                 |                                                              |
|  200   |               OK                | 请求成功。一般用于GET与POST请求                              |
|  201   |             Created             | 已创建。成功请求并创建了新的资源                             |
|  202   |            Accepted             | 已接受。已经接受请求，但未处理完成                           |
|  203   |  Non-Authoritative Information  | 非授权信息。请求成功。但返回的meta信息不在原始的服务器，而是一个副本 |
|  204   |           No Content            | 无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档 |
|  205   |          Reset Content          | 重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域 |
|  206   |         Partial Content         | 部分内容。服务器成功处理了部分GET请求                        |
|        |                                 |                                                              |
|  300   |        Multiple Choices         | 多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择 |
|  301   |        Moved Permanently        | 永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替 |
|  302   |              Found              | 临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI |
|  303   |            See Other            | 查看其它地址。与301类似。使用GET和POST请求查看               |
|  304   |          Not Modified           | 未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源 |
|  305   |            Use Proxy            | 使用代理。所请求的资源必须通过代理访问                       |
|  306   |             Unused              | 已经被废弃的HTTP状态码                                       |
|  307   |       Temporary Redirect        | 临时重定向。与302类似。使用GET请求重定向                     |
|        |                                 |                                                              |
|  400   |           Bad Request           | 客户端请求的语法错误，服务器无法理解                         |
|  401   |          Unauthorized           | 请求要求用户的身份认证                                       |
|  402   |        Payment Required         | 保留，将来使用                                               |
|  403   |            Forbidden            | 服务器理解请求客户端的请求，但是拒绝执行此请求               |
|  404   |            Not Found            | 服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页面 |
|  405   |       Method Not Allowed        | 客户端请求中的方法被禁止                                     |
|  406   |         Not Acceptable          | 服务器无法根据客户端请求的内容特性完成请求                   |
|  407   |  Proxy Authentication Required  | 请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权 |
|  408   |        Request Time-out         | 服务器等待客户端发送的请求时间过长，超时                     |
|  409   |            Conflict             | 服务器完成客户端的 PUT 请求时可能返回此代码，服务器处理请求时发生了冲突 |
|  410   |              Gone               | 客户端请求的资源已经不存在。410不同于404，如果资源以前有现在被永久删除了可使用410代码，网站设计人员可通过301代码指定资源的新位置 |
|  411   |         Length Required         | 服务器无法处理客户端发送的不带Content-Length的请求信息       |
|  412   |       Precondition Failed       | 客户端请求信息的先决条件错误                                 |
|  413   |    Request Entity Too Large     | 由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息 |
|  414   |      Request-URI Too Large      | 请求的URI过长（URI通常为网址），服务器无法处理               |
|  415   |     Unsupported Media Type      | 服务器无法处理请求附带的媒体格式                             |
|  416   | Requested range not satisfiable | 客户端请求的范围无效                                         |
|  417   |       Expectation Failed        | 服务器无法满足Expect的请求头信息                             |
|        |                                 |                                                              |
|  500   |      Internal Server Error      | 服务器内部错误，无法完成请求                                 |
|  501   |         Not Implemented         | 服务器不支持请求的功能，无法完成请求                         |
|  502   |           Bad Gateway           | 作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应 |
|  503   |       Service Unavailable       | 由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中 |
|  504   |        Gateway Time-out         | 充当网关或代理的服务器，未及时从远端服务器获取请求           |
|  505   |   HTTP Version not supported    | 服务器不支持请求的HTTP协议的版本，无法完成处理               |



# 请求方法

截止到HTTP1.1共有下面几种方法：
<img src="file:////private/var/folders/nq/c0jgjyfj3ld3xjykp1z9dhlw0000gn/T/com.kingsoft.wpsoffice.mac/wps-arvin/ksohtml/wpszeMcEQ.png" alt="img" style="zoom: 67%;" />



# content-type

### 常见的媒体格式类型

- text/html ： HTML格式
- text/plain ：纯文本格式
- text/xml ： XML格式
- image/gif ：gif图片格式
- image/jpeg ：jpg图片格式
- image/png：png图片格式



### 以application开头的媒体格式类型

- application/xhtml+xml ：XHTML格式
- application/xml： XML数据格式
- application/atom+xml ：Atom XML聚合格式
- application/json： JSON数据格式
- application/pdf：pdf格式
- application/msword ： Word文档格式
- application/octet-stream ： 二进制流数据（如常见的文件下载）
- application/x-www-form-urlencoded ： <form encType=””>中默认的encType，form表单数据被编码为key/value格式发送到服务器（表单默认的提交数据的格式）

另外一种常见的媒体格式是上传文件之时使用的：

- multipart/form-data ： 需要在表单中进行文件上传时，就需要使用该格式