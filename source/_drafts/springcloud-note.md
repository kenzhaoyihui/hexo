title: springcloud-note
author: Kenzhaoyihui
date: 2018-07-29 10:51:20
tags:
---
#### springcloud4 - 服务注册与发现

*  ##### 服务发现/注册
服务发现协议（Service Discovery Protocol)
	* Java: Jini (Apache River)
    * REST: HATEOAS
    * Web Services: UDDI(Universal Description Discovery and Integration)统一描述发现和整合
    
服务注册， 常见的注册中心：
	* Apache Zookeeper, 高一致性， 但是可能损失了高可用这个特性
    * Netflix Eureka ，高可用， 但是一致性方面差一些
    * Consul， 折中方案， 高可用和一致性都有兼顾

*  ##### 高可用架构

一种系统特性， 致力于确保可接受程度的操作执行， 通常采用上线时间作为基准。其中，以一年上线时间与自然事件的比率来描述可用性。

基本原则：
消灭单点故障
可靠性交迭
故障探测
--故障自动化恢复

*  ##### springcloud Netflix Eureka
Eureka server是Eureka client的注册服务中心， 管理所有的注册服务以及实例信息和状态

*  ##### springcloud config 