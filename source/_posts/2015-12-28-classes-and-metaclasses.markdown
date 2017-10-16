---
layout: post
title: "Classes and Metaclasses"
date: 2015-12-28 13:53:36 +0800
comments: true
categories: 
---
这是一篇译文，原文[在这里](http://www.sealiesoftware.com/blog/archive/2009/04/14/objc_explain_Classes_and_metaclasses.html)，水平有限，翻译难免有错误的地方。  

`Objective-C`是一个基于类别对象的系统。每一个对象都是某个类的实例，对象的`isa`指向它自己的类(Class),这个类描述了该对象所有的数据信息：开辟的空间大小，成员变量类型以及其他信息的排列，这个类同样也描述了该对象可以执行的一些行为：可以响应哪些`selector`以及它自己实例方法的实现。  

类的方法列表里面其实就是该类的所有实例方法的集合，通俗的讲就是该对象可以响应的所有`selectors`。当你向一个类发送消息时，会通过`objc_msgSend()`去查找它自身的类的方法列表(如果它有父类的话，也会在它的父类方法列表中去查找)来决定调用哪一个方法。  
<!--more-->
`Objective-C`的每一个类也是一个对象，它也有`isa`指针以及其他的一些信息，同样它也能响应`selector`，当你调用类方法时，比如：`[NSObject alloc]`，其实你是在向该类发送消息。  
  
上面说了，类也是一个对象，那它必定也是其他某一个类的一个实例，这就是`metaclass`，`metaclass`里面存放了类对象的所有信息，类和`metaclass`的关系就和刚才上面讲的实例对象和该实例的类的关系一样，`metaclass`的方法列表里面存放的该类对象能够响应的方法。当你向一个类(这个类是某个metaclass的一个实例对象)发送一个消息时，`objc_msgSend()`回去遍历`metaclass`的方法列表，如果它有父类的话，同样也会遍历父类的所有方法列表，从而决定调用哪一个方法。类方法的定义存放与类对象的`metaclass`，与之相对应的是实例方法的定义存放于实例对象对应的类里面。  
  
`metaclass`究竟是个啥？它也是一直向下的吗？NO，一个`metaclass`又是某个类的根类的`metaclass`实例对象，根`metaclass`又是它自身的一个实例对象，在这里`isa`指针的指向形成了一个环：实例对象->类->metaclass->根metaclass->它自己(根metaclass)，在实际的使用中我们很少和metaclass直接打交道。  

`metaclass`的父类和类的父类直接形成了两条平行线，所有类方法的继承类似于实例方法的继承，并且根`metaclass`的父类就是根类，所以每个类对象都能响应根类的实例方法，最后需要说一下的是一个类对象是它根类或者根类的一个子类的实例。  
  
是不是觉得有些迷惑，下面的图或许能给你一点帮助，你需要记住的是，当向一个对象发送任何消息是，都是从该对象的`isa`指针开始，然后按着它的父类一直寻找，直到找到合适的方法去调用，实例方法定义在类里面，类方法定义在`metaclass`和根类里面。  
![image](http://ww4.sinaimg.cn/mw690/8f7a6fe0jw1ezfb7ybjzqj20gw0hm40e.jpg)
  
`Objective-C`为了它的实用性，将类方法由`metaclass`管理，但另一方面它又想把`metaclass`影藏起来，例如`[NSObject class]`和`[NSObject self]`是完全等价的，其实它这个时候是返回的`metaclass`，也就是`NSObject->isa`所指向的。