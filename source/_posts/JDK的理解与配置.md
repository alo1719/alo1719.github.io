---
title: JDK的理解与配置
date: 2019-03-08 19:46:32
tags: [java, jdk]
categories: 入门教程
---

## 什么是JDK

Sun公司在创造Java这一门语言时便有提到Java技术体系至少包括4个部分:

1. Java程序设计语言
2. 各平台上的Java虚拟机 (Java Virtual Machine，JVM)
3. Java API类库
4. 一系列辅助工具，最知名的如javac

而 2+3 构成了JRE（Java Runtime Environment，Java运行时环境），是Java程序运行依赖的最小环境。

1+2+3+4 构成了JDK (Java Development Kit)，也即 JRE+1+4 是Java开发所依赖的最小环境。

从低级向高级，从底层到高层而言，是JVM -> JRE -> JDK的顺序。
<!-- more -->

## Oracle JDK / Open JDK

市面上有两种常见的JDK，一种是Oracle JDK，另一种是Open JDK。两者的授权协议不一样，Open JDK大部分代码是开源的，Oracle JDK相比于Open JDK包含更多的东西，两者的核心功能是没有多少差别的。

## JavaSE / JavaEE / JavaME

JavaSE (Standard Edition) 通俗理解就是桌面端的软件。

JavaEE (Enterprise Edition) 通俗理解是Web后端服务软件，实际上基于JavaSE而发展出来的一套 *规范* 接口。 像Spring Boot，Spring Cloud等用的都是JavaEE。

JavaME (Micro Edition) 用于嵌入式开发，很少能用到。

三者所依赖的都是JDK。

## Oracle JDK配置流程（Windows）

1. 到[Oracle官网](https://www。oracle。com/technetwork/java/javase/downloads/index。html)下载JDK (Java Development Kit)，JDK中包含了JRE (Java Runtime Environment)。

   推荐安装Java8最新的奇数版本号版本(例如8u201)，综合而言是最稳定最泛用的版本。

2. 安装JDK以及JRE

   安装完毕可以进入JDK文件夹。

   bin目录中 `java.exe` 用来运行java程序，`javac.exe` 用来编译java程序。

3. 配置环境变量

   打开 `计算机-属性-高级系统设置-高级-环境变量`

   将JDK中的bin目录添加到环境变量。