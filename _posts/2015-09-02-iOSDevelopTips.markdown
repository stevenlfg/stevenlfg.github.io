---
layout: post
title: "iOS develop tips"
date: 2015-09-02
comments: true
categories: ios
tags: [tips]
keywords: iOS 问题
---
 
 iOS开发过程中，总有那么一些个小问题让人纠结，它们不会让程序崩溃，但是会让人崩溃
 
 1.返回输入键盘

{% highlight ruby %}
  <UITextFieldDelegate>

 - (BOOL)textFieldShouldReturn:(UITextField *)textField {
    [textField resignFirstResponder];
     return YES;
  }
{% endhighlight %}

 2.CGRect

{% highlight ruby %}
CGRectFromString(<#NSString *string#>)//有字符串恢复出矩形
CGRectInset(<#CGRect rect#>, <#CGFloat dx#>, <#CGFloat dy#>)//创建较小或者较大的矩形
CGRectIntersectsRect(<#CGRect rect1#>, <#CGRect rect2#>)//判断两巨星是否交叉，是否重叠
CGRectZero//高度和宽度为零的，位于（0，0）的矩形常量
{% endhighlight %}

 3.隐藏状态栏

{% highlight ruby %}
[UIApplication sharedApplication] setStatusBarHidden:<#(BOOL)#> withAnimation:<#(UIStatusBarAnimation)#>//隐藏状态栏
{% endhighlight %}

4.自动适应父视图大小

{% highlight ruby %}
self.view.autoresizesSubviews = YES;
self.view.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
{% endhighlight %}

5. UITableView的一些方法

{% highlight ruby %}
缩进级别设置为行号，row越大，缩进越多
<UITableViewDelegate>

- (NSInteger)tableView:(UITableView *)tableView indentationLevelForRowAtIndexPath:(NSIndexPath *)indexPath {
NSInteger row = indexPath.row;
return row;
}
{% endhighlight %}

6.把plist文件中的数据赋给数组

{% highlight ruby %}
NSString *path = [[NSBundle mainBundle] pathForResource:@"States" ofType:@"plist"];
NSArray *array = [NSArray arrayWithContentsOfFile:path];
{% endhighlight %}

7.获取触摸的点

{% highlight ruby %}
- (CGPoint)locationInView:(UIView *)view;
- (CGPoint)previousLocationInView:(UIView *)view;
{% endhighlight %}

8.获取触摸的属性

{% highlight ruby %}
@property(nonatomic,readonly) NSTimeInterval      timestamp;
@property(nonatomic,readonly) UITouchPhase        phase;
@property(nonatomic,readonly) NSUInteger          tapCount;
{% endhighlight %}

9.从plist中获取数据赋给字典

{% highlight ruby %}
NSString *plistPath = [[NSBundle mainBundle] pathForResource:@"book" ofType:@"plist"];
NSDictionary *dictionary = [NSDictionary dictionaryWithContentsOfFile:plistPath];
{% endhighlight %}

10.NSUserDefaults注意事项

{% highlight ruby %}
设置完了以后如果存储的东西比较重要的话，一定要同步一下
[[NSUserDefaults standardUserDefaults] synchronize];
{% endhighlight %}

11.获取Documents目录

{% highlight ruby %}
NSString *documentsDirectory = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)[0];
{% endhighlight %}

12.获取tmp目录

{% highlight ruby %}
NSString *tmpPath = NSTemporaryDirectory();
{% endhighlight %}

13.利用Safari打开一个链接
{% highlight ruby %}
NSURL *url = [NSURL URLWithString:@"http://baidu.com"];
[[UIApplication sharedApplication] openURL:url];
{% endhighlight %}

14.利用UIWebView显示pdf文件，网页等等

{% highlight ruby %}
<UIWebViewDelegate>

UIWebView *webView = [[UIWebView alloc]initWithFrame:self.view.bounds];
webView.delegate = self;
webView.scalesPageToFit = YES;
webView.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
[webView setAllowsInlineMediaPlayback:YES];
[self.view addSubview:webView];

NSString *pdfPath = [[NSBundle mainBundle] pathForResource:@"book" ofType:@"pdf"];
NSURL *url = [NSURL fileURLWithPath:pdfPath];
NSURLRequest *request = [NSURLRequest requestWithURL:url cachePolicy:(NSURLRequestUseProtocolCachePolicy) timeoutInterval:5];
{% endhighlight %}

15.UIWebView和html的简单交互

{% highlight ruby %}
myWebView = [[UIWebView alloc]initWithFrame:self.view.bounds];
[myWebView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]]];
NSError *error;
NSString *errorString = [NSString stringWithFormat:@"<html><center><font size=+5 color='red'>AnError Occurred;<br>%@</font></center></html>",error];
[myWebView loadHTMLString:errorString baseURL:nil];

