---
title: PVZ项目bug解决汇总
copyright: true
date: 2023-08-28 23:12:59
updated: 2023-08-28 23:12:59
tags:
    - 项目
    - bug
categories:
    - 项目
    - PVZ
excerpt: 记录在编写,重构PVZ时遇到的一些bug以及解决方法
---
## 构建项目

### 抽象类的构建

**描述:**

在编写按钮时, 我抽象了一个按钮基类(`IButton`), 在其中包含了按钮基本的操作(初始化, 显示, 判断点击, 点击事件四个方法). `Button`类继承自 `IButton`, 是(不可旋转的)矩形按钮, 其中添加了成员 `SDL_Rect`代表按钮的范围并实现 `PressButton`函数判断鼠标点击位置是否在矩形中. 最后由 `Button`派生一系列矩形的按钮对象.

构建时出现了找不到 `Button`中定义的 `PressButton`方法的bug, 我起初认为 `Button`也是抽象基类, 便使用 `add_library(button INTERFACE Button.cpp)`去构建该库, 然后在链接时使用 `target_link_library(XXX INTERFACE button)`.

**解决方法:**

使用 `add_library(button OBJECT Button.cpp)`同时链接时使用 `PUBLIC`的方式去链接, 这样子才能找到 `PressButton`方法.

### OBJECT库文件

我在项目中全部生成 `OBJECT`库(接口生成 `INTERFACE`库), 虽然避免了生成大量静态库文件, 但导致主程序中链接库文件时必须把继承过程中产生所有 `OBJECT`库全部链接(就算连接了派生类, 其父类也要链接). 如果不链接就会报错: 找不到某某函数的定义.
