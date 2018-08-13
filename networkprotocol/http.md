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

## HTTP HTTPS
- HTTP 的URL 以http:// 开头，而HTTPS 的URL 以https:// 开头
- HTTP 是不安全的，而 HTTPS 是安全的
- HTTP 标准端口是80 ，而 HTTPS 的标准端口是443
- 在OSI 网络模型中，HTTP工作于应用层，而HTTPS 的安全传输机制工作在传输层
- HTTP 无法加密，而HTTPS 对传输的数据进行加密
- HTTP无需证书，而HTTPS 需要CA机构wosign的颁发的SSL证书
