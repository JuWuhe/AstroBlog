---
layout: '../../../layouts/MarkdownPost.astro'
title: 腾讯游戏学院 | UMG界面
pubDate: 2021-10-17
description: 'UE的UMG'
author: 'Insyent居无何'
cover:
    url: 'https://oss.insyent.today/4.jpg'
    square: 'https://oss.insyent.today/4.jpg'
    alt: 'cover'
tags: ["游戏开发", "客户端", "UnrealEngine"]
theme: 'light'
featured: ture
---

# UMG

- UE4提供的界面开发系统
  - HUD：例如准星
  - Slate：代码，是虚幻真正的UI框架
  - UMG：动效图形系统

- UMG是对Slate的封装，方便界面编辑以及支持虚幻的垃圾回收机制

# 基本概念

- 控件Widget
- UI蓝图
- 槽Slot：面板控件中用于摆放子控件，包含了定义如何摆放的数据信息
- 锚点Anchor：定位摆放在Slot中的控件

- 可见性
  - 折叠：不占据空间，其余空间会自动拉伸
  - 隐藏：仍然占据空间，只是不显示
  - 不可点击
  - 完全显示

- 渲染变形

# 常用控件

- 按钮 Button
- 复选框 Check Box
- 图像 Image
- 进度条 Progress Bar：经验值、血条等
- 文本 Text：显示文本
- 文本框 Text Box：用户输入框

- 面板Panel控件
  - 画布面板 Canvas Panel：随意摆放，按z序排序层叠
  - 网格面板 Grid Panel：平均分割可用空间
  - 水平框 Horizontal Box
  - 纵向框 Vertical Box
  - 滚动框 Scroll Box：任意滚动的控件，例如服务器列表中滚动查看，不支持虚拟化
  - 控件切换器 Widget Switcher：切页
- 优化控件
  - 无效框 Invalidation Box
  - 限位框 Retainer Box

# 界面动画

- 添加动画轨道(每个控件可以有多个轨道)
- 添加动画关键帧

# 使用和控制编辑好的界面

- 蓝图
  - 创建：`Create New Widget`
  - 显示：`Add to Viewport`
  - 控件变量：通过控件变量来操作控件，设置属性 (控件 is Variable)
  - 调用动画：创建动画时也会创建一个变量，`Play Animation`
  - 绑定事件

- LUA
  - [SLuaunreal插件](https://github.com/Tencent/sluaunreal)
  - [Sluaunreal的完整demo](https://github.com/IriskaDev/slua_unreal_demo)

# 拓展控件库

> 自定义控件，避免重复造轮子

- 纯UMG实现
  - 创建控件`NewWidget`
  - 在其他界面中使用 `User Created -> New Widget`
- C++实现
  - 实现`SNewWidget`类
  - 初始化布局
  - 事件绑定
  - 实现`UNewWidget`类
  - 在`RebuildWidget`方法中创建`SNewWidget`实例
  - 在`SynchronizePropertie`s方法中同步U对象与S对象的对应属性
  - 在UMG编辑页面中使用此控件

# 内存与性能

## 内存

资源引用形成、解耦

- 引用形成
  - 直接摆放在UI的Tree中
  - 变量类型
  - 创建UI时选择的类型

- 解耦
  - 避免在蓝图中直接访问类型中的方法，尽量通过基类、接口调用
  - 不要将各种使用的资源都配置在UI上或者蓝图里
  - 使用弱引用
  - 使用类似MVC的设计模式，独立实现界面的数据、视图以及控制逻辑

## 性能

-  不用或少用属性绑定
- 尽量不在UI蓝图中实现Tick
- 复杂蓝图逻辑转C++，然后在蓝图中调用C++实现的函数
- 正确设置“隐藏” (Hidden <==> Collapsed)
- 无效框：无效框可以缓存UI绘制的中间数据，减少绘制的CPU消耗
- 合理结构提升渲染性能
  - 减少UI层级，从而减少UI绘制过程中的递归调用(UI渲染是从根节点深度优先虚函数不断重载调用)
  - 合理规划UI层次，合并图集，批量渲染以减少DrawCall
