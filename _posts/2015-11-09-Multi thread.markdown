---
layout: post
title: "多线程总结"
date: 2015-11-09
comments: true
categories: iOS
tags: [multi thread]
keywords: 多线程 性能 block
description:  多线程总结
---

iOS多线程的使用对于App性能优化有很重要的意义。使用线程可以把程序中占据时间长的任务放到后台去处理，如图片、视频的下载。
发挥多核处理器的优势，并发执行让系统运行的更快、更流畅，用户体验更好。总结一下iOS里面常用的三种线程编程的技术。

###NSThread

这种方案是经过苹果封装后的，并且完全面向对象的。可以直接操控线程对象，非常直观和方便。但是它的生命周期还是需要我们手动管理。
使用NSThread创建有以下几种方式:
1.创建一个NSTread类实例，然后调用start方法。

{% highlight ruby %}
NSThread* aThread = [[NSThread alloc] initWithTarget:self selector:@selector(threadRoutine:) object:nil];  
[aThread start];
{% endhighlight %}
2.+detachNewThreadSelector:toTarget:withObject:类方法直接生成一个子线程

{% highlight ruby %}
[NSThread detachNewThreadSelector:@selector(threadRoutine:) toTarget:self withObject:nil];
{% endhighlight %}

3.调用NSObject的+performSelectorInBackground:withObject:方法生成子线程。
{% highlight ruby %}
[myObj performSelectorInBackground:@selector(threadRoutine:) withObject:nil];
{% endhighlight %}

4.创建一个NSTread子类，然后调用子类实例的start方法。

第-种和第四种方法创建的线程有个好处就是拥有线程的对象，因此可以使用performSelector:onThread:withObject:waitUntilDone:在该线程上执行方法，这是一种非常
方便的线程间通讯的方法，所要执行的方法可以添加到目标线程的Runloop中。Apple建议使用这个接口运行的方法不要是耗时或者频繁的操作，以免子线程的负载过重。
第三种方法其实和第二种方法是一样的，都会直接生成一个子线程。

上面四种方法生成的子线程都是detached状态，即主线程结束时这些线程都会被直接杀死；如果要生成joinable状态的子线程，只能使用pthread接口。
如果需要，可以设置线程的优先级(-setThreadPriority：)；如果要在线程中保存一些状态信息，还可以用到-threadDictionary得到一个NSMutableDictionary,以
key-value的方式保存信息于线程内读写。

###NSOperation

