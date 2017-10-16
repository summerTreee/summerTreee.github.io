---
layout: post
title: "iOS中如何在一个工程创建多个App"
date: 2016-01-23 20:11:40 +0800
comments: true
categories: 
---

一般情况下，我们是一个App应用对应一个`Xcode Project`，但是如果你需要同时开发多个产品，而这些产品90%的数据结构以及交互方式都一样，呈现在用户面前的这两个产品，最大的不一样就是UI元素以及某些配色时，如果这个时候还是一个App一个`Xcode Project`，普遍的做法是：  
  
你会先开发完成一个产品，然后在复制到其他的产品中，非常的麻烦而且效率很低，你需要一个一个文件去比对。如果你使用模块的方式，开发完一个模块，然后再利用`Pod`的方式导入到其他产品中，虽然这样可行，但是涉及到产品的迭代开发以及产品的随时会变的交互，模块的细化分很难实现。  
  
如果你也面临这样的问题，不放考虑一下下面讲的`一个工程来开发多个App`：  

<!--more-->

####1、创建新的Target
如果现在你已经有个一个产品叫`MultipleTargetA`，如下图所示：  
  
![image](http://ww3.sinaimg.cn/mw690/8f7a6fe0gw1f09onfv2lnj20e709xabz.jpg)

这时你想添加一个叫`MultipleTargetB`的产品，你需要做的是按如下的步奏进行创建：
选择`Project -> Targets -> 右击MultipleTargetsA -> 选择Duplicate`，这时我们就按照`MultipleTargetsA`复制了一个产品`MultipleTargetsA copy`，并且你会看到多出了一个文件`MultipleTargetsA copy-Info.plist`，如下图所示：    

![image](http://ww3.sinaimg.cn/mw690/8f7a6fe0gw1f09onggesdj20cr0b8q53.jpg)

将**TARGETS**里面的`MultipleTargetsA copy`改名为`MultipleTargetsB`（选中回车进入编辑）。

####2、编辑plist文件
上面讲到了当我们创建新的`Target`后会多出一个`MultipleTargetsA copy-Info.plist`文件，这个`plist`文件就是控制`MultipleTargetsB`的名称，版本等信息的文件，我们为了统一将他改为`MultipleTargetsB-Info.plist`，在修改名字之前你需要在`MultipleTargetsB`的`Build Settings`中找到`MultipleTargetsA copy-Info.plist`一项，待会儿我们修改完这个`plist`文件以后，还需要在这里填入它正切的位置信息。这样程序执行时才能找到它，不然程序是不能通过编译的：

![image](http://ww3.sinaimg.cn/mw690/8f7a6fe0gw1f09ongryr7j20oq0algnp.jpg)

处理完上面的操作后，程序能正常通过编译后我们还需要修改`plist`文件里面的一下符合我们预期的信息：

![image](http://ww2.sinaimg.cn/mw690/8f7a6fe0gw1f09onhbzvjj20h20aydj0.jpg)

####3、定义Preprocessor Macros
现在我们的工程里面同时包含了两个`Target`，现在工程里面的类是这两个`Target`公用的，如果你想在一个类里面区分是`MultipleTargetsA`还是`MultipleTargetsB`，这时你就需要用到`Preprocessor Macros`了，它的定义很简单，选中一个`Target`，然后在`Build Settings`里面搜索`Preprocessor Macros`一项，然后在里面添加表明是`MultipleTargetsA`的宏：  

![image](http://ww2.sinaimg.cn/mw690/8f7a6fe0gw1f09oni1sa6j20ol0fbwhe.jpg)

假如我们为`MultipleTargetsA`和`MultipleTargetsB`分别按如上步奏添加了标识：`MultipleTargetsA`和`MultipleTargetsB`,那在某个类里面判断现在编译的目标是哪个`Target`就变得简单了

```
- (void)someFunction
{
#ifdef MultipleTargetsA
    NSLog(@"Build For Target MultipleTargetsA!");
#else
    NSLog(@"Build For Target MultipleTargetsB!");
#endif
}

```

####4、修改Scheme
现在`MultipleTargetsB`的`Scheme`还是`MultipleTargetsA copy`:   
 
![image](http://ww4.sinaimg.cn/mw690/8f7a6fe0gw1f09onihy4tj20c20820uh.jpg)

我们可以通过`Manage Schemes`，选择`MultipleTargetsA copy`，然后按回车键进行编辑，将它改为`MultipleTargetsB`：

![image](http://ww4.sinaimg.cn/mw690/8f7a6fe0gw1f09onj571gj20m00d4dh4.jpg)

####5、资源文件和类文件
上面说了几个`APP`%90是相同的，不同的地方的资源文件和类文件如果要加以区分，比如`A.m`类属于`MultipleTargetsA`而不属于`MultipleTargetsB`，这时我们可以利用`Xcode`的`Target Membership`功能，来选择该类属于哪个`Target`。  

![image](http://ww2.sinaimg.cn/mw690/8f7a6fe0gw1f09onjlay4j20dw0a5dh8.jpg)

对于`Assets`文件来说，它只能整体的选择某个`Target`，所以不同的`Target`你可能需要建立不同的`Assets`文件。