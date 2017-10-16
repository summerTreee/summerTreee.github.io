---
layout: post
title: "App中拨打电话一点小总结"
date: 2015-08-23 12:27:08 +0800
comments: true
categories: 
---
###App内发起电话拨打的一点小技巧
####一、拨打电话
在App内发起电话拨打主要有两种方式：

1、利用这种方式发起的电话拨打，通话结束后不会直接返回App内，而是停留在通信录里面：
	
	 NSString *str = [[NSString alloc] initWithFormat:@"tel:%@",@"131xxxx1909"];
    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:str]];
<!--more-->
2、利用`UIWebView`实现电话拨打，会弹出拨打提示，并且拨打完成后会返回App内：

	NSString *phone = @"131****1909";
    NSString *cleanedString =[[phone componentsSeparatedByCharactersInSet:[[NSCharacterSet characterSetWithCharactersInString:@"0123456789-+()"] invertedSet]] componentsJoinedByString:@""];
    NSString *escapedPhoneNumber = [cleanedString stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    NSURL *telURL = [NSURL URLWithString:[NSString stringWithFormat:@"tel://%@",escapedPhoneNumber]];
    UIWebView *mCallWebview = [[UIWebView alloc] init] ;
    
    [self.view addSubview:mCallWebview];
    [mCallWebview loadRequest:[NSURLRequest requestWithURL:telURL]];
    
    
####二、检测通话时间
利用`CTCallCenter`我们可以检测在使用App期间拨打电话出去以及电话打入的时机，以及通话结束的时机。实现如下：

     #import <CoreTelephony/CTCallCenter.h>
     #import <CoreTelephony/CTCall.h>
     
	 @interface AppDelegate : UIResponder<UIApplicationDelegate>
     @property (nonatomic, strong) CTCallCenter *callCenter;
     @end

    @implementation AppDelegate

    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
    {
      self.callCenter = [[CTCallCenter alloc] init];
      [self handleCall];
      return YES;
    }

    -(void)handleCall
    {
       self.callCenter.callEventHandler = ^(CTCall *call) {
        if ([call.callState isEqualToString: CTCallStateConnected])
        {
            NSLog(@"接通电话");
        }
        else if ([call.callState isEqualToString: CTCallStateDialing])
        {
            NSLog(@"发起呼叫");
        }
        else if ([call.callState isEqualToString: CTCallStateDisconnected])
        {
            NSLog(@"结束电话");
        }
        else if ([call.callState isEqualToString: CTCallStateIncoming])
        {
            NSLog(@"打入电话");
        }
       };
    }
    
