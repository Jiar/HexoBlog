---
title: 从此不再担心键盘遮住输入框OC1
date: 2015-11-15 16:33:00
categories: iOS
tags:
- iOS
- 键盘
- 输入框
- 遮盖
---

文章可能有更新，如需了解，请查看原文：[从此不再担心键盘遮住输入框OC(一)](http://www.jianshu.com/p/48993ff982c1)

------

### 新版本在这里：[从此不再担心键盘遮住输入框OC(二)](http://blog.jiar.vip/2015/12/07/%E4%BB%8E%E6%AD%A4%E4%B8%8D%E5%86%8D%E6%8B%85%E5%BF%83%E9%94%AE%E7%9B%98%E9%81%AE%E4%BD%8F%E8%BE%93%E5%85%A5%E6%A1%86OC2/)

------

想必大家在iOS开发中都有遇到过这种问题。点击输入框后，弹出的键盘遮挡了输入框，然后你就无法看见你输入了什么。为了解决这个问题，我也在 [Github](https://github.com/)、[CocoaChina](http://www.cocoachina.com/)以及[Code4App](http://code4app.com/)上花了不少时间去找相关的代码以及实现。
##### 找到的相关内容很多，但是都有一个共同点，是通过将底部的View上滑至键盘之上，从而可以看见输入框内的内容。在这方面做得好的有[IQKeyboardManager](https://github.com/hackiftekhar/IQKeyboardManager)，喜欢的可以去看看，但是我不是就直接采用了IQKeyboardManager，而是自己写了一个键盘组件[KeyboardToolBar](https://github.com/Jiar/KeyboardToolBar/)，优点是小巧易使用，支持[CocoaPods](https://cocoapods.org/)，侵入性小，作者爱交友~  

先来一张效果图
![KeyboardToolBar1 show](https://github.com/Jiar/KeyboardToolBar/raw/master/images/KeyboardToolBar1.gif) 

<!--more-->

### 如何使用、源码分析

下面我通过`如何使用`和`源码分析`两个方面来介绍[KeyboardToolBar](https://github.com/Jiar/KeyboardToolBar/)。

#### 如何使用

------

##### 就是不想用[CocoaPods](https://cocoapods.org/)
- 去[KeyboardToolBar](https://github.com/Jiar/KeyboardToolBar/)下载zip。将`Classes`文件夹下的代码复制到你的项目中去。

##### 如果你也用[CocoaPods](https://cocoapods.org/)

###### Podfile 
    platform :ios, '7.0'
    pod 'KeyboardToolBar', '~> 1.0.0'
###### import 
    /// 不要忘了先导入.h 
    #import "KeyboardToolBar.h"
###### 注册使用KeyboardToolBar 
    /// 使用该方法给UITextField注册使用KeyboardToolBar 
    /// @param textField 需要注册的UITextField 
    [KeyboardToolBar registerKeyboardToolBar:self.textField];
###### 反注册(移除)eyboardToolBar 
    /// 不想让UITextField使用KeyboardToolBar的时候 
    /// 强烈建议使用该方法给UITextField移除KeyboardToolBar 
    /// @param textField 需要移除的UITextField
    [KeyboardToolBar unregisterKeyboardToolBar:self.textField];
###### unregisterAllKeyboardToolBar 
    /// 如果嫌一个一个给UITextField移除KeyboardToolBar麻烦 
    /// 使用这个方法一次性将所有的UITextField移除KeyboardToolBar
    [KeyboardToolBar unregisterAllKeyboardToolBar];

#### 源码分析

------

##### 实现思路

我的设想是在键盘上方的工具栏处做文章，哪怕是键盘挡住了输入框，但是如果将输入框上的`placeholder`以及输入的内容时刻在工具栏上显示，那么哪怕键盘挡住了输入框，依旧可以清楚知道我现在要输入哪方面的内容以及我现在输入的内容是什么。

##### 源码分析

为了看起来清楚些，以下代码中我使用`KTB`代表`KeyboardToolBar单例对象`。

###### .h文件 
    /// 继承iOS自带的UIToolbar
    @interface KeyboardToolBar : UIToolbar
###### 宏 
    /// KeyboardToolBar宽度 
    #define KeyboardToolBarWidth [UIScreen mainScreen].bounds.size.width 
    /// KeyboardToolBar高度 
    #define KeyboardToolBarHeight 44 
    /// KeyboardToolBar上UIScrollView组件的宽度 
    #define KeyboardScrollViewWidth (KeyboardToolBarWidth-80)
###### 属性 
    @property(nonatomic, strong)UIScrollView *scrollView;
    @property(nonatomic, strong)UITextField *toolBarTextField; 
    /// 字典用于存放注册使用KeyboardToolBar的所有UITextField
    @property(nonatomic, retain)NSMutableDictionary *allRegisterTextFields;
###### KeyboardToolBar构造单例方法
```
/// KTBstatic KeyboardToolBar *keyboardToolBar = nil;
+ (instancetype)shareKeyboardToolBar {
    if (keyboardToolBar == nil) { 
        /// KTB是否初始化，如果没有，则进行初始化 
        /// KeyboardToolBar上需要有一个UIScrollView组件，UIScrollView内部有一个UITextField，如果UITextField内的内容过多，UIScrollView就派上用场了。
        /// KeyboardToolBar的右侧还需要有一个'完成'按钮，点击该按钮后关闭键盘
        keyboardToolBar = [[KeyboardToolBar alloc]initWithFrame:CGRectMake(0, 0, KeyboardToolBarWidth, KeyboardToolBarHeight)];
        [keyboardToolBar setBarStyle:UIBarStyleDefault];
        keyboardToolBar.scrollView = [[UIScrollView alloc] initWithFrame:CGRectMake(0, 0, KeyboardScrollViewWidth, KeyboardToolBarHeight)];
        keyboardToolBar.scrollView.backgroundColor = [UIColor clearColor]; 
        keyboardToolBar.scrollView.contentSize = CGSizeMake(KeyboardScrollViewWidth, KeyboardToolBarHeight); 
        keyboardToolBar.scrollView.bounces = NO;
        keyboardToolBar.toolBarTextField = [[UITextField alloc] initWithFrame:CGRectMake(0, 0, KeyboardScrollViewWidth, KeyboardToolBarHeight)];
        keyboardToolBar.toolBarTextField.textAlignment = NSTextAlignmentLeft;
        keyboardToolBar.toolBarTextField.userInteractionEnabled = NO；
        [keyboardToolBar.scrollView addSubview:keyboardToolBar.toolBarTextField];
        UIBarButtonItem *textFieldItem = [[UIBarButtonItem alloc] initWithCustomView:keyboardToolBar.scrollView];
        UIBarButtonItem *finishBtnItem = [[UIBarButtonItem alloc]initWithTitle:@"完成" style:UIBarButtonItemStyleDone target:keyboardToolBar action:@selector(resignKeyboard)];
        NSArray * buttonsArray = [NSArray arrayWithObjects:textFieldItem,finishBtnItem,nil]; [keyboardToolBar setItems:buttonsArray];
    } 
    return keyboardToolBar;
}
/// 关闭键盘
- (void)resignKeyboard {
    keyboardToolBar.toolBarTextField.text = @"";
    [[[UIApplication sharedApplication] keyWindow] endEditing:YES];
}
```
###### KeyboardToolBar注册方法
```
+ (void)registerKeyboardToolBar:(UITextField *)textField {
    if([KeyboardToolBar shareKeyboardToolBar].allRegisterTextFields == nil) {
        keyboardToolBar.allRegisterTextFields = [NSMutableDictionary dictionaryWithCapacity:10];
    }
    /// 将KTB赋予传入的textField
    [textField setInputAccessoryView:keyboardToolBar];
    /// 为传入的textField对象addTarget
    [textField addTarget:keyboardToolBar action:@selector(textFieldDidBegin:) forControlEvents:UIControlEventEditingDidBegin];
    [textField addTarget:keyboardToolBar action:@selector(textFieldDidChange:) forControlEvents:UIControlEventEditingChanged];
    /// 将传入的textField保存于KTB
    [keyboardToolBar.allRegisterTextFields setValue:textField forKey:[NSString stringWithFormat:@"%p",textField]];
}
- (void)textFieldDidBegin:(UITextField *)textField {
    [self reSetTextField:textField];
}
- (void)textFieldDidChange:(UITextField *)textField {
    [self reSetTextField:textField];
}
/// 将textField的placeholder以及textField上的文字及时显示在KTB内部的UITextField上
- (void)reSetTextField:(UITextField *)textField {
    UITextField *tempTextField = [keyboardToolBar.allRegisterTextFields objectForKey:[NSString stringWithFormat:@"%p",textField]];
    CGFloat textWidth = [KeyboardToolBar widthForString:tempTextField.text withFont:keyboardToolBar.toolBarTextField.font];
    if(textWidth > KeyboardScrollViewWidth) {
        keyboardToolBar.toolBarTextField.frame = CGRectMake(0,0,textWidth,KeyboardToolBarHeight);
        keyboardToolBar.scrollView.contentSize = CGSizeMake(textWidth, KeyboardToolBarHeight);
        [self.scrollView scrollRectToVisible:CGRectMake(textWidth-KeyboardScrollViewWidth,0,KeyboardScrollViewWidth,KeyboardToolBarHeight) animated:YES];
    } else {
        keyboardToolBar.toolBarTextField.frame = CGRectMake(0, 0, KeyboardScrollViewWidth, KeyboardToolBarHeight);
        keyboardToolBar.scrollView.contentSize = CGSizeMake(KeyboardScrollViewWidth, KeyboardToolBarHeight);
    }
    keyboardToolBar.toolBarTextField.text = tempTextField.text;
    keyboardToolBar.toolBarTextField.textColor = tempTextField.textColor;
    if(tempTextField.placeholder != nil) {
        NSAttributedString *attribute = textField.attributedPlaceholder;
        NSDictionary *dictionary = [attribute attributesAtIndex:0 effectiveRange:nil];
        keyboardToolBar.toolBarTextField.attributedPlaceholder = [[NSAttributedString alloc] initWithString:tempTextField.placeholder attributes:@{NSForegroundColorAttributeName: [dictionary valueForKey:NSForegroundColorAttributeName]}];
    }
}
/// 根据文本内容和字体计算NSString长度用于设置KTB内部的UIScrollView以及UITextField的宽度
+ (CGFloat)widthForString:(NSString *)str withFont:(UIFont *)font {
    NSDictionary *attribute = @{NSFontAttributeName: font};
    CGSize size = [str boundingRectWithSize:CGSizeMake(CGFLOAT_MAX, CGFLOAT_MAX) options: NSStringDrawingTruncatesLastVisibleLine | NSStringDrawingUsesLineFragmentOrigin | NSStringDrawingUsesFontLeading attributes:attribute context:nil].size;
    return size.width;
}
```
###### KeyboardToolBar反注册(移除)方法
```
/// 直接将KTB中所有已注册的UITextField移除
+ (void)unregisterKeyboardToolBar:(UITextField *)textField {
    if(keyboardToolBar == nil || keyboardToolBar.allRegisterTextFields.count == 0) {
        return;
    }
    UITextField *tempTextField = [keyboardToolBar.allRegisterTextFields objectForKey:[NSString stringWithFormat:@"%p",textField]];
    [tempTextField setInputAccessoryView:nil];
    [tempTextField removeTarget:keyboardToolBar action:@selector(textFieldDidBegin:) forControlEvents:UIControlEventEditingDidBegin];
    [tempTextField removeTarget:keyboardToolBar action:@selector(textFieldDidChange:) forControlEvents:UIControlEventEditingChanged];
    [keyboardToolBar.allRegisterTextFields removeObjectForKey:[NSString stringWithFormat:@"%p",textField]];
    if(keyboardToolBar.allRegisterTextFields.count == 0) {
        keyboardToolBar.allRegisterTextFields = nil;
        keyboardToolBar = nil;
    }
}
/// 根据传入的UITextField兑现。从KTB内移除
+ (void)unregisterAllKeyboardToolBar {
    if(keyboardToolBar == nil || keyboardToolBar.allRegisterTextFields.count == 0) {
        return;
    }
    NSEnumerator *enumeratorValue = [keyboardToolBar.allRegisterTextFields objectEnumerator];
    for(UITextField *tempTextField in enumeratorValue) {
        [tempTextField setInputAccessoryView:nil];
        [tempTextField removeTarget:keyboardToolBar action:@selector(textFieldDidBegin:) forControlEvents:UIControlEventEditingDidBegin];
        [tempTextField removeTarget:keyboardToolBar action:@selector(textFieldDidChange:) forControlEvents:UIControlEventEditingChanged];
    } 
    [keyboardToolBar.allRegisterTextFields removeAllObjects];
    keyboardToolBar.allRegisterTextFields = nil;
    keyboardToolBar = nil;
}
```
好了，以上就是我的介绍，欢迎大家来我的[KeyboardToolBar](https://github.com/Jiar/KeyboardToolBar/)主页进行Star、Issues或Pull requests，我是Jiar，我热爱交朋友~


### 微信订阅号
欢迎关注我的个人微信订阅号，我将不定期分享开发方面的干货。

![Jiar's 微信订阅号](Dingyuehao.jpg)