title: JAVA note
author: Kenzhaoyihui
date: 2018-08-08 14:03:10
tags:
---
总结：
1. 源码分析
常见的设计模式： Proxy, Factory, Singleton, Delegate, Strategy, Prototype, Template
Spring5
Mybatis

2. 工程
Maven, Jenkins, Sonar, Git

3. 多线程与并发
见博客整理

4. 性能优化
JVM， JMM， GC， GC日志 
Tomcat, 运行机制和框架， tomcat线程模型， 系统参数和调优
MySQL(数据库优化）， MySQL底层B+ Tree， SQL执行计划， 索引优化， SQL语句优化

5. 微服务
springboot， springCloud(见视频）， Docker虚拟化， Kubernetes， 
本章问题：
SOA架构喝微服务之间的架构有什么区别和联系
如何设计微服务及其设计原则
Springboot为什么这么流行， 以及具体能够解决什么问题
什么是SpringCloud， 为何要选择SpringCloud
基于全局分析SpringCloud各个组件所解决的问题

6. 分布式
分布式架构原理
	分布式原理？如何构建分布式？构建分布式的重要因素？CDN？
分布式架构策略
	分布式架构网络通信原理
	通信协议中的序列化和反序列化
	基于框架的RPC技术WebService/RMI/Hessian
	深入分析Zookeeper在disconf配置中心的作用
	基于Zookeeper实现分布式服务器动态上下线感知
	深入分析Zookeeper Zab协议以及选举机制源码
	Dubbo管理中心以及监控平台安装部署
	基于Dubbo的分布式系统架构实战
	Dubbo容错机制以及高扩展性分析

分布式架构中间件
	分布式消息通信ActiveMQ/Kafka/RabbitMQ
	Redis主从复制原理以及无磁盘复制分析
	Redis中AOF和RDB持久化策略原理
	MongoDB企业级集群解决方案
	MongoDB数据分片，转存以及恢复策略
	基于OpenResty部署应用层Nginx以及Nginx+Lua实战
	Nginx反向代理服务器以及负载均衡服务器
	基于Netty实现高性能IM聊天
	基于Netty实现Dubbo多协议通信支持
	Netty无锁化串行设计以及高并发处理机制

分布式架构实战
	分布式全局ID生成方案
	Session跨域共享以及企业级单点登陆解决方案实战
	分布式事物解决方案实战
	高并发下的服务降级，限流实战
	基于分布式架构下分布式锁的解决方案实战
	分布式架构下实现分布式定时调度
