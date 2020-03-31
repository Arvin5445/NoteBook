# HTTP 的安全问题		

​		目前大多数网站和app的接口都是采用http协议，但是http协议很容易就通过抓包工具监听到内容，甚至可以篡改内容，为了保证数据不被别人看到和修改，可以通过以下几个方面避免。

​		重要的数据，要**加密**，比如用户名密码，我们需要加密，这样即使被抓包监听，他们也不知道原始数据是什么（如果简单的md5，是可以暴力破解），所以加密方法越复杂越安全，根据需要，常见的是 md5(不可逆)，aes（可逆），自由组合吧,你还可以加一些特殊字符啊，没有做不到只有想不到， 举例：username = aes(username), pwd = MD5(pwd + username);。。。。。

​		非重要数据，要**签名**，签名的目的是为了防止篡改，比如http://www.xxx.com/getnews?id=1，获取id为1的新闻，如果不签名那么通过id=2,就可以获取2的内容等等。怎样签名呢？通常使用sign，比如原链接请求的时候加一个sign参数，sign=md5(id=1)，服务器接受到请求，验证sign是否等于md5(id=1)，如果等于说明正常请求。这会有个弊端，假如规则被发现，那么就会被伪造，所以适当复杂一些，还是能够提高安全性的。

​		**登录态**怎么做，http是无状态的，也就是服务器没法自己判断两个请求是否有联系，那么登录之后，以后的接口怎么判定是否登录呢，简单的做法，在数据库中存一个token字段（名字随意），当用户调用登陆接口成功的时候，就将该字段设一个值，（比如aes(过期时间)），同时返回给前端，以后每次前端请求带上该值，服务器首先校验是否过期，其次校验是否正确，不通过就让其登陆。（redis 做这个很方便哦，key有过期时间）

 

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

​		HTTP（超文本传输协议）是应用层上的一种客户端/服务端模型的通信协议,它由请求和响应构成，且是无状态的。

​		早在 HTTP 建立之初，主要就是为了将超文本标记语言(HTML)文档从Web服务器传送到客户端的浏览器。也是说对于前端来说，我们所写的HTML页面将要放在我们的 web 服务器上，用户端通过浏览器访问url地址来获取网页的显示内容，但是到了 WEB2.0 以来，我们的页面变得复杂，不仅仅单纯的是一些简单的文字和图片，同时我们的 HTML 页面有了 CSS，Javascript，来丰富我们的页面展示，当 ajax 的出现，我们又多了一种向服务器端获取数据的方法，这些其实都是基于 HTTP 协议的。同样到了移动互联网时代，我们页面可以跑在手机端浏览器里面，但是和 PC 相比，手机端的网络情况更加复杂，这使得我们开始了不得不对 HTTP 进行深入理解并不断优化过程中。

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

![img](https://images2017.cnblogs.com/blog/1090126/201711/1090126-20171115074903718-2136115327.png)







# 状态码

​		HTTP状态码由三个十进制数字组成，第一个十进制数字定义了状态码的类型，后两个数字没有分类的作用。HTTP状态码共分为5种类型：
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





# HTTP 1.1

​		HTTP1.0最早在网页中使用是在1996年，那个时候只是使用一些较为简单的网页上和网络请求上，而HTTP1.1则在1999年才开始广泛应用于现在的各大浏览器网络请求中，同时HTTP1.1也是当前使用最为广泛的HTTP协议。 主要区别主要体现在：

1. **缓存处理**，在HTTP1.0中主要使用header里的If-Modified-Since,Expires来做为缓存判断的标准，HTTP1.1则引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since, If-Match, If-None-Match等更多可供选择的缓存头来控制缓存策略。

2. **带宽优化及网络连接的使用**，HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。

3. **错误通知的管理**，在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。

4. **Host头处理**，在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。

5. **长连接**，HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection： keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。

   

   

# HTTPS

- HTTPS协议需要到CA申请证书，一般免费证书很少，需要交费。
- HTTP协议运行在TCP之上，所有传输的内容都是明文，HTTPS运行在SSL/TLS之上，SSL/TLS运行在TCP之上，所有传输的内容都经过加密的。
- HTTP和HTTPS使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
- HTTPS可以有效的防止运营商劫持，解决了防劫持的一个大问题。





# HTTP 2.0

**HTTP2.0和SPDY的区别**

1. HTTP2.0 支持明文 HTTP 传输，而 SPDY 强制使用 HTTPS
2. HTTP2.0 消息头的压缩算法采用 **HPACK** http://http2.github.io/http2-spec/compression.html，而非 SPDY 采用的 **DEFLATE** http://zh.wikipedia.org/wiki/DEFLATE

 

**HTTP2.0和HTTP1.X相比的新特性**

- **新的二进制格式**（Binary Format），HTTP1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑HTTP2.0的协议解析决定采用二进制格式，实现方便且健壮。

- **多路复用**（MultiPlexing），即连接共享，即每一个request都是是用作连接共享机制的。一个request对应一个id，这样一个连接上可以有多个request，每个连接的request可以随机的混杂在一起，接收方可以根据request的 id将request再归属到各自不同的服务端请求里面。

- **header压缩**，如上文中所言，对前面提到过HTTP1.x的header带有大量信息，而且每次都要重复发送，HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。

- **服务端推送**（server push），同SPDY一样，HTTP2.0也具有server push功能。

   

**HTTP2.0的升级改造**

- 前文说了HTTP2.0其实可以支持非HTTPS的，但是现在主流的浏览器像chrome，firefox表示还是只支持基于 TLS 部署的HTTP2.0协议，所以要想升级成HTTP2.0还是先升级HTTPS为好。
- 当你的网站已经升级HTTPS之后，那么升级HTTP2.0就简单很多，如果你使用NGINX，只要在配置文件中启动相应的协议就可以了，可以参考**NGINX白皮书，NGINX配置HTTP2.0官方指南** https://www.nginx.com/blog/nginx-1-9-5/。
- 使用了HTTP2.0那么，原本的HTTP1.x怎么办，这个问题其实不用担心，HTTP2.0完全兼容HTTP1.x的语义，对于不支持HTTP2.0的浏览器，NGINX会自动向下兼容的。

 

**HTTP2.0的多路复用和HTTP1.X中的长连接复用有什么区别？**

- HTTP/1.* 一次请求-响应，建立一个连接，用完关闭；每一个请求都要建立一个连接；
- HTTP/1.1 Pipeling解决方式为，若干个请求排队串行化单线程处理，后面的请求等待前面请求的返回才能获得执行机会，一旦有某请求超时等，后续请求只能被阻塞，毫无办法，也就是人们常说的线头阻塞；
- HTTP/2多个请求可同时在一个连接上并行执行。某个请求任务耗时严重，不会影响到其它连接的正常执行。



**HTTP2.0多路复用有多好？**

​		HTTP 性能优化的关键并不在于高带宽，而是低延迟。TCP 连接会随着时间进行自我「调谐」，起初会限制连接的最大速度，如果数据成功传输，会随着时间的推移提高传输的速度。这种调谐则被称为 TCP 慢启动。由于这种原因，让原本就具有突发性和短时性的 HTTP 连接变的十分低效。

​		HTTP/2 通过让所有数据流共用同一个连接，可以更有效地使用 TCP 连接，让高带宽也能真正的服务于 HTTP 的性能提升。

