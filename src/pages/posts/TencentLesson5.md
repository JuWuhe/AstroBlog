---
layout: '../../layouts/MarkdownPost.astro'
title: 腾讯游戏学院 | 骨骼动画
pubDate: 2021-10-17
description: 'UE的Animation'
author: 'Insyent居无何'
cover:
    url: 'https://oss.insyent.today/5.jpg'
    square: 'https://oss.insyent.today/5.jpg'
    alt: 'cover'
tags: ["游戏开发", "客户端", "UnrealEngine"]
theme: 'light'
featured: ture
---

# 动画概述

​	逐帧拍摄对象并连续播放而形成运动的影像技术

- 序列帧动画：早期，逐帧画
- 顶点动画：模型顶点运动，一些列关键帧保存成关键帧序列
- 骨骼动画：绑骨，编辑骨骼
- 2D骨骼动画：角色图片拆分成若干图元，每个图元绑定对应骨骼

# UE4动画蓝图

- 继承AnimInstance，或写一个继承AnimInstance的Cpp类后动画蓝图继承该Cpp类
- 动画融合
- EventGraph
  - `Blueprint Initialize Animation`
  - `Blueprint Upadate Animation`
  - `Animation Notify`
- AnimGraph：树形结构来排列组合各动画控制节点

# 常用动画类型

- Animation Sequences：关键帧序列
  - 指定某一帧 `AnimNotify`
  - 动画曲线：随着动画播放改变变量
  - 动画叠加 AnimAdditive
    - 输出的Pose为此动画当前帧的Pose与设置的BasePose的差
    - AdditvieAnimType决定了此结果的Pose中骨骼变换的数据属于哪个空间
    - `Apply Mesh Spcae Additive`
    - Local Space下，Pose中保存的每个骨骼的变换数据为相对父骨骼的变换
    - Mesh Space下，Pose中保存的每个骨骼的变换数据为相对骨骼模型组件的变换

- Blend Spaces
  - 由若干个Anim Sequences构成，所以当所有Anim Sequences都为叠加动画时，即可输出为叠加下Pose

- AimOffset：Blend Spaces的子集，一个在MeshSpace下具有叠加属性的BlendSpace

- Montage

  - 在编辑器中创建的动画资源，由若干个Anim Sequence组成

  - 智能循环、基于逻辑的动画切换

  - 一个Montage可以设置若干个Slot，具体哪个生效由AnimGraph决定

  - 每个Slot中可以拖入若干个Anim Sequence，顺序可以按需更改。如果Anim Sequence为叠加型动画，则这个Montage也为叠加型Montage

  - Montage可以有若干个Section，Section把整个Montage拆分成若干块，这些块之间可以自由的衔接和跳转

  - 可以添加自己的Notifies，不影响Anim Sequence的Notify

  - BlendIn、BlendOut 淡入淡出

  - 如果Montage所使用的Anim Sequence勾选了RootMotion，则这个动画在播放过程中根骨骼的位移会使角色产生移动。这个功能要与Character Movement组件配合。移动组件在更新速度时会优先使用根骨骼位移来作为移动的值。

    > 注意:根骨动画需要客户端和服务器同时播，如果服务器不播动画或者时间差太多，客户端会被拉扯

# 常见动画节点

- 混合节点

  - `ApplyAdditive`：在LocalSpace下动画叠加
  - `ApplyMeshSpaceAdditive`：MeshSpace下执行叠加
  - `Blend`：把两个Pose根据Alpha参数作为权重进行混合
  - `BlendBoneByChannel`：可以指定一根骨骼与另一根骨骼混合（不常用）
  - `BlendMulti`：同时对多个Pose进行混合
  - `BlendPosesByBool`和`BlendPosesByInt`：类似Switch Case
  - `LayeredBlendPerBone`：可以由指定某个骨骼开始对BasePose进行覆盖，覆盖时可以选择LocalSpace或MeshSpace
  - `MakeDynamicAdditive`：动态生成叠加型Pose，ApplyAdditive的反向操作，输出为两个输入Pose的差
  - ![image-20211107103837952](C:\Users\10499\AppData\Roaming\Typora\typora-user-images\image-20211107103837952.png)

- 空间转换 Space Controls

  - 会把输入的Pose中存储的每个骨骼的变换信息全部按新的空间进行重新计算，所以应该尽量减少这种节点的使用

- 状态机 State Machine

  - 状态机提供了图形化的方法来控制动画的切换，比如姿势切换、武器、跳跃等等。状态之间可以设置转换条件以及转换的融合相关数据

    > 注意:频繁的快速转换不适合设置太长的转换融合

# 动画Debug

- `ShowDebug Animation`
- `ShowDebugToggleSubcategoryGraph`
- `slomo`
