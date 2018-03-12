---
layout: post
title: "使用CocoaPods管理第三方类库"
date: 2015-08-04
comments: true
categories: ios
tags: CocoaPods
keywords: CocoaPods
---
 在项目开发中，一些常用的第三方类库如AFNetworking，MBProgressHUD，ReactiveCocoa，Masonry，MJRefresh等，如何去管理它们呢，
 这些类库每次更新都要手动导入项目中吗？如何知道它们版本更新了呢？用CocoaPods解决以上问题。

###安装CocoaPods
* 查看是否安装了Ruby和git，ruby -v命令查看Ruby的版本，以及git --version
* 更新{% highlight ruby %}gemsudo gem update --system{% endhighlight %}
* 移除官方镜像源{% highlight ruby %}gem sources --remove https://rubygems.org/{% endhighlight %}
* 添加淘宝的镜像源{% highlight ruby %}gem sources -a http://ruby.taobao.org/{% endhighlight %}
* 验证gem sources -l，出现
![image](/images/Tool/tool_5.png)
才算成功。如果无法移除https://rubygems.org/,可以替换成gem sources --remove https://rubygems.org/

* 安装:{% highlight ruby %}sudo gem install cocoapods{% endhighlight %}
* 配置:{% highlight ruby %}pod setup{% endhighlight %}
* 查看第三方类库是否可用，比如pod search AFNetworking,如果能搜索到就表明这个第三库支持cocoapods 
![image](/images/Tool/tool_6.png)

###使用CocoaPods
* 进入项目所在目录, 输入vim Podfile(创建Podfile文件)，或者输入touch podfile然后open -e podfile，这时候就会用文本编辑器打开podfile文件供编辑
* 输入相关的第三方类库的信息，比如 pod 'MBProgressHUD', '~> 0.8'
 
  注意:保存vim文件方式是:先按ESC退出编辑模式，然后输入:wq(:q!表示不保存，强制退出),保存并退出
  
* 输入pod install，等待第三方库安装完成(cocoapods会列出安装的第三库和版本)
* 退出当前项目，从目录中打开xcworkspace后缀的和项目同名的文件，或者直接在终端输入open xxxxx.xcworkspace
* 注意#import如果用""无法导入，可以用<>代替
* 需要添加新的第三库的时候，需要编辑podfile文件，输入vim podfile，如果遇到输入无效的时候，左下角显示红色的字E353：Nothing in register，输入i或a即可进入编辑模式。
* 再次输入pod update，即可添加新的第三库

###注意事项
* 更新单独的库 pod update xxxx
*  删掉某个类库xxxx, 打开Podfile文件，删除xxxx该行,cd到当前项目目录下，重新执行pod install命令即可。
* 如果使用CocoaPods来添加第三方类库，无论是执行pod install还是pod update都卡在了Analyzing dependencies不动,原因在于当执行以上两个命令的时候会升级CocoaPods的spec仓库，加一个参数可以省略这一步，然后速度就会提升不少。加参数的命令如下：
{% highlight ruby %}
pod install --verbose --no-repo-update
{% endhighlight %}
{% highlight ruby %}
pod update --verbose --no-repo-update
{% endhighlight %}

*如果mac升级到10.11，cocoapods没了，使用命令行sudo gem install cocoa pods出错，换成sudo gem install -n /usr/local/bin cocoa pods即可

###备注
   参考文章[CocoaPods安装和使用教程](http://code4app.com/article/cocoapods-install-usage)