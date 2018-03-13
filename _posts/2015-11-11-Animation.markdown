---
layout: post
title: "iOS动画总结"
date: 2015-11-11
comments: true
categories: iOS
tags: [Animation]
keywords: CALayer 绘图 
description:  iOS动画总结
--- 

### CALayer
CALayer简介

在iOS系统中，能看见摸得着的东西基本上都是UIView,UIView是iOS系统中界面元素的基础，所有的界面元素都是继承自它。比如一个文本标签，一个按钮等等，这些都是UIView。其实UIView之所以能显示在屏幕上，完全是因为它内部的一个层。在创建UIView对象时，UIView内部会自动创建一个层(即CALayer对象),通过UIView的layer属性可以访问这个层。当UIView需要显示到屏幕上时，会调用drawRect：方法进行绘图，并且会将所有内容绘制在自己的层上，绘图完毕后，系统会将层拷贝到屏幕上，于是就完成了UIView的显示，也就是说UIView本身不具备显示的功能，是它内部的层才有显示功能。

CALayer使用

UIView之所以能够使用，完全是因为内部的CALayer对象。因此通过操作这个CALayer对象，可以很方便地调整UIView的一些界面属性，如:圆角大小，阴影，边框宽度和颜色等。

如果想在UIView中画东西，需要自定义view，创建一个类与之关联，让这个类继承自UIView，然后重写它的DrawRect:方法，然后在该方法中画图。
绘制图形的步骤:1.获取上下文 2.绘制图形 3.渲染图形
新建一个类，让该类继承自CALayer.

{% highlight ruby %}

 @implementation Mylayer
 //重写该方法，在该方法内绘制图形
 -(void)drawInContext:(CGContextRef)ctx
 {
     //1.绘制图形
     //画一个圆
     CGContextAddEllipseInRect(ctx, CGRectMake(50, 50, 100, 100));
     //设置属性（颜色）
 //    [[UIColor yellowColor]set];
    CGContextSetRGBFillColor(ctx, 0, 0, 1, 1);
     
     //2.渲染
     CGContextFillPath(ctx);
}
 @end
{% endhighlight %}

在控制器中，创建一个自定义的类

{% highlight ruby %}

@interface YYViewController ()
@end
 @implementation YYViewController
 
 - (void)viewDidLoad
 {
     [super viewDidLoad];
     
     //1.创建自定义的layer
     YYMylayer *layer=[YYMylayer layer];
     //2.设置layer的属性
     layer.backgroundColor=[UIColor brownColor].CGColor;
     layer.bounds=CGRectMake(0, 0, 200, 150);
     layer.anchorPoint=CGPointZero;
     layer.position=CGPointMake(100, 100);
     layer.cornerRadius=20;
     layer.shadowColor=[UIColor blackColor].CGColor;
     layer.shadowOffset=CGSizeMake(10, 20);
     layer.shadowOpacity=0.6;
     
     [layer setNeedsDisplay];
     //3.添加layer
     [self.view.layer addSublayer:layer];
    
}
 @end
{% endhighlight %}

在自定义layer中的-(void)drawInContext：方法不会自己调用，只能自己通过setNeedDisplay方法调用。

CALayer常用属性

在iOS中CALayer的设计主要是为了内容展示和动画操作，CALayer本身并不包含在UIKit中，它不能响应事件。由于
CALayer在设计之初就考虑它的动画操作功能，CALayer很多属性在修改时都能形成动画效果，这种属性称之为"隐式动画属性"。但是对于UIView的根图层而言属性的修改并不形成动画效果，因为很多情况下根图层更多的充当容器的使用，如果它的属性变动形成动画效果会直接影响子图层。

以下是CALayer常用的属性:
anchorPoint：和中心点position重合的一个点，称为"锚点",锚点的描述是相对于x,y位置比例而言的，默认在
图像中心点(0.5,0.5)的位置

backgroundColor：图层背景颜色

borderColor：边框颜色

borderWidth：边框宽度

bounds：图层大小

contents：图层显示内容，例如可以将图片作为图层的显示内容

contentsRect：图层显示内容的大小和位置

cornerRadius：圆角半径

doubleSided：图层背面是否显示，默认为YES

frame：图层大小和位置，不支持隐式动画，所以CALayer很少使用frame，通常使用bounds和position代替。

transform：图层形变

