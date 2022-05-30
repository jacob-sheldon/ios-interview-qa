---
layout: default
title: Cocoa Touch
parent: iOS 相关
nav_order: 2
---

# Cocoa Touch

## layoutIfNeeded 和 setNeedsLayout 的区别

- 如果不是为了更新动画可以无脑用 `setNeedsLayout`
- `setNeedsLayout`没有动画能力，`layoutIfNeeded`用来动画。因此，`layoutIfNeeded`需要放在动画的 block 里面来触发动画。

## layoutSubviews 调用时机

- init 初始化不会触发 layoutSubviews
- addSubview 会触发 layoutSubviews
- 设置 view 的 frame 会触发 layoutSubviews
- 滚动一个 UIScrollView 会触发 layoutSubviews
- 旋转屏幕会触发父 view 上的layoutSubviews。
- 改变一个 view 的大小也会触发父 view 上的 layoutSubviews
- 调用 setNeedLayout 和 layoutIfNeeded 会触发

## UIView 和 CALayer 的关系和区别

- UIView 是被 CALayer 支持的
- UIView 的绘制需要 CPU 在主线程进行，CALayer 的绘制是由 GPU 在子线程i进行。
- UIView 可以响应事件，有响应链，CALayer 没有。
- UIView 的显示属性修改不会产生隐式动画，CALayer 的属性修改会产生默认 0.25s 的隐式动画。

## UIView 动画

`[UIView animationWithDuration:delay:options:animations:completion]` 方法中的 animations block 会立即调用不会等待延迟 dealy 时间后再调用。并且在 animations 执行结束后这个方法才会返回。

## drawRect: 什么情况下会被触发

- 调用时机：`layoutView` -> `viewDidLoad` -> `drawRect:`
- 如果在 UIView 初始化时没有设置 rect 大小，将直接导致 drawRect: 不被自动调用
- 通过设置 contentMode 属性值为 `UIViewContentModeRedraw`，那么将在每次设置或更改frame的时候自动调用 `drawRect:`
- 直接调用`setNeedDisplay`或者`setNeedsDisplayInRect:`会触发 drawRect:，条件是view 当前的 rect 不能为空。
- 该方法在调用 `sizeThatFits` 后被调用，所以可以先调用 sizeToFit 计算出 size，然后系统自动调用 drawRect: 方法。

