# Http介绍
&emsp;&emsp;简单的来说，Http协议是应用层的一个超文本传输协议。通俗点来讲，就是在网络的应用层中，所有传输的报文必须严格遵守这个协议规定的格式，关于什么是网络的应用层，前面也都讲过不再赘述。Http协议也有很多版本（如http 1.0，http 1.1，http 2.x等），不同版本的Http在报文格式上有所差别，在意义和作用上差别更明显。  

### Http报文基本格式:
&emsp;&emsp;Http报文分两种：请求报文（Request）和响应报文（Response），请求报文就是client发送给server的报文，响应报文就是server返回给client的报文，它们的报文结构大致相同。  

&emsp;&emsp;Http报文的数据结构包括3部分：**行（Line）**，**头（Header）**，**主体（Body）**，根据报文的种类又分为请求行、响应行、请求头......balabala的。下面逐个对这些做介绍。  

&emsp;&emsp;先说下Request的各部分：  

#####  请求行
&emsp;&emsp;请求行的格式为：**${method} ${path-to-resource} HTTP/ version-number（CRLF）**，如下是一个标准的HTTP请求行：  

&emsp;&emsp;`POST http://www.baidu.com/ HTTP/1.1`  

&emsp;&emsp;method表示请求方法，包括get，post，head等等。  

&emsp;&emsp;path-to-resource代表请求的资源在服务器的url。  

&emsp;&emsp;version-number代表协议版本。  

&emsp;&emsp;可以看到，请求行包含了一个请求报文的基本信息。其中请求方法get/post对应的请求报文的格式又有点区别，后面再说。请求行末尾是\r\n回车换行，这里为了方便用（CRLF）缩写表示.  

#####  请求头
&emsp;&emsp;请求头在http中的作用是为了**定义报文的一些属性**的，它的格式类似于HashMap的遍历输出，以键值对按行输出（即每个键值对末尾以CRLF结尾），中间以冒号隔开，多值以逗号隔开。例如下：  

&emsp;&emsp;Accept:text/plain,text/html  

&emsp;&emsp;Referer:http://www.baidu.com/  

&emsp;&emsp;Accept-Language:zh-cn,en  

&emsp;&emsp;Accept-Encoding:compress,gzip  

&emsp;&emsp;User-Agent:Mozilla/5.0 (Linux; X11)  

&emsp;&emsp;Host:www.baidu.com  

&emsp;&emsp;Connection: Keep-Alive  

&emsp;&emsp;Cookie:	Cookie: $Version=1; Skin=new;  

&emsp;&emsp;所有的请求头属性的作用和形式如下：  

![Http请求头对照表](https://urt1rsliu.github.io//images/post/Basic/http所有请求头.PNG)

&emsp;&emsp;其中有不少重要的请求头在我们实际使用中非常常见，比如Range这个header，当我们需要向服务器请求一个500KB的文件时，由于文件太大，我们需要分多部分发送多个请求，而不是一次性请求完，这时我们可以发送多个request，给每个request header中的range设置不同值，来实现请求同一个文件的不同内容。或者说我至想要某个文件的部分数据而不是整个文件，同样也可以用range来实现。  

&emsp;&emsp;还有Cache-Control通常用来设置Request的缓存，Cache-Control后面通常用max-age表示当前request的剩余过期时间。这里需要对request的缓存作一个简单的介绍：考虑到效率问题，Http协议在设置了缓存的情况下，会把每次请求都缓存下来，当下一次request与当前request相同时，则会复用上次request后返回的结果，而不会重新发送。当然每个request缓存都是有"保质期"的，max-age就是当前时间下request缓存的保质期，如果max-age为0，就会重新发送请求。  

&emsp;&emsp;还有Content-Type，这里Content-Type值不同，会导致Request Body的格式不同，所以通过Http报文发送不同类型的数据时，应该选择不同的Content-Type。Content-Type值很多，常见的Content-Type有如下几种：   

1. application/s-form-urlencoded，Http默认的Content-Type，会将body里的内容编码为键值对。
2. 