sublayerTransform：子图层形变

sublayers：子图层

shadowColor：阴影颜色

shadowOffset：阴影漂移量

shadowOpacity：阴影透明度，注意默认为0,如果设置阴影必须设置此属性

shadowPath：阴影的形状

shadowRadius：阴影模糊半径

maskToBounds：子图层是否剪切图层边界,默认为NO

opacity：透明度，类似于UIView的alpha

position：图层中心点位置，类似于UIView的center

隐式属性动画的本质是这些属性的变动默认隐含了CABasicAnimation动画实现。

### Core Animation
iOS中实现一个动画相当简单，只要调用UIView的块代码即可实现一个动画效果。
{% highlight ruby %}
+ (void)animateWithDuration:(NSTimeInterval)duration delay:(NSTimeInterval)delay options:(UIViewAnimationOptions)options animations:(void (^)(void))animations completion:(void (^)(BOOL finished))completion NS_AVAILABLE_IOS(4_0);

+ (void)animateWithDuration:(NSTimeInterval)duration animations:(void (^)(void))animations completion:(void (^)(BOOL finished))completion NS_AVAILABLE_IOS(4_0);

+ (void)animateWithDuration:(NSTimeInterval)duration animations:(void (^)(void))animations NS_AVAILABLE_IOS(4_0);

{% endhighlight %}
使用方法也很简单,就是在API的Block中改变需要实现动画的UIView属性值。iOS7中又增加了针对Keyframe动画的API:

{% highlight ruby %}
+ (void)animateKeyframesWithDuration:(NSTimeInterval)duration delay:(NSTimeInterval)delay options:(UIViewKeyframeAnimationOptions)options animations:(void (^)(void))animations completion:(void (^)(BOOL finished))completion NS_AVAILABLE_IOS(7_0);


+ (void)addKeyframeWithRelativeStartTime:(double)frameStartTime relativeDuration:(double)frameDuration animations:(void (^)(void))animations NS_AVAILABLE_IOS(7_0);

{% endhighlight %}

使用上面UIView封装的方法进行动画设置固然十分方便，但是具体动画如何实现我们是不清楚的，而且上面的代码还有一些问题是无法解决的，例如:如何
控制动画的暂定？如何进行动画的组合？这就需要了解iOS的核心动画Core Animation(包含在Quartz Core框架中)。
在iOS核心动画分为几类:基础动画、关键帧动画、动画组、转场动画。

#### 基础动画

动画创建一个分为以下几步:

1.初始化动画并设置动画属性

2.设置动画属性初始值，结束值以及其他动画属性

3.给图层添加动画

{% highlight ruby %}
//设置背景(注意这个图片其实在根图层)
UIImage *backgroundImage=[UIImage imageNamed:@"background.jpg"];
self.view.backgroundColor=[UIColor colorWithPatternImage:backgroundImage];


//自定义一个图层
_layer=[[CALayer alloc]init];
_layer.bounds=CGRectMake(0, 0, 10, 20);
_layer.position=CGPointMake(50, 150);
_layer.contents=(id)[UIImage imageNamed:@"petal.png"].CGImage;
[self.view.layer addSublayer:_layer];

// 移动动画
-(void)translatonAnimation:(CGPoint)location{
//1.创建动画并指定动画属性
CABasicAnimation *basicAnimation=[CABasicAnimation animationWithKeyPath:@"position"];

//2.设置动画属性初始值和结束值
basicAnimation.fromValue=[NSNumber numberWithInteger:50];
basicAnimation.toValue=[NSValue valueWithCGPoint:location];
basicAnimation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
//设置其他动画属性
basicAnimation.duration=5.0;
basicAnimation.repeatCount=HUGE_VALF;//设置重复次数,HUGE_VALF可看做无穷大，起到循环动画的效果
basicAnimation.removedOnCompletion=NO;

//3.添加动画到图层，注意key相当于给动画进行命名，以后获得该动画时可以使用此名称获取
[_layer addAnimation:basicAnimation forKey:@"KCBasicAnimation_Translation"];
}

//点击事件
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
UITouch *touch=touches.anyObject;
CGPoint location= [touch locationInView:self.view];
//创建并开始动画
[self translatonAnimation:location];
}
{% endhighlight %}

duration表示动画的持续时间，默认为0秒；


