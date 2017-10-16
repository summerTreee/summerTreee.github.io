---
layout: post
title: "动画黄金搭档:CADisplayLink&amp;CAShapeLayer"
date: 2016-08-07 17:36:13 +0800
comments: true
categories: 
---
我们在开发中有时会遇到一些看似非常复杂的动画，不知该如何下手，今天的这篇文章中我会讲到如何利用`CADisplayLink`和`CAShapeLayer`来构建一些复杂的动画，希望能在你下次构建动画中，给你一些启发。    
  
在接下来的文章中，我们会构建如下的一个动画：  

![image](http://ww4.sinaimg.cn/mw690/8f7a6fe0gw1f6lb4ym7ovg207002z3ym.gif)  

该动画是在`du`的轮廓中进行，类似一个镂空效果，轮廓的填充是用双波浪的形式，类似于水流慢慢注入容器的过程。  
动画使用`CADisplayLink`来进行刷新，保证了动画的流程性，利用`CAShapeLayer`来构建波浪的轮廓，最后利用`CALayer`的`mask`属性来实现逐渐填充的过程。  
<!--more-->
####背景知识介绍
在动画创建过程的讲解之前，先介绍一下会使用到的一些知识点：      

1. CADisplayLink
2. UIBezierPath
3. CAShapeLayer
4. mask
  
如果你已经对这些概念有了充分的了解，可以略过`背景知识介绍`这一节。  
  
######1、CADisplayLink
用绘制的方式构建的动画，必然需要不断的刷新绘制的内容来呈现流畅的动画，`CADisplayLink`就像是一个定时器，每隔几毫秒刷新一次屏幕。能让我们以和屏幕刷新频率相同的频率去刷新我们绘制到屏幕上的内容。  
`CADisplayLink`的使用方式如下：  
  
```  
  _displayLink = [CADisplayLink displayLinkWithTarget:self
                                            selector:@selector(updateContent:)];
   [_displayLink addToRunLoop:[NSRunLoop currentRunLoop]
                      forMode:NSRunLoopCommonModes];
  
  
```    

当`CADisplayLink`注册到**runloop**以后，屏幕刷新的时候就会调用绑定到它上面的**target**所拥有的**selector**方法。停止`CADisplayLink`的运行非常的简单，只需要调用它的`invalidate`方法。    

**NSTimer和CADisplayLink有什么不同？**    
  
iOS设备的屏幕每秒会刷新60次，正常情况下`CADisplayLink`在屏幕每次刷新时都会调用，精确度非常高，并且`CADisplayLink`的使用场合相对专一，适合做UI的不停重绘，比如动画的连续绘制。  
  
`NSTimer`的使用范围要广泛很多，可以做单次或者循环处理某个任务，精度相比**CADisplayLink**要低。  
  
#####2、UIBezierPath  
使用`UIBezierPath`类可以创建基于矢量的路径，它是**Core Graphics**框架关于`CGPathRef`类型数据的封装，利用它创建直线或者曲线来构建我们想要的形状，每一个直线段或者曲线段的结束位置就是下一个线段开始的地方。这些连接的直线或者曲线的集合成为**subpath**。一个`UIBezierPath`对象的完整路径包括一个或者多个**subpath**。  
  
创建一个完整的**UIBezierPath**对象的完整步骤如下：  
  
* 创建一个Bezier Path对象。
* 使用方法**moveToPoint:**去设置初始线段的起点。
* 添加**line**或者**curve**去定义一个或者多个**subpath**。
* 修改**UIBezierPath**对象跟绘图相关的属性。

#####3、CAShapeLayer
`CAShapeLayer`是一个通过矢量图形而不是**bitmap**来绘制的图层子类。`CAShapeLayer`可以用来绘制所有通过**CGPath**来表示的形状，上面讲到了可以用`UIBezierPath`来创建任何你想要的路径，使用`CAShapeLayer`的属性`path`配合`UIBezierPath`创建的路径，就可以呈现出我们想要的形状。  
这个形状不一定要闭合，图层路径也不一定是连续不断的，你可以在`CAShapeLayer`的图层上绘制好几个不同的形状，但是你只有一次机会去设置它的`path、lineWith、lineCap`等属性，如果你想同时设置几个不同颜色的多个形状，你就需要为每个形状准备一个图层。  
  
下面的示例代码是通过`UIBezierPath`和`CAShapeLayer`来创建一个简单的火柴人。  
  
```  
  - (void)viewDidLoad {
    [super viewDidLoad];
    
    UIBezierPath *path = [[UIBezierPath alloc] init];
    [path moveToPoint:CGPointMake(175, 100)];
    
    [path addArcWithCenter:CGPointMake(150, 100) radius:25 startAngle:0 endAngle:2*M_PI clockwise:YES];
    [path moveToPoint:CGPointMake(150, 125)];
    [path addLineToPoint:CGPointMake(150, 175)];
    [path addLineToPoint:CGPointMake(125, 225)];
    [path moveToPoint:CGPointMake(150, 175)];
    [path addLineToPoint:CGPointMake(175, 225)];
    [path moveToPoint:CGPointMake(100, 150)];
    [path addLineToPoint:CGPointMake(200, 150)];
    
    //create shape layer
    CAShapeLayer *shapeLayer = [CAShapeLayer layer];
    shapeLayer.strokeColor = [UIColor colorWithRed:147/255.0 green:231/255.0 blue:182/255.0 alpha:1].CGColor;
    shapeLayer.fillColor = [UIColor clearColor].CGColor;
    shapeLayer.lineWidth = 5;
    shapeLayer.lineJoin = kCALineJoinRound;
    shapeLayer.lineCap = kCALineCapRound;
    shapeLayer.path = path.CGPath;
    //add it to our view
    [self.view.layer addSublayer:shapeLayer];
}  

```  
显示的形状如下：  
  
![image](http://ww1.sinaimg.cn/mw690/8f7a6fe0gw1f6lb4z3e4oj209y0aqglo.jpg)  
  
#####4、mask  
`CALayer`有一个属性叫做`mask`，通常被称为蒙版图层，这个属性本身也是**CALayer**类型，有和其他图层一样的绘制和布局属性。它类似于一个子视图，相对于父图层（即拥有该属性的图层）布局，但是它却不是一个普通的子视图。不同于一般的**subLayer**，`mask`定义了父图层的可见区域，简单点说就是最终父视图显示的形态是父视图自身和它的属性`mask`的交集部分。  

![image](http://ww3.sinaimg.cn/mw690/8f7a6fe0gw1f6lb50rgqbj20go06mwet.jpg)  
  
`mask`图层的**color**属性是无关紧要的，真正重要的是它的轮廓，`mask`属性就像一个切割机，父视图被`mask`切割，相交的部分会留下，其他的部分则被丢弃。  
**CALayer**的蒙版图层真正厉害的地方在于蒙版图层不局限于静态图，任何有图层构成的都可以作为`mask`属性，这意味着蒙版可以通过代码甚至是动画实时生成。这也为我们实现示例中波浪的变化提供了支持。  
  
####绘制波浪轮廓
我们利用`UIBezierPath`来绘制波浪的轮廓，通过正弦函数和余弦函数来创建顶部的波浪曲线，在单位为1的右手直角坐标系中的曲线变化如下：  
  
![image](http://ww2.sinaimg.cn/mw690/8f7a6fe0gw1f6lb51vscuj20v007sq4f.jpg)    

可以看到在**(-2π , 2π )**的范围类，`y`值在`[-1, 1]`之间变化。  
以正弦曲线为例，它可以表示为`y=Asin(ωx+φ)+k`，公式中各符号表示的含义：  
  
* **A**--振幅，即波峰的高度。
* **(ωx+φ)**--相位，反应了变量*y*所处的位置。
* **φ**--初相，*x=0*时的相位，反映在坐标系上则为图像的左右移动。
* **k**--偏距，反映在坐标系上则为图像的上移或下移。
* **ω**--角速度，控制正弦周期(单位角度内震动的次数)。


通过上面的函数，我们就能计算出波浪曲线上任意位置的坐标点。通过`UIBezierPath`的函数`addLineToPoint`来把这些点连接起来，就构建了波浪形状的**path**。只要我们的设定相邻两点的间距够小，就能构建出平滑的正弦曲线，构建正弦波浪的代码如下：  
  
```  
  - (UIBezierPath *)createSinPath
{
    UIBezierPath *wavePath = [UIBezierPath bezierPath];
    CGFloat endX = 0;
    for (CGFloat x = 0; x < self.waveWidth + 1; x += 1) {
        endX=x;
        CGFloat y = self.maxAmplitude * sinf(360.0 / _waveWidth * (x  * M_PI / 180) * self.frequency + self.phase * M_PI/ 180) + self.maxAmplitude;
        if (x == 0) {
            [wavePath moveToPoint:CGPointMake(x, y)];
        } else {
            [wavePath addLineToPoint:CGPointMake(x, y)];
        }
    }
    
    CGFloat endY = CGRectGetHeight(self.bounds) + 10;
    [wavePath addLineToPoint:CGPointMake(endX, endY)];
    [wavePath addLineToPoint:CGPointMake(0, endY)];
    
    return wavePath;
}
  
```  
  
显示的效果如下：  
  
![image](http://ww1.sinaimg.cn/mw690/8f7a6fe0gw1f6lb53gapqj20jg06ejrc.jpg)  
  
在这里我们设定了两个正弦曲线上的点的横坐标间距是**1**，现在来解释一下通过横坐标**x**来得出**y**的计算过程：
  
  	y = self.maxAmplitude * sinf(360.0 / _waveWidth * (x  * M_PI / 180) * self.frequency + self.phase * M_PI/ 180) + self.maxAmplitude;  
  
第一个`self.maxAmplitude`表示曲线的波峰值，`360.0 / _waveWidth`计算出单位间距**1pixel**代表的度数，`x  * M_PI / 180`表示将横坐标值转换为角度。`self.frequency`表示角速度，即单位面积内波动次数，波浪的大小。`self.phase * M_PI/ 180`代表上面公式中的初相，通过规律的变化初相，可以制造出曲线上的点动起来的效果，`self.maxAmplitude`代表偏距，由于我们需要让波浪曲线的波峰在`layer`的范围内显示，所以需要将整个曲线向下移动波峰大小的单位，因为`CALayer`使用左手坐标系，所以向下移动需要加上波峰的大小。  
  
####让波浪曲线动起来
正弦或者余弦曲线上的点，不论角度如何，在**y**轴上的变化范围在它的波峰与波谷之间。拿单位正交直角坐标系来说，只要我们规律性的增加角度值，那么曲线上的点就会在**[1, -1]**之间变化，我们以曲线上**x=0**的点为例，角度的不断增加，会让它的**y**值规律性的来回变化：  
  
![image](http://ww3.sinaimg.cn/mw690/8f7a6fe0gw1f6lb548i4ig205304idft.gif)  
  
如若曲线上的点都能这样规律的变化，我们就能让波浪曲线浪起来。  
要让曲线上所有的点都动起来，在这里我们使用`CADisplayLink`来不断刷新由`UIBezierPath`创建的形状，两次刷新之间曲线的变化通过增加初相来实现，代码如下：

```  
- (void)startLoading
{
    [_displayLink invalidate];
    self.displayLink = [CADisplayLink displayLinkWithTarget:self
                                                   selector:@selector(updateWave:)];
    [_displayLink addToRunLoop:[NSRunLoop currentRunLoop]
                       forMode:NSRunLoopCommonModes];
}

- (void)updateWave:(CADisplayLink *)displayLink
{
    self.phase += 8;//逐渐累加初相
    self.waveSinLayer.path = [self createSinPath].CGPath;
}
  
- (UIBezierPath *)createSinPath
{
    UIBezierPath *wavePath = [UIBezierPath bezierPath];
    CGFloat endX = 0;
    for (CGFloat x = 0; x < self.waveWidth + 1; x += 1) {
        endX=x;
        CGFloat y = self.maxAmplitude * sinf(360.0 / _waveWidth * (x  * M_PI / 180) * self.frequency + self.phase * M_PI/ 180) + self.maxAmplitude;
        if (x == 0) {
            [wavePath moveToPoint:CGPointMake(x, y)];
        } else {
            [wavePath addLineToPoint:CGPointMake(x, y)];
        }
    }
    
    CGFloat endY = CGRectGetHeight(self.bounds) + 10;
    [wavePath addLineToPoint:CGPointMake(endX, endY)];
    [wavePath addLineToPoint:CGPointMake(0, endY)];
    
    return wavePath;
} 
  
  
```    

把`CAShapeLayer`的背景色设置为淡红色，波浪曲线会在`Layer`的**bounds**类波动，动起来的波浪曲线如下：  
  
![image](http://ww4.sinaimg.cn/mw690/8f7a6fe0gw1f6lb55j45eg208r021dg0.gif)     
  
####构建波浪上升的镂空效果  
到目前为止，我们利用`CAShapeLayer`、`UIBezierPath`以及`CADisplayLink`实现了动起来的波浪效果，下面我们需要实现的是在`du`的轮廓里，水波不断上升填充的过程，整个动画过程中，会呈现一个`du`的镂空效果，在它轮廓之外的水波则不会显示，这样的效果需要借助`CALayer`的`mask`属性来实现。  
  
我们需要的动画素材如下：  
  
![image](http://ww1.sinaimg.cn/mw690/8f7a6fe0gw1f6lb55xrpgj20bm04iaa5.jpg)  
  
将这三个图片位置设置为一样，底层放置动画中一直显示的底层轮廓，中间层用以实现余弦波浪，最外层用于实现正弦波浪，将中间层和最外层图片的`mask`属性赋值为我们创建的用来呈现余弦波浪和正弦波浪的`CAShapeLayer`，这样动画开始后，利用`CADisplayLink`来不断刷新两个`CAShapeLayer`的**path**来让波浪浪起来，再利用`CABasicAnimation`来对两个`CAShapeLayer`的`position`进行动画，实现从下到上的填充效果。我们想要的效果就完成了：

![image](http://ww4.sinaimg.cn/mw690/8f7a6fe0gw1f6lb4ym7ovg207002z3ym.gif)
  
完整的代码示例在[这里](https://github.com/liangwei518/WaterWave)  
  
####参考  
* [UIBezierPath](http://blog.csdn.net/crayondeng/article/details/11093689)
* [CAShapeLayer](http://www.cnblogs.com/YouXianMing/p/3678709.html)
* [图层蒙版](https://zsisme.gitbooks.io/ios-/content/chapter4/layer-masking.html)
