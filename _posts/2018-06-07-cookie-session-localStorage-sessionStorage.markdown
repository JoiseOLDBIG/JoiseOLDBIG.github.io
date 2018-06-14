---
layout: post
title: "浅谈cookie、session、localStorage、sessionStorage"
date: 2018-06-07 20:22:00 +0800
categories: post
---

## cookie和session：

cookie技术是客户端的解决方案，cookie是服务端发给客服端的特殊信息，客服端以文本文件的方式存储在本地，客服端每次向服务器发送请求的时候都会带上这些特殊信息。

### cookie介绍：

#### cookie的由来：

在程序中，会话跟踪是很重要的事情。理论上，一个用户的所有请求操作都属于同一个会话，另一个用户的所有请求操作都属于另一个会话，两者不能混淆。Web应用程序使用HTTP协议进行数据传输，但HTTP协议是无状态的协议，一旦数据交换完毕，客户端和服务端的链接就会关闭，这意味着服务端无法从HTTP协议上跟踪会话。于是出现了cookie机制，弥补HTTP的不足。有两个HTTP头部专门用来设置以及传送cookie的，分别是Set-Cookie和Cookie。Set-Cookie是服务端传给客户端的，用来告诉客户端建立一个Cookie，并且在后续连接服务端的时候，带上这个Cookie；Cookie就是客户端连接服务端的时候，传给服务端的，告诉服务端这属于哪个会话，直到cookie过期之前，每次连接都会传cookie。

#### cookie的工作原理：

由于HTTP是一种无状态的协议，服务器单从网络连接上无从知道客户身份。怎么办呢？就给客户端们颁发一个通行证吧，每人一个，无论谁访问都必须携带自己通行证。这样服务器就能从通行证上确认客户身份了。这就是Cookie的工作原理。

#### cookie设置及发送过程：

客户端发送一个http请求到服务器端 -> 服务器端发送一个http响应到客户端，其中包含Set-Cookie头部 -> 客户端发送一个http请求到服务器端，其中包含Cookie头部 ->服务器端发送一个http响应到客户端

![png1]

#### cookie的不可跨域名性：

cookie在客户端是由游览器来管理的，不能的域名的cookie是不可混淆的，从而保证用户的隐私。

#### cookie的属性：

String name：该Cookie的名称。Cookie一旦创建，名称便不可更改。 

Object value：该Cookie的值。如果值为Unicode字符，需要为字符编码。如果值为二进制数据，则需要使用BASE64编码。

int maxAge：该Cookie失效的时间，单位秒。如果为正数，则该Cookie在>maxAge秒之后失效。如果为负数，该Cookie为临时Cookie，关闭浏览器即失效，浏览器也不会以任何形式保存该Cookie。如果为0，表示删除该Cookie。默认为–1。 

boolean secure：该Cookie是否仅被使用安全协议传输。安全协议。安全协议有HTTPS，SSL等，在网络上传输数据之前先将数据加密。默认为false。 

String path：该Cookie的使用路径。如果设置为“/sessionWeb/”，则只有contextPath为“/sessionWeb”的程序可以访问该Cookie。如果设置为“/”，则本域名下contextPath都可以访问该Cookie。注意最后一个字符必须为“/”。 

String domain：可以访问该Cookie的域名。如果设置为“.google.com”，则所有以“google.com”结尾的域名都可以访问该Cookie。注意第一个字符必须为“.”。 

String comment：该Cookie的用处说明。浏览器显示Cookie信息的时候显示该说明。 

int version：该Cookie使用的版本号。0表示遵循Netscape的Cookie规范，1表示遵循W3C的RFC 2109规范。

#### cookie的有效期：

Cookie的maxAge决定着Cookie的有效期，单位为秒（Second）。Cookie中通过getMaxAge()方法与setMaxAge(int maxAge)方法来读写maxAge属性。 如果maxAge属性为正数，则表示该Cookie会在maxAge秒之后自动失效。浏览器会将maxAge为正数的Cookie持久化，即写到对应的Cookie文件中。无论客户关闭了浏览器还是电脑，只要还在maxAge秒之前，登录网站时该Cookie仍然有效。

