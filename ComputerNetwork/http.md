# 浅析HTTP协议

[TOC]

HTTP即 HyperText Transfer Protocol，web应用的基础，应该是最常用的应用层协议。有如下几个特点：

1. 客户/服务器模式
2. 基于TCP传输
3. 无连接：每次连接只处理一个请求，响应后断开。（keepalive）
4. 无状态：服务器不会保存客户的任何信息。（cookie与session）



# 一、HTTP的报文格式

HTTP报文分为请求报文和响应报文，二者的格式非常相似。都由三部分组成：起始行、包含属性的首部与以及可选的主体部分。



## 请求报文

从客户端发到服务器的是请求报文

从客户端发到服务器的是请求报文

> <方法>< URL ><版本>
> <首部>
>
> <主体>

请求报文的起始行包含方法，请求URL以及版本号。
接下一行的是首部，首部可以有一后者多行。
并不是所有报文都包含主体部分，主体和前面的首部之间有一个空行。如下：

> GEt /special/te.html HTTP/1.0
> HOST:  www.som.com
> Connection: close
> User-agent: Mozilla/5.0
> Accept-language: fr



## 响应报文

服务器发回给客户端的叫响应报文

> <版本>< 状态码 ><状态信息>
> <首部>
>
> <主体>

格式和求情报文差不多，只是首行不同。

> HTTP/1.0 200 OK
> Connection: close
> Date: Tue, 09 Aug 2011 15:44:04 GMT
> Sever: Apache/2.3.3
> Content-Length: 6821
> Content-Type: text/html
>
> (date date date .......)



## 各字段的意义

### 1、方法

- GET： 用于请求访问已经被URI（统一资源标识符）识别的资源，可以通过URL传参给服务器
- POST：用于传输信息给服务器，主要功能与GET方法类似，但一般推荐使用POST方式。
- PUT： 传输文件，报文主体中包含文件内容，保存到对应URI位置。
- HEAD： 获得报文首部，与GET方法类似，只是不返回报文主体，一般用于验证URI是否有效。
- DELETE：删除文件，与PUT方法相反，删除对应URI位置的文件。
- OPTIONS：查询相应URI支持的HTTP方法。



#### get与head的区别

head方法与get方法类似，但是服务器响应中没有主体部分，一般用于：

- 在不获取资源的情况下了解资源情况
- 通过查看响应中的状态码，判断某个对象是否存在
- 通过查看首部，测试资源是否被修改



#### get与post的区别

1.  get用于从服务器获取资源，post重点在于向服务器发送数据
2. get传输数据是通过URL请求，以field（字段）= value的形式，置于URL后，并用"?"连接，多个请求数据间用"&"连接，如http://127.0.0.1/Test/login.action?name=admin&password=admin，这个过程用户是可见的；
   post传输数据通过Http的post机制，将字段与对应值封存在请求实体中发送给服务器，这个过程对用户是不可见的；
3. Get传输的数据量小，因为受URL长度限制，但效率较高；
   Post可以传输大量数据，所以上传文件时只能用Post方式；
4. get是不安全的，因为URL是可见的，可能会泄露私密信息，如密码等；
   post较get安全性较高；
5. get方式只能支持ASCII字符，向服务器传的中文字符可能会乱码。
   post支持标准字符集，可以正确传递中文字符。



### 2、版本

版本主要就是HTTP/1.0和HTTP/1.1。1.1版本中，默认使用的是长链接，而在11.0中，如果要使用长连接，必须要在首部中指定Connection： keep-alive



### 3、状态码

HTTP中的状态码分为五大类

#### 100-199 信息性状态码   (指示信息--表示请求已接收，继续处理)

* 100 Continue：收到了请求的初始部分，请客户端继续

#### 200-299 成功状态码

* 200 OK：请求被正常处理
* 204：请求被受理但没有资源可以返回
* 206：客户端只是请求资源的一部分，服务器只对请求的部分资源执行GET方法，相应报文中通过Content-Range指定范围的资源。

#### 300-399 重定向状态码

* 301 Moved Permanently：永久性重定向
* 302：临时重定向
* 303：与302状态码有相似功能，只是它希望客户端在请求一个URI的时候，能通过GET方法重定向到另一个URI上
* 304：发送附带条件的请求时，条件不满足时返回，与重定向无关
* 307：临时重定向，与302类似，只是强制要求使用POST方法

#### 400-499 客户端错误状态码

* 400 Bad Request：请求报文语法有误，服务器无法识别
* 401 Unauthorized：请求需要认证
* 403 Forbidden：请求的对应资源禁止被访问
* 404 Not Found：服务器无法找到对应资源

#### 500-599 服务器错误状态码

