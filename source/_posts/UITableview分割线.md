---
title: UITableview分割线
date: 2016-06-02 12:23:48
categories: iOS
tags:
- iOS
- cell
- UITableview
- UITableview分割线
- UITableview分割线补全
- UITableview分割线隐藏
---
1. cell分割线补全
	- 指定cell分割线补全：
	在`cellForRowAtIndexPath`代理中加入如下代码：
	```
	cell.preservesSuperviewLayoutMargins = NO;
	cell.layoutMargins = UIEdgeInsetsZero;
	cell.separatorInset = UIEdgeInsetsZero;
	```
	<!--more-->
	- 全局cell分割线补全：
	在`UIControllerView`中加入重写如下方法：
	```
	- (void)viewDidLayoutSubviews {
	    if ([self.table respondsToSelector:@selector(setSeparatorInset:)]) {
	        [self.table setSeparatorInset:UIEdgeInsetsMake(0,0,0,0)];
	    }
	    if ([self.table respondsToSelector:@selector(setLayoutMargins:)]) {
	        [self.table setLayoutMargins:UIEdgeInsetsMake(0,0,0,0)];
	    }
	}
	
	- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath {
	    if ([cell respondsToSelector:@selector(setSeparatorInset:)]) {
	        [cell setSeparatorInset:UIEdgeInsetsZero];
	    }
	    if ([cell respondsToSelector:@selector(setLayoutMargins:)]) {
	        [cell setLayoutMargins:UIEdgeInsetsZero];
	    }
	}
	```
2. cell分割线隐藏
	- 指定cell分割线隐藏：
	首先保证指定`cell分割线已经补全`，然后加入如下代码：
	```
	cell.separatorInset = UIEdgeInsetsMake(0, 0, 0, kDeviceWidth);
	```

	- 全局cell分割先隐藏：
	```
	self.table.separatorStyle = UITableViewCellSeparatorStyleNone;
	```

欢迎关注我的个人微信订阅号，我将不定期分享开发方面的干货。

![Jiar's 微信订阅号](Dingyuehao.jpg)