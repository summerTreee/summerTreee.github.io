---
layout: post
title: "Objective-C库文件使用小记"
date: 2017-01-22 22:36:42 +0800
comments: true
categories: 
---
  
#####静态库 VS 动态库

- 静态库：静态库在Objective-C里面以`.a`或者`.framework`作为后缀，目前开发者自己创建的库文件（Framework）其实都是以静态库的形式链接到执行文件的。链接时完整的拷贝到了可执行文件中，被多次使用就会有多份拷贝(eg：iOS8+的Extention中使用)。静态库文件一般都会比较大，因为所有要使用的数据都会被编译进去，而且如果库文件的某个函数改变了，那么就又需要重新编译新的库文件了，优点就是编译后的执行程序不需要外部的函数库支持，因为所有的函数都已经被编译进去了。
- 动态库：动态库在Objective-C里面以`dylib`或者`.framework`最为后缀，系统为我们提供的framework就是动态库，目前开发者是不允许使用动态库的，因为我们自己创建的库文件虽然`buildSetting`中的`Mach-O Type`设置为`Dynamic Library`，但是使用时直接链接到程序里面的，而不是放在服务器上进行更新，开发者如果使用动态库放在服务器上，然后动态的加载`dlopen`是不会通过审核的，不然Apple的审核就没有意义了。动态库在链接时不复制，程序运行时由系统动态加载到内存，系统只加载一次，多个程序间共用，节省内存，而且升级方便。
<!--more-->
我们创建framework库文件时，系统默认是动态库的格式，如果想做成静态库，需要在`buildSetting`中将`Mach-O Type`选项设置为`Static Library`就行了！

![image1](http://wx4.sinaimg.cn/mw690/8f7a6fe0gy1fbzs4ohq7gj20he048t94.jpg)

#####framework VS .a 
- .a：.a是纯二进制文件，不能直接拿来使用，需要配合头文件、资源文件一起使用。代码资源、图片、json资源、xib文件等是无法打包进去的，所以使用`.a`静态库的时候需要三个组成部分：.a文件+开放的头文件+资源文件。
- .framework：相当于一个文件夹，可以直接拿来使用，所需要的资源、头文件、源文件都在里面。

![image2](http://wx3.sinaimg.cn/mw690/8f7a6fe0gy1fbzs4nzfa3j20go03o74v.jpg)

#####Static Library/Framework VS Embedded Framework
`Embedded Framework`是**iOS8**引入的为了方便`Extention`和宿主APP公用一份代码库而引入的，`Embedded Framework`必须是`Dynamic framework`(在buildSeting中设置为Dynamic)。如果你想限制在`Extention`中不可用的API放入你的`Embedded Framework`，你可以勾选`Allow app extension API only`选框。

![image3](http://wx3.sinaimg.cn/mw690/8f7a6fe0gy1fbzs4nfpb9j20gm07oaao.jpg)

#####-framework的妙用
有些静态库文件我们只是在`DEBUG`模式下，调试使用。而不想打入`release`包中，因为这样会增加安装包的大小，这时可以在`buildSetting`中的`Other Linker Flags`下对应的模式中添加需要的库文件，以`-framework`标记，这样程序编译的时候就会根据里面的标记来编译进执行文件中。

![image4](http://wx3.sinaimg.cn/mw690/8f7a6fe0gy1fbzs4mwnfhj20kx08qabd.jpg)

在使用时可以利用`runtime`的反射来判断库文件有没有被加载，或者利用`buildSetting`中的预编译宏`Preprocessor Macros`来标记。

```
    if (NSClassFromString(@"MyFramework") != Nil) {
        //MyFramework被加载了
    }
    //or
#if DEBUG
    /*该模式下可使用DEBUG模式下‘Other Linker Flags’
     通过‘-framework’引入的静态库*/
#endif

```

######参考文章
* <https://hacking-ios.cocoagarage.com/working-with-a-static-library-framework-vs-embedded-framework-9ca7cd77b4f9#.ign0m484w>
* <http://www.knowstack.com/framework-vs-library-cocoa-ios/>
* <http://stackoverflow.com/questions/27015154/link-binary-with-libraries-vs-embed-frameworks>