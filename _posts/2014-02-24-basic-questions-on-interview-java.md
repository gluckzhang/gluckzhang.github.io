---
date: 2014/2/24 17:25:57
layout: post
title: 技术一面基本问题梳理-Java篇
thread: 201402241726
categories: 学习
tags: Interview Java
---

###Java的基本数据类型？我们能否继承String类？
基本数据类型包括byte、int、char、long、float、double、boolean和short。  
java.lang.String类是final类型的，因此不可以继承这个类、不能修改这个类。

###int 和 Integer 有什么区别？
Java 提供两种不同的类型：引用类型和原始类型（或内置类型）。Int是java的原始数据类型，Integer是java为int提供的封装类，Java为每个原始类型提供了封装类。  
引用类型和原始类型的行为完全不同，并且它们具有不同的语义。两者之间的差别包括大小和速度问题、以哪种类型的数据结构存储、当引用类型和原始类型用作某个类的实例数据时所指定的缺省值。对象引用实例变量的缺省值为null，而原始类型实例变量的缺省值与它们的类型有关。

###String 和 StringBuffer 的区别
String和StringBuffer均可以储存和操作字符串，即包含多个字符的字符数据。String类提供了数值不可改变的字符串，而StringBuffer类提供的字符串可以进行修改，当你知道字符数据要改变的时候你就可以使用StringBuffer。典型地，你可以使用StringBuffer来动态构造字符数据。

###运行时异常与一般异常有何异同？
异常表示程序运行过程中可能出现的非正常状态，运行时异常表示虚拟机的通常操作中可能遇到的异常，是一种常见运行错误。java编译器要求方法必须声明抛出可能发生的非运行时异常，但是并不要求必须声明抛出未被捕获的运行时异常。


###说出Servlet的生命周期，以及Servlet和CGI的区别
Servlet被服务器实例化后，容器运行其init方法，请求到达时运行其service方法，service方法自动派遣运行与请求对应的doXXX方法（doGet，doPost）等，当服务器决定将实例销毁的时候调用其destroy方法。  
与CGI的区别在于servlet处于服务器进程中，它通过多线程方式运行其service方法，一个实例可以服务于多个请求，并且其实例一般不会销毁，而CGI对每个请求都产生新的进程，服务完成后就销毁，所以效率上低于servlet。