//页面跳转了以后，停止载入

-(void)viewWillDisappear:(BOOL)animated {
if (myWebView.isLoading) {
[myWebView stopLoading];
}
myWebView.delegate = nil;
[UIApplication sharedApplication].networkActivityIndicatorVisible = NO;
}
{% endhighlight %}

16.汉字转码

{% highlight ruby %}
NSString *oriString = @"\u67aa\u738b";
NSString *escapedString = [oriString stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
{% endhighlight %}

17.处理键盘通知

{% highlight ruby %}
- (void)keyboardWasShown:(NSNotification *)notification {
if (keyboardShown) {
return;
}
NSDictionary *info = [notification userInfo];

NSValue *aValue = [info objectForKey:UIKeyboardFrameBeginUserInfoKey];
CGSize keyboardSize = [aValue CGRectValue].size;
CGRect viewFrame = scrollView.frame;
viewFrame.size.height = keyboardSize.height;

CGRect textFieldRect = activeField.frame;
[scrollView scrollRectToVisible:textFieldRect animated:YES];
keyboardShown = YES;
}

- (void)keyboardWasHidden:(NSNotification *)notification {
NSDictionary *info = [notification userInfo];
NSValue *aValue = [info objectForKey:UIKeyboardFrameEndUserInfoKey];
CGSize keyboardSize = [aValue CGRectValue].size;

CGRect viewFrame = scrollView.frame;
viewFrame.size.height += keyboardSize.height;
scrollView.frame = viewFrame;

keyboardShown = NO;
}
{% endhighlight %}

18.点击键盘的next按钮，在不同的textField之间换行

{% highlight ruby %}
- (BOOL)textFieldShouldReturn:(UITextField *)textField {
if ([textField returnKeyType] != UIReturnKeyDone) {
NSInteger nextTag = [textField tag] + 1;
UIView *nextTextField = [self.tableView viewWithTag:nextTag];
[nextTextField becomeFirstResponder];
}else {
[textField resignFirstResponder];
}
return YES;
}
{% endhighlight %}

19.设置日期格式

{% highlight ruby %}
dateFormatter = [[NSDateFormatter alloc]init];
dateFormatter.locale = [NSLocale currentLocale];
dateFormatter.calendar = [NSCalendar autoupdatingCurrentCalendar];
dateFormatter.timeZone = [NSTimeZone defaultTimeZone];
dateFormatter.dateStyle = NSDateFormatterShortStyle;
NSLog(@"%@",[dateFormatter stringFromDate:[NSDate date]]);
{% endhighlight %}

20.加载大量图片的时候，可以使用

{% highlight ruby %}
NSString *imagePath = [[NSBundle mainBundle] pathForResource:@"icon" ofType:@"png"];
UIImage *myImage = [UIImage imageWithContentsOfFile:imagePath];
{% endhighlight %}

21.有时候在iPhone游戏中，既要播放背景音乐，同时又要播放比如枪的开火音效

{% highlight ruby %}
NSString *musicFilePath = [[NSBundle mainBundle] pathForResource:@"xx" ofType:@"wav"];
NSURL *musicURL = [NSURL fileURLWithPath:musicFilePath];
AVAudioPlayer *musicPlayer = [[AVAudioPlayer alloc]initWithContentsOfURL:musicURL error:nil];
[musicPlayer prepareToPlay];
musicPlayer.volume = 1;
musicPlayer.numberOfLoops = -1;//-1表示一直循环
{% endhighlight %}

22.从通讯录中读取电话号码，去掉数字之间的-

{% highlight ruby %}
NSString *originalString = @"(123)123123abc";
NSMutableString *strippedString = [NSMutableString stringWithCapacity:originalString.length];
NSScanner *scanner = [NSScanner scannerWithString:originalString];
NSCharacterSet *numbers = [NSCharacterSet characterSetWithCharactersInString:@"0123456789"];

while ([scanner isAtEnd] == NO) {
NSString *buffer;
if ([scanner scanCharactersFromSet:numbers intoString:&buffer]) {
[strippedString appendString:buffer];
}else {
scanner.scanLocation = [scanner scanLocation] + 1;
}
}
NSLog(@"%@",strippedString);
{% endhighlight %}

23.正则判断：字符串只包含字母和数字

{% highlight ruby %}
NSString *myString = @"Letter1234";
NSString *regex = @"[a-z][A-Z][0-9]";
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"SELF MATCHES %@",regex];

