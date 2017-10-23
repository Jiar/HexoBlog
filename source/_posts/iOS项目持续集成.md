---
title: iOS项目持续集成
date: 2016-08-28 14:59:39
categories: iOS
tags:
- iOS
- 持续集成
- 打包
- Jenkins
- Shell
---

### 使用Jenkins持续集成

#### 使用Jenkins持续集成

最近老是看到关于使用Jenkins持续集成方面的文章，于是也去折腾了，稍作整理，这里分享下我是如何使用Jenkins持续集成iOS项目的。

##### 相关博文

* [fir.im weekly - 「 持续集成 」实践教程合集](http://blog.fir.im/fir_im_weekly160505/)
* [从自动化测试到持续部署，你需要了解这些](http://blog.fir.im/testing_cd/)
* [手把手教你利用Jenkins持续集成iOS项目](http://www.jianshu.com/p/41ecb06ae95f)
* [一步一步构建iOS持续集成:Jenkins+GitLab+蒲公英+FTP](http://www.jianshu.com/p/c69deb29720d)
* [*jenkins*iOS项目持续集成（SVN+Cocoapods+Workspace）实战扩展（修改版）](http://www.jianshu.com/p/269d8d66472d)
* [iOS使用Jenkins进行持续集成](http://www.jianshu.com/p/4ce6649973e6)
* [Jenkins+GitHub+Xcode+fir搭了一个持续集成环境](http://www.jianshu.com/p/a17167274463)
* [iOS持续集成简述](http://www.jianshu.com/p/f44d746ff8a9)
* [iOS项目自动打包(一)](http://xu01.github.io/ios/2016/04/26/iOS-automates-the-integration-1.html)
* [使用 jenkins 进行 iOS 项目持续集成与自动化构建](http://blog.ilovejuly.com/2015/06/20/use-jeninks-to-build-daily.html)
* [使用jenkins + git + 蒲公英 对 iOS 项目进行持续集成](http://tttpeng.com/2015/11/14/jenkins-1/)
* [iOS 下如何自动化打包 App](http://reviewcode.cn/article.html?reviewId=11)
* [Jenkins+Github+Testflight在Mac下搭建持续集成环境](http://www.itiger.me/?p=30)
* [Xcode&Jenkins持续集成的几种实现方式](http://huos3203.github.io/MyBlog/blog/2015/09/18/xcode-and-jenkinschi-xu-ji-cheng-de-ji-chong-shi-xian-fang-shi/)
* [Jenkins、Git、CocoaPods、Fir.im 实现 iOS 应用持续集成](http://www.linfuyan.com/continuous-integration-of-ios-application-with-jenkins-git-cocoapods-fir.im/)
* [Jenkins+Cocoapods+Coding+Git+Fir iOS项目持续集成](http://www.jianshu.com/p/1fe8652918df)
* [Jenkins/git/KeyChains & Provisioning, 记录CI中的一些坑](http://www.jianshu.com/p/e19c8327b167)
* [Jenkins+Cocoapods+Coding+Git+Fir iOS项目持续集成](http://www.jianshu.com/p/1fe8652918df)
* [Jenkins构建Android项目持续集成之创建项目](http://blog.csdn.net/it_talk/article/details/50261229)(这个虽然是安卓，但是参考了构建失败与构建成功分别发送给不同的接受者邮件的方法)
* [配置jenkins发送邮件](http://liuhongjiang.github.io/hexotech/2015/12/04/jenkins-send-email-after-build/)
* [Jenkins 邮件配置 (使用 Jenkins Email Extension Plugin)](http://www.cnblogs.com/GGHHLL/p/jenkins.html)
* [Jenkins进阶系列之——01使用email-ext替换Jenkins的默认邮件通知](http://www.cnblogs.com/zz0412/p/jenkins_jj_01.html)
* [Using OCLint with Jenkins CI](http://docs.oclint.org/en/stable/guide/jenkins.html)


<!--more-->


#### 使用shell命令

利用Xcode自带的命令xcodebuild、xcrun通过shell对iOS项目进行打包上传至第三方测试平台(fir.im、蒲公英等平台)

##### 相关博文

* [xcodebuild官方文档](https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man1/xcodebuild.1.html)
* [xcrun官方文档](https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man1/xcrun.1.html)
* [Shell教程](http://www.runoob.com/linux/linux-shell.html)
* [shell打包到fir/蒲公英](http://www.jianshu.com/p/5abbe0d61cef)


### Jenkins持续集成的理解

在你的mac上搭建好Jenkins，安装好相应的插件，在项目配置中关联代码仓库(git、svn)，你可以设置固定时间点检查仓库或者固定时间循环检查仓库的方式来检查你的代码仓库，如仓库有变动，则执行编译等操作。每次编译，首先会将仓库中的代码update到本地，然后在本地编译，因此你可以使用本机的所有命令。

如果你的iOS项目使用了`Cocoapods`，建议在提交代码的时候`ignore`如下:`Pods文件夹、`Podfile.lock文件`、`XXX.xcworkspace文件`。如下图：

![ignore](ignore.jpeg)

这样做的原因是pod来的代码，本来就是在第三方仓库，没必要再拉到自己的仓库里面占空间，这样设置之后，需要在`Jenkins`的`构建`步骤添加`Execute shell`操作，并输入脚本`pod install --verbose --no-repo-update`。如下图：

![ExecuteShell](ExecuteShell.jpeg)


### 我踩过的坑

折腾这东西还真没少踩过坑，在这里分享一下，给入坑的同学提供一个解决的思路。

#### Keychains and Provisioning Profiles Management

iOS打包需要签名文件和证书，所以我们借助了这个插件`Keychains and Provisioning Profiles Management`，然而你会发现，我上面提供的博文中，人家的`Keychains and Provisioning Profiles Management`插件设置中的最下方有`Filename`和`UUID`这两个选项，如下图：

![HaveFileNameAndUUID](HaveFileNameAndUUID.png)

而你的设置中却没有，如下图：

![NoFileNameAndUUID](NoFileNameAndUUID.jpeg)

这里要注意下，我们要先上传自己的`login.keychain`文件，才能在下面的`Keychains`中显示出来。但是你点击图片中最上方的`选择文件`按钮时，你会发现你要选择的`login.keychain`文件在这个路径下面(`/Users/Jiar/Library/Keychains`)，你根本无法选择到，这里一个简单的操作就是先把`login.keychain`拷贝到桌面再上传这个文件。


进入Jenkins的全局设置，如下图：

![ProvisioningProfilesDirectoryPath](ProvisioningProfilesDirectoryPath.jpeg)

这里设置好签名文件以及证书路径后，到时候在项目中引用这个路径即可，如下图，在项目中引用这里设置的路径。（首先你要在项目的`构建`步骤里面添加`Xcode`步骤，然后在`Xcode`的`Code signing & OS X keychain options`项里面勾选`Unlock Keychain?`，进行配置路径）

![CodeSigning&OSXKeychainOptions](CodeSigning&OSXKeychainOptions.jpeg)

上面的步骤中，我们已经设置好了签名文件和证书的路径，也上传了`login.keychain`文件，通过这个文件可以为项目选择签名文件对应的证书。同时也在`Xcode`中设置了`Unlock Keychain`的path（path实际路径我们在系统设置中设置好了，这里只是一个引用），同时输入了unlock密码（就是你mac的登录密码）。这些做好了，还差一步，那就是我们需要告诉这个项目，选择那个签名文件，才可以通过签名文件找到对应的证书。我们在项目的`构建环境`中勾选`Keychains and Code Signing Identities`，然后执行以下操作，如下图：

![KeychainsAndCodeSigningIdentities](KeychainsAndCodeSigningIdentities.jpeg)


#### 上传至第三方测试平台

这里的第三方测试平台主要是[fir.im](http://fir.im)、[蒲公英](https://www.pgyer.com/)

##### 上传至fir.im
首先你得先安装fir.im提供的上传工具`fir-cli`，使用如下命令安装：`sudo gem install fir-cli --no-ri --no-rdoc`。
你可以通过两种方式来使用这个工具上传app至fir.im。
- 使用fir.im为Jenkins提供的插件。[教程在这里](http://blog.fir.im/jenkins/)
- 直接用命令来上传，命令如下：`fir p ${WORKSPACE}/build/TestJenkins.ipa -T #API Token#`

##### 上传至蒲公英
这里提供一个蒲公英官方api，[要看点这里](https://www.pgyer.com/doc/api#uploadApp)


#### 邮件通知
首先要知道Jenkins自带一个邮件通知，但是无法自定义邮件内容的样子，然后都会使用一个第三方插件`Extended E-mail Notification`。

首先进入到系统设置里面，找到`邮件通知`，我这里使用的是163邮箱，网易163免费邮箱相关服务器信息如下图：

![163EmailHelp](163EmailHelp.png)

我们会发现使用`SMTP`服务，如果启用`SSL`协议，则有`465`或`994`两个端口可以选择。如果不用这个协议，则使用`25`端口。

我的设置如下图：

![邮件通知](邮件通知.jpeg)

注意了，这里的密码不能使用你的邮箱登录密码，原因是163邮箱第三方登录需要设置授权码，同时，我们使用`SMTP`服务，也需要去邮箱设置好先，不然会不成功。如下两张图，分别对`POP3/SMTP/IMAP`和`客户端授权密码`进行设置（没办法，为了安全，只好那么麻烦，记得把设置的授权码作为密码填入到`邮件通知`的密码栏里面去）

![POP3SMTPIMAP](POP3SMTPIMAP.jpeg)

![客户端授权密码](客户端授权密码.jpeg)

这些都设置好后，不出意外没什么问题了，点击`邮件通知`下面的`通过发送测试邮件测试配置`，填入一个邮箱测试一下是否成功。

如果成功了，那么恭喜你，你可以把`邮件通知`这一块丢弃了，哈哈，因为刚才说了，我们不使用
Jenkins自带的邮件通知服务，“那你还让我们这么配置干啥？逗我们玩？” 别急，这里主要是用了Jenkins自带的邮件通知服务进行邮箱测试，如果这里成功了，再把这些内容配置到第三方插件`Extended E-mail Notification`中，那就可以了。好了，接下来看`Extended E-mail Notification`，同样的，在Jenkins的系统设置中找到`Extended E-mail Notification`，由于`Extended E-mail Notification`内容很长，我分图片显示：

![ExtendedE-mailNotificationPart1](ExtendedE-mailNotificationPart1.jpeg)

![ExtendedE-mailNotificationPart2](ExtendedE-mailNotificationPart2.jpeg)

![ExtendedE-mailNotificationPart3](ExtendedE-mailNotificationPart3.jpeg)

在这个链接里面有邮件内容的说明：[Jenkins进阶系列之——01使用email-ext替换Jenkins的默认邮件通知](http://www.cnblogs.com/zz0412/p/jenkins_jj_01.html)。当然，你可以查看`Extended E-mail Notification`自带的说明，在系统设置中找到`Extended E-mail Notification`，滚动到`Extended E-mail Notification`的底部，找到`Content Token Reference`选项，点击它最右边的问号图标，即可展开内容。如下图所示：

![ContentTokenReference](ContentTokenReference.jpeg)

在系统设置中设置好默认值后，打开项目设置，在`构建后操作`中添加`Editable Email Notification`。配置如图所示：

![EditableEmailNotificationStep1](EditableEmailNotificationStep1.jpeg)

![EditableEmailNotificationStep2](EditableEmailNotificationStep2.jpeg)

![EditableEmailNotificationStep3](EditableEmailNotificationStep3.jpeg)

展开 `Failure-Any`

![EditableEmailNotificationStep4](EditableEmailNotificationStep4.jpeg)

展开 `Success`

![EditableEmailNotificationStep5](EditableEmailNotificationStep5.jpeg)


到此如何在Jenkins中使用邮件服务就已经配置好了，这里再次强调一点。我们的邮件服务使用的是`Extended E-mail Notification`插件，而不是Jenkins自带的邮箱服务，也就是说，不要在`构建后操作`中添加`E-mail Notification`操作。


### 结束语

以上就是我在使用Jenkins持续集成iOS项目中的分享，感谢我在文章中提到的博文的博主的分享。读者在阅读本文时如有发现错误或不恰当指出，欢迎在评论中之处。如果读者还有一些相关方面的疑问，也欢迎在评论中提出。


欢迎关注我的个人微信订阅号，我将不定期分享开发方面的干货。

![Jiar's 微信订阅号](Dingyuehao.jpg)

