---
title: Android群英传读书笔记-第三章控件架构与自定义控件讲解
date: 2016-7-9 21:02:25
categories: 读书笔记
tags: [Android,群英传]
---

# Android控件架构
Android的每个控件都是占一块矩形的区域，大致的分两类，继承View和ViewGroup，ViewGroup相当于一个容器，他可以管理多个字View，，整个界面上的控件形成了一个树形结构，也就是我们常说的控件树，上层控件负责下层控件的测量和绘制，并且传递交互事件，通过findviewbyid（）这个方法来获取，其实就是遍历查找，在树形图的顶部都有一个ViewParent对象，这就是控制核心，所有的交互管理事件都是由它统一调度和分配，从而进行整个视图的控制。
<!--more-->

![](http://oeiu2t0ur.bkt.clouddn.com/20160308223320045.png)

# Android界面的架构图
![](http://oeiu2t0ur.bkt.clouddn.com/20160308223337794.png)
我们可以看到，每个activity都有一个window对象，在Android中，window对象通常由一个phonewindow来实现的，phonewindow将一个DecorView设置为整个窗口的根View，DecorView作为窗口界面的顶层视图，封装了一些通用的方法，可以说，DecorView将要显示的内容呈现在了phonewindow上，这里面所有的View监听，都是通过WindowManagerService来接收的，并通过Activity对象来回调相应的OnClicListener。在显示上，他将屏幕分成了两部分，一个title一个content，看到这里，大家应该能看到一个熟悉的界面ContentView，它是一个ID为content的framelayout，activity_main.xml就是设置在这个framelayout里面。

调用requestwindowFeature(Window.FEATURE_NO_TITLE)方法一定要在setContentView()之前调用才有效果。

# View的测量
# View的绘制
# ViewGroup的测量
# ViewGroup的绘制
# 自定义View
# 自定义ViewGroup
# 事件拦截机制分析
见View绘制流程分析的文章