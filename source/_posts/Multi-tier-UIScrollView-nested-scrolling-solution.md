---
title: 多层 UIScrollView 嵌套滚动解决方案
date: 2019-02-13 18:22:01
categories: iOS
tags:
- iOS
- UIScrollView
- 嵌套
---

[掘金阅读地址在这里](https://juejin.cn/post/6844903776130695175)

本文旨在对于[SegementSlide](https://github.com/Jiar/SegementSlide)库实现原理的讲解，有兴趣的同学，欢迎前往[Github地址](https://github.com/Jiar/SegementSlide)浏览。

![SegementSlide](Logo.png)

------

#### 背景

如今的app中，越来越多地采用如下图所示的设计，一般用在诸如『用户主页』、『话题详情页』、『专题详情页』等这些场景。通常，这些场景会带有头部视图（头部视图可能要求支持滚动渐变），下面紧接着的是分页控件，最下面是滚动列表。

如下图所示：
![SegementSlide](transparent.gif)

<!--more-->

#### 各种方案以及优缺点

为了方便下面的说明，在开始之前，先约定几个说法，下面的各种方案，大都离不开在最底层放上一个`UIScrollView`（竖直方向滚动），我们称之为`rootScrollView`。无论分页控件下方有多少个子界面，总有一个当前界面，我们称当前界面下的`UIScrollView`（竖直方向滚动）为`childScrollView`。

##### I 控制`isScrollEnabled`属性

这是我们第一时间能想到的方案，通过给`rootScrollView`和`childScrollView`实现`UIScrollViewDelegate`，并在`func scrollViewDidScroll(_ scrollView: UIScrollView)`方法中实时将`scrollView.contentOffset.y`与临界值进行对比从而修改两者`scrollView`的`isScrollEnabled`属性值来达到目的。

大致代码如下
```swift
func scrollViewDidScroll(_ scrollView: UIScrollView) {
    if scrollView == rootScrollView {
        if scrollView.contentOffset.y >= headerStickyHeight {
            scrollView.contentOffset.y = headerStickyHeight
            rootScrollView.isScrollEnabled = false
            childScrollView.isScrollEnabled = true
        }
    } else {
        if scrollView.contentOffset.y <= 0 {
            scrollView.contentOffset.y = 0
            childScrollView.isScrollEnabled = false
            rootScrollView.isScrollEnabled = true
        }
    }
}
```

方法简单，但是有个不太能接受的交互问题，但凡将`isScrollEnabled`设置为`false`，这次的滑动手势就会被打断，从表现上来看，就是滑动到临界值时滑动会被中断。

##### II 自定义滑动手势

在这篇文章[这篇文章](https://www.jianshu.com/p/df01610b4e73)中，作者提供了一种利用自定义手势的方式来实现。
但是，只是添加普通的滑动手势是不够的，`UIScrollView`是自带阻尼效果的，因此引入了`UIDynamicAnimator `来实现阻尼效果。
这是一种不错的思路。不过完全自定义手势来实现`UIScrollView`的效果，需要考虑的细节过多，挺难处理得跟系统的效果一致（写这篇文章的时候，下载了作者提供的[源码](https://github.com/Junlau/ScrollViewInScrollView)，`commitID`为`ff7b76f8468bc87fea8ea6975d8b9fe1173ab031`，在真机`iPhone X`上运行，感觉还是有交互上的问题）。此外，因为是自定义手势，手势不是直接作用在`UIScrollView`上的，`UIScrollView`的`ScrollIndicator`是无法显示的，通过改变`UIScrollView`的`contentOffset`，其`ScrollIndicator`也是无法显示的，必须要手势作用在`UIScrollView`上才行。使用`UIScrollView`的`flashScrollIndicators()`来强迫`ScrollIndicator`显示出来？...可能还真行，不过我没试过，感觉太粗暴了。

##### III 手势穿透

这应该是目前相对主流的一种实现方式，比如在[这篇文章中](https://www.jianshu.com/p/8bf6c2953da3)，便是介绍了这种方式。据我观察Twitter和微博的用户主页可能是使用这种方式实现的（写这篇文章的时候，Twitter版本为：7.41.2，微博版本为：9.2.0，推测错了的话还望见谅）

该方案的核心为有两点：

- 让滑动手势穿透使得`rootScrollView`和`childScrollView`都能接收到滑动手势（因为手势是作用到`UIScrollview`上的，自然是能显示`ScrollIndicator`的）。做法是让`rootScrollView`实现`UIGestureRecognizerDelegate`的代理方法`func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, shouldRecognizeSimultaneouslyWith otherGestureRecognizer: UIGestureRecognizer) -> Bool`，并在适当的时机返回`true`。

这部分的代码大致如下：
```swift
class SegementSlideScrollView: UIScrollView, UIGestureRecognizerDelegate {
    
    func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, shouldRecognizeSimultaneouslyWith otherGestureRecognizer: UIGestureRecognizer) -> Bool {
        return true
    }
    
}
```

当然只是如此的话，是不够的，这样的结果是滑动的时候，导致`rootScrollView`和`childScrollView`一起滚动。

- 增加两个标志位来控制何时允许`rootScrollView`滚动，以及何时允许`childScrollView`。

这部分代码大致如下：
```swift
func scrollViewDidScroll(_ scrollView: UIScrollView) {
    if scrollView == rootScrollView {
        if !canParentViewScroll {
            rootScrollView.contentOffset.y = headerStickyHeight // point A
            canChildViewScroll = true
        } else if scrollView.contentOffset.y >= headerStickyHeight {
            rootScrollView.contentOffset.y = headerStickyHeight
            canParentViewScroll = false
            canChildViewScroll = true
        }
    } else {
        if !canChildViewScroll {
            childScrollView.contentOffset.y = 0 // point B
        } else if scrollView.contentOffset.y <= 0 {
            canChildViewScroll = false
            canParentViewScroll = true
        }
    }
}
```

如上代码所示，控制`rootScrollView`或者是`childScrollView`不可滚动的方式是将两者的`contentOffset.y`设置为一个固定值（见注释`point A`和`point B`），并不是简单地将`isScrollEnabled`设置`false`而已。

 没问题了？不，也是有不足之处的：
***在第一个界面使用手指向上滑动，让头部视图完全被隐藏后再向上滑动一些，让`childScrollView`的`contentOffset.y`处于大于`0`的状态，随后，左右切换到第二个界面，使用手指向下滑动，完全拉出头部视图，然后再切换回第一个界面，这个时候，使用手指在屏幕上稍微滑动一下，`rootScrollView`或是`childScrollView`的`contentOffset.y`会突变，从表现上看，就是发生『位置突变现象』***

问题产生的原因是什么？
`canParentViewScroll`和`childScrollView`始终为一对相反的值，浏览上诉代码，会发现在`point A`和`point B`处，将`rootScrollView`或者是`childScrollView`的`contentOffset.y`设置为了一个固定值。这样的处理，当始终在同一个界面滑动的时候，不会有问题，但是，在切换界面后，由于`rootScrollView`是共用的，在新界面改动了`rootScrollView`的`contentOffset.y`，切换回原界面后，稍做滑动，定会执行`point A`或是`point B`其中的一处代码，从而导致『位置突变现象』。

在微博和Twitter中对此问题做了简单的处理。微博上，在切换至新界面之前，将原界面的`childScrollView`的`contentOffset.y`值重置为了`0`。Twitter上，则是在合适的时机做了重置。这也是推测两者可能是使用了该方案的原因。

如下图所示：
![weibo](weibo.gif)
![twitter](twitter.gif)

#### SegementSlide的需求
[SegementSlide](https://github.com/Jiar/SegementSlide)是使用 **方案III** 来实现的。

此外我希望它还能支持一些别的特性：
1. 简单易用的接口
2. 一般使用 方案III 实现的例子，大都只是支持在`rootScrollView`上实现阻尼效果，我希望也能在`childScrollView`上实现，可以选择任意一个阻尼来使用。（有阻尼，就可以配套下拉刷新工具来使用了）
3. 一般使用 方案III 实现的例子，大都是需要手指在子视图部分滑动才能实现联动，希望也能在头部滑动实现联动
4. 既可以支持使用头部视图，也可以不需要头部视图
5. 头部视图可以使用简单的接口实现滚动渐变效果（`navigation`上随着滚动改变背景色、标题、leftItem颜色、rightItem颜色，或是背景色透明之类的），也可以自定义渐变效果
6. 子控件既可结合一起使用，也可以单独使用
7. 分页标题旁可以显示红点
...

对此，大都已经实现：
1. 看下如下示例代码，是否还算简单易用：

```swift
import SegementSlide

class HomeViewController: SegementSlideViewController {

    ......

    override var headerHeight: CGFloat? {
        return view.bounds.height/4
    }
    
    override var headerView: UIView? {
        return UIView()
    }

    override var titlesInSwitcher: [String] {
        return ["Swift", "Ruby", "Kotlin"]
    }

    override func segementSlideContentViewController(at index: Int) -> SegementSlideContentScrollViewDelegate? {
        return ContentViewController()
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        canCacheScrollState = true
        reloadData()
        scrollToSlide(at: 0, animated: false)
    }

}
```

```swift
import SegementSlide

class ContentViewController: UITableViewController, SegementSlideContentScrollViewDelegate {

    ......

    @objc var scrollView: UIScrollView {
        return tableView
    }

}
```

2. 已经能否支持“父阻尼”和“子阻尼”效果了

重写`SegementSlideViewController`的属性`bouncesType`，它是一个枚举类型：

```swift
enum BouncesType {
    case parent
    case child
}
```

默认值为`.parent`，如下重写，即可实现『子阻尼』效果：

```
class HomeViewController: SegementSlideViewController {

    ......

    override var bouncesType: BouncesType {
        return .child
    }
}
```

3. 如何使得在头部滑动也能实现滚动联动效果？
我在`SegementSlideHeaderView`中重写了方法`func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView?`，在合适的情况下返回了`childScrollView`。目前这不是一个最优的方法，因为我没能够在这个方法中判断出这个事件是滑动还是点击事件，这里还可以优化。

4. 既可以支持使用头部视图，也可以不需要头部视图
`SegementSlideViewController`是实现这套方案的基类，其中有一个`headerView`属性，该属性为可选值，返回`nil`则表示不需要头部视图。我在项目配套的`Example`工程中，其中的首页便是没有头部视图的示例，不过增加了下拉显示`navigation`、上滑隐藏`navigation`的效果。一般使用 方案III 的例子，在`rootScrollView`上使用了`UITableView`，为了使用`UITableView`的`tableHeaderView`属性，以及吸顶效果。`SegementSlide`在`v1`版本的时候，使用了`UICollectionView`，也是处于同样的目的，现`v2`已经改成了`UIScrollView`，吸顶效果的话，可以通过增加一条到`view.safeAreaLayoutGuide.topAnchor`的约束来实现。

5. 快速应用头部渐变效果？
`TransparentSlideViewController`是继承于`SegementSlideViewController`的子类，其中的`headerView`属性已被改成非可选值。其中另外定义了一些属性，用于头部视图处于『显示状态』或是『嵌入状态』时，`titleView`和`navigationBar`对应属性的改动。

如下所示：

```
typealias DisplayEmbed<T> = (display: T, embed: T)

override var isTranslucents: DisplayEmbed<Bool> {
    return (true, false)
}

override var attributedTexts: DisplayEmbed<NSAttributedString?> {
    return (nil, nil)
}

override var barStyles: DisplayEmbed<UIBarStyle> {
    return (.black, .default)
}

override var barTintColors: DisplayEmbed<UIColor?> {
    return (nil, .white)
}

override var tintColors: DisplayEmbed<UIColor> {
    return (.white, .black)
}
```

其中`DisplayEmbed`为一个`typealias`表示『显示状态』或是『嵌入状态』时的值。

需要注意的是：
- `TransparentSlideViewController`中的`titleView`是使用自定义的方式并赋值给`navigationItem.titleView`来实现的，最先考虑的是修改`navigationBar`的`titleTextAttributes`属性，实践下来，发现会出现`titleTextAttributes`已经修改完毕，但是效果没有改变的情况。
- `TransparentSlideViewController`会在`viewWillAppear`时保存`navigation`上对应样式的状态，并在`viewWillDisappear`时进行还原，来保证从一个`TransparentSlideViewController`（A）进入到另一个`TransparentSlideViewController`（B）时，`navigation`上样式的状态不会有错误，所以也不该在`viewDidLoad`时修改`navigation`上的样式，因为`B`的`viewDidLoad`先于`A`的`viewWillDisappear`执行。

如果需要自定义渐变效果，可以模仿`TransparentSlideViewController`继承`SegementSlideViewController`来实现需要的效果。`Example`中使用的是原生的`UINavigationController`，和`TransparentSlideViewController`配合起来，可以做到还算满意的效果。但是，实际情况下每个项目中可能会去改动默认的`navigation`，如果`TransparentSlideViewController`不适用，则需要使用自定义的方式来支持已有项目。

6. 子控件既可结合一起使用，也可以单独使用
目前`SegementSlideSwitcherView`和`SegementSlideContentView`既可以作为`SegementSlideViewController`的子控件来使用，也可以单独拿出来使用，`Example`工程中的`NoticeViewController`便是单独使用的例子，实现了将`switcher`放在`navigation`上的效果。

7. 红点显示？
`SegementSlideSwitcherView`支持了红点显示

```swift
enum BadgeType {
    case none
    case point
    case count(Int)
}
```

红点类型为枚举值，从上述代码可以看出红点是支持『普通红点显示』还有『带数字红点显示』。

#### 还需要优化的点

1. 上面在第3点已经提到，『头部滑动也能实现滚动联动效果』目前对此的解决方法不是最优。

2. 方案III 所提到的『位置突变现象』，我在`SegementSlideViewController`中提供了`canCacheScrollState`属性，值为`true`时，在切换界面的时候会缓存当前的`canParentViewScroll`、`canChildViewScroll`以及`rootScrollView`的`contentOffset.y`值，并在切换回该界面的时候恢复；值为`false`时，即为类似微博的处理，在切换到新界面前将当前界面的`childScrollView`的`contentOffset.y`值置为`0`。设置为`true`时会有一个效果，担心这个效果难以被接受，故将该值的默认值设置为了`false`。

效果如下：
![canCacheScrollState](canCacheScrollState.gif)

但这仍不是一个很好的处理方式。

3. 联动滚动切换的时候，还没有达到完美的流畅效果。由于`point A`和`point B`处将`contentOffset.y`强制设值来阻止滚动，同时也导致了滚动切换时『动能』不足的结果，也就是还不够流畅。

#### 接下去要做的事

自然是要解决上面提到的三点不足的地方，要想让联动完美般流畅，还是需要使用一个滚动，而不是两个。我在本地开了个`v3`分支做了个尝试，在视图顶层覆盖一层透明的`UIScrollView`，借用它的手势、它的`contentOffset`来控制`rootScrollView`和`childScrollView`的`contentOffset`，可以解决上述提到的三个需要优化的点，但是同时也带来了其他好多问题，这里就不细说了，哪天问题都解决了，更新了`v3`版本，再来补充说明吧。


#### 参考
- [iOS 嵌套UIScrollview的滑动冲突另一种解决方案](https://www.jianshu.com/p/df01610b4e73)
- [iOS scrollView嵌套tableView的手势冲突解决方案](https://www.jianshu.com/p/8bf6c2953da3)

#### 结束语
编写本文时，[SegementSlide](https://github.com/Jiar/SegementSlide)的版本号为`2.0-beta-13`。另外，本站还未开通评论功能，如对本文中的内容存在疑问，或者发现文中的不正确之处，欢迎在本文的[掘金地址](https://juejin.im/post/6844903776130695175)评论区中**友善**提出。如对本项目有任何疑问，欢迎前往[issues](https://github.com/Jiar/SegementSlide/issues)提出，同时也欢迎来[Pull requests](https://github.com/Jiar/SegementSlide/pulls)，为本项目做贡献。

欢迎关注我的个人微信订阅号，我将不定期分享开发方面的干货。

![Jiar's 微信订阅号](/images/Dingyuehao.jpg)

