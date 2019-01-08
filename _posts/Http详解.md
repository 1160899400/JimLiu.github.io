# Http介绍
&emsp;&emsp;简单的来说，Http协议是应用层的一个超文本传输协议。通俗点来讲，就是在网络的应用层中，所有传输的报文必须严格遵守这个协议规定的格式，关于什么是网络的应用层，前面也都讲过不再赘述。Http协议也有很多版本（如http 1.0，http 1.1，http 2.x等），不同版本的Http在报文格式上有所差别，在意义和作用上差别更明显。  

### Http报文基本格式:
&emsp;&emsp;Http报文分两种：请求报文（Request）和响应报文（Response），请求报文就是client发送给server的报文，响应报文就是server返回给client的报文，它们的报文结构大致相同。  

&emsp;&emsp;Http报文的数据结构包括3部分：**行（Line）**，**头（Header）**，**主体（Body）**，根据报文的种类又分为请求行、响应行、请求头......balabala的。下面逐个对这些做介绍。  

&emsp;&emsp;先说下Request的各部分：  

#####  请求行
&emsp;&emsp;请求行