NSOperation本身是一个抽象类，定义要执行的工作，NSOperationQueue是一个工作队列，当工作加入到队列后，NSOperationQueue会自动按照优先顺序及工作的从属
依赖关系(如果有的话）组织执行。NSOperation是没法直接使用的，它只是提供了一个工作的基本逻辑，具体实现还是需要你通过定义自己的NSOperation子类来活得。如果
有必要也可以不将NSOperation加入到一个NSOperationQueue中去执行，直接调用起-start也可以直接执行。在继承NSOperation后，对于非并发的工作，只需要实现NSOperation子类的main方法:
由于NSOperation的工作是可以取消Cancel的，那么你在main方法处理工作时就需要不断轮询[self isCancelled]确认当前的工作是否被取消了。如果要支持并发工作，
那么NSOperation子类需要至少override这四个方法:start,isConcurrent,isExecuting,isFinished.
NSOperation和NSOperationQueue其他特性：
工作是由优先级的，可以通过NSOperation的两个接口读取或者设置：

{% highlight ruby %}
- (NSOperationQueuePriority)queuePriority;
- (void)setQueuePriority:(NSOperationQueuePriority)p;
{% endhighlight %}

工作之间也可以有从属依赖关系，只有依赖的工作完成后才会执行:

{% highlight ruby %}
- (void)addDependency:(NSOperation *)op;
- (void)removeDependency:(NSOperation *)op;
{% endhighlight %}

还可以通过下面接口设置运行NSOpration的子线程优先级：

{% highlight ruby %}
- (void)setQueuePriority:(NSOperationQueuePriority)priority;
{% endhighlight %}

如果要设置Queue的并发操作数:

{% highlight ruby %}
- (void)setMaxConcurrentOperationCount:(NSInteger)cnt;
{% endhighlight %}

iOS4之后还可以再NSOperation上添加一个结束block,用于在工作执行结束之后的操作:

{% highlight ruby %}
- (void)setCompletionBlock:(void(^)(void))block;
{% endhighlight %}

如果需要阻塞等待NSOperation工作结束(别在主线程这么干),可以使用接口:

{% highlight ruby %}
- (void)waitUntilFinished;
{% endhighlight %}

NSOperationQueue除了添加NSOperation外,也支持直接添加一个Block(iOS4之后):

{% highlight ruby %}
- (void)addOperationWithBlock:(void (^)(void))block
{% endhighlight %}

NSOperationQueue可以取消所有添加的工作:

{% highlight ruby %}
- (void)cancelAlloperations;
{% endhighlight %}

也可以阻塞式的等待所有工作结束(别在主线程这么干):

{% highlight ruby %}
- (void)waitUntilAllOperationAreFinished;
{% endhighlight %}

在NSOperation对象中获得被添加的NSOperationQueue队列:

{% highlight ruby %}
+ (id)currentQueue
{% endhighlight %}

要获得一个绑定在主线程的NSOperationQueue队列:

{% highlight ruby %}
+ (id)mainQueue
{% endhighlight %}

其实除非必要，简单的工作完全可以使用官方提供的NSOperation两个子类NSInvocationOperation和NSBlockOperation来实现
NSInvocationOperation:

{% highlight ruby %}
NSInvocationOperation* theOp = [[NSInvocationOperation alloc]initWithTarget:self
selector:@selector(myTaskMethod:)
object:data];
{% endhighlight %}

NSBlockOperation:

{% highlight ruby %}
NSBlockOperation* theOp = [NSBlockOperation blockOperationWithBlock: ^{  
NSLog(@"Beginning operation.\n");
// Do some work.
}];
{% endhighlight %}

###GCD

GCD(Grand Central Dispatch),是一组用于实现并发编程的C接口。GCD是基于Objective-C的block特性开发的，基本业务逻辑和NSOperation很像，都是
将工作添加到一个队列，由系统来负责线程的生成和调度。由于直接使用Block,因此此NSOperation子类使用起来更方便，大大降低了多线程开发的门槛。
基本用法:

{% highlight ruby %}
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{  
[self doTask];
NSLog(@"Fisinished");
});
{% endhighlight %}

GCD的调用接口非常简单，就是将Job提交至Queue中，主要的提交Job接口为:
1.dispatch_sync(queue,block)同步提交job

2.dispatch_async(queue,block)异步提交job

3.dispatch_after(time,queue,block)同步延时提交job

其中第一个参数类型是dispatchqueuet,就是一个表示队列的数据结构typedef struct dispatch_queue_s *dispatch_queue_t;
block就是表示任务的Block typedef void(^dispatch_block_t)(void);
dispatch_async函数是异步非阻塞的，调用后会立刻返回，工作由系统在线程池中分配线程去执行工作。
dispatch_sync和dispatch_after是阻塞式的，会一直等到添加的工作完成后才会返回。
除了添加Block到Dispatch Queue,还有添加函数到Dispatch Queue的接口，例如dispatch_async对应的有dispatch_async_f：

{% highlight ruby %}
dispatch_async_f(dispatch_queue_t queue,  
void *context,
dispatch_function_t work);
{% endhighlight %}

其中第三个参数就是个函数指针，即typedef void(*dispatch_function_t)(void *);第二个参数就是传给这个函数的参数。

Dispatch Queue 
要添加工作到队列Dispatch Queue中，这个队列可以是串行或并行的，并行队列会尽可能的并发执行其中的工作任务，而串行队列每次只能
运行一个任务。
目前GCD中有三种类型的Dispatch Queue：

1.Main Queue：关联到主线程的队列，可以使用函数dispatch_get_main_queue()获得，加到这个队列中的工作都会分发到主线程运行。
主线程只有一个，因此很明显这个是串行队列，每次运行一个工作.

2.Global Queue：全局队列是并发队列，又根据优先级细分为高优先级、默认优先级和低优先级三种。通过dispatch_get_global_queue加上优先级
参数获得这个全局队列，例如dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0).

