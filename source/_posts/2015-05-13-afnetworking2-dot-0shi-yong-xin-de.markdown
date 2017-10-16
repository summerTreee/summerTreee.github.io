---
layout: post
title: "AFNetworking2.0使用心得"
date: 2015-05-13 13:56:23 +0800
comments: true
categories: 
---
这段时间要将公司项目中的网络引擎由[ASIHTTPRequest](http://allseeing-i.com/ASIHTTPRequest/)替换为[AFNetworking](https://github.com/AFNetworking/AFNetworking),替换的过程比较曲折，在此记录下自己替换过程中得心得：

### 1、建立数据请求中介者
建立`中介者`是指项目中的数据请求都通过它去实现，而不是每一个数据请求都直接与`AFNetworking`打交道，这样做的好处是：

* 将网络请求与第三方库依赖隔离开来，方便以后对第三方库的替换。
* 方便处理网络请求的公共逻辑。

<!--more-->

### 2、使用completionQueue
默认情况下`AFURLConnectionOperation`或者`AFHTTPRequestOperation`请求结束以后会在主线程将结果传递回来，如果你要将请求的结果做一些耗时的复杂的处理，就会*block*住主线程，所以这种情况下你就需要对请求的Operation传递`completionQueue`参数：

```
AFHTTPRequestOperation *request = [[AFHTTPRequestOperation alloc] initWithRequest:urlrequest];

request.responseSerializer = [AFJSONResponseSerializer serializer];
//设置回调的queue，默认是在mainQueue执行block回调
request.completionQueue = your_request_operation_completion_queue();
[request setCompletionBlockWithSuccess:^(AFHTTPRequestOperation *operation, id responseObject) {
         //设置了'completionQueue'后，就可以在这里处理复杂的逻辑
         //不用担心block住了主线程
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {

 }];
 [request start];

```
### 3、如何知道AFHTTPRequestOperationManager执行完成  
开始以为直接设置`AFHTTPRequestOperationManager`的*completionGroup*，然后利用`dispatch_group_notify`来获取operationQueue执行结束的通知，最后才发现，这样根本不行，看了源码才知道：`AFHTTPRequestOperationManager`直接将*completionGroup*赋值给了他的每一个*operation*，在*operation*的completionBlock里面利用*completionGroup*，来确保在operation的处理完成后，将completionBlock置为nil，防止循环引用。  

虽然说这样不能知道`AFHTTPRequestOperationManager`什么时候执行完成，但是生活还得继续下去啊！在`AFURLConnectionOperation`中有一个方法叫：  

```
+ (NSArray *)batchOfRequestOperations:(NSArray *)operations
                        progressBlock:(void (^)(NSUInteger numberOfFinishedOperations, NSUInteger totalNumberOfOperations))progressBlock
                      completionBlock:(void (^)(NSArray *operations))completionBlock;

```
这个方法就是能够发一组请求，跟`AFHTTPRequestOperationManager`比起来的缺点就是没法设置并发数，但是它却能实现检测一组请求什么时候结束，能检测完成了多少请求，它是怎么做到的呢？

原来他也是利用**dispatch_group_async**和**dispatch_group_notify**
  
一般的我们要把一个任务加入一个group里是这样：  

```
dispatch_group_async(group, queue, ^{
    block();
});

```
这个写法等价于  

```
dispatch_async(queue, ^{
    dispatch_group_enter(group);
    block()
    dispatch_group_leave(group);
});

```
如果要把一个异步任务加入group，这样就行不通了： 
 
```
dispatch_group_async(group, queue, ^{
    [self performBlock:^(){
        block();
    }];
    //未执行到block() group任务就已经完成了
});

```
这是就需要用到`batchOfRequestOperations`里的实现了： 
 
```
dispatch_group_enter(group);
[self performBlock:^(){
    block();
    dispatch_group_leave(group);
}];

```
其实这个和引用计数差不多，*dispatch_group_enter*时引用计数+1，*dispatch_group_leave*时引用计数-1，引用计数为0时执行*dispatch_group_notify*的内容。具体过程大致如下：  

```
AFHTTPRequestOperationManager *downloadManager = [AFHTTPRequestOperationManager manager];
    //设置最大并发数
    [downloadManager.operationQueue setMaxConcurrentOperationCount:([NSProcessInfo processInfo].processorCount) * 2];
    //创建一个group
    __block dispatch_group_t group = dispatch_group_create();
    for (NSURL *url in urlArray) {
        NSURLRequest *requestUrl = [NSURLRequest requestWithURL:url];
        AFHTTPRequestOperation *requestOperation = [[AFHTTPRequestOperation alloc] initWithRequest:requestUrl];
        [requestOperation setCompletionBlockWithSuccess:^(AFHTTPRequestOperation *operation, id responseObject) {
            dispatch_group_leave(group);
            
        } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
            dispatch_group_leave(group);
            
        }];
        //将请求加入队列中
        [downloadManager.operationQueue addOperation:requestOperation];
        dispatch_group_enter(group);
    }
    
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        //全部请求完成
        
    });  
```  
### 4、实现断点续传
`AFNetworking`虽然支持文件下载的暂停和继续，但是当缓存清空重新启动时，它并没有记录下下载的状态，无法续传，但是可以通过[AFDownloadRequestOperation](http://https://github.com/steipete/AFDownloadRequestOperation)来简单的实现，其实过程也不是很复杂，大致如下，感兴趣的朋友可以阅读`AFDownloadRequestOperation`是实现部分：  

1、设置`AFHTTPRequestOperation`的请求`NSMutableURLRequest`*HTTPHeader*的**Range**字段。
  
```  
NSMutableURLRequest *mutableURLRequest = [self.request mutableCopy];
//offset是指断点续传文件已经下载的大小
[mutableURLRequest setValue:[NSString stringWithFormat:@"bytes=%llu-", offset] forHTTPHeaderField:@"Range"];
```  

2、设置`AFHTTPRequestOperation`的*outputStream*属性。  

 ``` 
 //downloadPath是指文件下载存放的路径
downloadOperation.outputStream = [NSOutputStream outputStreamToFileAtPath:downloadPath append:YES];
```

### 结尾
对于`ASIHTTPRequest`和`AFNetworking`的比较网上有很多很好的文章，一搜一大把，对于普通的使用来说感觉区别不大，请求速度什么的也没什么感觉，以上若有错误，还望大家多多
指正😃。