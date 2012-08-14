---
layout: post
title: "减少xampp内存占用"
description: ""
category: Server
tags: [linux , xampp , apache , moodle]
---
{% include JB/setup %}


最近有个改写moodle系统插件的项目，所以我又重操起php的老行当。moodle是一个非常不错的考试系统，多种题型，多种计分方式，唯一的缺点就是文档太少，非常的缺...开发人员列表里面也看到了好几个中国人的名字。

下载，放到htdocs，`sudo lampp start`，结果发现他竟然要求php5.3以上。我的php还是5.1的老版本，那没法，唯有下载个最新的xampp吧。一切还算顺利，除了xampp好像怪怪的。因为我改了配置文件后，再重启，他竟然好像没有反应，这个到现在还是个不解之谜。

在开发的过程中，我还发现电脑卡了很多，`ps -A|grep httpd`一看，哇的一刷。内存给我用了几百兆了，虽然我有4G也不是这样浪费的嘛，我只是开发而已，没有必要开这么线程。下面记录一下给xampp减少线程的方法。

<!--more-->
# 减少xampp线程数，享受低碳生活


## 先看看Apache现在是在什么模式下工作

{% codeblock lang:bash %}
$ /opt/lampp/bin/apachectl -lCompiled in modules:
core.c
prefork.c
http_core.c
mod_so.c
{% endcodeblock %}

看到了吧，原来xampp的Apache默认是在*prefork*模式下跑的，那我们得相应修改prefox模式的进程数。


## 修改Apache配置

### 修改httpd.conf
{% codeblock lang:bash %}
$ vim etc/httpd.conf
{% endcodeblock %}
找到`#Include etc/extra/httpd-mpm.conf`去掉注释，保存。

### 修改httpd-mpm.conf
{% codeblock lang:bash %}
$ vim etc/extra/httpd-mpm.conf 
{% endcodeblock %}
找到
{% codeblock %}
<IfModule mpm_prefork_module>
    StartServers          2
    MinSpareServers       2
    MaxSpareServers      5
    MaxClients          5
    MaxRequestsPerChild   0
 </IfModule>
{% endcodeblock %}
修改其中的参数，下面是这几个参数的详细解释：

Apache一开始会新建*StartServers*个子进程后，为了满足*MinSpareServers*设置的需要，创建一个进程，等待一秒钟，继续创建两个，再等待一秒钟，继续创建四个……如此按指数级增加创建的进程数，最多达到每秒32个，直到满足*MinSpareServers*设置的值为止。


这种模式可以不必在请求到来时再产生新的进程，从而减小了系统开销以增加性能。*MaxSpareServers*设置了最大的空闲进程数，如果空闲进程数大于这个值，Apache会自动kill掉一些多余进程。这个值不要设得过大，但如果设的值比MinSpareServers小，Apache会自动把其调整为 MinSpareServers+1。


如果站点负载较大，可考虑同时加大*MinSpareServers*和*MaxSpareServers*。 *MaxRequestsPerChild*设置的是每个子进程可处理的请求数。每个子进程在处理了*MaxRequestsPerChild*个请求后将自动销毁。0意味着无限，即子进程永不销毁。虽然缺省设为0可以使每个子进程处理更多的请求，但如果设成非零值也有两点重要的好处：

> 1. 可防止意外的内存泄漏。
> 2. 在服务器负载下降的时侯会自动减少子进程数。

因此，可根据服务器的负载来调整这个值。*MaxClients*是这些指令中最为重要的一个，设定的是 Apache可以同时处理的请求，是对Apache性能影响最大的参数。其缺省值150是远远不够的，如果请求总数已达到这个值（可通过`ps -ef|grep http|wc -l`来确认），那么后面的请求就要排队，直到某个已处理请求完毕。这就是系统资源还剩下很多而HTTP访问却很慢的主要原因。虽然理论上这个值越大，可以处理的请求就越多，但Apache默认的限制不能大于256。
