---
title: JRAlertController
date: 2016-11-11 11:11:11
categories: iOS
tags:
- iOS
- Swift
- JRAlertController
- UIAlertController
- alert
- sheet
---

[JRAlertController](https://github.com/Jiar/JRAlertController/)：基于apple的UIAlertController控件api，用swift重新打造的UI控件，更符合主流app的风格。

### JRAlertController总体效果图

![UIAlertController_Main](https://raw.githubusercontent.com/Jiar/JRAlertController/master/Screenshot/JRAlertController_Main.gif)

<!--more-->

### UIAlertController 历史

在日常iOS开发中，我们经常能遇到这种情况（我们需要在某个地方让用户做一个选择），比如说：一个博客，点击右上角的按钮后，你可以执行“修改博客”、“删除博客”两个操作。既然是这么常用的操作，Apple当然给我们提供了常用的控件，那就是UIAlertController。

`UIAlertController`自iOS8出现，在那之前，我们用的是`UIAlertView`和`UIActionSheet`。iOS8之后，`UIAlertView`与`UIActionSheet`合并为`UIAlertController`，并以一个`style`属性来区分原来的`UIAlertView`和`UIActionSheet`的作用，还有就是用闭包回调的方式代替了之前的代理（我觉得闭包回调的方式写起来方便多了）。

### UIAlertController 不足

那么既然`UIAlertController`已经是在iOS8优化后的控件（至少api上优化了），那么为何还来个`JRAlertController`呢？

---

在开发中我们会发现，`UIAlertController`有以下几个不足之处：

1.无论是`alert`还是`sheet`下的界面，边角过于圆滑，尤其当`style`是`sheet`的时候，从底部弹出来那么一个过于圆滑的界面，反正我不觉得好看，不信你看微博、微信这些主流app是怎么做的
微博的效果：
![weibo](weibo.jpeg)

2.点击背景部分，无法`dismiss UIAlertController`

3.`alert`样式下，添加过多的`UITextField`和`Action`后，界面显示丑陋。（虽然不会有这种需求，也不该在`UIAlertController`过量添加，毕竟`UIAlertController`适用于"短暂"操作，但是过多添加后，界面确实不好看，后面会有效果图）

基于以上几点不足，我认为足以自定义一个控件来代替`UIAlertController`，所以`JRAlertController`诞生了。

### JRAlertController 与 UIAlertController 的不同

我在开头提到过`UIAlertController`的`api`,它的`api`还是不错的，所以我在写`JRAlertController`的时候，几乎完全采用了`UIAlertController`的`api`，一方面`api`不错，另一方面方便大家从`UIAlertController`迁移到`JRAlertController`，基本上你只需要把原来`UIAlertController`部分的`UI`开头的改成`JR`就可以了，我提供的Demo中，大家便能很清晰的看到这一点。

---

##### 还有的几处不同点：

1.
```
	// UIAlertController 的初始化方法
	public convenience init(title: String?, message: String?, preferredStyle: UIAlertControllerStyle)

	// JRAlertController 的初始化方法
	public convenience init(title: String? = nil, message: String? = nil, preferredStyle: JRAlertControllerStyle = .actionSheet)
```
你会发现，`JRAlertController`提供了`title`参数和`message`参数的默认值，恩......如果你不需要`title`或`message`中的某一个（或都不需要），这样可以帮你少写一点点代码。

2.
```
	// UIAlertController进入方法
	// 这里的alertController为UIAlertController的实例，self为当前UIViewController
	self.present(alertController, animated: true, completion: nil)

	// JRAlertController进入方法
	// 这里的alertController为JRAlertController的实例，self为当前UIViewController
	alertController.jr_show(onRootView: self)
```
至于这里为什么要做，这里涉及到`JRAlertController`从底部上移的动画效果。（目前我能想到的方法是在`JRAlertController`的`viewWillAppear`中执行一个上移动画，如果调用系统的`present`进入的话，如果又给animated参数设置true，那么`JRAlertController`的进入效果会比较丑陋。可能有朋友会问，为什么不使用iOS的转场动画，我有尝试去用过，但是也需要第一个执行`present`的`UIViewController`做很多其他的动作，写更多的代码。如果有朋友看了我的代码后，有更好的方式来处理进入效果，欢迎到[本项目的Github地址](https://github.com/Jiar/JRAlertController/)来`Pull requests`）

3.
`UIAlertController`里面有一个属性`preferredAction`，要求系统版本至少为iOS9。而在`JRAlertController`中，你只需要在iOS8下就可以使用了（如果不是`UIViewController`的`modalPresentationStyle`属性的`.overCurrentContext`值要求iOS8，我们就可以兼容到iOS7了）


### JRAlertController 与 UIAlertController 相同点以及说明

因为`JRAlertController`是采用几乎和`UIAlertController`一样的`api`来实现的，所以`JRAlertController`的大体功能效果会和`UIAlertController`一样。同时也是为了方便打算使用`JRAlertController`的朋友们能够在迁移到`JRAlertController`的时候没有后顾之忧。

---

##### 他们的相同点说明：

1.只有`alert`样式下，才可以添加`UITextField`。

2.`action`也有样式，但是`cancel`样式的`action`只能添加一个，添加多了`assert`。

3.`preferredAction`属性也只能在`alert`样式下才能使用。

以上三点别问我为什么这么规定，Apple的`UIAlertController`就是这么定的。

---

##### 其他说明：

1.`JRAlertController`的`preferredAction`属性补充说明：除了只能在`alert`样式下才能使用外，如果存在`UITextField`，那么在`UITextField`列表的最后一个`UITextField`的键盘中点击`return`按钮，将触发`preferredAction`回调，并且`dismiss`当前`JRAlertController`。

2.`JRAlertController`里面会有一个属性是这样的：`open var textFields: [UITextField]?`。所有你添加的`UITextField`都会在这里。注意的是，`JRAlertController`已经对所以添加的`UITextField`进行了代理操作。如果你覆盖了代理，影响也不是很大，但是至少会影响你以下两点：

①.在`UITextField`中点击键盘上`return`按钮时无法从当前`UITextField`进入到下一个`UITextField`。
②.如果当前`UITextField`已经是最后一个`UITextField`，同时你又设置了`preferredAction`，则无法触发`preferredAction`的回调，以及无法`dismiss`当前`JRAlertController`。

3.如果需要在`JRAlertController`里，主动`dismiss`，建议调用`jr_dismiss()`。

### JRAlertController 与 UIAlertController Gif图效果对比

说了那么多，来几张效果图对比下`JRAlertController`与`UIAlertController`的区别


#### JRAlertController 实现效果图

##### JRAlertController在alert样式下简单显示
![JRAlertController_alert_simple](https://raw.githubusercontent.com/Jiar/JRAlertController/master/Screenshot/JRAlertController_alert_simple.gif)

##### JRAlertController在alert样式下复杂显示
![JRAlertController_alert_multiple](https://raw.githubusercontent.com/Jiar/JRAlertController/master/Screenshot/JRAlertController_alert_multiple.gif)

##### JRAlertController在sheet样式下简单显示
![JRAlertController_sheet_simple](https://raw.githubusercontent.com/Jiar/JRAlertController/master/Screenshot/JRAlertController_sheet_simple.gif)

##### JRAlertController在sheet样式下复杂显示
![JRAlertController_sheet_multiple](https://raw.githubusercontent.com/Jiar/JRAlertController/master/Screenshot/JRAlertController_sheet_multiple.gif)

#### UIAlertController 实现效果图

##### UIAlertController在alert样式下简单显示
![UIAlertController_alert_simple](https://raw.githubusercontent.com/Jiar/JRAlertController/master/Screenshot/UIAlertController_alert_simple.gif)

##### UIAlertController在alert样式下复杂显示
![UIAlertController_alert_multiple](https://raw.githubusercontent.com/Jiar/JRAlertController/master/Screenshot/UIAlertController_alert_multiple.gif)

##### UIAlertController在sheet样式下简单显示
![UIAlertController_sheet_simple](https://raw.githubusercontent.com/Jiar/JRAlertController/master/Screenshot/UIAlertController_sheet_simple.gif)

##### UIAlertController在sheet样式下复杂显示
![UIAlertController_sheet_multiple](https://raw.githubusercontent.com/Jiar/JRAlertController/master/Screenshot/UIAlertController_sheet_multiple.gif)

以上就是对`JRAlertController`的一些说明，下面放是官方性信息：

---


### Requirements

- iOS 8.0+
- Xcode 8.0+
- Swift 3.0+

### Installation

#### CocoaPods

[CocoaPods](http://cocoapods.org) is a dependency manager for Cocoa projects. You can install it with the following command:

```bash
$ gem install cocoapods
```

> CocoaPods 1.1.0+ is required to build JRAlertController 1.0.0

To integrate JRAlertController into your Xcode project using CocoaPods, specify it in your `Podfile`:

```ruby
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '8.0'
use_frameworks!

target '<Your Target Name>' do
    pod 'JRAlertController', '~> 1.0。0'
end
```

Then, run the following command:

```bash
$ pod install
```

#### Manually

If you prefer not to use either of the aforementioned dependency managers, you can integrate JRAlertController into your project manually.

##### Embedded Framework

- Open up Terminal, `cd` into your top-level project directory, and run the following command "if" your project is not initialized as a git repository:

  ```bash
$ git init
```

- Add JRAlertController as a git [submodule](http://git-scm.com/docs/git-submodule) by running the following command:

  ```bash
$ git submodule add https://github.com/Jiar/JRAlertController.git
```

- Open the new `JRAlertController` folder, and drag the `JRAlertController.xcodeproj` into the Project Navigator of your application's Xcode project.

    > It should appear nested underneath your application's blue project icon. Whether it is above or below all the other Xcode groups does not matter.

- Select the `JRAlertController.xcodeproj` in the Project Navigator and verify the deployment target matches that of your application target.
- Next, select your application project in the Project Navigator (blue project icon) to navigate to the target configuration window and select the application target under the "Targets" heading in the sidebar.
- In the tab bar at the top of that window, open the "General" panel.
- Click on the `+` button under the "Embedded Binaries" section.
- You will see two different `JRAlertController.xcodeproj` folders each with two different versions of the `JRAlertController.framework` nested inside a `Products` folder.

    > It does not matter which `Products` folder you choose from, but it does matter whether you choose the top or bottom `JRAlertController.framework`.

- Select the top `JRAlertController.framework` for iOS.
- And that's it!

  > The `JRAlertController.framework` is automagically added as a target dependency, linked framework and embedded framework in a copy files build phase which is all you need to build on the simulator and a device.

---

### Usage

#### JRAlertController_alert_simple
```swift
        let alertController = JRAlertController(title: "login tip", message: "please input account and password", preferredStyle: .alert)
        let cancelAction = JRAlertAction(title: "cancel", style: .cancel, handler:  {
            (action: JRAlertAction!) -> Void in
            print("cancel")
        })
        let loginAction = JRAlertAction(title: "login", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("login")
        })
        alertController.addAction(cancelAction)
        alertController.addAction(loginAction)
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.keyboardType = .default
            textField.placeholder = "please input account"
        })
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.keyboardType = .default
            textField.isSecureTextEntry = true
            textField.placeholder = "please input password"
        })
        // must use this function to show JRAlertController
        // self is a UIControllerView
        alertController.jr_show(onRootView: self)
```

#### JRAlertController_alert_multiple
```swift
        let alertController = JRAlertController(title: "I am title,I am title,I am title,I am title,I am title", message: "I am message, I am message, I am message, I am message, I am message, I am message, I am message, I am message, I am message", preferredStyle: .alert)
        let cancelAction = JRAlertAction(title: "cancel", style: .cancel, handler:  {
            (action: JRAlertAction!) -> Void in
            print("cancel")
        })
        let deleteAction = JRAlertAction(title: "delete", style: .destructive, handler: {
            (action: JRAlertAction!) -> Void in
            print("delete")
        })
        let archiveAction = JRAlertAction(title: "archive", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive")
        })
        let archiveAction1 = JRAlertAction(title: "archive1", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive1")
        })
        let archiveAction2 = JRAlertAction(title: "archive2", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive2")
        })
        let archiveAction3 = JRAlertAction(title: "archive3", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive3")
        })
        let archiveAction4 = JRAlertAction(title: "archive4", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive4")
        })
        let archiveAction5 = JRAlertAction(title: "archive5", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive5")
        })
        let archiveAction6 = JRAlertAction(title: "archive6", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive6")
        })
        alertController.addAction(cancelAction)
        alertController.addAction(deleteAction)
        alertController.addAction(archiveAction)
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.textColor = .black
            textField.text = "black"
        })
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.textColor = .darkGray
            textField.text = "darkGray"
        })
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.textColor = .lightGray
            textField.text = "lightGray"
        })
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.backgroundColor = .black
            textField.textColor = .white
            textField.text = "white"
        })
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.textColor = .gray
            textField.text = "gray"
        })
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.textColor = .red
            textField.text = "red"
        })
        alertController.addAction(archiveAction1)
        alertController.addAction(archiveAction2)
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.textColor = .green
            textField.text = "green"
        })
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.textColor = .blue
            textField.text = "blue"
        })
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.textColor = .cyan
            textField.text = "cyan"
        })
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.textColor = .yellow
            textField.text = "yellow"
        })
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.textColor = .magenta
            textField.text = "magenta"
        })
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.textColor = .orange
            textField.text = "orange"
        })
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.textColor = .purple
            textField.text = "purple"
        })
        alertController.addTextField(configurationHandler: { (textField: UITextField) -> Void in
            textField.textColor = .brown
            textField.text = "brown"
        })
        alertController.addAction(archiveAction3)
        alertController.addAction(archiveAction4)
        alertController.addAction(archiveAction5)
        alertController.addAction(archiveAction6)
        alertController.preferredAction  = archiveAction6
        // must use this function to show JRAlertController
        // self is a UIControllerView
        alertController.jr_show(onRootView: self)
```

#### JRAlertController_sheet_simple
```swift
        let alertController = JRAlertController(title: "blog tip", message: "Please select the option to use the corresponding option to operate your blog", preferredStyle: .actionSheet)
		// let alertController = JRAlertController(title: "blog tip")
		// let alertController = JRAlertController(message: "Please select the option to use the corresponding option to operate your blog")
		// let alertController = JRAlertController()
        let addAction = JRAlertAction(title: "add", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("add blog")
        })
        let modifyAction = JRAlertAction(title: "modify", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("modify blog")
        })
        let deleteAction = JRAlertAction(title: "delete", style: .destructive, handler: {
            (action: JRAlertAction!) -> Void in
            print("delete blog")
        })
        let cancelAction = JRAlertAction(title: "cancel", style: .cancel, handler:  {
            (action: JRAlertAction!) -> Void in
            print("cancel")
        })
        alertController.addAction(addAction)
        alertController.addAction(modifyAction)
        alertController.addAction(deleteAction)
        alertController.addAction(cancelAction)
        // must use this function to show JRAlertController
        // self is a UIControllerView
        alertController.jr_show(onRootView: self)
```

#### JRAlertController_sheet_multiple
```swift
        let alertController = JRAlertController(title: "I am title,I am title,I am title,I am title,I am title", message: "I am message, I am message, I am message, I am message, I am message, I am message, I am message, I am message, I am message", preferredStyle: .actionSheet)
        let cancelAction = JRAlertAction(title: "cancel", style: .cancel, handler: nil)
        let deleteAction = JRAlertAction(title: "delete", style: .destructive, handler: nil)
        let archiveAction = JRAlertAction(title: "archive", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive")
        })
        let archiveAction1 = JRAlertAction(title: "archive1", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive1")
        })
        let archiveAction2 = JRAlertAction(title: "archive2", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive2")
        })
        let archiveAction3 = JRAlertAction(title: "archive3", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive3")
        })
        let archiveAction4 = JRAlertAction(title: "archive4", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive4")
        })
        let archiveAction5 = JRAlertAction(title: "archive5", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive5")
        })
        let archiveAction6 = JRAlertAction(title: "archive6", style: .default, handler: {
            (action: JRAlertAction!) -> Void in
            print("archive6")
        })
        alertController.addAction(cancelAction)
        alertController.addAction(deleteAction)
        alertController.addAction(archiveAction)
        alertController.addAction(archiveAction1)
        alertController.addAction(archiveAction2)
        alertController.addAction(archiveAction3)
        alertController.addAction(archiveAction4)
        alertController.addAction(archiveAction5)
        alertController.addAction(archiveAction6)
        // must use this function to show JRAlertController
        // self is a UIControllerView
        alertController.jr_show(onRootView: self)
```


### License

JRAlertController is released under the Apache-2.0 license. See [LICENSE](https://raw.githubusercontent.com/Jiar/JRAlertController/master/LICENSE) for details.



### 结束语

本文完，大家如何喜欢`JRAlertController`，欢迎来[JRAlertController](https://github.com/Jiar/JRAlertController/)对本项目Star。读者在阅读本文时如有发现错误或不恰当指出，欢迎在评论中指出。如果读者还有一些相关方面的疑问，也欢迎在评论中提出。


欢迎关注我的个人微信订阅号，我将不定期分享开发方面的干货。

![Jiar's 微信订阅号](/images/Dingyuehao.jpg)

