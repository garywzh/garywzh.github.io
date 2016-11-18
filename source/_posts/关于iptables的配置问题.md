---
title: 关于iptables配置的问题
date: 2016-03-17 11:01:10
tags: Linux
---

我们一般通过给iptables配置一些rule以提升服务器的安全性。常见做法就是在入口上只打开需要的端口如80、443、22等等，命令如下：

`sudo iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT`

然后默认drop非法数据包：

`sudo iptables -P INPUT DROP`

但是一般服务器都需要联网功能，比如下载、更新文件（apt-get、git），转发数据（shadowsocks），这个时候服务器主动建立连接请求数据时返回的数据包会被drop。举例说明：

前提：

通过shadowsocks访问Google，服务器iptables打开了8888端口给shadowsocks

流程：

客户端与vps 8888端口建立连接并发送访问www.google.com的请求 - 成功

vps请求与Google服务器建立连接 - 失败

分析：

shadowsocks接收到数据后会通过其他端口（如52333）与Google服务器建立连接，而这个端口（52333）并没有在iptables中配置 input accept，这就导致在vps与Google服务器建立连接开始的三次握手中Google服务器响应的ACK数据包会被iptables drop，导致连接建立失败，使得shadowsocks无法正常工作

解决方法：

我们需要让vps主动发起的连接请求所返回的响应通过iptables，命令如下：

`sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT`
