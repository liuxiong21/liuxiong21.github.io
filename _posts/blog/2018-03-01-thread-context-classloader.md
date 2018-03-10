---
layout: post
title: 关于Thread的ContextClassLoader
categories: Java
description: SOA vs. Microservices
keywords: SOA, Microservices
---

看springframework源码，看到很多获取ClassLoader的地方都会使用到Thread.currentThread().getContextClassLoader()这段代码的存在，那么到底它和class.getClassLoader()有什么不一样的地方呢？

先看下下面这段代码：
```
  @Nullable
	public static ClassLoader getDefaultClassLoader() {
		ClassLoader cl = null;
		try {
			cl = Thread.currentThread().getContextClassLoader();
		}
		catch (Throwable ex) {
			// Cannot access thread context ClassLoader - falling back...
		}
		if (cl == null) {
			// No thread context class loader -> use class loader of this class.
			cl = ClassUtils.class.getClassLoader();
			if (cl == null) {
				// getClassLoader() returning null indicates the bootstrap ClassLoader
				try {
					cl = ClassLoader.getSystemClassLoader();
				}
				catch (Throwable ex) {
					// Cannot access system ClassLoader - oh well, maybe the caller can live with null...
				}
			}
		}
		return cl;
	}
```
通过上面的代码，我们可以看到Spring在获取ClassLoader的时候，会进行三次尝试，分别为当前线程的Context、当前Class以及系统的ClassLoader。那么这三种方式获取到的ClassLoader有什么不一样呢？为什么要首先从线程的上下文中获取？


## 首先来说说Java的类加载机制
Java中的类加载器分为两种，一种是系统提供的，一种是开发人员自己提供的。  
系统提供了三种类加载器：
1. 引导类加载器（bootstrap class loader）：它用来加载 Java 的核心库，是用原生代码来实现的，并不继承自 java.lang.ClassLoader。
2. 扩展类加载器（extensions class loader）：它用来加载 Java 的扩展库。Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java 类。
3. 系统类加载器（system class loader）：它根据 Java 应用的类路径（CLASSPATH）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader()来获取它。上面代码中最后就是采用的这个。

大家都知道Java采用代理模式来加载class的字节码，也就是当类加载器在尝试自己去查找某个类的字节代码并定义它时，会先代理给其父类加载器，由父类加载器先去尝试加载这个类，依次类推。这种机制在大多数时候是没有问题，但是在一些特殊情况下就失效了。

## 会遇到什么问题

比如Java提供了很多SPI，允许第三方类库定义其实现。然后这些SPI的接口都是由核心类库提供的，也就是这些类的加载是由引导类加载器加载的，而实现是由第三方实现的，引导类是无法加载到第三方库里面的类的，这个时候线程Context ClassLoader就能够解决这个问题。

## 安全问题
除了从本地加载类以外我们还可以从网络上下载class字节数据来实现类的加载，如果不采用代理委派的方式来加载类就有可能造成Jdk里面的类被恶意修改的情况。

另外默认如果不设置线程的Context ClassLoader，那么将继承其父线程的类加载器，而Java应用运行的初始线程的上下文类加载器是系统类加载器。
