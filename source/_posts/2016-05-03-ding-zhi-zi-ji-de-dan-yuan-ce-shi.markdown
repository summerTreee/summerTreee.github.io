---
layout: post
title: "定制自己的单元测试"
date: 2016-05-03 09:18:10 +0800
comments: true
categories: 
---
我们在进行开发的过程中，会对新开发的功能进行但愿测试，不管是TDD还是BDD，我们都需要利用单元测试来保证我们新功能的正确性，这其中包括执行结果是否正确，对边界处理是否周全，一些特殊情况的测试，这对于一些功能较少的工程可能写好一个`testXXX`函数，执行一遍单元测试，不会花太多时间，但是对于一些大的工程，可能测试的类会有成百甚至上千个，当你想专注于测试新开发的功能是，每次测试都全部跑一次完整的单元测试，这显然是很耗费时间的行为。
<!--more-->
对于这种情况，我们可以利用`Xcode`来新建一个`scheme`，专门只是测试当前新开发的功能的单元测试，步骤如下：  
  
######1、新建一个scheme

![image](http://ww1.sinaimg.cn/mw690/8f7a6fe0jw1f3hxfjso7oj208u062dgf.jpg)  
  
######2、对scheme命名
在新的窗口中，`target`的下拉框中选择`*Tests`，这里命名为`MyCustomTests`。  
  
![image](http://ww3.sinaimg.cn/mw690/8f7a6fe0jw1f3hxfkav8jj20r409ugmw.jpg)  
  
######3、定制自己的单元测试
选择刚才新创建的`scheme`，进入`Edit Scheme...`,进入`Test`选项中，右边就能对需要测试的文件进行筛选：  
  
![image](http://ww2.sinaimg.cn/mw690/8f7a6fe0jw1f3hxfknumnj21dk0rwagm.jpg)  
  
这是我们再进行单元测试时，就只会执行我们刚才勾选的单元测试文件：  
  
![image](http://ww3.sinaimg.cn/mw690/8f7a6fe0jw1f3hxflboq8j20ea0a276i.jpg)