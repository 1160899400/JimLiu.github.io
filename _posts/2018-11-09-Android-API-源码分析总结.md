---
layout: post
title:  "Android API 源码分析总结"
date:   2018-11-09
tags:
- Source Code Learning
- Android
- API
---

### Handler
&emsp;&emsp;总结起来就是每个Handler对象都会有一个Looper和一个MessageQueue成员对象，Handler在初始化时会通过Looper.MyLooper()获取到当前线程对应的Looper对象，如果当前是UI线程，则Looper已经初始化，可以正常取到；但是如果当前线程不是UI线程，则需要调用Looper.Prepare()初始化Looper，这样才能正常初始化Handler。  

&emsp;&emsp;再看Looper的内部，ThreadLocal<Looper> sThreadLocal成员维护着所有线程与对应的Looper（ThreadLocal有一个ThreadLocalMap内部类，通过这个类获取，具体见源码），通过调用该对象的get方法就可以获取当前线程的Looper对象，Looper内部还有一个MessageQueue成员。当new Handler时，会在获得Looper对象以后将Looper.MessageQueue赋给Handler.MessageQueue。调用Handler.sendMessage()方法就是将Message放入该队列，最后在当前线程调用Looper.loop()即可循环读取Handler的MessageQueue来实现消息的获取。

&emsp;&emsp;再看发送Message时Message的初始化，我们使用时一般不是通过new message来创建一个message，而是调用`Message.obtain()`方法。而且obtain内部通过对象锁实现同步，obtain方法有多种重载形式，具体实现也可以查看源码。