---
layout: post
title: "iOS测试之UIAutomation介绍"
date: 2016-02-29 07:47:18 +0800
comments: true
categories: 
---

UIAutomation是随着iOS SDK 4.0引入的，帮助开发者在`while you sleep`的时候也能帮你进行自动化的UI测试，它的测试代码使用Javascript编写，不过别担心，如果你对Javascript不熟悉的话，可以使用Instrument中`UIAutomation`的录制功能，它能将你的操作转换为测试代码，你只需要保存这些自动生成的代码就行，稍后我们将介绍该功能。  

使用`UIAutomation`做自动化UI测试时，使用者需要做最基本的两件事情是：  

- 如何找到界面上的UI元素
- 如何针对找到的UI元素进行测试操作  

<!--more-->
####如何找到界面上的UI元素
在`UIAutomation`中，界面就是一堆UI元素构建的层级结构，这些元素需要一个叫`Accessibility label`去标记，它可以在如下所示的地方进行设置：    

![image](http://ww4.sinaimg.cn/mw690/8f7a6fe0gw1f1fv1q07rxj20bn0d83zs.jpg)  

`UI Accessibility`在iOS3.0就被引入了，它是用来辅助身体不便的人士使用APP的，VoiceOver是Apple的屏幕阅读技术，而`UI Accessibility`的基本原则就是对屏幕上的UI元素进行分类和标记，两者配合，通过阅读和聆听这些元素，用户就可以在不接触屏幕的情况下通过声音使用APP。  

`Accessibility` 的核心思想是对 UI 元素进行分类和标记，将屏幕上的UI分类为像是按钮，文本框，Cell或者是静态文本(也就是label) 这样的类型，然后使用`identifier`来区分不同的UI元素。用户可以通过语音控制app的按钮点击，或是询问某个label的内容等等，十分方便。iOS SDK 中的控件都实现了默认的`Accessibility`支持，而我们如果使用自定义的控件的话，则需要自行使用 `Accessibility` 的 API 来进行添加。  
  
####对找到的UI元素进行测试操作
#####1、启动Instruments
由于`UIAutomation`被集成到了`Instruments`中，所以要进行`UIAutomation`测试，首先打开如下所示的工具`Automation`   
 
![image](http://ww2.sinaimg.cn/mw690/8f7a6fe0gw1f1fv1qinz2j20l90bz774.jpg)  

######2、编写测试代码
打开`Automation`如下图所示：    

![image](http://ww3.sinaimg.cn/mw690/8f7a6fe0gw1f1fv1r6ehaj20ou0jdq6l.jpg)  
  
在这里我们可以进行测试代码的编写或者在右边区域直接导入已经写好的测试代码，正如前面提到的，这些测试代码需要使用JavaScript进行编写，如果你不熟悉或者嫌麻烦，可以使用底部的录制功能，启用录制后你对屏幕的测试操作都会被自动的转为测试代码，非常方便。  

测试如果不通过，就会在顶部的时间轴上标红，测试也会自动停止。  

######3、测试代码相关的简介
在`UIAutomation`中所有元素都继承自`UIAElement`，这个对象提供了每个UI元素所应具备的如下属性：    

- name
- value
- elements
- parent  
  
一、Target application  
获取方法：    

```  
UIAtarget.localTarget().ftontMostApp()  
```  
![image](http://ww3.sinaimg.cn/mw690/8f7a6fe0gw1f1fv1rmjppj208u0dw769.jpg)  

二、Main Window  
获取到他的方法如下：  
  
```  
UIAtarget.localTarget().ftontMostApp().mainWindow()  
```  
![image](http://ww3.sinaimg.cn/mw690/8f7a6fe0gw1f1fv1rmjppj208u0dw769.jpg)  
  
三、View  
获取主视图的方法如下(eg:UITableView):  
  
```  
UIATarget.localTarget().frontMostApp().mainWindow ().tableViews()[0]  
```  
![image](http://ww1.sinaimg.cn/mw690/8f7a6fe0gw1f1fv1se2hij208u0dwac1.jpg)  
  
四、Element  
例如获取tableView下的第一行：  
  
```  
UIATarget.localTarget().frontMostApp().mainWindow ().tableViews()[0].cells()[0]  
```
  
![image](http://ww3.sinaimg.cn/mw690/8f7a6fe0gw1f1fv1t0c7rj208w0dwtao.jpg)  
  
五、Child Element  
例如想获取第一行里面的显示标题的label：  

```  
UIATarget.localTarget().frontMostApp().mainWindow ().tableViews()[0].cells()[0].elements()[“第一章 会计法律制度”]  
```  
![image](http://ww2.sinaimg.cn/mw690/8f7a6fe0gw1f1fv1te5d2j208u0dwabz.jpg)  
  
六、button的点击事件  
例如一个叫`Edit`的button，我们要模拟对他的点击，可以如下测试：    

```  
UIATarget.localTarget().frontMostApp().navigationBar().buttons()["Add"].tap();  
```  
七、文本的输入  
例如界面上只有一个`UITextField`，我们要测试对它输入一段文字：    

```  
 var name = “Turtle Pie”; UIATarget.localTarget().frontMostApp().mainWindow().textFields()[0].setValue(name);  
```
八、Logging  
开始测试和结束测试：  
  
```  
 var testName = "My first test"; UIALogger.logStart(testName); ... // test code ... UIALogger.logPass(testName);
```  
  
测试过程中打印Log信息：  
  
```  
 var testName = "My first test"; UIALogger.logStart(testName); ... UIALogger.logMessage("Logging about my test"); ... UIALogger.logPass(testName);  
```  
  
九、模拟点击Home键进入后台一段时间再自动唤起：  
  
```  
 UIALogger.logMessage("Deactivating app");
 //10s后自动唤起 UIATarget.localTarget().deactivateAppForDuration(10); UIALogger.logMessage("Resuming test after deactivation");
```  
十、模拟Orientation切换：  
   
```  
 var target = UIATarget.localTarget(); var app = target.frontMostApp(); // set landscape left target.setDeviceOrientation(UIA_DEVICE_ORIENTATION_LANDSCAPELEFT); UIALogger.logMessage("Current orientation is " + app.interfaceOrientation()); // portrait target.setDeviceOrientation(UIA_DEVICE_ORIENTATION_PORTRAIT); UIALogger.logMessage("Current orientation is " + app.interfaceOrientation());   
```    
#####模拟手势操作
一、Taps  
  
```  
UIATarget.localTarget().tap({x:100, y:200});UIATarget.localTarget().doubleTap({x:100, y:200});UIATarget.localTarget().twoFingerTap({x:100, y:200});
```    
  
二、Pinches  
  
```    
//指定pinch在2秒内完成
UIATarget.localTarget().pinchOpenFromToForDuration({x:20, y:200}, {x:300, y:200},2);UIATarget.localTarget().pinchCloseFromToForDuration({x:20, y:200}, {x:300, y:200}, 2);
```  

三、Drag and Flick  
  
```  
UIATarget.localTarget().dragFromToForDuration({x:160, y:200}, {x:160, y:400}, 1);UIATarget.localTarget().flickFromTo({x:160, y:200}, {x:160, y:400});
```

虽然从iOS7开始，`UIAutomatation`苹果不在维护了，转而支持Xcode中的UI Testing，但是它却可以测试复杂的手势操作，这是目前UI Testing无法完成的。  
  
#####参考文章
1、https://developer.apple.com/videos/play/wwdc2010/306/  
2、http://blog.manbolo.com/2012/04/08/ios-automated-tests-with-uiautomation  
3、http://onevcat.com/2015/09/ui-testing/ 
  