* 500 Internal Server Error：服务器内部错误
* 503 Server Unavailable：服务器正忙
* 505：服务器不支持请求报文的HTTP版本





### 4、首部

首部和方法配合工作，共同决定了客户端和服务器能做什么。首部主要分为以下几类：

#### 通用首部

客户端和服务器都可以使用

- Date：创建报文时间
- Connection：连接的管理
- Cache-Control：缓存的控制
- Transfer-Encoding：报文主体的传输编码方式



#### 请求首部

只在请求报文中有意义的首部

- Host：请求资源所在服务器
- Accept：可处理的媒体类型
- Accept-Charset：可接收的字符集
- Accept-Encoding：可接受的内容编码
- Accept-Language：可接受的自然语言



#### 响应首部

只在响应报文中有意义的首部

- Accept-Ranges：可接受的字节范围
- Location：令客户端重新定向到的URI
- Server：HTTP服务器的安装信息



#### 实体首部

请求报文与响应报文实体部分的首部，提供有关实体及内容的信息

- Allow：资源可支持的HTTP方法
- Content-Type：实体主类的类型
- Content-Encoding：实体主体适用的编码方式
- Content-Language：实体主体的自然语言
- Content-Length：实体主体的的字节数
- Content-Range：实体主体的位置范围，一般用于发出部分请求时使用

以上就是HTTP报文的相关知识点。



# 二、持续连接与非持续连接

由于HTTP协议是基于TCP的，所以客户端与服务器响应的时候必须要考虑连接问题。



## 1、定义

**非持续连接**：每个请求/响应对都是经一个单独的TCP连接发送
**持续连接**：所有的请求及响应经相同的TCP连接发送



## 2、区别

如果使用非持续连接，如果打开一个包含一个HTML文件和10个内联图象对象的网页时，HTTP就要建立11次TCP连接才能把文件从服务机传送到客户机。而如果采用持续连接，HTTP建立一次TCP连接就可把文件从服务机传送到客户机。

- 每次TCP连接必需要建立和断开。客户机和服务机建立一次连接需要执行三向沟通连接法(three-way handshake)，服务机在对象递送之后要断开TCP连接。在建立和断开连接时要占用CPU的资源。如果使用一次连接代替11次连接的话，占用客户机和服务机的CPU时间可大大减少。
- 对每次连接，客户机和服务机都必须分配发送和接收缓存。这就意味着要影响客户机和服务机的存储器资源，这同样要占用CPU的时间。
- 对由大数量对象组成的文件，TCP的低速启动算法(slow start-up algorithm)会限制服务机向客户机传送对象的速度。使用HTTP/1.1之后，大多数对象都可以尽最大的速率传送。



## 3、HTTP/1.0+keep-alive

HTTP/1.1中默认了保持持久连接。但是在1.0的时候并没有。
如果HTTP1.1版本的HTTP请求报文不希望使用长连接，则要在HTTP请求报文首部加上Connection: close。

所以，在1.0中客户端如果想保持长连接，必须在首部中添加：Connection：keep-alive
如果客户端请求中没有这条消息，那么服务器在响应之后就会关闭连接

《HTTP权威指南》提到，HTTP首部的Connection: Keep-alive是HTTP1.0浏览器和服务器的实验性扩展，当前的HTTP1.1 RFC2616文档没有对它做说明，因为它所需要的功能已经默认开启，无须带着它，但是实践中可以发现，浏览器的报文请求都会带上它。
应该忽略所有来自HTTP/1,0设备的Connection首部字段，因为他们可能由比较老的代理服务器转发。



# 三、cookie与session

由于HTTP协议是无状态的，而web站点通常希望能够识别用户，于是就有了cookie。



## 1、cookie

Cookies是服务器在本地机器上存储的小段文本并随每一个请求发送至同一个服务器。IETF RFC 2965 HTTP State Management Mechanism 是通用cookie规范。网络服务器用HTTP头向客户端发送cookies，在客户终端，浏览器解析这些cookies并将它们保存为一个本地文件，它会自动将同一服务器的任何请求缚上这些cookies 。



### 内容

cookie的内容主要包括：名字，值，过期时间，路径和域。路径与域一起构成cookie的作用范围。



### 过期时间

**会话cookie**：若不设置过期时间，则表示这个cookie的生命期为浏览器会话期间，关闭浏览器窗口，cookie就消失。这种生命期为浏览器会话期的cookie被称为会话cookie。

**持久cookie**：若设置了过期时间，浏览器就会把cookie保存到硬盘上，关闭后再次打开浏览器，这些cookie仍然有效直到超过设定的过期时间。



## 2、session

session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。

