####0x01 (Origin:P3)XSS跨站脚本攻击的发生
- 理解代码<script>eval(location.hash.substr(1);</script>
例如：运用在页面 http：//www.baidu.com#alert(1)
 * substr(0):  结果弹窗 #alert(1)
 * substr(1):  结果弹窗 alert(1)
 * substr(2):  结果弹窗 lert(1)


####0x02  (Origin:P5)  同源策略  表1-1 是否同域情况判断
- 同源策略规定： 不同域的客户端脚本在没明确授权的情况下，不能读写对方的资源。
- 同域要求两站点同协议、同域名、同端口  
http  ://  www.foo.com  &nbsp;&nbsp;:80  &nbsp;&nbsp;  /a/  
协议&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;域名&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;端口&nbsp;&nbsp;目录  
- 两个站点同域就是指它们同源  

####0x03  （origin:P6）客户端脚本
- 客户端脚本主要指JavaScript、ActionScript，都遵循ECMAScript脚本标准

####0x04  (origin:P6 P25) 理解AJAX 异步传输  AJAX风险
- 全称为Asynchronous Javascript And XML，即异步的Javascript和XML，  是前端黑客攻击中必用的技术模式
  +  这里有三个点： 异步、Javascript、 XML
  +  AJAX严格遵守同源策略
- 常用场景：对网页的局部数据进行更新时，不需要刷新整个网页，以节省带宽资源

####0x05  (origin: P7) 资源
- 同源策略里的资源是指web客户端的资源，一般来说，资源包括：http消息头、整个DOM树、浏览器存储（cookies、flash cookies、localstorage等）

####0x06 (origin: P7) DOM
- 全称Document Object Model，即文档对象模型，就是浏览器将HTML/XML这样的文档抽象成一个树形结构
- 树上的每个节点都代表HTML/XML中的标签、标签属性或标签内容等

####0x07 (origin: P7-9)  信任与信任关系
- 默认情况下，不同源则不信任  
下面两个“信任”场景：
  * 场景一：  
一个web服务器上有两个网站A、B，且在同一个文件系统里。黑客的目标是A，但直接入侵A太难，而入侵B成功了。如果AB没进行有效的文件权限配置，攻克A就轻而易举了。
     + 缺陷：A与B之间过于信任，未做很好的分离。
     + 启示：安全类似木桶原理，短的那块板决定了木桶实际能装多少水。一个web服务器，如果其上的网站没做好权限分离，没控制好信任关系，则整体安全性就由安全性最差的那个网站决定。
  * 场景二：   
很多网站都嵌入了第三方的访问统计脚本，嵌入的方式是使用`<script>`标签引用，这时就等于建立了信任关系，如果第三方的统计脚本被黑客挂马，那么这些网站也都会被危及。  
这种信任关系很普遍：服务器与服务器、网站与网站、web服务的不同子域、web层面与浏览器第三方插件、web层面与浏览器特殊API、浏览器特殊API与本地文件系统、嵌入的flash与当前DOM树、不同协议之间等等。一个安全性非常好的网站有可能会因为建立了不可靠的信任关系，导致网站被黑。


####0x08  (origin: P10)  CSRF跨站请求伪造
- CSRF借用目标用户权限做一些借刀杀人的事情
  + CSRF：借用
  + XSS：盗取

####0x09  （origin:P13）导致web安全事件的角色
- W3C
- 浏览器厂商
- web厂商
- 攻击者（黑客）
- 被攻击者（或用户）

####0x0A  (origin:P15-18)  HTTP协议 请求头 响应头
- URL的请求协议几乎都是HTTP，它是一种无状态的请求相应，即每次的请求响应之后，连接会立即断开或者延时断开（保持一定的连接有效期），断开后，下一次请求再重新建立。
请求与相应一般都分为头部与体部（它们之间以空行分隔）。对于请求体来说，一般出现在POST方法中。响应体就是在浏览器中看到的内容，比如，HTML/JSON/JavaScript/XML等。
头部的每一行都有自己的含义，key与value之间以冒号分隔。
- 请求头中的几个关键点如下：
  - GET  http://www.foo.com/http/1.1  
这一行必不可少，常见的请求方法有GET/POST，最后的“HTTP/1.1”表示1.1版本的HTTP协议。
  - Host：www.foo.com  
这一行也必不可少，表示请求的主机是什么。
  - User-Agent：Mozilla/5.0(Windows NT 6.1)  AppleWebKit/535.19(KHTML, like Gecko)  Chrome/18.0.1025.3  Safari/535.19  
User-Agent很重要，用于表明身份（我是谁）。从这里可以看到操作系统、浏览器、浏览器内核及对应的版本号等信息。
  - Referer：http://www.baidu.com/  
Referer很重要，表明从哪里来，比如从http://www.baidu.com/页面点击过来。
  - Cookies：SESSIONID=58DAFAB45FDAASD45SDF45ASG453SD:FG=1  
HTTP是无状态的，那么每次在连接时，服务端如何知道你是上一次的那个？这里通过Cookies进行会话跟踪，第一次响应时设置的Cookies在随后的每次请求中都会发送出去。Cookies还可以包括登录认证后的身份信息。
- 响应头中的几个关键点如下：
  - HTTP/1.1 200 OK  
这一行肯定有 200是状态码 OK是状态描述
  - Server：Apache/2.2.8 (Win32) PHP/5.2.6  
Web容器、操作系统、服务端语言及对应的版本。
  - X-Powered-By: PHP/5.2.6  
服务端语言的信息
  - Content-Length：3635  
响应体的长度
  - Content-Type：text/html；charset=gbk  
响应资源的类型与字符集
- Set-Cookies  
每个Set-Cookies都设置一个Cookie（key=value这样），随后是如下内容：
     - expires：过期时间，如果过期时间的过去，那就表明这个Cookie要被删
     - path：相对路径，只有这个路径下的资源可以访问这个Cookie
     - domain：域名，有权限设置为更高一级的域名
     - HttpOnly：标志（默认无，如果有的话，表明Cookie存在于HTTP层面，不能被客户端脚本读取）
     - Secure：标志（默认无，如果有的话，表明Cookie仅通过HTTPS协议进行安全传输

####0x0B  (origin:P20-21)  DOM树
- 一段HTML代码

```
<html>
<head>
<title>HTML</title>
<metahttp-equiv=”Content-Type” content=”text/html”;  charset=”uft-8”
<style>
    Body{font-size:14px;}
</style>
<script>
    a = 1; //这里是脚本
</script>
</head>
<body>
<div>
    <h1>这些都是HTML</h1> <br />
    <img src=http://www.foo.com/logo.jpg title=”这里是图片引用” />
</div>
</body>
</html>
```
- 用DOM树形结构描述

```
<html>
 -<head>
   -<title>
     -HTML
 -<mate>
     -@http-equiv
       -Content-Type
     -@content
       -text/html
     -@charset
       -uft-8
 -<style>
    -body{font-size:14px;}
  -<script>
    -a=1;//这里是脚本
 -<body>
   -<div>
     -<h1>
       -这些都是HTML
     -<br />
     -<img>
       -@src
         -http://www.foo.com/logo.jpg
       -@title
       -这里是图片引用
```

####0x0C  (origin: P21)  隐私数据可能存储的位置
- HTML内容中
- 浏览器本地存储中，如Cookies等
- URL地址中

####0x0D (origin: p25)  获取URL地址中的数据
 - 从window.location或location处可以获取URL地址中的数据
   + 如 `<script> document.write(location.href);</script>`
   + 用于获取当前页面的地址（URL）
   
####
