<h1 align="center">Interview Http</h1>

[toc]

## http 与 https

http 是超文本传输协议，而 https 可以简单理解为安全的 http 协议。https 通过在 http 协议下添加了一层 ssl 协议对数据进行加密从而保证了安全。https 的作用主要有两点：建立安全的信息传输通道，保证数据传输安全；确认网站的真实性。

### http 与 https 的区别主要如下：

- https 需要到 CA 申请证书，很少免费，因而需要一定的费用
- http 是明文传输，安全性低；而 https 在 http 的基础上通过 ssl 加密，安全性高
- 二者的默认端口不一样，http 使用的默认端口是80；https使用的默认端口是 443

### https 的工作流程

提到 https 的话首先要说到加密算法，加密算法分为两类：对称加密和非对称加密。

- **对称加密：** 加密和解密用的都是相同的秘钥，优点是速度快，缺点是安全性低。常见的对称加密算法有 DES、AES 等等。
- **非对称加密：** 非对称加密有一个秘钥对，分为公钥和私钥。一般来说，私钥自己持有，公钥可以公开给对方，优点是安全性比对称加密高，缺点是数据传输效率比对称加密低。采用公钥加密的信息只有对应的私钥可以解密。常见的非对称加密包括RSA等。

在正式的使用场景中一般都是对称加密和非对称加密结合使用，使用非对称加密完成秘钥的传递，然后使用对称秘钥进行数据加密和解密。二者结合既保证了安全性，又提高了数据传输效率。

### https 的具体流程如下：

1. 客户端（通常是浏览器）先向服务器发出加密通信的请求
   * 支持的协议版本，比如 TLS 1.0版
   * 一个客户端生成的随机数 random1，稍后用于生成"对话密钥"
   * 支持的加密方法，比如 RSA 公钥加密
   * 支持的压缩方法
2. 服务器收到请求,然后响应
   - 确认使用的加密通信协议版本，比如 TLS 1.0版本。如果浏览器与服务器支持的版本不一致，服务器关闭加密通信
   - 一个服务器生成的随机数 random2，稍后用于生成"对话密钥"
   - 确认使用的加密方法，比如 RSA 公钥加密
   - 服务器证书
3. 客户端收到证书之后会首先会进行验证
   - 首先验证证书的安全性
   - 验证通过之后，客户端会生成一个随机数 pre-master secret，然后使用证书中的公钥进行加密，然后传递给服务器端
4. 服务器收到使用公钥加密的内容，在服务器端使用私钥解密之后获得随机数 pre-master secret，然后根据 radom1、radom2、pre-master secret 通过一定的算法得出一个对称加密的秘钥，作为后面交互过程中使用对称秘钥。同时客户端也会使用 radom1、radom2、pre-master secret，和同样的算法生成对称秘钥。
5. 然后再后续的交互中就使用上一步生成的对称秘钥对传输的内容进行加密和解密。

### http头部的字段以及含义

- **Accept :** 浏览器（或者其他基于HTTP的客户端程序）可以接收的内容类型（Content-types）,例如 `Accept: text/plain`
- **Accept-Charset：**浏览器能识别的字符集，例如 `Accept-Charset: utf-8`
- **Accept-Encoding：**浏览器可以处理的编码方式，注意这里的编码方式有别于字符集，这里的编码方式通常指gzip,deflate等。例如 `Accept-Encoding: gzip, deflate`
- **Accept-Language：**浏览器接收的语言，其实也就是用户在什么语言地区，例如简体中文的就是 `Accept-Language: zh-CN`
- **Authorization：**在HTTP中，服务器可以对一些资源进行认证保护，如果你要访问这些资源，就要提供用户名和密码，这个用户名和密码就是在Authorization头中附带的，格式是“username:password”字符串的base64编码
- **Cache-Control：**这个指令在request和response中都有，用来指示缓存系统（服务器上的，或者浏览器上的）应该怎样处理缓存，因为这个头域比较重要，特别是希望使用缓　存改善性能的时候
- **Connection：**告诉服务器这个user agent（通常就是浏览器）想要使用怎样的连接方式。值有keep-alive和close。http1.1默认是keep-alive。keep-alive就是浏览器和服务器　的通信连接会被持续保存，不会马上关闭，而close就会在response后马上关闭。但这里要注意一点，我们说HTTP是无状态的，跟这个是否keep-alive没有关系，不要认为keep-alive是对HTTP无状态的特性的改进。
- **Cookie：**浏览器向服务器发送请求时发送cookie，或者服务器向浏览器附加cookie，就是将cookie附近在这里的。例如：`Cookie:user=admin`
- **Content-Length：**一个请求的请求体的内存长度，单位为字节(byte)。请求体是指在HTTP头结束后，两个CR-LF字符组之后的内容，常见的有POST提交的表单数据，这个Content-Length并不包含请求行和HTTP头的数据长度。
- **Content-MD5：**使用base64进行了编码的请求体的MD5校验和。例如：`Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==`
- **Content-Type：**请求体中的内容的mime类型。通常只会用在POST和PUT方法的请求中。例如：`Content-Type: application/x-www-form-urlencoded`
- **Date：**发送请求时的GMT时间。例如：`Date: Tue, 15 Nov 1994 08:12:31 GMT`
- **From：**发送这个请求的用户的email地址。例如：`From: user@example.com`
- **Host：**被服务器的域名或IP地址，如果不是通用端口，还包含该端口号，例如：`Host: www.some.com:182`
- **Proxy-Authorization：**连接到某个代理时使用的身份认证信息，跟Authorization头差不多。例如：`Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==`
- **User-Agent：**通常就是用户的浏览器相关信息。例如：`User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:12.0) Gecko/20100101 Firefox/12.0`
- **Warning：**记录一些警告信息。

## TCP、UDP

* [TCP和UDP详解](../Computer/TCP/TCP.md)
* [TCP23个疑难问题详解](../Computer/TCP/TCP23Problem.md)