3.自定义Queue：自己创建一个队列,通过函数dispatch_queue_creat创建，例如，dispatch_queue_creat("com.kiloapp.test",NULL).第一个参数
是队列的名字，Apple建议使用烦DNS型的名字命名，防止重名；第二个参数是创建的queue的类型，iOS4.3以前只支持串行，即DISPATCHQUEUESERIAL(就是NULL)
iOS4.3以后也开始支持串行队列，即参数DISPATCHQUEUECONCURRENT.

由于有这些种不同类型的队列，一种常见的使用模式是:

{% highlight ruby %}
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{  
[self doHardWorkInBackground];
dispatch_async(dispatch_get_main_queue(), ^{
[self updateUI];
});
});
{% endhighlight %}

将一些耗时的工作添加到全局队列，让系统分配线程去做，工作完成后再次调用GCD的主线程队列去完成UI相关的工作，这样做就不会因为大量的非UI相关工作加重主线程负担，从而加快UI事件响应。
其他几个可能用到的接口:
dispatch_get_current_queue()获取当前队列，一般在提交的Block中使用。在提交的Block之外调用时，如果在主线程中就返回主线程Queue；如果是在其他子线程，返回的是
默认的并发队列。
dispatch_queue_get_label(queue)获取队列的名字，如果你自己创建的队列没有设置名字，那就是返回NULL.
dispatch_set_target_queue(object,queue)设置给定对象的目标队列。这是一个非常强大的接口，目标队列负责处理这个GCD Object.注意这个Object还可以是另个队列。
dispatch_main()会阻塞主线程等待主队列Main Queue中的Block执行结束。

Dispatch Group
GCD的另一种使用场景：例如

{% highlight ruby %}
for(id obj in array)  
{
[self doWorkOnItem:obj];
}
[self doWorkOnArray:array];
{% endhighlight %}
前半部分可以用GCD得到处理性能的提升:

{% highlight ruby %}
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);  
for(id obj in array)  
dispatch_async(queue, ^{
[self doWorkOnItem:obj];
});
[self doWorkOnArray:array];
{% endhighlight %}

问题是[self doWorkOnArray:array];原先是在全部数组各个成员的工作完成后才会执行的，现在由于dispatchasync是异步的，[self doWorkOnArray:array];很有可能在各个成员的工作完成前就开始运行，这明显不符合原先的语义。如果将dispatchasync改成dispatch_sync可以解决问题，但是和原来的方法一样没有并行处理数组，使用GCD也就没有意义了。
针对这种情况，GCD提供了Dispatch Group可以将一组工作集合在一起，等待这组工作完成后再继续运行。dispatchgroupcreate函数可以用来创建这个Group：

{% highlight ruby %}
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);  
dispatch_group_t group = dispatch_group_create();  
for(id obj in array)  
dispatch_group_async(group, queue, ^{
[self doWorkOnItem:obj];
});
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);  
dispatch_release(group);  
[self doWorkOnArray:array];
{% endhighlight %}

方法是不是很简单，将并发的工作用dispatchgroupasync异步添加到一个Group和全局队列中，dispatchgroupwait会等待这些工作完成后再返回，这样你就可以再运行[self doWorkOnArray:array];。

不过有点不好的是dispatchgroupwait会阻塞当前线程，如果当前是主线程岂不是不好，有更绝的dispatchgroupnotify接口：

{% highlight ruby %}
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);  
dispatch_group_t group = dispatch_group_create();  
for(id obj in array)  
dispatch_group_async(group, queue, ^{
[self doWorkOnItem:obj];
});
dispatch_group_notify(group, queue, ^{  
[self doWorkOnArray:array];
});
dispatch_release(group);  
{% endhighlight %}

dispatchgroupnotify函数可以将这个Group完成后的工作也同样添加到队列中（如果是需要更新UI，这个队列也可以是主队列），总之这样做就完全不会阻塞当前线程了。

Dispatch Group还有两个接口可以显式的告知group要添加block操作： 
dispatchgroupenter(group)和dispatchgroupleave(group)，这两个接口的调用数必须平衡，否则group就无法知道是不是处理完所有的Block了。
