---
layout: post
title: "我所理解的Runtime"
date: 2016-05-22 14:46:14 +0800
comments: true
categories: 
---
Objective-C是一门基于对象的动态语言，它里面所有的继承自`NSObject`的类本身以及类所实例化的对象都是对象，好像有点儿绕，大意就是这是一门基于对象的语言，就连类自己也是一个对象，程序运行起来以后类本身也会被初始化，也占有内存空间。  
在这篇笔记里，我会记录如下的一些信息：    

- 消息传递
- runtime如何处理对象
- self 和 super
- Method Swizzling
- 消息转发
    
<!--more-->
####消息传递
Objective-C是一门动态的语言，当我们在某个对象上调用方法是，`Objective—C`里面叫做`消息传递`，和`C语言`的函数调用在本质上就不一样，`C语言`使用`静态绑定`，在编译器就能决定运行时应该调用的函数：  
  
```
//代码一
void printHello() {
    printf("Hello, world!\n");
}

void printGoodbye() {
    printf("Goodbye, world!\n");
}

void doSomeThing(int type) {
    if (type == 0) {
        printHello();
    } else {
        printGoodbye();
    }
}  
  
```  

以上的代码在编译器编译代码的时候就已经知道程序中有 **printHello** 和 **printGoodbye** 这两个函数了，于是就直接生成调用这些函数的指令。而函数地址实际上是硬编码在指令之中的。但是，对于`Objective—C`来说，如果像下面这样写，情况会完全不一样：    
  
``` 
  //代码二 
  void printHello() {
    printf("Hello, world!\n");
}

void printGoodbye() {
    printf("Goodbye, world!\n");
}

void doSomeThing(int type) {
    void (*funcation)();
    if (type == 0) {
        funcation = printHello;
    } else {
        funcation = printGoodbye;
    }
    
   funcation();
}
  
```  

这时就需要使用`动态绑定`了，因为实际调用的方法在运行期根据**type**的值才能确定，在**代码一**中，**if**和**else**里面都有函数调用指令，而在`代码二`中，只有一个函数调用指令，而且待调用的函数地址无法硬编码到指令中，而是需要在运行期间读取出来。  
在底层，所有的方法都是普通的`C语言`函数，当对象接收到消息之后，究竟该调用哪个方法完全是运行期决定的，甚至可以在运行期改变以及交换某些函数的实现，这就是`runtime`所体现出来的异于其他编程语言的特点。  
当给某个对象传递一个消息：  
  
	id returnValue = [someObject messageName:parameter];  
  
在这个消息中，`someObject`叫接收者(receiver)，`messageName`叫做选择子(selector)，选择子和参数结合起来叫`消息`(message)。当编译器看到此消息后，会将其转换为标准的`C语言`函数调用，这是就引出了我们消息传递中最核心的一个函数`objc_msgSend`，其原型如下：  
  
	id objc_msgSend(id self, SEL op, ...)  
它是一个参数可变的函数，可以接受两个及两个以上的参数。有了这个函数，编译器会把刚才的消息转换为如下的函数：  
  
	id returnValue = objc_msgSend(someObjetc,
                                  @selector(messageName:),
                                  parameter);  
  
到了运行期间，为了完成此操作，该方法需要到接受者所属的类中搜索其**方法列表**(后续会有讲到)，如果能找到与**选择子**名称相符的方法，就跳至其实现代码，如果找不到则按照其继承关系向上查找，如果到最终(NSObject)还是没能找到匹配的方法，那就执行`消息转发`(后续会讲到)。  
    
除了刚才讲到的`objc_msgSend`外，还有其他的几种处理`边界情况`的函数：  
  
    //处理待发送的消息要返回结构体
	objc_msgSend_stret(id self, SEL op, ...)
	
	//处理待发送的消息返回值是浮点数
	objc_msgSend_fpret(id self, SEL op, ...) 
	
	//处理将消息发送给超类的情况 
	objc_msgSendSuper(struct objc_super *super, SEL op, ...)
  
上面讲到了，`objc_msgSend`函数会根据**选择子**去查找应该调用的方法实现，**选择子**可以理解为一个标记，它其实就是`char *`字符串，选择字和方法的实现`IMP`之间是一对一的关系，`runtime`里面通过结构体`objc_method`来存储它们：  
    
	typedef struct objc_method *Method;
	struct objc_method {
    	SEL method_name     OBJC2_UNAVAILABLE;
    	char *method_types  OBJC2_UNAVAILABLE;
    	IMP method_imp      OBJC2_UNAVAILABLE;
	}  
  
