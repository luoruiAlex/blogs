## 报文
- HTTP报文大致可分为报文首部和报文主体两块，两者由空行（就相当于用了两个换行符rnrn）来划分。报文主体并不是一定要有的。
- 请求报文
  - METHOD \+ URI \+ HTTP/1.1(协议版本)
  - 请求首部
  - 报文主体
- 响应报文
  - HTTP/1.1(协议版本) \+ statusCode \+ OK(状态码原因短语)
  - 响应首部
  - 响应内容实体
  
## URI URL
- URI，是uniform resource identifier，统一资源标识符，用来唯一的标识一个资源
- URL，是uniform resource locator，统一资源定位器，它是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何locate这个资源。URL包含了协议

## 状态
- HTTP是无状态协议，前后两次请求不能标识为一个
- 可用cookie来解决状态问题

## HTTP1.1
- 默认长连接
- 客户端可同时发送多个HTTP请求
- 支持断点续传：实际上就是利用HTTP消息头使用分块传输编码，将实体主体分块传输。

### HTTP2.0
- 二进制格式而非文本格式
- header数据压缩
- 多路复用(HTTP1.1中，浏览器客户端在同一时间，针对同一域名下的请求有一定数量限制。超过限制数目的请求会被阻塞）即一个连接即可并行
- 支持服务器主动推送给客户端缓存

## HTTP HTTPS
- HTTP 的URL 以http:// 开头，而HTTPS 的URL 以https:// 开头
- HTTP 是不安全的，而 HTTPS 是安全的
- HTTP 标准端口是80 ，而 HTTPS 的标准端口是443
- 在OSI 网络模型中，HTTP工作于应用层，而HTTPS 的安全传输机制工作在传输层
- HTTP 无法加密，而HTTPS 对传输的数据进行加密
- HTTP无需证书，而HTTPS 需要CA机构wosign的颁发的SSL证书

## GET POST差别
- GET的请求数据在URL，而POST的请求数据在请求体中
- GET请求的数据受到浏览器和服务器的限制，而POST的只受服务器的限制(HTTP协议本身没有做限制)
- POST请求安全性更高
- 语义上的区别：GET POST PUT DELETE

## Session Cookie
- 特点：
  - 都是用于解决HTTP协议的无状态的问题
  - Server：Set-Cookie；Client：Cookie
  - Session、Cookie都有失效时间，过期后会自动删除，减少系统开销
  - Cooike不能跨域，使用Unicode字符必须编码
  - 可用HttpOnly禁止JS获取cookie，可将cookie持久化到硬盘中
  - Session依赖Cookie，cookie的值为session的id，Java里面为JSESSIONID
- Session由应用服务器维护的一个服务器端的存储空间；Cookie是客户端的存储空间，由浏览器维护
