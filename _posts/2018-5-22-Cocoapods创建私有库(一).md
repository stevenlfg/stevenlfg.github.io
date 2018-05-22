# Cocoapods创建私有库(一)

---

随着我们公司的业务的扩展，项目不断增加，由原来的学生端增加至家长端，老师端等项目组。可复用代码的需求越来越大，同时组件化管理等标准流程也开始正式应用起来了。目前组件化管理用的比较多的就是Cocoapods，将项目中底层的上课音视频层从项目代码中抽离出来，组装成SDK，供其他业务端调用。本篇博客是我在探索制作私有库制作过程中的一些心得记录，仅供参考。

---
##一.创建私有Spec Repo  ##

1.Spec Repo类似一个容器，里面装有所有公开的Pods,当使用Cocoapods后，他就会被clone到本地的~/.cocoapods/repos目录下。终端执行以下命令查看：

    pod repo

![1@2x.png-48kB][1]


2.我是在[Coding][2]上创建的仓库,因为在GitHub上面创建私有库是收费的，所以选择了免费的git服务。在Coding上创建远程私有索引库，用了存放.podspec文件：
 
 ![2.png-60.8kB][3]
 
3.创建本地的私有索引库文件夹，并与远程的私有索引库进行关联

    pod repo add 本地文件名 sourceURL （上一步建好的远程仓库）
    
![3@2x.png-28.8kB][4]

之后再查看repos目录:已经多了一个我们添加的ZMTestSpec文件夹

![4@2x.png-59.6kB][5]

---

##二.创建一个用了存放项目组件的仓库##

![5.png-60.4kB][6]

1.在一个合适的目录下创建工程模板

  

    pod lib create ZMCommonBackButton
    
 ![6@2x.png-214.9kB][7]

2.以上信息配置好之后，会自动打开之前创建的测试工程，在工程目录下，我们要替换掉之前的ReplaceMe.m文件

![7@2x.png-111.2kB][8]  
  
3.替换完成之后，重新回到Example路径下，重新执行pod install操作

![8.png-24.9kB][9]

4.将能够运行的测试代码工程提交的远端代码管理仓库

![9@2x.png-138.5kB][10]
![10@2x.png-82.1kB][11]

此处有冲突，要解决冲突重新提交

![11@2x.png-141.1kB][12]

5.编辑模板工程的.podspec文件

![12.1@2x.png-71.1kB][13]

6.验证.podspec

![12@2x.png-112.9kB][14]

7.将podspec文件提交到本地的私有索引库

    pod repo push ZMTestSpec ZMCommonBackButton.podspec --allow-warnings
    
![13@2x.png-126.3kB][15]

8.提交成功的话，就可以用pod search ZMCommonBackButton查看自己的库了

![14@2x.png-59.2kB][16]

9.可以在Demo项目里面引用我们的私有库测试一下

![15@2x.png-43.1kB][17]
![16.png-18.5kB][18]

OK，此时大功告成了。

##三.总结##

测试的过程中，还是遇到了一些坑的，参考了一些[博客][19]，帮我梳理了整个流程。

1.关于spec repository 和code repository

中间有个过程是验证spec的，老是出错。最后发现是把spec文件和代码放在同一个git仓库里面了。
**code repository**：代码仓库。我们把包代码上传到这个仓库里。
**spec repository**：配置仓库。所有的配置按照包名、版本号分门别类的存放在这个仓库。注意： 这个仓库只用来存放spec文件，不存放代码。

2.提交代码工程的时候出错

此时一定要看清出错在哪里，有冲突解决冲突，然后在重新提交。不要盲目去解决问题。

3.关于编辑.podspec文件

s.source_files的文件路径一定要对比下你的工程项目目录，不然一直报错。



  [1]: http://static.zybuluo.com/stevenlfg/rbpvwnnbz766h5jkmva74yy8/1@2x.png
  [2]: https://coding.net
  [3]: http://static.zybuluo.com/stevenlfg/wbh6hnee3x4ix84fss1ik4fv/2.png
  [4]: http://static.zybuluo.com/stevenlfg/4t3ho2evhpu6fk33sgyb79r5/3@2x.png
  [5]: http://static.zybuluo.com/stevenlfg/dm8oojqh527f1ak57ogubnws/4@2x.png
  [6]: http://static.zybuluo.com/stevenlfg/w1w4lm91mz9roakljwuth5ty/5.png
  [7]: http://static.zybuluo.com/stevenlfg/pvgrm1dtmvwr4xd2rnht67hp/6@2x.png
  [8]: http://static.zybuluo.com/stevenlfg/99aabk5otbepflxd31gwoy54/7@2x.png
  [9]: http://static.zybuluo.com/stevenlfg/0z4byoe8ruiae0nfo2mlc9dq/8.png
  [10]: http://static.zybuluo.com/stevenlfg/ralnkbmmn8kccjodh12kgy8z/9@2x.png
  [11]: http://static.zybuluo.com/stevenlfg/6ylr3mukenum14ojv9l9jyyz/10@2x.png
  [12]: http://static.zybuluo.com/stevenlfg/g684owhxrv83ofe3dui8qz5t/11@2x.png
  [13]: http://static.zybuluo.com/stevenlfg/bvtj656gpof9v086iroixjhm/12.1@2x.png
  [14]: http://static.zybuluo.com/stevenlfg/7ty5ifs1btfi88cih7oega24/12@2x.png
  [15]: http://static.zybuluo.com/stevenlfg/a0m3kll632ppv88xbeiosgb6/13@2x.png
  [16]: http://static.zybuluo.com/stevenlfg/1qqu0z3kquot9w878vxto8wk/14@2x.png
  [17]: http://static.zybuluo.com/stevenlfg/kap60h5rzqgs5aju6l5pwggg/15@2x.png
  [18]: http://static.zybuluo.com/stevenlfg/38ecumzv5lgh7v7lpgbavelf/16.png
  [19]: https://imciel.com/2016/07/25/create-private-pods/