####runtime如何处理对象
`Objective-C`是一门基于对象的动态语言，它的动态性是建立在对象上面的，从`runtime` 源码我们可以看到它是由`C语言`编写，这可以保证它执行的高效性，编译器在编译期会将对象转换为它里面定义的结构体：    

	struct objc_object {
    	Class isa  OBJC_ISA_AVAILABILITY;
	};
由此可见，每个对象结构体的首个成员变量是`Class`类的变量，通过它可以知道对象所属的类，通过`isa`指针可以访问到。  
比如说我们定义了如下一个类`MyClass`：  
  
	@interface MyClass : NSObject {
    	NSString *name;
    	NSUInteger age;
	}
  
编译器编译`MyClass`类以后，它的实例变量布局如下：  
  
```  
  struct MyClass {
    struct objc_class {
        Class isa;
    }
    NSString *name;
    NSUInteger age;
}
  
```  
  
精简一下，去除`struct`的壳： 
   
```  
  struct MyClass {
    Class isa;
    NSString *name;
    NSUInteger age;
}

```  

`isa`变量的类型是`Class`，它的定义在`runtime`程序库里可以找到，它的定义如下：  
  
	typedef struct objc_class *Class;  
  
所以由此我们知道`isa`本身就是一个`objc_class`类型的指针，同样我们可以在`runtime`程序库里面找到`objc_class`的定义：  
  
```  
  struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
  
```
  
这个结构体存放了类的所有`元数据`，例如定义了哪些实例变量，定义了哪些方法，超类等。此结构体的首个变量也是`isa`指针，说明`Class`本身也是一个`Objective-C`对象，`Class`里面的`isa`指向的类就是`元类`，用它来定义类对象本身的`元数据`。也就是说实例对象的`isa`指针指向它唯一的类对象，而类对象的`isa`指针指向它唯一的元类。我们知道，类和实例对象都可以发送`class`消息，从`runtime`的源码中有这样的定义：  
  
```  
  - class
  {
      return (id)isa;
  }
  
  + class
  {
     return self;
  }
  
``` 

由上可以看出实例对象的class就是`isa`指向的class对象，而实例对象的类的class就是自己，这也印证了上面的说法。  
  
在`Objective-C`中，对象的实例方法存放在它的`isa`指向的类对象中，类方法存放于类对象的`isa`指向的元类中，某个类在应用程序范围内，类对象以及它所属的元类只有一个，它们都是`单例`。
  
