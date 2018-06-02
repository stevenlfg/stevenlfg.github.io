



---

接着上面文章，用Cocoapods创建私有库,完成了我们自己的私有库制作，在接下来的开发工作中，可能会根据需求去更新我们的私有库，本篇文章分享下更新私有库的经历。

##一.更新##

具体做法是将要添加的源文件放到Pod/Classes中，然后编辑.podspec文件。

![1@2x.png-63.4kB][1]


![8@2x.png-44.6kB][2]

编程完成后，本地验证下

![3@2x.png-80.4kB][3]

本地验证通过之后，就可以推送到远端了。这里面有个坑，就是要把push tag,由于我漏了这一步，出现了一下问题。

![4@2x.png-134.3kB][4]

可以尝试如下代码

git tag -m "update tag" 1.0.1

git push --tags 

然后再重新执行pod repo push ** **.podspec --allow-warnings

![5@2x.png-261.4kB][5]

##二.查看sepc##

上面的更新操作完成之后，就可以在~/.cocoapods/repos目录下查看

![6@2x.png-39kB][6]

执行pod search 查看

![7@2x.png-57.8kB][7]

如果想要删除一个私有Spec Repo，只需要执行一条命令即可

pod repo remove **

当然我们还可以通过一下命令添加回来。

pod repo add ** git@coding.net:**/**.git



  [1]: http://static.zybuluo.com/stevenlfg/4l9vywh1r0p5edsdb0ok7rzh/1@2x.png
  [2]: http://static.zybuluo.com/stevenlfg/btctktd27do1y1fs2nbx3ef0/8@2x.png
  [3]: http://static.zybuluo.com/stevenlfg/d3e2copthpjog84duo1aw0yq/3@2x.png
  [4]: http://static.zybuluo.com/stevenlfg/tfhm97de70wjg0jmzesceqsx/4@2x.png
  [5]: http://static.zybuluo.com/stevenlfg/ji43jcgs62gejrrr909a1soj/5@2x.png
  [6]: http://static.zybuluo.com/stevenlfg/s9neo1oj6t6gf3ngh7851pgu/6@2x.png
  [7]: http://static.zybuluo.com/stevenlfg/ic3a08v516u570kbiwymyl7t/7@2x.png