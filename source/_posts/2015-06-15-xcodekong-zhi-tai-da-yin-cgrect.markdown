---
layout: post
title: "Xcode控制台打印CGRect"
date: 2015-06-15 21:15:58 +0800
comments: true
categories: 
---
在用Xcode调试开发的时候，我们经常需要在控制台打印某个视图的frame，以前的时候相当麻烦，但是从Xcode6.3以后，打印frame变得很简单：

```
// Before
(lldb) p (CGRect) [self frame]
(CGRect) $0 = (origin = (x = 0, y = 0), size = (width = 50, height = 100))
 
// After
(lldb) expr @import UIKit
(lldb) p self.frame
(CGRect) $1 = (origin = (x = 0, y = 0), size = (width = 50, height = 100))

```