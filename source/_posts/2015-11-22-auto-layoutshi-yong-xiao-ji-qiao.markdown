---
layout: post
title: "Auto Layout使用小技巧"
date: 2015-11-22 22:58:51 +0800
comments: true
categories: 
---
目前由于Apple自己的Auto Layout写法比较啰嗦，所以出现了许多对原生语句进行封装的第三方开发库，这其中`Masonry`广受开发者的喜爱，所以以下都以`Masonry`来做演示说明，但对于Apple原生的写法也同样适用。
### 1、图片+文字居中显示
很多时候我们都会遇到这样的需求：一张图片旁边接上一段文字，然后让他们相对于父视图居中显示，以前我的做法是先知道图片的尺寸，然后来计算他们相对于父视图中心的偏移量，再进行布局。需求的样式大致如下：    

![大致效果](http://ww3.sinaimg.cn/mw690/8f7a6fe0jw1eya4nyp8qbj208w014a9v.jpg)  
  
但是这样的缺点是每次都需要去据算距离中心的偏移量，很麻烦，对于这种情况其实可以添加一个辅助视图，让这个辅助视图的左边等于图片的左边，右边和文字的右边对齐，然后这个辅助视图相对于父视图居中显示就行。
	  
<!--more-->
	- (void)layoutImageAndText {
    UIImageView *imageView = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"av_colum"]];
    [self.view addSubview:imageView];
    [imageView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.top.equalTo(@10);
        make.left.greaterThanOrEqualTo(self.view);
    }];
    
    UILabel *label = [UILabel new];
    label.text = @"梁朝伟";
    label.font = [UIFont systemFontOfSize:17];
    label.textAlignment = NSTextAlignmentLeft;
    label.backgroundColor = [UIColor clearColor];
    [self.view addSubview:label];
    [label mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(imageView.mas_right).offset(5);
        make.centerY.equalTo(imageView);
    }];
    
    UIView *limitView = [UIView new];
    [self.view addSubview:limitView];
    [limitView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(imageView);//和图片左边对齐
        make.right.equalTo(label);//和文字右边对齐
        make.centerX.equalTo(self.view);//指定limitView居中显示
    }];
	}

### 2、处理复合型布局
直接上图说明需求，页面上有四个可见的视图：红、绿、蓝三个`UIView`，最下面是一个`UIButton`,想要达到的效果是点击button触发事件将中间的绿色视图隐藏，下面的蓝色视图和button移动到红色视图下，我们想要的效果如下：

