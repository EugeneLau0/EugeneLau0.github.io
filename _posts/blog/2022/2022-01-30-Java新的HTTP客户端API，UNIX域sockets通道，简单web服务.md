---
title: 'Java新的HTTP客户端API，UNIX域sockets通道，简单web服务'
categories:
  - translate
tags:
  - java
---

这是一篇在甲骨文官网上，两个架构师关于Java 18中网络新特性的一段对话，个人觉得挺有启发，故翻译出来。

<!--more-->

### 下面是两个架构师关于Java 18中网络新特性的一段对话

> 网络能力是从一开始就被构建到Java核心库。对于开发工程师而言，一个事实是网络模块在开发平台中默认被创建是很受欢迎的，因为企业或机构没有必要去依赖第三方库或者安排资源去开发他们自己的网络服务以支持公用网络协议。
> 
> 当然，在90年代，并不是每一个应用都需要接入网络，而如今支持联网对所有平台而言是一个刚需。确实，如今大多数不能使用HTTP协议的商业或消费型应用将不复存在。
> 
> 从另一方面说，Java非常广泛地使用http Client API也不见得是一个很有意义的提升，直到这个新的API携带到JDK9并且作为JDK11的标准。
> 
> David Delabassée是oracle中Java平台的一名开发者，最近与oracle 的架构师Daniel Fuchs、Michael McMahon就Java平台的网络更新展开一些讨论，下面是本次讨论内容的一些要点。

### Delabassée：为什么要引入新的Java Client API呢？
*McMahon*：起源，25年的Java HTTP Client API已经太旧了，并且有一些局限。很难使用，也很难维护。部分API支持的协议不再使用，比如说Gopher，它增加了复杂性。
另外，旧的client是阻塞式，并且在整个请求中都维护一个线程，无论整个过程花费多久。额外的，旧的client是基于HTTP/1.1协议，并需要去逐步地支持更新的HTTP/2协议。

*Fuchs*：新版HTTP client API最先在JDK9中支持，并且作为一个标准及在JDK11中永久支持。它同时支持HTTP/1.1和HTTP/2，也支持WebSockets，同时提升了安全性。更多的，新的client更现代化、更易用，比如说，它使用了流行的构建者模式。

### Delabassée：这个新的client是如何运行的呢？
*Fuchs*：为了发送一个请求，你需要先创建一个HTTP client；为此有一个创建者。使用创建者，你创建了一个client实例并配置它的状态。比如说，你可以指定用哪一个HTTP协议和哪一种重定向方式。你也设置代理选择器，为TLS设置必要的上下文等等。
一旦创建，你的HTTP client就可以快速发送多个请求。请求可以被发送耦次，并且你可以选择是同步还是异步的方式。

### Delabassée：在HTTP client API之下，UNIX-domain socket通道被加入到Java 16。对此你们怎么看？
*McMahon*：UNIX-domain socket就像TCP/IP sockets，这些sockets仅仅使用内部调用（IPC），它们通过系统文件路径名称寻址，而不是通过IP地址加端口号。
UNIX域sockets通道在[JEP 380](https://openjdk.java.net/jeps/380)第一次出现，对本地 调用或者容器间调用有几个好处。第一个是性能：通过剪切TCP/IP堆，可以提升吞吐量及更低的CPU消耗。也因为是本地调用，不需要连接网络，可以提升安全。
使用UNIX域sockets，你也可以使用通过使用文件控制系统和用户认证来提升安全。在docker中测试时，我们发现在容器间通过UNIX域sockets建立通信更容易。
为了达成这些，JEP 380额外增加了下面这些API元素：
* 新的socket地址类，`java.net.UnixDomainSocketAddress`
* 一个UNIX枚举常量值，`java.net.StandardProtocolFamily`
* 新开发的工厂方法，`SocketChannel`和`ServerSocketChannel`
* 更新了`SocketChannel`和`ServerSocketChannel`的规范，允许你定义UNIX域的行为
还有一点需要声明，尽管它名字叫UNIX域，但它也能在Windows上使用。

### Delabassée：通常的，在一个网络连接中，安全取决于网络等级，比如说通过使用防火墙和反向代理。那么UNIX域sockets是如何保障安全的呢？
*McMahon*：一个安全等级是原来的Java平台安全模式。它仅是一个简单的许可，启用或关闭取决于安全管理是否运行。
UNIX域sockets的主要方式是通过操作系统，它的用户和用户组，以及它的文件权限，因此你可以通过用户ID和用户组ID来限制文件系统节点。
另外一个有用的特性是测试用户认证，它可以运行在UNIX系统上，但是不能在windows系统上。如果一个client socket通道连接到一个服务端socket通道，另外一边也可反向请求用户身份。

### Delabassée：现在有一个新的web服务和API 目标请求在Java18中，你能给我们做一个简单的概述吗？
*McMahon*：[JEP 408](https://openjdk.java.net/jeps/408)发送一个静态内容到web服务端。JEP 408包括一个自动生成的API和服务及组件的自定义。
正如它名字一样，这个简单的web服务设计是相当的迷你。它的目的是以开发目的，也为了测试和教学使用，但是它肯定不是所有的特性你都应该在生产上使用。比如说，它没有提供安全特性。
这个简单的web服务是基于`com.sun.net.httpserver`包来实现，这个服务包在2006年就已经被包含在JDK中了。这个包是官方支持的，并且它的拓展API以实现简易的服务创建及提高请求并发处理。这个简单的web服务可以被用在命令行工具`jwebserver`上或者通过它的API实现程序自动化。
在重申下，该简易web服务预期是学习、开发测试和调试为目的。

### Delabassée：关于这个新的网络能力，有任何的终极思想吗？
*Fuchs*：我们讨论这个HTTP客户端API，UNIX域socket通道以及这个简单的web服务。这些特性是显而易见的，因为开发者们可以直接使用它们。在所有这些特性中，Java团队持续在它自身的平台上工作，比如一些特性不是那么显而易见，尽管如此还是在往前发展、进步。
举例来说，缓慢但确定的Project Loom正向我们走来，并且我们也期待我们的网络能力可以在虚拟线程中有更好的表现。这是JEP 353的动机，它重新实现了socket API的缺陷，在JEP 373中，它重新实现了DatagramSocket API的缺陷。

[来源](https://blogs.oracle.com/javamagazine/post/java-httpclient-unixdomainsockets-simplewebserver)