---
title: 适配Xcode9.0-beta与Swift4.0
date: 2017-06-09 15:12:09
categories: iOS
tags:
- iOS
- Swift4
- Xcode9.0-beta
- OC(Objective-C)与Swift4混编
- OC(Objective-C)调用Swift4
---

# 适配Xcode9.0-beta与Swift4.0

[简书阅读地址在这里](http://www.jianshu.com/p/1f702d59e54b)

这几天苹果在开`WWDC2017`大会，期间放出了`Xcode9.0-beta`以及`Swift4`。为了响应苹果爸爸的号召，我果断下载了`Xcode9.0-beta`，并在项目中拉出了新的分支，准备搞事。

![Xcode9.0-beta](Xcode9.0-beta-icon.png)

<!--more-->

## 如何适配

`Xcode9.0-beta`内置的`Swift`版本不止一个，它同时支持`Swift4.0`和`Swift3.2`。而我们正在用的`Xcode8`，最高只支持`Swift3.1`。基于这个事实，我先拉一个`Xcode9.0-beta-Swift3.2`的分支，待适配好`Swift3.2`后，再起分支`Xcode9.0-beta-Swift4.0`去支持`Swift4.0`。

### 适配`Swift3.2`

首先，对于`Swift3.2`，我的理解是：既然版本命名为`3.2`，那么应该只是基于`3.1`版本上的微调（我去查`Swift`，查到更多的是关于`Swift4.0`方面的信息）。适配`Swift3.2`的过程中，我的项目代码不需要任何改动，唯一出问题的是一个第三方库：[Eureka](https://github.com/xmartlabs/Eureka)，报错的原因是`Collection`协议的`subscript`返回值从`Array`变成了`ArraySlice`，关于这个问题，已有人在[Eureka](https://github.com/xmartlabs/Eureka)的issues中提出([#1082](https://github.com/xmartlabs/Eureka/issues/1082))。随后有人[commit](https://github.com/xmartlabs/Eureka/commit/89b0326fe79aeec1f9fef90a4f57c95bd1931089)修复了这个问题，并开出新分支来适配`Swift3.2`。

![Eureka-commit](Eureka-commit.jpeg)

最后，我在`Podfile`中修改`pod 'Eureka'`为`pod 'Eureka', :git => 'https://github.com/xmartlabs/Eureka.git', :branch => 'swift3.2'`，完成了适配`Swift3.2`。

由此可见，适配`Swift3.2`几乎是没有什么压力的，我也就看到`Collection`协议的`subscript`返回值变动这个情况。

### 适配`Swift4.0`

并不是所有库都能做到及时支持`Swift4.0`，更何况是在现在连`Xcode9`也还是`beta`的状态，所以我们仅能做到将自己的业务代码（主工程代码）部分升级到`Swift4.0`，然后同时保留各种`pod`库在`Swift3.2`版本。没办法，谁叫`Swift4.0`也还无法做到`ABI`兼容呢（但愿能在`Swift5`之前实现吧）。至于我说的同时使用两个版本的`Swift`，这是没问题的，`Xcode9`支持在项目中同时使用`Swift3.2`和`Swift4.0`。

#### 具体要怎么做呢？(修改`Swift`版本)

第一步，如下图指定主工程的`Swift`版本为`4.0`
![Project-Build-Settings-Swift-Language-Version](Project-Build-Settings-Swift-Language-Version.png)
第二步，如下所示，在`Podfile`文件的最下方加入如下代码，指定`pod`库的`Swift`版本为`3.2`(这样会使得所有的第三方`pod`库的`Swift`版本都为`3.2`)
```
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['SWIFT_VERSION'] = '3.2'
    end 
  end
end
```
#### 做完以上处理，剩下的就是主工程中的代码修改了。

从`Swift3.2`到`Swift4.0`的过程，比从`Swift3.1`到`Swift3.2`的过程要麻烦一点，但是比当年从`Swift2.3`到`Swift3`的过程要好太多了。

下面我列举一下`Swift3.2`到`Swift4.0`的改变(只是我项目中遇到的)：

- `Swift4.0`中对于扩展的属性(包括实例属性、`static`属性、`class`属性)，都只能使用`get`方法，不可使用`set`方法
- `Swift4.0`中不再允许复写扩展中的方法(包括实例方法、`static`方法、`class`方法)
- `swift3`使用`#selector`指定的方法，只有当方法权限为`private`时需要加`@objc`修饰符，现在全都要加`@objc`修饰符
- 字体方面的一些重命名(`NSFontAttributeName`重命名为`NSAttributedStringKey.font`、`NSForegroundColorAttributeName`重命名为`NSAttributedStringKey.foregroundColor`、`NSStrikethroughStyleAttributeName`重命名为`NSAttributedStringKey.strikethroughStyle`、`size(withAttributes:)`方法重命名为`size(withAttributes:)`)
- ...

#### `OC`与`Swift4.0`混编才是坑

由于历史原因，我负责的项目，还有好大一部分`OC`的代码，新写的`Swift`需要被`OC`调用。所以，问题来了...

#####  `OC`调用`Swift4.0`问题一：编译不通过

我在`Swift4`的代码中写了不少`class`和`extension`，有些也给`OC`调用。在`OC`的代码中，我们通过`#import "ModuleName-Swift.h"`导入了`Swift`文件，以给`OC`调用。如果是`Swift3.2`，一切都能正常工作，但是在`Swift4.0`上，编译通不过了。

一：在`OC`中调用一个`Swift4.0`类的方法（包括实例方法、`static`方法、`class`方法），你需要：

- 在该`Swift4.0`类前加上修饰符`@objc`
- 该`Swift4.0`类必须继承`NSObject`(否则，无法在前面加上修饰符`@objc`。当然，这里指的是普通类，`@objc`也是可以修饰`UI`开头的一系列`UIKit`框架下的`UI`类，只是修饰了这些类，不会产生什么影响)
- 在需要调用的方法前加上修饰符`@objc`
  示例如下：

```
@objc class SampleObject: NSObject {

    @objc func sampleFunc  {
        print("sampleFunc")
    }
    
    @objc static func sampleStaticFunc  {
        print("sampleStaticFunc")
    }
    
    @objc class func sampleClassFunc  {
        print("sampleClassFunc")
    }
    
```

如此一来，便可在`OC`文件中调用，示例如下：

```
#import "OCSample.h"
#import "ModuleName-Swift.h"

@implementation OCSample

- (void)callSwiftFunc {
    // 调用实例方法
    SampleObject *object = [[SampleObject alloc] init];
    [object sampleFunc];
    // 调用static方法
    [SampleObject sampleStaticFunc];
    // 调用class方法
    [SampleObject sampleClassFunc];
}

@end
```

二：在`OC`中调用一个`Swift4.0`扩展的属性（包括实例属性、`static`属性、`class`属性）、方法（包括实例方法、`static`方法、`class`法），你有如下两种选择方式：

- 在该`Swift4.0`扩展前加上修饰符`@objc`(这样的话，该扩展下的所有的属性、方法，都可被`OC`调用)。

示例如下：

```
@objc extension UIViewController {

    var name: String {
        reutrn "name"
    }
    
    static var staticName: String {
        reutrn "staticName"
    }
    
    class var className: String {
        reutrn "className"
    }
    
    func nameFunc() {
        print("nameFunc")
    }
    
    static func staticNameFunc() {
        print("staticNameFunc")
    }
    
    class func classNameFunc() {
        print("classNameFunc")
    }
    
}
```

- 在需要的属性、方法前直接加上`@objc`修饰，也可达到目的。

示例如下：

```
extension UIViewController {

    @objc var name: String {
        reutrn "name"
    }
    
    @objc static var staticName: String {
        reutrn "staticName"
    }
    
    @objc class var className: String {
        reutrn "className"
    }
    
    @objc func nameFunc() {
        print("nameFunc")
    }
    
    @objc static func staticNameFunc() {
        print("staticNameFunc")
    }
    
    @objc class func classNameFunc() {
        print("classNameFunc")
    }
    
}
```

#####  `OC`调用`Swift4.0`问题二：运行时找不到属性

这个问题藏得比较深，恰巧项目中有着相关的实现，让我看出发现这个潜在因素。
项目中有这么一种实现：有一个`Swift4.0`的类，是继承`UIViewController`的。然后我在`OC`里面对这个继承而来的`UIViewController`进行操作，我用了`[viewController valueForKey:@"iconURL"]`这一`KVC`方法去获取这个自定义`UIViewController`中的`iconURL`这一属性的属性值。这种方式，编译时是无法检查出问题的。但是在运行时，问题就来了，找不到这个属性。因为这个属性没有暴露给`OC`来进行调用。

解决方式：仅需要在自定义的`UIViewController`类中给需要暴露给`OC`调用的属性前加上`@objc`修饰符便可。如此一来，在`OC`代码中就能访问到这个属性。(注意：这里可不像上面提到的`extension`一样，在这个已定义的`UIViewController`类前面加上`@objc`修饰符没有任何意义)。

示例如下：
```
class SampleViewController: UIViewController {
    @objc var iconURL: String?
}
```

除了在`OC`里通过`valueForKey:`方法调用到一些未经过`@objc`修饰的`Swift4.0`的`UI`类的属性会导致`crash`。其他比如你在`Swift4.0`代码中，通过`setValuesForKeys`这种通过`KVC`来操作未经过`@objc`修饰的属性，也会导致`crash`。

##### 关于混编方面的更多信息

更多关于混编方面的内容，可以访问查看Apple官方提供的这篇文章：[Using Swift with Cocoa and Objective-C (Swift 4)](https://developer.apple.com/library/content/documentation/Swift/Conceptual/BuildingCocoaApps/index.html#//apple_ref/doc/uid/TP40014216-CH2-ID0)，篇幅不少，不单单介绍了`Swift4.0`与`OC`的混用，也介绍了与`C`的`api`的交互、还有更多关于`@objc`修饰符的用法。

## 关于`Xcode9-beta`的更多

### `Xcode9-beta`局域网调试

#### 要求

- 必须是`Xcode9-beta`
- `iPhone`系统需`iOS11`以上

#### 操作

1. 在`Xcode9-beta`菜单的`Window`选项中选择`Devices and Simulators`
2. 通过连接线让你的`Mac`识别到你的`iPhone`
3. 在`Devices and Simulators`面板的左侧`Connected`菜单中选择连接的设备，然后在顶部的`Devices`和`Simulators`选项中选择`Devices`(这里其实默认就是选择了`Devices`)，最后勾选`Connect via network`选项。

来自[`stackoverflow`回答](https://stackoverflow.com/questions/44382841/how-to-do-wireless-debug-on-xcode-9-and-ios-11?answertab=votes#tab-top)

## 结束语

### 关于本文

- 本文为作者这几天在`Xcode9-beta`以及`Swift4.0`方面的学习记录与分享，作者会视情况对内容进行补充。
- 如果您在阅读本文中发现内容存在错误，希望您积极指出。如果您有其他建议，也欢迎在评论去区留言。
- 作者接受指正，但是希望彼此之间保留敬意。
- 欢迎转载，但请保留博文的原地址或者博文在简书上的地址。

### 关于本人

比起 [微博@Jiar](https://weibo.com/u/2268197591/) ，更喜欢 [推特@JiarYoo](https://twitter.com/JiarYoo/) ，求一波关注。😝

### 微信订阅号

欢迎关注我的个人微信订阅号，我将不定期分享开发方面的干货。

![Jiar's 微信订阅号](/images/Dingyuehao.jpg)

