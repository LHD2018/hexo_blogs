---
uuid: 9e5de499-5b45-3119-ea37-75ab542120e3
title: Socket封装
date: 2020-11-25 10:44:04
tags: code
---

# 写在前面
	因为经常用到套接字通信，故对socket函数进行封装，方便后面直接调用，涉及到的平台包括linux和windows。

## Linux

[github开源地址](https://github.com/LHD2018/socket_for_linux)

+ 目录结构
> include 		
>> common.h	   		需要的头文件
>> mysocket.h	      socket基类头文件（包含发送以及接收函数）
>> tcpclient.h	 	客户端类头文件
>> tcpserver.h		服务端类头文件

> src 
>> client_main.cpp	客户端测试代码
>> server_main.cpp	服务的测试代码
>> tcpclient.cpp		客户端实现
>> tcpserver.cpp		服务端实现

> CMakeLists.txt

+ 项目说明
	参考C语言技术网的freecplus框架实现的linux上套接字通信，自定义报文格式，解决粘包问题， 多线程实现小并发。
	
---

## Windows

[github开源地址](https://github.com/LHD2018/socket_for_windows)

**详情请看github**
