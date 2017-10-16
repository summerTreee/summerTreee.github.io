---
layout: post
title: "nullable和__kindof的使用"
date: 2016-08-15 10:59:30 +0800
comments: true
categories: 
---
###nullable和__kindof的使用记录
2015年的WWDC里面介绍了几个新的关键字：`__kindof`和`nullable`。如果能很好的使用它们，编译器在我们犯错误的时候就能给予我们提示，将bug扼杀在摇篮里。
<!--more-->
#####__kindof
`__kindof`关键字的使用很简单，直接上一个示例代码就能说明白：
  
```  
NSMutableArray<UIView *> *subviews = [[NSMutableArray alloc] init];

[subviews addObject:[[UIView alloc] init]]; // Works
[subviews addObject:[[UIImageView alloc] init]]; // Also works

UIView *sameView = subviews[0]; // Works
UIImageView *sameImageView = subviews[1]; // Incompatible pointer types initializing 'UIImageView *' with an expression of type 'UIView *'

NSLog(@"%@", NSStringFromClass([sameView class])); // UIView
NSLog(@"%@", NSStringFromClass([sameImageView class])); // UIImageView
  
```  
  
上面的代码在编译的时候会给出警告，运行的时候不会crash，下面是使用`__kindof`的示例：  
  
```  
NSMutableArray<__kindof UIView *> *subviews = [[NSMutableArray alloc] init];

[subviews addObject:[[UIView alloc] init]]; // Works
[subviews addObject:[[UIImageView alloc] init]]; // Also works

UIView *sameView = subviews[0]; // No problem
UIImageView *sameImageView = subviews[1]; // No complaints now!

NSLog(@"%@", NSStringFromClass([sameView class])); // UIView
NSLog(@"%@", NSStringFromClass([sameImageView class])); // UIImageView
  
```  
  
可以看到在使用了`__kindof`关键字后，不在有编译警告了，所以对于`Objective-C`新提供的这个关键字来说，它可以限定返回值或者参数以及像我们例子中的这种泛型的类型，要么是这种类型本身的类型，要么是它的subClass，反之就会给出警告。  
  
#####nullable、_Nullable、__nullable
这些关键字可以限定参数或者返回值可以空。它们的作用相同，但是也有细微的使用差别。  
  
1. **_Nullable**是**Xcode7**以后用来替代**__nullable**的，所以能使用**__nullable**的地方就可以使用**_Nullable**。
2. **nullable**放在类型之前，**_Nullable**放在类型之后。  
  
以下的示例都是正确并且作用都是等价的：  
  
1、修饰返回值：  

```  
- (nullable NSNumber *)result
- (NSNumber * __nullable)result
- (NSNumber * _Nullable)result
  
```  
  
2、修饰参数：  
  
```  
- (void)doSomethingWithString:(nullable NSString *)str
- (void)doSomethingWithString:(NSString * _Nullable)str
- (void)doSomethingWithString:(NSString * __nullable)str
  
```  
  
3、修饰property：  
  
```  
@property(nullable) NSNumber *status
@property NSNumber *__nullable status
@property NSNumber * _Nullable status
  
```  
  
4、修饰指针的指针，不能使用*nullable*：  
  
```  
- (void)compute:(NSError *  _Nullable * _Nullable)error
- (void)compute:(NSError *  __nullable * _Null_unspecified)error
  
```  
  
5、修饰block：  
  
```  
- (void)executeWithCompletion:(nullable void (^)())handler
- (void)executeWithCompletion:(void (^ _Nullable)())handler
- (void)executeWithCompletion:(void (^ __nullable)())handler
  
```

6、结论：  
`__nullable、_Nullable、nullable`以及与它们对应的`__nonnull、_Nonnull、nonnull`一样，只需要注意它们相对于修饰类型的位置。