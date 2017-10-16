---
layout: post
title: "如何检测iPhone设备处于低电量模式"
date: 2016-06-12 20:52:53 +0800
comments: true
categories: 
---
在iOS9中，苹果为iPhone增加了低电量模式，开启低电量模式后，系统会为了节约电量而停止一些耗电的行为，例如接收邮件，通过`Hey Siri`唤起，后台消息推送等。  
  
很重要的一点是系统不会为用户自动打开低电量模式，而是由用户自己去决定是否进入低电量模式，进入低电量模式后状态栏中的电池会变为黄色：  
<!--more-->
![image](http://ww2.sinaimg.cn/mw690/8f7a6fe0jw1f4sqcia5uuj20jq0f4gn2.jpg)  
  
当你充电量达到80%时会自动关闭低电量模式。  
  
####检测低电量模式
在iOS9里面你可以通过如下方法得知当前设备是不是处于低电量模式：  
  
```  
// Objective-C
if ([[NSProcessInfo processInfo] isLowPowerModeEnabled]) {
  // stop battery intensive actions
}
  
```  
  
并且你可以通过注册`NSProcessInfoPowerStateDidChangeNotification`通知来检测低电量模式的变化：  
  
```  
// Objective-C
[[NSNotificationCenter defaultCenter] addObserver:self
  selector:@selector(didChangePowerMode:)
  name:NSProcessInfoPowerStateDidChangeNotification
  object:nil];
  
  
  // Objective-C
- (void)didChangePowerMode:(NSNotification *)notification {
  if ([[NSProcessInfo processInfo] isLowPowerModeEnabled]) {
    // low power mode on
  } else {
    // low power mode off
  }
}
  
```     
  
######注意：  
1. 该通知和`lowPowerModeEnabled`只有在iOS9里面才有，支持iOS8的App需要自己在代码里面判断。
2. 低电量模式是iPhone设备独有的，在iPad里面会返回false。  
  
开启低电量模式后，我们需要采取一些措施去节约电量，比如Apple给的一些建议：  
  
- 停止定位位置更新。
- 限制动画的使用。
- 限制后台的网络请求。
- 关闭`motion effects`。

参考【[链接](http://useyourloaf.com/blog/detecting-low-power-mode/?utm_campaign=iOS%2BDev%2BWeekly&utm_medium=web&utm_source=iOS_Dev_Weekly_Issue_252)】

  
