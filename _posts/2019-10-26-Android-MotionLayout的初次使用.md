---
layout:     post
title:      MotionLayout的初次使用
date:       2019-11-26
author:     guuguo
catalog:    true
tags:
    - JetPack

---

## 引用constraintlayout 2.0 依赖  

>此布局支持MotionScenes中定义的约束集之间的过渡
>
>`MotionLayout`是`ConstraintLayout`，它允许您在各种状态之间设置布局动画。
>
>**注意：**`MotionLayout`是作为支持库提供的，您可以在从API级别14开始的Android系统上使用。
>
>MotionLayout链接并需要一个MotionScene文件。 该文件包含一个顶级标签“ MotionScene”

---

> 最新版本见 [maven仓库](https://mvnrepository.com/artifact/androidx.constraintlayout/constraintlayout)

```groovy
//androidx
implementation "androidx.constraintlayout:constraintlayout:2.0.0-beta3"
//或者support
implementation "com.android.support.constraint:constraintlayout:2.0.0-beta3"
```



## 准备步骤

在xml 布局中新增 MotionLayout 

