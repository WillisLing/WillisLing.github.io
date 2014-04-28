---
layout: post
title: iOS 中 Touch Cancelled 的触发方式
categories:
- Programming
tags:
- iOS

---

今天在项目中需要自己处理点击事件，看了 UIView 的官方文档，主要有四个方法需要实现：

	– touchesBegan:withEvent:
	– touchesMoved:withEvent:
	– touchesEnded:withEvent:
	– touchesCancelled:withEvent:

前面3个方法都很容易理解，但是最后一个方法的 touchesCancelled 什么时候能够触发呢？我一开始以为，手指移出屏幕、或者移出 View 的 Frame 会触发 touchesCancelled，但是实际调试的时候，发现移出屏幕触发的是 touchesEnded，移出 View 的 Frame 不会导致触摸终止，依然是 Moved 状态。

那么 touchesCancelled 是干什么用的？
于是又认真看了下官方文档：

**touchesCancelled:withEvent:**<br>
Sent to the receiver when a system event (such as a low-memory warning) cancels a touch event.<br>
- (void)touchesCancelled:(NSSet * )touches withEvent:(UIEvent * )event<br>
...<br>
*Discussion*<br>
This method is invoked when the Cocoa Touch framework receives a system interruption requiring cancellation of the touch event; for this, it generates a UITouch object with a phase of UITouchPhaseCancel. The interruption is something that might cause the application to be no longer active or the view to be removed from the window.

当 Cocoa Touch framework 接到系统中断通知，需要取消触摸事件的时候，会调用此方法。同时会将导致一个 UITouch 对象的 phase 改为UITouchPhaseCancel。这个中断往往是因为 app 长时间没有响应，或者当前 view 从 window 上移除了。

据网上统计，有这么几种情况会导致触发Cancelled：

1. 官方所说长时间无响应，view被移除
1. 低内存警告
1. 触摸的时候来电话，弹出 UIAlertView(低电量、短信、推送之类)，按了 Home 键，也就是说程序进入后台。
1. 屏幕关闭，触摸的时候，某种原因导致距离传感器工作，例如脸靠近。
1. 手势的权限盖掉了Touch, UIGestureRecognizer 有一个属性：
{% highlight objc %}
@property(nonatomic) BOOL cancelsTouchesInView;
// default is YES. causes touchesCancelled:withEvent: to be sent to the view 
// for all touches recognized as part of this gesture immediately before
// the action method is called
{% endhighlight %}

