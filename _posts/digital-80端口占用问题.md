---
name: 80port
title: 如何定位端口占用问题
date: 2024-10-24
categories: 数码
---



80端口占用问题

找到占用者，其pid为4

![image-20241025203515838](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025203515838.png)

根据pid找到其名为System，不能直接taskKill

![image-20241025203538420](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241025203538420.png)

netsh http show servicestate

发现是defaultAppPool在占用，控制器进程的pid为4671



任务管理器中找到4671，发现是iis Publishing 服务，禁用