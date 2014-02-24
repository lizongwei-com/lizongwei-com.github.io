---
layout: post
title: "新objc之Modules和@import"
date: 2013-09-22 23:37
comments: true
categories: objc
---
若熟悉python的话`Module`的概念应该也不会陌生，如:
``` python
	import json
	import os.path
	from sys import * // 这随便写的
```
如今新objc也加入了`Module`的概念  
- - -
还记得过去的时候如何引入并使用一个framework么？首先，需要把需要的framework添加到工程里来:  
{% img center /images/2-1.png 400 460 %}   
然后就可以在代码里`#import`并开始使用  
``` objc
	#import <QuartzCore/QuartzCore.h>
	...
	self.view.layer.borderWidth = 1.0f;
```
**会带来什么问题?**  

问题1. 预编译时大量重复的文本拷贝  
{% img center /images/2-2.png 600 400 "原引WWDC2013 Session404 PDF"%}   
如上图，MyApp.m直接引用了`iAd`和`UIKit`两个framework，而`iAd`内部同时又间接引用了`UIKit`。当编译器开始预处理MyApp.m时，会将所有`iAd`和`UIKit`的**.h**文件一并copy到MyApp.m头部，随后进行编译和链接过程，这直接造成的结果是，预处理之后的**.m**文件的体积远不止里面写着的代码：  
{% img center /images/2-3.png 600 400 "原引WWDC2013 Session404 PDF"%}   
由此引发的编译时间复杂度也会成比例增长，而同时一个工程内也不可能只有一个**.m**文件和只引用一个framework，Session中指出这个趋势大概为：  
`M source files + N headers ⇒ M x N compile time`  

问题2. 头文件容易受污染  
这里就不引图了，意思是外部引用一个framework前不会清楚里面都用到过哪些符号啊，变量啊，define啊，在不知情的情况下可能会定义了一个和内部冲突的符号从而引发framework内部的异常，而这经常是很莫名其妙的。（我好像就遇到过）  

问题3. pch预编译头也不能完全解决问题  
工程里的`.pch`(pre-compiled-header)里可以加入`UIKit`和`Fundation`使工程里面所有文件都可以引用到，缓解了大量重复framework头文件拷贝的问题，但是Session中还提到了些带来的问题，比如**Maintenance burden(维护负担)**和**Namespace pollution(命名域污染)**，维护这个不太理解，namesapce污染大概就是指明明不需要而且不想用到这个framework的地方也一样能引用到。我自己理解的一个问题是使得代码结构变得不合理，明明只和一个framework耦合的模块变成了和所有pch中的framework耦合。  
- - -
**说了这么多没用的终于到主题了**
`Modules`简而言之就是将所有系统framework`单例化`了，不知道这么理解对不对，**不论是代码中还是框架中甚至是系统运行依赖中所指向的同一个framework都是一份**，与代码分开编译，通过api调用直接指向dylib的实现部分  
{% img center /images/2-4.png 300 "原引WWDC2013 Session404 PDF"%}   
与此而来的是新的语法`@import`:  
```objc
	@import UIKit	// #import <UIKit/UIKit.h>
	@import UIKit.UIView // #import <UIKit/UIView.h>
	// 上面两种写法与注释的写法等价
```
语法很简单，Xcode5的编译器已经可以识别并自动补全了。  
使用新的`Modules`和`@import`已经解决了之前提出的问题，与此同时还带了些**很赞的好处**  

**1. 不用去手动添加framework啦！Module自动给干了**
{% img center /images/2-5.png 400 "原引WWDC2013 Session404 PDF"%} 

**2. 不用多写一行代码就能完成转换**  
{% img center /images/2-6.png 800 "原引WWDC2013 Session404 PDF"%}   
当然前提是只能适用于系统framework而且得是iOS7 SDK/OS 10.9 SDK

**3. 带来的编译时间的提升**
{% img center /images/2-7.png 600 "原引WWDC2013 Session404 PDF"%}   
{% img center /images/2-8.png 600 "原引WWDC2013 Session404 PDF"%}  
这也是感觉Xcode5编ios7感觉很快的原因之一  

- - -
**总结**  
WWDC2013中介绍的新`Module`和`@import`确实是不错的改进，相比于其带来的性能提升，它的成本基本可以不计。我是决定以后的项目都用`@import`来引用系统框架了，清晰明了，和自己的header能区分开来，使用新SDK的同学们赶快入手吧~
- - -
by`sunnyxx`**欢迎转载，转载请注明作者和来源地址(本地址)** ：）
