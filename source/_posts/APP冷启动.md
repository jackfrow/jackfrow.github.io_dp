---
title: APP冷启动
date: 2019-8-23 11:21:40
tag: iOS
categories: 学习总结
---

#### 什么是APP冷启动

APP冷启动是指，APP点击启动前，它的进程不在系统里，需要系统新创建一个进程分配给它启动的过程。这是一次完整的启动过程。APP冷启动时间的长短是衡量一个APP好坏的重要标志。

#### 前言

网上好多文章完全就是乱扯，抄过去抄过来，在didFinishLaunchingWithOptions方法完成之前，你的APP怎么可能完成首屏渲染，吐槽一下。（手机运行会做一定程度的优化，并不会完全等didFinishLaunchingWithOptions执行完再去渲染首屏界面）

#### APP启动阶段

APP启动阶段主要分为两个阶段：

1.main()函数执行之前

2.main()函数之后

#### main函数执行之前

在main()函数执行之前，系统主要会做下面几件事情：

1. 加载可执行文件（APP的.o文件的集合）
2. 加载动态链接库,进行rebase指针调整和bing符号绑定
3. Objc运行时的初始化处理，包括 Objc 相关类的注册、category 注册、selector唯一性检查等
4. load()方法初始化

#### 优化方案

1. 对多个非系统动态库进行合并（最多支持6个）
2. 减少load()方法中的耗时操作

#### main()函数之后

main()函数指的是didFinishLaunchingWithOptions方法中执行的操作。

#### 优化方案

1. 将非首屏渲染必须的操作异步执行。
2. 如果有广告页面，可以考虑将耗时操作放广告页中执行一半(或者全部)，在首屏中再执行剩下的一半。

#### 检测手段

1. TimerProfile :使用指导:[Instruments Tutorial with Swift: Getting Started](https://link.juejin.im/?target=https%3A%2F%2Fwww.raywenderlich.com%2F397-instruments-tutorial-with-swift-getting-started)

2. 打点测试。