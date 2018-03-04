---
layout:     post
title:      "Android群英传读书笔记-第八章Activity与Activity调用栈分析"
date:       2016-07-15 22:10:25
author:     "afayp"
catalog:    true
tags:
    - Android
    - 读书笔记
---


# 一.Activity
系统采用Activity栈的方式来管理Activity

<!--more-->
## Activity形态
Activity一个最大的特点就是拥有多种形态，他可以在多种形态中自由切换，以此来控制自己的生命周期
> 
• Active/Running 处于栈的最顶层，可见，可交互
• Paused 失去焦点，被一个非全屏Activity覆盖或者一个透明的Activity覆盖（？）时转换成Paused状态，不可交互，所有状态信息，成员变量都保留。
• Stopped 被一个Activity完全覆盖就会进入Stopped状态，不可见，依然保留所有状态信息和成员变量。
• Killed 被销毁

## 生命周期

# 二.Android任务栈简介

# 三.AndroidManifest启动模式

这章大部分内容与`Android开发艺术探索`的第一章重复，就不写了。
见：[Activity的生命周期和启动模式，task(android-art第一章)](http://afayp.github.io/2016/04/14/Activity%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%92%8C%E5%90%AF%E5%8A%A8%E6%A8%A1%E5%BC%8F-android-art%E7%AC%AC%E4%B8%80%E7%AB%A0/)