[关于绘制的一篇掘金文章](https://juejin.cn/post/6844903712771555341)

## 渲染

### 渲染过程

界面从上到下渲染，每开始一行渲染会发出水平同步信号（HSync）。

从上到下渲染出来就是一帧画面。

一帧渲染完成开始下一帧前会发出垂直同步信号（VSync）。

显示器按照一定时间间隔刷新屏幕（界面）这个频率就是 VSync 信号产生的频率 —— 刷新率。平时说的 60Hz 和 120Hz 表示的就是每秒刷新 60 次和 120 次。

### CPU 和 GPU 协同工作

CPU 负责计算显示内容并将计算结果提交到 GPU，GPU 在渲染完成后将结果放入帧缓冲区。随后视频控制器根据 VSync 信号从缓冲区逐帧读取数据并显示到屏幕上。

**双（三）缓冲区**：如果只有一个缓冲区的话，GPU 只有等到视频控制器将内容读取完才能把下一帧放入缓冲区，这样延迟太大、画面就会卡顿。所以引入了多缓冲区机制，GPU 渲染完一帧放入缓冲区内，让视频控制器读取，当下一帧渲染完成后，GPU 会直接把视频控制器的指针指向第二个缓冲区，这样效率可以得到很大提升。

**GPU 的垂直（Vsync）同步**：但是双缓存机制会带来新的问题，如果视频控制器还没有读取完前一帧的内容时 GPU 就切换回来了，这样就会出现在一个画面同时出现两帧的内容——画面撕裂。为了解决这个问题 GPU 引入了垂直同步。当开启垂直同步后，GPU 会等待显示器的 VSync 信号发出后，才进行新的一帧渲染和缓冲区更新。但是这样也带来了其他问题。

iOS 系统的垂直同步是保持开启的，所以垂直同步机制是造成画面卡顿的根本原因。**当 CPU 和 GPU 的渲染速度慢于屏幕的刷新速度时，就会造成掉帧卡顿**。

### CPU 消耗资源的原因和解决方案

**在界面渲染过程中 CPU 主要的工作：对象创建、布局计算、图片解码、文本绘制**，如果这些工作量过大会造成界面卡顿。

- 对象创建：**使用更轻量的对象比如 CALayer 代替 UIView；尽量使用缓存池复用对象**。
- 对象调整：避免调整视图层次、添加和移除视图。
- 局部计算：视图布局计算是 App 中最为常见的消耗 CPU 的地方，尽量在后台提前计算好布局并进行缓存。
- 文本计算：减少文本宽高计算。 `[NSAttributedString boundingRectWithSize:options:context:]` `[NSAttributedString drawWithRect:options:context:]`
- 文本渲染：屏幕上看到的所有文本内容控件，包括 UIWebView，在底层都是通过 CoreText 排版绘制为 Bitmap 显示的。常见的文本控件（UILabel、UITextView 等）其排版和绘制都是在主线程进行的，当显示大量文本时，CPU 的压力会非常大。解决方法： **自定义文本控件，使用 TextKit 或者 CoreText 对文本异步绘制。**
- 图片的解码：使用 UIImage 或 CGImageSource 创建图片时不会立即解码，设置到 UIImageView 或者 CALayer.contents 中且 CALayer 被提交到 GPU 前，CGImage 中的数据才会得到解码。这一步是发生在主线程的，并且不可避免。解决方法：后台线程先把图片绘制到 CGBitmapContext 中，然后从 Bitmap 直接创建图片（SDWebImage 就是这样做的）。

### GPU 消耗资源的原因和解决方案

- GPU 干的事情：接受 CPU 提交的纹理（Texture）和顶点描述（三角形），应用变换（transform）、混合并渲染，然后输出到缓冲。
- 纹理的渲染：较短时间显示大量图片（TableView 存在大量图片快速滚动）。解决方案：减少短时间内大量图片的显示，可以在滚动过程中先不显示图片，等到停止滚动的过程中再把图片显示出来。
- 视图的混合：视图结构很复杂时会拖累 GPU 的渲染速度，应尽量减少视图层级。把多个图片合并成一张显示。
- 图形的生成：减少圆角，使用圆角图片覆盖或者把需要显示的图片在后台线程绘制为图片。

主要参考了 [ibreme 的 iOS 保持界面流程的技巧](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

## 界面优化技巧

### 预排版

当获取到 API JSON 数据后，把每条 cell 需要的数据在后台线程计算并封装为一个布局对象 cellLayout。并把它们缓存到内存，后面可以直接使用这个对象。

### 预渲染

为了得到圆角图像，把下载下来的图片在后台线程预先渲染为圆形并单独保存到一个 ImageCache 中。

### 异步绘制

## view 显示到屏幕上的过程

根据上面的渲染原理可以得到这个问题的答案：

1. CPU 创建 view 对象并计算位置和文本宽高，之后进行文本的渲染和图片的解码（如果是图片的话）
2. GPU 接受 CPU 提交过来的纹理和顶点描述，应用变换、混合和渲染，然后输出到缓冲区

## 事件传递链

RunloopSource0 -> UIKitCore 的事件队列 -> UIWindow -> UIView ... -> 最底下的能够响应的 view

从UIWindow递归寻找子视图，并且对于同一层级的子视图使用倒叙遍历，分别调用每一个 view 的 `hittest` 方法。

## 一个 view 在下列情况不能响应事件：

- alpha < 0.01
- userInteractionEnable = NO
- hidden = YES

## 事件响应链

1. RunloopSource0
2. UIKitCore 事件队列
3. Application `sendEvent`
4. 触发事件的 UIControl `sendAction:to:forEvent:`
5. Application `sendAction:from:to:forEvent:`, 这里有参数target如果不为空则直接给target发送action，否则沿着响应链查询能够影响action的UIResponder。查询每个视图的方法是通过调用 `canPerformAction:withSender` 方法，如果当前 view 实现了 action 那么就返回 YES，否则继续沿着 nextResponder 找能成功响应的 responder。

![](../../images/responder-chain.png)
