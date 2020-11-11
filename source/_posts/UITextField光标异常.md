---
title: UITextField光标异常
date: 2016-07-05 10:02:38
categories: iOS
tags:
- iOS
- UITextField
---

## UITextField光标异常如何解决

在`iOS`开发中，用`xib`写了那么多的界面，我也是头一次遇到这样一个问题，如下图所示：
![UITextField光标异常](UITextField光标异常.gif)

<!--more-->

这是一个普通的登录界面，因为界面简单，采用xib实现，还有可能会对`UITextField`光标产生影响的是项目使用了`IQKeyboardManager`。
我在`viewDidLoad`加入了`[self.phoneField becomeFirstResponder];`使得输入手机号的文本框获取焦点，于是产生了上图显示的问题。

解决方法：首先去掉`[self.phoneField becomeFirstResponder];`，然后用如下两种方案替换：
方案一：
```
	// 刚进入该界面时，IQKeyboardManager的控件会先变黑一下，再变回来
	self.automaticallyAdjustsScrollViewInsets = NO;
	[self.phoneField becomeFirstResponder];
```
方案二：
```
	// 如果在xib中启用了clearsOnBeginEditing属性，则该属性有一定概率失效
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
		[self.phoneField becomeFirstResponder];
	});
```
方案三（推荐）：
```
	self.automaticallyAdjustsScrollViewInsets = NO;
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.5 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
		[self.phoneField becomeFirstResponder];
	});
```


欢迎关注我的个人微信订阅号，我将不定期分享开发方面的干货。

![Jiar's 微信订阅号](/images/Dingyuehao.jpg)


