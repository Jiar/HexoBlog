---
title: 从此不再担心键盘遮住输入框OC2
date: 2015-12-07 00:13:00
categories: iOS
tags:
- iOS
- 键盘
- 输入框
- 遮盖
---

文章可能有更新，如需了解，请查看原文：[从此不再担心键盘遮住输入框OC(二)](http://www.jianshu.com/p/f33fd3f927f6)

在我发布这篇文章没多久之前，我发布了一篇叫 [从此不再担心键盘遮住输入框OC(一)](http://blog.jiar.me/2015/11/15/%E4%BB%8E%E6%AD%A4%E4%B8%8D%E5%86%8D%E6%8B%85%E5%BF%83%E9%94%AE%E7%9B%98%E9%81%AE%E4%BD%8F%E8%BE%93%E5%85%A5%E6%A1%86OC1/)的文章。我在那篇文章中介绍了我的键盘组件[KeyboardToolBar](https://github.com/Jiar/KeyboardToolBar/)。

新版效果图
![KeyboardToolBar2 show](https://github.com/Jiar/KeyboardToolBar/raw/master/images/KeyboardToolBar2.gif) 

<!--more-->

> 当时的[KeyboardToolBar](https://github.com/Jiar/KeyboardToolBar/)还只是支持`UITextField`。后来也有收到别人的建议，希望增加支持`UITextField`之类的。其实本人也早就想着再完善一下。正好这个周末不忙，我就稍微优化了下。发布了V2版本。

> 现在的[KeyboardToolBar](https://github.com/Jiar/KeyboardToolBar/)支持`UITextField`、`UITextView`和`UISearchBar`。并且支持运行时(`runtime`)，你只要在项目中导入`"KeyboardToolBar.h"`即可开始使用，无需额外代码。

### KeyboardToolBar 是什么

KeyboardToolBar的主旨：从此不再担心键盘遮住输入框。目前是V2版本，如果想了解V1版本，请移步[V1版本](http://www.jianshu.com/p/48993ff982c1)。

### 如何开始使用
- **下载[KeyboardToolBar](https://github.com/Jiar/KeyboardToolBar/)并尝试在你的iPhone上运行DEMO。**

### 使用CocoaPods安装

#### Podfile 

      platform :ios, '7.0' 
      pod "KeyboardToolBar"

### Usage

现在，[KeyboardToolBar](https://github.com/Jiar/KeyboardToolBar/)支持`UITextField`、`UITextView`和`UISearchBar`。并且支持运行时(`runtime`)，你只要在项目中导入`"KeyboardToolBar.h"`即默认为所有的`UITextField`、`UITextView`和`UISearchBar`自动注册使用了`KeyboardToolBar`，你无需使用额外的代码来开启。如果你不想用`KeyboardToolBar`，你可以使用相应的`unregisterKeyboardToolBar`方法来反注册即可移除`KeyboardToolBar`。如果你已经为某个控件移除了`KeyboardToolBar`，又想要继续使用，你可以使用相应的`registerKeyboardToolBar`方法为控件重新注册使用`KeyboardToolBar`。

#### import 
      /// 导入就是使用 
      /// 导入后，将自动为UITextField、UITextView和UISearchBar注册使用KeyboardToolBar 
      #import "KeyboardToolBar.h"

#### 注册使用KeyboardToolBar 
      /// 以下均为可选方法，你可以不使用。 
      /// 为UITextField注册使用KeyboardToolBar. 
      [KeyboardToolBar registerKeyboardToolBarWithTextField:self.textField]; 
      /// 为UITextView注册使用KeyboardToolBar. 
      [KeyboardToolBar registerKeyboardToolBarWithTextView:self.textView]; 
      /// 为UISearchBar注册使用KeyboardToolBar.
      [KeyboardToolBar registerKeyboardToolBarWithSearchBar:self.searchBar];
#### 反注册取消KeyboardToolBar 
      /// 以下均为可选方法，你可以不使用。 
      /// 你可以为目标UITextField反注册取消使用KeyboardToolBar.
      [KeyboardToolBar unregisterKeyboardToolBarWithTextField:self.textField]; 
      /// 你可以为目标UITextView反注册取消使用KeyboardToolBar 
      [KeyboardToolBar unregisterKeyboardToolBarWithTextView:self.textView]; 
      /// 你可以为目标UISearchBar反注册取消使用KeyboardToolBar.
      [KeyboardToolBar unregisterKeyboardToolBarWithSearchBar:self.searchBar]; 

### License
KeyboardToolBar is released under the MIT license.


### 微信订阅号
欢迎关注我的个人微信订阅号，我将不定期分享开发方面的干货。

![Jiar's 微信订阅号](/images/Dingyuehao.jpg)