speed表示动画的速度，用于将父对象的时间对应当前时间，默认为1，如果设置为2，就表示动画速度是父对象的2倍；

repeatCount表示动画重复的次数，默认为0；repeatDuration表示重复时候的持续时间，默认为0秒；autoreverses表示
是否反向播放动画，默认为NO;

beginTime表示动画开始的时间，默认为0，它是相对于父对象的时间空间而言的，即如果一个Animation Group中的一个
animation设置beginTime为2，则表示这个animation在animation开始的2秒后才会开始执行。

+(instancetype)animationWithKeyPath:(NSString *)path;表示需要修改的Layer属性，例如"position","tansform.scale",可以对哪些属性进行修改，可以参考官方文档。timingFunction是至动画的时间函数，就是个简单的贝塞尔曲线。
四类时间函数简单点理解就是动画是匀速的(kCAMediaTimingFunctionLinear),还是先快后慢(kCAMediaTimingFunctionEaseOut),还是先慢后快(kCAMediaTimingFunctionEaseIn),还是慢快慢(kCAMediaTimingFunctionEaseInEaseOut).

#### 关键帧动画
关键帧动画就是在动画控制过程中开发者指定主要的动画状态，至于各个状态间动画如何进行则由系统自动运算补充(每两个关键帧之间系统形成的动画称之为“补充动画”)，这种动画的好处就是开发者不用逐个控制每个动画帧，而只要关心几个关键帧的状态即可。

关键帧动画开发分为两种形式:一种是通过设置不同的属性进行关键帧控制，另一种是通过绘制路径进行关键帧控制。后者优先级高于前者，如果设置了路径则属性值
就不再起作用。
设置四个关键帧,具体代码如下(动画创建过程同基本动画完全一致):
{% highlight ruby %}
- (void)viewDidLoad {
[super viewDidLoad];

//设置背景(注意这个图片其实在根图层)
UIImage *backgroundImage=[UIImage imageNamed:@"background.jpg"];
self.view.backgroundColor=[UIColor colorWithPatternImage:backgroundImage];

//自定义一个图层
_layer=[[CALayer alloc]init];
_layer.bounds=CGRectMake(0, 0, 10, 20);
_layer.position=CGPointMake(50, 150);
_layer.contents=(id)[UIImage imageNamed:@"petal.png"].CGImage;
[self.view.layer addSublayer:_layer];

//创建动画
[self translationAnimation];
}

//关键帧动画
-(void)translationAnimation{
//1.创建关键帧动画并设置动画属性
CAKeyframeAnimation *keyframeAnimation=[CAKeyframeAnimation animationWithKeyPath:@"position"];

//2.设置关键帧,这里有四个关键帧
NSValue *key1=[NSValue valueWithCGPoint:_layer.position];//对于关键帧动画初始值不能省略
NSValue *key2=[NSValue valueWithCGPoint:CGPointMake(80, 220)];
NSValue *key3=[NSValue valueWithCGPoint:CGPointMake(45, 300)];
NSValue *key4=[NSValue valueWithCGPoint:CGPointMake(55, 400)];
NSArray *values=@[key1,key2,key3,key4];
keyframeAnimation.values=values;
//设置其他属性
keyframeAnimation.duration=8.0;
keyframeAnimation.beginTime=CACurrentMediaTime()+2;//设置延迟2秒执行


//3.添加动画到图层，添加动画后就会执行动画
[_layer addAnimation:keyframeAnimation forKey:@"KCKeyframeAnimation_Position"];
}

{% endhighlight %}

keyTimes：各个关键帧的时间控制。前面使用values设置了四个关键帧，默认情况下每两帧之间的间隔为:8/(4-1)秒。如果想要控制动画从第一帧到
第二帧占用时间4秒，从第二帧到第三帧时间为2秒，而从第三帧到第四帧时间2秒的话，就可以通过keyTimes进行设置。keyTimes中存储的是时间占用比例点，
此时可以设置keyTimes的值为0.0,0.5,0.75,1.0,也就是说1到2帧运行到总时间的50%，2到3帧运行到总时间的75%，3到4帧运行到8秒结束。