如果maxAge为负数，则表示该Cookie仅在本浏览器窗口以及本窗口打开的子窗口内有效，关闭窗口后该Cookie即失效。maxAge为负数的Cookie，为临时性Cookie，不会被持久化，不会被写到Cookie文件中。Cookie信息保存在浏览器内存中，因此关闭浏览器该Cookie就消失了。Cookie默认的maxAge值为–1。

如果maxAge为0，则表示删除该Cookie。Cookie机制没有提供删除Cookie的方法，因此通过设置该Cookie即时失效实现删除Cookie的效果。

### session:

session技术则是服务端的解决方案，是服务端使用的一种记录客户端状态的机制，存储在服务器的内存中，相应的增加了服务器的存储压力。客户端浏览器再次访问时只需要从该Session中查找该客户的状态就可以了。

如果说Cookie机制是通过检查客户身上的“通行证”来确定客户身份的话，那么Session机制就是通过检查服务器上的“客户明细表”来确认客户身份。Session相当于程序在服务器上建立的一份客户档案，客户来访的时候只需要查询客户档案表就可以了。

#### session的有效期:

由于会有越来越多的用户访问服务器，因此Session也会越来越多。为防止内存溢出，服务器会把长时间内没有活跃的Session从内存删除。这个时间就是Session的超时时间。如果超过了超时时间没访问过服务器，Session就自动失效了。

session的超时时间为maxInactiveInterval属性，可以通过对应的getMaxInactiveInterval()获取，通过setMaxInactiveInterval(longinterval)修改。


### cookie和session都是为了验证用户身份，他们的区别在于：

> 1）cookie用于客户端，session用于服务端。
> 
> 2）cookie不是很安全，别人可以分析存放在本地的COOKIE并进行COOKIE欺骗; session相对来说更安全。
> 
> 3）单个cookie的大小限制是4k，cookie数一般不超过20个。


## localStorage和sessionStorage是html5新增的特性，本地存储。

### localStorage:

localStorage生命周期是永久，这意味着除非用户显示在浏览器提供的UI上清除localStorage信息，否则这些信息将永远存在。存放数据大小为一般为5MB,而且它仅在客户端（即浏览器）中保存，不参与和服务器的通信。

### sessionStorage:

sessionStorage仅在当前会话下有效，关闭页面或浏览器后被清除。存放数据大小为一般为5MB,而且它仅在客户端（即浏览器）中保存，不参与和服务器的通信。源生接口可以接受，亦可再次封装来对Object和Array有更好的支持。

### Cookie:

生命期为只在设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭。 存放数据大小为4K左右 。有个数限制（各浏览器不同），一般不能超过20个。与服务器端通信：每次都会携带在HTTP头中，如果使用cookie保存过多数据会带来性能问题。但Cookie需要程序员自己封装，源生的Cookie接口不友好。

### localStorage、seesionStorage、cookie的区别。

> 1）大小限制不同：cookie的大小限制为4k，同个domain下不超过20个；localStorage和sessionStorage的大小限制为5M左右。
> 
> 2）有效期不同：cookie的有效期是通过设置过期时间来决定的，过期时间之前一直有效，即使游览器关闭。localStorage的有效期是永久的，除非手动删除。sessionStorage仅在当前会话下有效，关闭页面后就删除。
> 
> 3）和服务器的通信： cookie在每次与服务器通信的时候都会携带在http头中，cookie保存过多，对通信性能会产生影响；localStorage和sessionStorage都不参与和服务器的通信。
> 
> 4）作用域不同：不同浏览器无法共享localStorage或sessionStorage中的信息。相同浏览器的不同页面间可以共享相同的 localStorage和cookie（页面属于相同域名和端口），但是不同页面或标签页间无法共享sessionStorage的信息。这里需要注意的是，页面及标签页仅指顶级窗口，如果一个标签页包含多个iframe标签且他们属于同源页面，那么他们之间是可以共享sessionStorage的。


[png1]: {{ site.baseurl }}/assets/images/20180607-1.png