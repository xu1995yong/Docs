### 

### HTTPS介绍
HTTPS是以安全为目标的HTTP通道，简单讲是HTTP的安全版，即



HTTPS协议的主要作用可以分为两种：一种是建立一个信息安全通道，来保证数据传输的安全；另一种就是确认网站的真实性。
HTTPS协议使用的是443端口。

### 二、HTTPS的工作原理



![wKioL1PvZ8uDsC6RAAHgZCg1uV0854.jpg](https://s3.51cto.com/wyfs02/M02/46/39/wKioL1PvZ8uDsC6RAAHgZCg1uV0854.jpg)



以共享密钥方式加密时必须将密钥也发给对方。安全转交密钥就需要使用非对称加密算法对密钥进行加密。



为什么不直接用非对称算法





## Cookie

### Cookie的介绍

由于HTTP协议是无状态的，服务器端无法知道每次用户请求的区别。这时就需要使用Cookie技术保存用户的状态信息。

Cookie是一小段K-V键值对形式的字符串。当浏览器第一次访问Web服务器时，Web服务器会创建Cookie并返回给浏览器，之后浏览器将Cookie存储在用户本地。这样当浏览器再次向该Web服务器发送请求时就会附带这些Cookie，Web服务器就可以使用这些Cookie来识别不同的用户。

### Cookie常用属性

```
name ：一个唯一确定cookie的名称，不区分大小写，cookie的名字必须是经过URL编码的。
value：存储在cookie中的字符串值，必须经过被URL编码
domain：即cookie所属的域。Cookie是不可跨域名的，一个域名下的cookie不会被提交到其他域名。
expires：即cookie的有效期。
max-age：即cookie的最大生存时间，以秒为单位。
```

### Cookie的缺陷

1. 每个域的cookie总数是有限的，不同浏览器之间各有不同。
2. Cookie会被附加在每个HTTP请求中，所以无形中增加了流量。
3. 由于在HTTP请求中的Cookie是明文传递的，所以安全性成问题，除非用HTTPS。
4. Cookie的大小限制在4KB左右，对于复杂的存储需求来说是不够用的。

## Session

### Session的介绍

由于Cookie存在各种缺陷，服务器端采用Session技术来保存用户的状态信息，Session指的是服务器端为客户端所开辟的存储空间。

当浏览器第一次访问Web服务器时，Web服务器会在内部创建一个Session保存用户的状态信息，并将该Session的Id保存在Cookie中返回给浏览器。这样当浏览器再次向该Web服务器发送请求时就会附带保存了SessionId的cookie，Web服务器就可以根据SessionId获取保存在服务器内的浏览器对应的Session。

### 分布式集群中的Session共享

