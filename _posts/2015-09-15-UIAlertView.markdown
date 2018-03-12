---
layout: post
title: "从UIAlertView实现来看对象关联和ReactiveCocoa"
date: 2015-09-15
comments: true
categories: iOS
tags: [UIAlertView]
keywords: 对象关联 视图 UIAlertView
description:  懒加载的使用
---

 开发ios时经常用到UIAlertView类,该类提供了一种标准视图,可向用户展示警告信息。当用户按下按钮关闭视图时，需要用视图协议来处理此动作，但是，
 要想设置好这个委托机制，就得把创建警告视图和处理按钮动作的代码分开。由于代码分作两块，所以读起来有点乱(更可怕的是一个控制器里面出现多个UIAlertView)，比方说我们使用
 UIAlertView时，都会这么写:

{% highlight ruby %}
- (void)askUserAQuestion{
UIAlertView *alert = [[UIAlertView alloc]initWithTitle:@"Question" message:@"What dou you want to do?" delegate:self cancelButtonTitle:@"Cancel" otherButtonTitles:@"Continue", nil];
alert.delegate = self;
[alert show];
}

- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
{
if (buttonIndex ==0) {
[self doCancel];
}else{
[self doContinue];
}
}

{% endhighlight %}

如果想在同一个类里处理多个警告信息视图,那么代码就会变得更为复杂，我们必须在delegate方法中检查传人的alertView参数，并据此选用相应的逻辑。要是能再创建警告视图时把处理每个按钮的逻辑都写好，岂不完美？这可以通过关联对象来做。创建完警告视图之后，设定一个与之关联的"块"(block)，等到执行delegate方法时再将其读出来。方案如下:
{% highlight ruby %}
// #import <objc/runtime.h>
static void *EOCMyAlertViewKey = "EOCMyAlertViewKey";
- (void)askUserAQuestion{
UIAlertView *alert = [[UIAlertView alloc]initWithTitle:@"Question" message:@"What do you want to do" delegate:self cancelButtonTitle:@"Cancel" otherButtonTitles:@"Continue", nil];
void(^block)(NSInteger) = ^(NSInteger buttonIndex){
if (buttonIndex ==0) {
[self doCancel];
}else{
[self doContinue];
}
};
objc_setAssociatedObject(alert, EOCMyAlertViewKey, block, OBJC_ASSOCIATION_COPY);
[alert show];
}
- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
{
void(^block)(NSInteger) = objc_getAssociatedObject(alertView, EOCMyAlertViewKey);
block(buttonIndex);
}

{% endhighlight %}
以这种方式改下之后，创建警告视图与处理操作结果代码都放在一起了,这样比原来更容易读懂，我们无须在两部分代码之间来回游走了。但是采用该方案时要注意:块可能要捕获某些变量,也许会造成"保留环"。

下面来看另一种实现方式:

{% highlight ruby %}
- (void)askUserAQuestion{
UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:nil message:@"rac demo" delegate:nil cancelButtonTitle:@"取消" otherButtonTitles:@"确定", nil];
[[alertView rac_buttonClickedSignal] subscribeNext:^(id x) {
if ([x integerValue] == 0) {
[self doCancel];
} else {
[self doContinue];
}  
}];  
[alertView show];
}

{% endhighlight %}

 RAC为UIAlertView提供了- (RACSignal *)rac_buttonClickedSignal方法，此方法为警告框的按钮点击时间创建了一个signal.
