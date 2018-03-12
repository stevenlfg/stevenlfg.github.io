---
layout: post
title: "运用OC运行时和Method Swizzing来设置返回按钮"
date: 2015-07-31
comments: true
categories: iOS
tags: [返回按钮]
keywords: 自定义返回按钮
---
 在项目开发中，随着界面的增多，为每个控制器自定义返回按钮，每一个控制器中都要写一次，较为麻烦啰嗦。

{% highlight ruby %}
+ (UIBarButtonItem *)itemWithIcon:(NSString *)icon title:(NSString *)title highLightIcon:(NSString *)highLightIcon target:(id)target action:(SEL)action
{
UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
[button setTitle:title forState:UIControlStateNormal];
button.titleLabel.font = [UIFont systemFontOfSize:14];
[button setBackgroundImage:[UIImage imageNamed:icon] forState:UIControlStateNormal];
[button setBackgroundImage:[UIImage imageNamed:highLightIcon] forState:UIControlStateHighlighted];
button.frame = (CGRect){CGPointZero, button.currentBackgroundImage.size};
[button addTarget:target action:action forControlEvents:UIControlEventTouchUpInside];
return [[UIBarButtonItem alloc] initWithCustomView:button];
}
{% endhighlight %}

 然后再每个控制器的viewDidLoad方法中
{% highlight ruby %}
 self.navigationItem.leftBarButtonItem = [UIBarButtonItem itemWithIcon:@"backImg.png" title:nil highLightIcon:@"backImgHight.png" target:self action:@selector(TapLeftItemAction) ];
{% endhighlight %}

后来无意中看到了这两篇文章(http://wxgbridgeq.github.io)，(http://southpeak.github.io/blog/2014/11/06/objective-c-runtime-yun-xing-shi-zhi-si-:method-swizzling/)，受到启发.废话少说，直接上代码:

{% highlight ruby %}
+ (void)load{
swizzleAllViewController();
}
void swizzleAllViewController()
{
Swizzle([UIViewController class], @selector(viewDidAppear:), @selector(customViewDidAppear:));
Swizzle([UIViewController class], @selector(viewWillDisappear:), @selector(customViewWillDisappear:));
Swizzle([UIViewController class], @selector(viewWillAppear:), @selector(customviewWillAppear:));
}
- (void)customviewWillAppear:(BOOL)animated{
if ([[self.navigationController childViewControllers] count] > 1) {
self.navigationController.interactivePopGestureRecognizer.delegate = self;
self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc]initWithCustomView:self.backButton];

}
[self customviewWillAppear:animated];
}
- (UIButton *)backButton
{
UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
[button setBackgroundImage:[UIImage imageNamed:@"btn_back.png"] forState:UIControlStateNormal];
[button setBackgroundImage:[UIImage imageNamed:@"btn_back_highLight.png"] forState:UIControlStateHighlighted];
button.frame = (CGRect){CGPointZero, button.currentBackgroundImage.size};
[button addTarget:self action:@selector(goBack_Swizzle) forControlEvents:UIControlEventTouchUpInside];
return button;
}
- (void)goBack_Swizzle
{
[self.navigationController popViewControllerAnimated:YES];
}
{% endhighlight %}

经过试验,改写返回按钮会让苹果自带的返回手势失效，故开启手势

{% highlight ruby %}
self.navigationController.interactivePopGestureRecognizer.delegate = self;
{% endhighlight %}