if ([predicate evaluateWithObject:myString]) {
//implement
}
{% endhighlight %}

24.设置UITableView的滚动条颜色

{% highlight ruby %}
self.tableView.indicatorStyle = UIScrollViewIndicatorStyleWhite;
{% endhighlight %}

25.使用NSURLConnection下载数据

{% highlight ruby %}
1. 创建对象
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:[NSURL URLWithString:@"http://www.baidu.com"]];
[NSURLConnection connectionWithRequest:request delegate:self];

2. NSURLConnection delegate 委托方法
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {

}

- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {

}

- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {

}

- (void)connectionDidFinishLoading:(NSURLConnection *)connection {

}

3. 实现委托方法
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
self.receiveData.length = 0;//先清空数据
}

- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
[self.receiveData appendData:data];
}

- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
//错误处理
}

- (void)connectionDidFinishLoading:(NSURLConnection *)connection {
[UIApplication sharedApplication].networkActivityIndicatorVisible = NO;
NSString *returnString = [[NSString alloc]initWithData:self.receiveData encoding:NSUTF8StringEncoding];
firstTimeDownloaded = YES;
}
{% endhighlight %}

26.隐藏状态栏

{% highlight ruby %}
[UIApplication sharedApplication].statusBarHidden = YES;
{% endhighlight %}

27..m文件与.mm文件的区别

{% highlight ruby %}
.m文件是objective-c文件
.mm文件相当于c++或者c文件
{% endhighlight %}

28.读取一般性文件

{% highlight ruby %}
- (void)readFromTXT {
NSString *tmp;
NSArray *lines;//将文件转化为一行一行的
lines = [[NSString stringWithContentsOfFile:@"testFileReadLines.txt"] componentsSeparatedByString:@"\n"];

NSEnumerator *nse = [lines objectEnumerator];

//读取<>里的内容
while (tmp == [nse nextObject]) {
NSString *stringBetweenBrackets = nil;
NSScanner *scanner = [NSScanner scannerWithString:tmp];
[scanner scanUpToString:@"<" intoString:nil];
[scanner scanString:@"<" intoString:nil];
[scanner scanUpToString:@">" intoString:&stringBetweenBrackets];
NSLog(@"%@",[stringBetweenBrackets description]);
}
}
{% endhighlight %}

29.隐藏UINavigationBar

{% highlight ruby %}
[self.navigationController setNavigationBarHidden:YES animated:YES];
{% endhighlight %}

30.调用电话，短信，邮件
{% highlight ruby %}
[[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"mailto:apple@mac.com?Subject=hello"]];
sms://调用短信
tel://调用电话
itms://打开MobileStore.app
{% endhighlight %}

31.获取版本信息

{% highlight ruby %}
UIDevice *myDevice = [UIDevice currentDevice];
NSString *systemVersion = myDevice.systemVersion;
{% endhighlight %}

32.UIWebView的使用

{% highlight ruby %}
<UIWebViewDelegate>
webView.delegate = self;
-(BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
NSURL *url = request.URL;
NSString *urlStirng = url.absoluteString;
NSLog(@"%@",urlStirng);
return YES;
}
{% endhighlight %}

33.NSNotificationCenter带参数发送

{% highlight ruby %}
MPMoviePlayerController *theMovie = [[MPMoviePlayerController alloc]initWithContentURL:[NSURL fileURLWithPath:moviePath]];
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(myMovieFinishedCallback:) name:MPMoviePlayerPlaybackDidFinishNotification object:theMovie];
[theMovie play];

- (void)myMovieFinishedCallback:(NSNotification *)aNotification {
MPMoviePlayerController *theMovie = [aNotification object];
[[NSNotificationCenter defaultCenter] removeObserver:self name:MPMoviePlayerPlaybackDidFinishNotification object:theMovie];
}
{% endhighlight %}

34.用NSDateFormatter调整时间格式代码

{% highlight ruby %}
NSDateFormatter *dateFormatter = [[NSDateFormatter alloc]init];
dateFormatter.dateFormat = @"yyyy-MM-dd HH:mm:ss";
NSString *currentDateStr = [dateFormatter stringFromDate:[NSDate date]];
{% endhighlight %}

35.UIView设置成圆角的方法

{% highlight ruby %}
mainView.layer.cornerRadius = 6;
mainView.layer.masksToBounds = YES;
{% endhighlight %}