#### 动画组
实际开发中一个物体的运动往往是复活运动，单一属性的运动情况比较少，但恰恰属性动画每次进行动画设置时一次只能设置一个属性进行动画控制(不管是基础动画
还是关键帧动画都是如此)，这样一来要做一个复合运动的动画就必须创建多个属性动画进行组合。对于一两种动画的组合或许处理起来还比较容易，但是对于更多
动画的组合控制往往变得很复杂，动画组的产生就是基于这样一种情况而产生的。动画组是一系列动画的组合，凡是添加到动画组中的动画都受控与动画组，这样一来
各类动画公共的行为就可以统一进行控制而不必单独设置，而且放到动画组中的各个动画可以并发执行，共同构建出复杂的动画效果。动画组使用起来并不复杂，首先
单独创建个动画(可以是基础动画也可以是关键帧动画)，然后将基础动画添加到动画组，最后将动画组添加到图层即可。

{% highlight ruby %}
- (void)viewDidLoad {
[super viewDidLoad];

//设置背景(注意这个图片其实在根图层)
UIImage *backgroundImage=[UIImage imageNamed:@"background.jpg"];
self.view.backgroundColor=[UIColor colorWithPatternImage:backgroundImage];

//自定义一个图层
_layer=[[CALayer alloc]init];
_layer.bounds=CGRectMake(0, 0, 10, 20);
_layer.position=CGPointMake(50, 150);
_layer.contents=(id)[UIImage imageNamed:@"petal.png"].CGImage;
[self.view.layer addSublayer:_layer];

//创建动画
[self groupAnimation];
}

//基础旋转动画
-(CABasicAnimation *)rotationAnimation{

CABasicAnimation *basicAnimation=[CABasicAnimation animationWithKeyPath:@"transform.rotation.z"];

CGFloat toValue=M_PI_2*3;
basicAnimation.toValue=[NSNumber numberWithFloat:M_PI_2*3];

//    basicAnimation.duration=6.0;
basicAnimation.autoreverses=true;
basicAnimation.repeatCount=HUGE_VALF;
basicAnimation.removedOnCompletion=NO;

[basicAnimation setValue:[NSNumber numberWithFloat:toValue] forKey:@"KCBasicAnimationProperty_ToValue"];

return basicAnimation;
}

//关键帧移动动画
-(CAKeyframeAnimation *)translationAnimation{
CAKeyframeAnimation *keyframeAnimation=[CAKeyframeAnimation animationWithKeyPath:@"position"];

CGPoint endPoint= CGPointMake(55, 400);
CGPathRef path=CGPathCreateMutable();
CGPathMoveToPoint(path, NULL, _layer.position.x, _layer.position.y);
CGPathAddCurveToPoint(path, NULL, 160, 280, -30, 300, endPoint.x, endPoint.y);

keyframeAnimation.path=path;
CGPathRelease(path);

[keyframeAnimation setValue:[NSValue valueWithCGPoint:endPoint] forKey:@"KCKeyframeAnimationProperty_EndPosition"];

return keyframeAnimation;
}

//创建动画组
-(void)groupAnimation{
//1.创建动画组
CAAnimationGroup *animationGroup=[CAAnimationGroup animation];

//2.设置组中的动画和其他属性
CABasicAnimation *basicAnimation=[self rotationAnimation];
CAKeyframeAnimation *keyframeAnimation=[self translationAnimation];
animationGroup.animations=@[basicAnimation,keyframeAnimation];

animationGroup.delegate=self;
animationGroup.duration=10.0;//设置动画时间，如果动画组中动画已经设置过动画属性则不再生效
animationGroup.beginTime=CACurrentMediaTime()+5;//延迟五秒执行

//3.给图层添加动画
[_layer addAnimation:animationGroup forKey:nil];
}
//代理方法
// 动画完成
-(void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag{
CAAnimationGroup *animationGroup=(CAAnimationGroup *)anim;
CABasicAnimation *basicAnimation=animationGroup.animations[0];
CAKeyframeAnimation *keyframeAnimation=animationGroup.animations[1];
CGFloat toValue=[[basicAnimation valueForKey:@"KCBasicAnimationProperty_ToValue"] floatValue];
CGPoint endPoint=[[keyframeAnimation valueForKey:@"KCKeyframeAnimationProperty_EndPosition"] CGPointValue];

[CATransaction begin];
[CATransaction setDisableActions:YES];

//设置动画最终状态
_layer.position=endPoint;
_layer.transform=CATransform3DMakeRotation(toValue, 0, 0, 1);

[CATransaction commit];
}
{% endhighlight %}
