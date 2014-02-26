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

###对比ArrayList、Vector、LinkedList的存储性能和特性
ArrayList和Vector都是使用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，它们都允许直接按序号索引元素，但是插入元素要涉及数组元素移动等内存操作，所以索引数据快而插入数据慢，Vector由于使用了synchronized方法（线程安全），通常性能上较ArrayList差，而LinkedList使用双向链表实现存储，按序号索引数据需要进行前向或后向遍历，但是插入数据时只需要记录本项的前后项即可，所以插入速度较快。

###Collection 和 Collections的区别。
Collection是集合类的上级接口，继承于他的接口主要有Set和List。  
Collections是针对集合类的一个帮助类，他提供一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作。

###HashMap和Hashtable的区别。
HashMap是Hashtable的轻量级实现（非线程安全的实现），他们都实现了Map接口，主要区别在于HashMap允许空键值，由于非线程安全，效率上可能高于Hashtable。  
HashMap把Hashtable的contains方法去掉了，改成containsValue和containsKey，因为contains方法容易让人引起误解。  
Hashtable继承自Dictionary类，而HashMap是Java1.2引进的Map interface的一个实现。  
在多个线程访问Hashtable时，不需要自己为它的方法实现同步，而HashMap就必须为之提供外同步。Hashtable和HashMap采用的hash/rehash算法大致相同，所以性能不会有很大的差异。

###final、finally、finalize的区别
final用于声明属性，方法和类，分别表示属性不可变，方法不可覆盖，类不可继承。  
finally是异常处理语句结构的一部分，表示总是执行。  
finalize是Object类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法，可以覆盖此方法提供垃圾收集时的其他资源回收，例如关闭文件等。

###sleep()和wait()的区别
sleep是线程类（Thread）的方法，使此线程暂停执行指定时间，将执行机会给其他线程，但是监控状态依然保持，到时后会自动恢复，调用sleep不会释放对象锁。  
wait是Object类的方法，对此对象调用wait方法导致本线程放弃对象锁，进入等待此对象的等待锁定池，只有针对此对象发出notify方法（或notifyAll）后本线程才进入对象锁定池准备获得对象锁进入运行状态。

###Overload和Override的区别
方法的重写（Overriding）和重载（Overloading）是Java多态性的不同表现。  
重写是父类与子类之间多态性的一种表现，重载是一个类中多态性的一种表现。如果在子类中定义某方法与其父类有相同的名称和参数，我们说该方法被重写。子类的对象使用这个方法时，将调用子类中的定义，对它而言，父类中的定义如同被"屏蔽"了一样；如果在一个类中定义了多个同名的方法，它们或有不同的参数个数或有不同的参数类型，则称为方法的重载。Overloaded的方法是可以改变返回值的类型。

