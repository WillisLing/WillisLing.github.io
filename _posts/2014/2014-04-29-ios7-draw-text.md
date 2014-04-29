---
layout: post
title: iOS7 中文本绘制有细线的问题
categories:
- Programming
tags:
- iOS

---

今天的项目在适配 iOS7 时，发现一个很奇怪的问题。相同的代码，在 iOS7 系统上，会出现一条大约一像素的细线：<br>
![带细线](/media/images/2014-04-29_1.png)<br><br>
而在 iOS7 以下的版本却没有这个问题：<br>
![正常](/media/images/2014-04-29_2.png)<br><br>
这个文本框的背景使用的是 `resizableImageWithCapInsets:` 拉伸后的图片，而其他非拉伸背景上的文本却是正常显示的。于是，初步猜测是 `resizableImageWithCapInsets:` 的问题。

然后 google 发现 stackOverFlow 上也有人[碰到这个问题](http://stackoverflow.com/questions/19005668/resizableimagewithcapinsets-to-strech-chat-bubble-issue-in-ios7)，下面给出了一些浮点型取整的方案，但是问题并没有得到解决。另外提问者给出了一个 [github 类似问题的链接](https://github.com/jessesquires/MessagesTableViewController/issues/50)。结合这两个问题一看，猜测可能之前的方向搞错了，可能不是 `resizableImageWithCapInsets:` 的问题，而很可能是文本绘制的问题。

项目中绘制文本使用的是 NSString 的 `drawInRect:withFont:lineBreakMode:alignment:` 方法。**我根据猜测，修改了下代码，把第一个参数对应的 rect 取整之后，再传入到绘制函数，结果问题得到了完美的解决**。

为什么 iOS7 上会出现这个问题呢，目前猜测可能是底层文本绘制引擎换成 TextKit 导致的。而这个改变带来了一些 bug，可以参考 [Fixing UITextView on iOS 7](http://petersteinberger.com/blog/2014/fixing-uitextview-on-ios-7/) 这篇文章。