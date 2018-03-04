---
layout: post
title: SOA vs. Microservices
categories: 架构
description: SOA vs. Microservices
keywords: SOA, Microservices
---

大家有没有想过SOA和微服务有什么区别？它们又有什么联系吗？
答案是他们是有点相似又有很多不相似的地方，微服务其实是SOA架构的一种演变，SOA是微服务的SuperSet。

## 什么是SOA

Service Oriented Architecture是一种软件架构，应用组件通过通信协议为其他组件提供服务。


## 什么是Microservices

微服务架构风格是把单个应用拆分成一系列小服务来开发，而一个个服务单独运行并且使用轻量级机制来通信如HTTP，REST。


## 它们之间的区别

1. 关注点不同：

    SOA主要关注于业务功能重用而微服务主要关注于上下文边界，也就是说在一个SOA服务里面可能包含多个不同的业务功能，而微服务风格更倾向于把一个个业务功能拆分成单独的服务。

2. 架构方式不一样：

    微服务倾向于把一个大的单体应用按模块拆分成一个个独立的服务，而SOA架构的组成单元是一个个独立的单体应用。

3. 运维方式：

    SOA很少关注Devops，而微服务强调Devops和持续交付。