![需要的效果](http://ww3.sinaimg.cn/mw690/8f7a6fe0jw1eya4nz1i8hg208p0bjmxi.gif)

如果只是将中间的绿色的视图hidden掉，是达不到这样的效果的：

![影藏中间的绿色视图](http://ww1.sinaimg.cn/mw690/8f7a6fe0jw1eya4nza0mfj208w0a8748.jpg)

要达到这样的效果可以利用`MASConstraint`的`deactivate`和`activate`方法，他们的作用是让一个constraint**不生效**和**生效**，`Masonry`的每个Constraint都会产生一个`MASConstraint`，所以我们可以保存一些需要改变的constraint，在需要的时候使用这两个方法。
   
    //.h
	@interface ViewController ()

	@property (nonatomic, strong) UIView *greenView;
	@property (nonatomic, strong) MASConstraint *gHeightConstraint;//保存关键的constraint

	@end
	//.m
	@implementation ViewController

	- (void)testComplexLayout {
    NSNumber *viewHeight = @80;
    UIView *redView = [UIView new];
    redView.backgroundColor = [UIColor redColor];
    [self.view addSubview:redView];
    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.top.left.right.equalTo(self.view);
        make.height.equalTo(viewHeight);
    }];
    
    _greenView = [UIView new];
    [self.view addSubview:_greenView];
    [_greenView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.left.right.equalTo(redView);
        make.top.equalTo(redView.mas_bottom);
        self.gHeightConstraint = make.height.equalTo(@0).priorityHigh();//设为高优先级，首先满足它的需求
    }];
    [self.gHeightConstraint deactivate];//最开始让它不生效，就是高为0的限制暂不生效
    _greenView.clipsToBounds = YES;//不显示超过bounds本身的部分，因为在它里面加了subView
    //subGreenView是真实的显示绿色部分，它添加在greenView上
    UIView *subGreenView = [UIView new];
    subGreenView.backgroundColor = [UIColor greenColor];
    [_greenView addSubview:subGreenView];
    [subGreenView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(self.greenView).priorityLow();
        make.height.equalTo(viewHeight);
    }];
    
    UIView *blueView = [UIView new];
    blueView.backgroundColor = [UIColor blueColor];
    [self.view addSubview:blueView];
    [blueView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.height.left.right.equalTo(redView);
        make.top.equalTo(self.greenView.mas_bottom);
    }];
    
    UIButton *hiddenButton = [UIButton buttonWithType:UIButtonTypeCustom];
    [hiddenButton setTitle:@"影藏中间的View" forState:UIControlStateNormal];
    [hiddenButton addTarget:self action:@selector(hiddenAction:) forControlEvents:UIControlEventTouchUpInside];
    [hiddenButton setTitleColor:[UIColor blackColor] forState:UIControlStateNormal];
    [self.view addSubview:hiddenButton];
    [hiddenButton mas_makeConstraints:^(MASConstraintMaker *make) {
        make.centerX.equalTo(self.view);
        make.top.equalTo(blueView.mas_bottom).offset(40);
    }];
	}

	static bool isActive = NO;
	- (void)hiddenAction:(id)sender {
    if (isActive) {
        [_gHeightConstraint deactivate];
    } else {
        [_gHeightConstraint activate];
    }
    //动画展示
    [UIView animateWithDuration:.25 animations:^{
        [self.view layoutIfNeeded];
    }];
    
    isActive = !isActive;
	}
	  
### 3、Preview的使用
这段时间在学习storyBoard相关的一些知识，RW上的两篇文章：[Part1](http://www.raywenderlich.com/50308/storyboards-tutorial-in-ios-7-part-1)和[Part2](http://www.raywenderlich.com/50310/storyboards-tutorial-in-ios-7-part-2)非常适合初学者学习，已经有人翻译成了中文[Part1](http://www.cocoachina.com/industry/20131213/7537.html)，[Part2](http://blog.sina.com.cn/s/blog_5c5c87d80101dzyh.html)。

`Xcode`提供了一个很重要的功能`Preview`。当我们在`Storyboard`上使用`Auto Layout`上进行布局以后，由于目前的画布`canvas`已经没有尺寸的概念，我们布局完成后需要在3.5、4.0、4.7、5.5英寸的设备上测试，如果支持旋转或者支持iPad，一个效果我们会build多次在不同的设备上查看效果。非常的麻烦，但是有了`Preview`后一切都变得简单起来。
使用`Preview`的步奏如下：  

1、选中`Main.storyboard`,如下图所示，选取的时候按下`option+shift`按键。

![选择storyboard](http://ww2.sinaimg.cn/mw690/8f7a6fe0jw1eya4nzk6ywj20b204p74o.jpg)

2、选取`Preview`后会出来如下的界面，选择界面右边的`+`号，按`回车`。

![image](http://ww1.sinaimg.cn/mw690/8f7a6fe0jw1eya4nzzeuxj20bc05daae.jpg)

3、通过步骤`2`后，会出现以下的界面：  

![Preview界面](http://ww3.sinaimg.cn/mw690/8f7a6fe0jw1eya4o08srqj20f70gh0t6.jpg =500x520)
  

关键的几个地方我都用红方框圈了起来，左下角的`+`能模拟不同的设备，中间的红框部位可以模拟设备的旋转，
右下角的`English`部位可以模拟布局中的文学信息双倍后的布局表现，这几个功能都非常的方便。