当程序需要为某个客户端的请求创建一个session的时候，服务器首先检查这个客户端的请求里是否已包含了一个session标识 - 称为session id，如果已包含一个session id则说明以前已经为此客户端创建过session，服务器就按照session id把这个session检索出来使用（如果检索不到，可能会新建一个），如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个session id将被在本次响应中返回给客户端保存。



### sessionid的存储

1. 保存这个session id的方式可以采用cookie，这样在交互过程中浏览器可以自动的按照规则把这个标识返回给服务器。一般这个cookie的名字都是类似于SEEESIONID，而。比如weblogic对于web应用程序生成的cookie，JSESSIONID=ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764，它的名字就是JSESSIONID。 

2. 由于cookie可以被人为的禁止，必须有其他机制以便在cookie被禁止时仍然能够把session id传递回服务器。经常被使用的一种技术叫做URL重写，就是把session id直接附加在URL路径的后面，附加方式也有两种，一种是作为URL路径的附加信息，表现形式为http://...../xxx;jsessionid=ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764 
   另一种是作为查询字符串附加在URL后面，表现形式为http://...../xxx?jsessionid=ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764 
   这两种方式对于用户来说是没有区别的，只是服务器在解析的时候处理的方式不同，采用第一种方式也有利于把session id的信息和正常程序参数区分开来。 

3. 另一种技术叫做表单隐藏字段。就是服务器会自动修改表单，添加一个隐藏字段，以便在表单提交时能够把session id传递回服务器。比如下面的表单： 

   ```html
   <form name="testform" action="/xxx"> 
   <input type="text"> 
   </form> 

   在被传递给客户端之前将被改写成： 

   <form name="testform" action="/xxx"> 
   <input type="hidden" name="jsessionid" value="ByOK3vjFD75aPnrF7C2HmdnV6QZcEbzWoWiBYEnLerjQ99zWpBng!-145788764"> 
   <input type="text"> 
   </form> 
   ```



## 3、cookie与session的区别

### 存取方式不同

Cookie中只能保管ASCII字符串，假如需求存取Unicode字符或者二进制数据，需求先进行编码。Cookie中也不能直接存取Java对象。若要存储略微复杂的信息，运用Cookie是比拟艰难的。
而Session中能够存取任何类型的数据，包括而不限于String、Integer、List、Map等。Session中也能够直接保管Java Bean乃至任何Java类，对象等，运用起来十分便当。能够把Session看做是一个Java容器类。



### 隐私策略不同

Cookie存储在客户端阅读器中，对客户端是可见的，客户端的一些程序可能会窥探、复制以至修正Cookie中的内容。而Session存储在服务器上，对客户端是透明的，不存在敏感信息泄露的风险。
假如选用Cookie，比较好的方法是，敏感的信息如账号密码等尽量不要写到Cookie中。最好是像Google、Baidu那样将Cookie信息加密，提交到服务器后再进行解密，保证Cookie中的信息只要本人能读得懂。而假如选择Session就省事多了，反正是放在服务器上，Session里任何隐私都能够有效的保护。



### 有效时间不同

由于Session依赖于名为JSESSIONID的Cookie，而Cookie JSESSIONID的过期时间默许为–1，只需关闭了阅读器该Session就会失效，因而Session不能完成信息永世有效的效果。运用URL地址重写也不能完成。而且假如设置Session的超时时间过长，服务器累计的Session就会越多，越容易招致内存溢出。



### 服务器压力不同

Session是保管在服务器端的，每个用户都会产生一个Session。假如并发访问的用户十分多，会产生十分多的Session，耗费大量的内存。



### 浏览器支持不同

Cookie是需要客户端浏览器支持的。假如客户端禁用了Cookie，或者不支持Cookie，则会话跟踪会失效。关于WAP上的应用，常规的Cookie就派不上用场了。

假如客户端浏览器不支持Cookie，需要运用Session以及URL地址重写。需要注意的是一切的用到Session程序的URL都要进行URL地址重写，否则Session会话跟踪还会失效。关于WAP应用来说，Session+URL地址重写或许是它唯一的选择。



### 跨域支持不同

Cookie支持跨域名访问，例如将domain属性设置为“.biaodianfu.com”，则以“.biaodianfu.com”为后缀的一切域名均能够访问该Cookie。跨域名Cookie如今被普遍用在网络中，例如Google、Baidu、Sina等。而Session则不会支持跨域名访问。Session仅在他所在的域名内有效。
仅运用Cookie或者仅运用Session可能完成不了理想的效果。这时应该尝试一下同时运用Cookie与Session。Cookie与Session的搭配运用在实践项目中会完成很多意想不到的效果。





# 四、参考地址

<http://justsee.iteye.com/blog/1570652>

<http://www.lai18.com/content/407204.html>

<http://m.blog.csdn.net/article/details?id=51336564>



































