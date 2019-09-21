## 一次URL输入域名按下回车到底发生了什么


1.  浏览器的url输入栏发起一个请求，浏览器首先会看自己缓存中有没有对应的ip地址，如果有的话
就直接去访问；如果没有
2.  浏览器会去查看本地的hosts文件，看看有没有和这个域名匹配的ip地址，如果有的话就直接用
hosts文件的ip地址；
3.  如果本地的hosts 文件没有能够找到对应的 ip 地址，浏览器会发出一个 DNS请求到本地DNS服务
器 ，本地DNS服务器会首先查询它的缓存记录，如果缓存中有此条记录，就可以直接返回结果，
此过程是递归的方式进行查询。如果没有，本地DNS服务器还要向DNS根服务器进行查询。
4.  根DNS服务器没有记录具体的域名和IP地址的对应关系，而是告诉本地DNS服务器，你可以到域
服务器上去继续查询，并给出域服务器的地址
5.  域服务器最终会返回给本地的DNS服务器一个具体的ip地址；然后本地的DNS服务器把这个具体
的ip地址返回给浏览器，并且他自己也会把这个url请求对应的ip保存在自己本地，从而加快访问
速度；
6.  浏览器得到域名对应的ip地址后，会加上一个端口号去访问；
7.  一般的服务器网站都会通过反向代理来负载均衡用户们的请求服务，
这里拿Nginx举例，请求来到Nginx后，Nginx会通过负载均衡的算法，把请求分发到服务器集群
上的任意一台服务器，
8.  请求会申请和服务器建立连接这个连接的过程就是我们常说的三次握手；
9.  建立连接后 Java的servlet容器会去接受这个请求，接受后servlet容器会解析
这个请求，与此同时容器会创建一个servlet实例，也就是实例化；同时还会去创建
servletRequest. servletResponse；servletConfig对象
10. 实例化对象之后会马上调用servlet的init方法去初始化这个servlet对象，init方法只会调用一次；
在初始化的时候，容器会给这个servlet实例创建一个ServletConfig对象，这个ServletConfig会从
web应用中的配置文件（web. xml）读取配置信息，得到servlet初始化的时候所需要的参数信
息；
11. 在初始化失败的时候servlet会得到500的错误，也就是服务器内部错误；
如果没有找到初始化参数的话，会报404错误；
12. 用户的请求通常是这个情况：http://hostname: port /contextpath/servletpath
hostname 和 port 是用来与服务器建立 TCP 连接，而后面的 URL 才是用来选
择服务器中那个子容器服务用户的请求。
服务器会通过org. apache. tomcat. util. http. mapper来帮URL找到准确的servlet容
器；我们的springMVC框架中有一个dispatcherServlet去继承HTTPServlet，
得到用户的请求信息，比如GET,POST 还有附带的一些头信息，例如账号密码
然后再进行业务的处理，最终将结果返回给前端进行处理，按页面原路返回给
浏览器；
13. session和cookie;
情况一：如果浏览器支持Cookic的话，Tomcat这类的服务器就会去
解析Cookic得到一个sessionID，得到这个sessionId后，服务器端会创建一个
HttpSession对象，
情况二： 触发 request.  getSession() 方法 的时候，如果Session ID 还没有对
应的 HttpSession 对象那么就创建一个新的HttpSession ，HttpSession 这个
对象会被 org. apache. catalina.  Manager 放到 sessions 容器中保存，
，Manager 类将管理所有 Session 的生命周期，Session 过期将被回收，服务
器关闭，Session 将被序列化到磁盘等。只要这个 HttpSession 对象存在，用户
就可以根据 Session ID 来获取到这个对象，也就达到了状态的保持。我们常
说的session就是这个HttpSession ；
14. 经过上面的步骤，服务器收到了我们的请求，也处理我们的请求，到这一步，它会把它的处
理结果返回，也就是返回一个HTPP响应。
HTTP响应与HTTP请求相似，HTTP响应也由3个部分构成，分别是：
l 　状态行
l 　响应头(Response Header)
l 　响应正文
状态行：
状态行由协议版本. 数字形式的状态代码. 及相应的状态描述，各元素之间以空格分隔。
格式:    HTTP-Version Status-Code Reason-Phrase CRLF
例如:    HTTP/1. 1 200 OK \r\n
-- 协议版本：是用http1. 0还是其他版本
-- 状态描述：状态描述给出了关于状态代码的简短的文字描述。比如状态代码为200时的描述为 ok
-- 状态代码：状态代码由三位数字组成，第一个数字定义了响应的类别，且有五种可能取值。如下
响应正文
包含着我们需要的一些具体信息，比如cookie，html,image，后端返回的请求数据等等。
15. 浏览器得到了请求的结果，就会解析HTML，CSS. JS. 图片等文件了。