![image](http://ww2.sinaimg.cn/mw690/8f7a6fe0gw1f445qvvtffj21b00o6e81.jpg)  
  
当我们向某个类的实例化对象发送消息时，实例对象通过它的`isa`指针找到它所属的类对象，再去类对象的方法列表里面根据`选择子`查找，如果没有找到就去它父类的类对象里面查找。如果是向类对象调用类方法，类对象先通过它的`isa`指针找到它的`元类`，在元类的方法列表里面根据`选择子`查找需要调用的方法，如果找到了则跳转到方法实现，若没有找到则向它的父类的元类里面查找。  
  
实例对象的类以及元类的关系如下图(图片[来源](www.sealiesoftware.com/blog/archive/2009/04/14/objc_explain_Classes_and_metaclasses.html))：  
  
![image](http://ww4.sinaimg.cn/mw690/8f7a6fe0gw1f445qwqwm2j20tg0u4gs1.jpg)  
  
这里需要说明的是：图中类对象**Root class**，也就是`NSObject`，它的父类是`nil`，它的`元类`是`NSObject元类`，而`NSObject元类`的元类指向自己，父类指向`NSObject`类对象，这样刚好形成一个环，可能是苹果设计的需要，因为`NSObject`里面有很多定义好的行为，比如内存管理，空间开辟等，让所有的类对象都收敛于`NSObject`，便于统一管理。  
  
好了，说了这么多，下面来做个简单的练习，热热身😃，首先贴出`object_getClass`方法的实现：
  
```  
  Class object_getClass(id obj)
  {
     if(obj)  return obj->isa;
     else return Nil;
  }
  
```  

也就是说这个方法会返回传入对象的`isa`指向。
  
以下是输出每行对象的内存地址：  
  
```  
  Class class1 = [NSObject class];//0x90620c ①
  id obj1 = [NSObject valueForKey:@"isa"];//0x906220 ②
  id obj2 = [obj1 valueForKey:@"isa"];//0x906220  ③
  NSObject *object = [NSObject new];
  Class class2 = objc_getMetaClass("NSObject");//0x906220 ④
  Class class3 = object_getClass(object);//0x90620c ⑤
  Class class4 = class_getSuperclass(class1);//0x0 ⑥
  
```  
  
①⑤的指向相同，②③④指向相同，⑥指向nil。  
① 对`NSObject`调用`class`方法返回`NSObject`类对象本身。  
② 获取`NSObject`类对象的`isa`变量，它指向`NSObject`类对象的元类。  
③ 获取`NSObject`元类的`isa`变量，它和②的指向一样，说明**NSObject元类**的**isa**指向了**NSObject类对象**自身，和上图中的指向一致。  
④ 获取**NSObject类对象**的元类。  
⑤ 获取**NSObject**实例对象的`isa`指向，其实就是**NSObject类对象**，所以和①的值相同。  
⑥ 获取**NSObject类对象**的父类，打印**nil**，和上图中标明的一致。    
  
再来一个例子：  
  
```  
  @interface NSObject (Mycategory)
	+ (void)foo;
  @end
  
  @implementation NSObject (Mycategory)
  - (void)foo
  {
   	 NSLog(@"%s",__PRETTY_FUNCTION__);
  }
  
@end
  
```  
  
如果执行下面的代码会输出什么呢？ 😖   
  
```  
[NSObject foo]; //①
[[NSObject new] performSelector:@selector(foo)];  //②
  
```  

①:对`NSObject`发送`foo`消息，会到**NSObject类对象**的元类中查找，理所当然没能找到，然后去到**NSObject类对象元类**的父类里面查找，它的父类是`NSObject`，然后在`NSObject类对象`的方法列表里面查找，这时找到了`- (void)foo`方法，然后跳到函数入口处执行，输出：  

```  
-[NSObject(Mycategory) foo]    
```
    
②:首先初始化一个**NSObject**实例对象，然后发送`foo`消息，这是会到`NSObject类对象`里面查找，这是找到了`- (void)foo`方法，然后跳到函数入口处执行，输出：  
  
```  
 -[NSObject(Mycategory) foo]  
 
```  

####self和super
- 在实例方法中`self`代表着`对象`本身。  
- 在类方法中，`self`代表当前`类`。  

万变不离其宗，记住一句话就行了：`self`代表着当前方法的调用者。  
上面讲到了，在`Objective-C`中对`super`发送消息时，编译后会被转换为：  
  
```  
 objc_msgSendSuper(struct objc_super *super, SEL op, ...)
  
```  
  
接下来我们看看`super`指针的类型`objc_super`：  
  
```  
  /// Specifies the superclass of an instance. 
struct objc_super {
    /// Specifies an instance of a class.
    __unsafe_unretained id receiver;

    /// Specifies the particular superclass of the instance to message. 
#if !defined(__cplusplus)  &&  !__OBJC2__
    /* For compatibility with old objc-runtime.h header */
    __unsafe_unretained Class class;
#else
    __unsafe_unretained Class super_class;
#endif
    /* super_class is the first class to search */
};
  
```  

通过上面的结构我们可以看出：对`super`发送消息，`receiver`还是`self`，唯一的区别是查找方法时不是从本类开始，而是从`super_class`开始。下面我们来看看一个测试：  
  
```  
@interface Father : NSObject

@end

@implementation Father

@end

@interface Son : Father

@end

@implementation Son

- (instancetype)init
{
    self = [super init];
    if (self) {
        NSLog(@"%@",NSStringFromClass([self class]));
        NSLog(@"%@",NSStringFromClass([super class]));
    }
    
    return self;
}

@end
  
```

运行结果都是输出：`Son`。由于`class`在基类`NSObject`里面有实现，所以两个输出都是输出调用者自己所属的类，这里都输出了`Son`，而没有输出`Father`，所以说`super`只是一个标记符，标记消息的查找是从`superClass`开始，其他和`self`没有任何区别。  
  
####Method Swizzling
我们有时候为了调试方便或者想对系统的方法进行自定义的改造，这是我们可以利用`Method Swizzling`来进行方法实现的交换。  
对方法的交换我们可以在`load`或者`initialize`里面进行，但是需要根据自己的需要进行选择，它们的区别如下：  
  
![image](http://ww3.sinaimg.cn/mw690/8f7a6fe0gw1f445qxsptrj219m0oo1kk.jpg)  
  
方法的交换就是更改选择子(selector)所对应的`IMP`，如下：  
  
![image](http://ww2.sinaimg.cn/mw690/8f7a6fe0gw1f445qynrpmj20p40fsapj.jpg)  
  
下面贴上一段`Method Swizzling`(方法调配)的示例：  
  
```  
  - (void)load
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class currentClass = [self class];
        SEL originSelector = @selector(foo);
        SEL swizzedSelector = @selector(st_foo);
        
        Method originMethod = class_getInstanceMethod(currentClass, originMethod);
        Method swizzedMethod = class_getInstanceMethod(currentClass, swizzedSelector);
        if (class_addMethod(currentClass, originSelector, method_getImplementation(swizzedMethod), method_getTypeEncoding(swizzedMethod))) {
            class_addMethod(currentClass, swizzedSelector, method_getImplementation(originMethod), method_getTypeEncoding(originMethod));
        } else {
            method_exchangeImplementations(originMethod, swizzedMethod);
        }
    });
}
  
```  
  
这里需要说明的是`if`条件里面的判断，如果为真则表示`currentClass`没有`originSelector`的方法，然后进行该方法的添加，如果为假则表示`currentClass`里面已经有了`originSelector`方法。这里这么做的原因是我们只想对当前类进行方法调配，而不想对父类造成影响。比如当前类的父类里面有一个方法，当前类没有进行覆盖，当我们在当前类里进行`方法调配`对这个方法的实现进行重新定义，如果直接进行`method_exchangeImplementations`则交换的是父类的方法。结果会与我们预想的不符。  
  
####消息转发
前面讲了`Objective-C`里面的`消息传递`机制，当我们向一个对象传递一个消息，而当程序运行中无法找到该消息对应的实现，这是就会启动`消息转发`(message forwarding)机制，这个时候我们可以进行一些操作来防止程序crash。  
我们经常遇到程序异常，控制台输出：  
  
```  
   *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[ViewController TestMessage]: unrecognized selector sent to instance 0x7fd41075b6a0'
  
```
  
这就是向对象传递了无法识别的消息报的异常信息。我们在开发中，可以在进入`消息转发`后执行预定的逻辑，从而避免奔溃。  
  
`消息转发`大致分为三个步奏：  
######1、动态方法解析
对象在接收到无法识别的消息后，会首先调用所属类的下列方法：  
  
```  
  + (BOOL)resolveClassMethod:(SEL)sel;//对于类方法
  + (BOOL)resolveInstanceMethod:(SEL)sel;//对于实例方法
  
```  
  
参数就是无法识别的`选择子`，这时我们可以在该方法里面为该选择子添加一个实现，返回`YES`，系统会对该方法再进行一次传递，这时就能正常运行了，但它的前提是添加的实现必须是已经定义好的。如果返回`NO`，则会就行下一步(下面会说到)，这里贴出简单的示例代码：  
  
```  
+ (BOOL)resolveInstanceMethod:(SEL)sel
{
    NSString *selectorString = NSStringFromSelector(sel);
    if ([selectorString isEqualToString:@"foo"]) {
        Method myMethod = class_getInstanceMethod(self, @selector(myFoo));
        class_addMethod(self, sel, method_getImplementation(myMethod), method_getTypeEncoding(myMethod));
        return YES;
    }
    
    return NO;
}

  
```  
  
######2、备援接受者
如果步骤一未能处理，运行期系统则会看看能不能把消息转发到其他对象，因为当前类肯定`组合`了一些其他的对象，这时调用：  
  
```  
 - (id)forwardingTargetForSelector:(SEL)aSelector; 
  
```  
  
若当前对象能找到合适的消息接受者，则将该接受者返回，若找不到则返回nil，消息转发会进行下一步，这里需要注意的是，未知消息到了这里只能将消息的转发给其他对象，无法修改消息的内容。  
  
######3、完整的消息转发
如果前两步都没能将未知消息进行有效的处理，则会进入`完整的消息转发`，首先需要重写`methodSignatureForSelector:`方法来获取未知消息的参数和返回值类型，如果这个方法里返回`nil`则消息转发结束，系统发出`doesNotRecognizeSelector:`消息，然后就挂掉了，如果返回一个签名函数，`runtime`就会创建一个`NSInvocation`对象，把未知消息相关的全部细节封装在里面：选择子、target、参数。然后发送`-forwardInvocation:`消息。  
  
```  
  - (NSMethodSignature *) methodSignatureForSelector : (SEL)selector
  {  
    NSMethodSignature *signature = [super methodSignatureForSelector:selector];  
      
    if(nil == signature){  
        signature = [self.otherObject methodSignatureForSelector:selector];  
    }  
    
    return signature;  
} 
  
  - (void)forwardInvocation:(NSInvocation *)invocation {  
    SEL selector = [invocation selector];  
      
    if([self.otherObject respondsToSelector:selector]){  
        [invocation invokeWithTarget:self.otherObject];  
    }  
} 
  
```  
  
综合起来如下图所示：  
  
![image](http://ww3.sinaimg.cn/mw690/8f7a6fe0gw1f445r01c5qj21f80uk4qq.jpg)  
  
######参考
[runtime源码](http://opensource.apple.com//source/objc4/)  
http://tech.glowing.com/cn/objective-c-runtime/