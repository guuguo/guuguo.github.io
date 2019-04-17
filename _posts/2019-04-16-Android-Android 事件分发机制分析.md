---
layout:     post
title:      Android 事件分发机制分析
date:       2019-04-16
author:     guuguo
catalog: true
tags:
    - Android
    - View System
---


# Android 事件分发机制分析

## 前言

-  `Android`事件分发机制是`Android`开发者必须了解的基础
-  很多时候粗略了解总会有些遗漏，总是需要从头到尾梳理一遍，理清思路

> 1. 先通过基础认知，对分发机制有个初略了解，解开其神秘的面纱。然后再逐步深入，源码分析了解流程
> 2. 总结事件分发的流程图谱，对事件先后顺序有个清晰的认识
> 3. 结合现实开发中的一些案例，利用分发机制的知识定制化实现目标



![示意图](https://ws3.sinaimg.cn/large/006tNc79gy1g1zo74pqfej317k0nyjte.jpg)

## 1. 基础认知

### 1.1 事件分发的对象是谁？

答：点击事件（`Touch`事件）

- 定义 
  当用户触摸屏幕时（`View` 或 `ViewGroup`派生的控件），将产生点击事件（`Touch`事件）

  > `Touch`事件的相关细节（发生触摸的位置、时间等）被封装成`MotionEvent`对象

- 事件类型（4种）

| 事件类型                  | 具体动作                   |
| ------------------------- | -------------------------- |
| MotionEvent.ACTION_DOWN   | 按下View（所有事件的开始） |
| MotionEvent.ACTION_UP     | 抬起View（与DOWN对应）     |
| MotionEvent.ACTION_MOVE   | 滑动View                   |
| MotionEvent.ACTION_CANCEL | 结束事件（非人为原因）     |

- 特别说明：事件列

从手指接触屏幕 至 手指离开屏幕，这个过程产生的一系列事件

>  注：一般情况下，事件列都是以`DOWN`事件开始、`UP`事件结束，中间有无数的`MOVE`事件，如下图： 

![](https://ws2.sinaimg.cn/large/006tNc79ly1g24hgebzfjj30k107zjrj.jpg)


即当一个点击事件（`MotionEvent` ）产生后，系统需把这个事件传递给一个具体的 View 去处理。

### 1.2 事件分发的本质

答：将点击事件（`MotionEvent`）传递到某个具体的View & 处理的整个过程

> 即 事件传递的过程 = 分发过程。

### 1.3 事件在哪些对象之间进行传递？

答：`Activity`、`ViewGroup`、`View`

`Android`的`UI`界面由`Activity`、`ViewGroup`、`View` 及其派生类组成 

---------------------
