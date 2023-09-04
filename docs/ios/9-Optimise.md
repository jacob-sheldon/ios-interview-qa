---
layout: default
title: 应用优化
parent: iOS 相关
nav_order: 9
---

# 应用优化

## 包体积

- 资源压缩和优化
  - 图片压缩：使用 TinyPNG 来压缩图片，减少图片大小
  - 使用矢量图形：使用 SVG 来替代png，减少图像大小
  - 使用合适的图片格式：WebP < JPEG < PNG
- 移除不必要的代码和资源
  - 移除未使用的代码：定期检查代码
  - 移除未使用的资源：图片、视频等
- 使用 Bitcode
  - 在上传 AppStore 时开启 Bitcode，苹果服务器会对代码针对不同的设备和系统进行优化，可能会减少包体积，但是也有可能会增大
- 优化第三方库和框架
  - 只包含必要的部分：只包含第三方库中你实际需要的部分，避免将整个库包含在应用程序中
- 使用 Assets Catalogs
  - 使用 Asset Catalogs 来管理应用程序中的图片、图标等资源，以便于应用程序可以只包含当前设备所需的资源。
- 优化编译配置
  - 在 Xcode 中使用编译设置优化代码，如启用编译优化、去除调试符号等，以减小生成的二进制文件大小。
  - 启用优化选项：在 Xcode 项目的 "Build Settings" 中，确保启用了编译器优化选项。例如，选择 "Optimization Level" 为 "Fastest, Smallest"，这会使编译器尝试以更小的大小生成更快的代码。
  - 去除不必要的符号：在 "Build Settings" 中，将 "Strip Debug Symbols During Copy" 设置为 "Yes"。这将在构建过程中去除不必要的调试符号，减小可执行文件的大小。
  - 编译器选项：
    - 在 "Other C Flags" 和 "Other C++ Flags" 中，添加一些编译器标志来帮助进一步优化代码。例如，可以使用 "-Os" 来最小化代码大小。
    - 使用 "Dead Code Stripping"（死代码剥离）来去除未使用的代码片段。
- 使用 On-Demand Resources
  - 对于需要大量资源的应用，可以使用 On-Demand Resources 来将一些资源延迟加载，从而减小应用初始下载大小。


[App thinning, Bitcode, Slicing: tutorial (iOS app)](https://ankur-s20.medium.com/implementing-app-thinning-in-your-project-step-by-step-tutorial-ios-app-b3cfd139896d)
[On-Demand Resources Guide](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/On_Demand_Resources_Guide/index.html#//apple_ref/doc/uid/TP40015083-CH2-SW1)


## 启动时间

*待补充*

## 内存占用

[深入探索 iOS 内存优化](https://juejin.cn/post/6864492188404088846)

## 崩溃率

*待补充*