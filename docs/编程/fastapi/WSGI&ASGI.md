# WSGI&ASGI

参考：https://www.jianshu.com/p/65807220b44a

> 此文档用于解读理解`WSGI`和`ASGI`两个概念，目标解决一下几个问题：

- 什么是`WSGI`
- 什么是`ASGI`
- `WSGI`和`ASGI`的区别在哪

### 什么是 WSGI

先说一下`CGI`，（通用网关接口， Common Gateway Interface/CGI），定义客户端与Web服务器的交流方式的一个程序。例如正常情况下客户端发来一个请求，根据`HTTP`协议Web服务器将请求内容解析出来，进过计算后，再将加us安出来的内容封装好，例如服务器返回一个`HTML`页面，并且根据`HTTP`协议构建返回内容的响应格式。涉及到`TCP`连接、`HTTP`原始请求和相应格式的这些，都由一个软件来完成，这时，以上的工作需要一个程序来完成，而这个程序便是`CGI`。

那什么是`WSGI`呢？[维基](https://link.jianshu.com/?t=https://zh.wikipedia.org/wiki/Web服务器网关接口)上的解释为，**Web服务器网关接口(Python Web Server Gateway Interface，WSGI)**，是为`Python`语言定义的Web服务器和Web应用程序或框架之间的一种简单而通用的接口。从语义上理解，貌似`WSGI`就是`Python`为了解决**Web服务器端与客户端**之间的通信问题而产生的，并且`WSGI`是基于现存的`CGI`标准而设计的，同样是一种程序（或者`Web`组件的接口规范？）。

[WSGI](https://link.jianshu.com/?t=https://zh.wikipedia.org/wiki/Web服务器网关接口)区分为两部分：一种为“服务器”或“网关”，另一种为“应用程序”或“应用框架”。
所谓的`WSGI`中间件同时实现了`API`的两方，即在`WSGI`服务器和`WSGI`应用之间起调解作用：从`WSGI`服务器的角度来说，中间件扮演应用程序，而从应用程序的角度来说，中间件扮演服务器。中间件具有的功能：

- 重写环境变量后，根据目标URL，将请求消息路由到不同的应用对象。
- 允许在一个进程中同时运行多个应用程序或应用框架
- 负载均衡和远程处理，通过在网络上转发请求和相应消息。
- 进行内容后处理，例如应用`XSLT`样式表。（以上 from 维基）

看了这么多，总结一下，其实可以说`WSGI`就是基于`Python`的以`CGI`为标准做一些扩展。

### 什么是[ASGI](https://link.jianshu.com/?t=https://blog.ernest.me/post/asgi-draft-spec-zh)

异步网关协议接口，一个介于网络协议服务和`Python`应用之间的标准接口，能够处理多种通用的协议类型，包括`HTTP`，`HTTP2`和`WebSocket`。
然而目前的常用的`WSGI`主要是针对`HTTP`风格的请求响应模型做的设计，并且越来越多的不遵循这种模式的协议逐渐成为`Web`变成的标准之一，例如`WebSocket`。
`ASGI`尝试保持在一个简单的应用接口的前提下，提供允许数据能够在任意的时候、被任意应用进程发送和接受的抽象。并且同样描述了一个新的，兼容`HTTP`请求响应以及`WebSocket`数据帧的序列格式。允许这些协议能通过网络或本地`socket`进行传输，以及让不同的协议被分配到不同的进程中。

### WSGI和ASGI的区别在哪

以上，`WSGI`是基于`HTTP`协议模式的，不支持`WebSocket`，而`ASGI`的诞生则是为了解决`Python`常用的`WSGI`不支持当前`Web`开发中的一些新的协议标准。同时，`ASGI`对于`WSGI`原有的模式的支持和`WebSocket`的扩展，即`ASGI`是`WSGI`的扩展。