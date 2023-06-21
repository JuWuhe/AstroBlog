---
layout: '../../../layouts/MarkdownPost.astro'
title: 腾讯游戏学院 | 基本物理
pubDate: 2021-10-17
description: 'UE的Animation'
author: 'Insyent居无何'
cover:
    url: 'https://oss.insyent.today/6.jpg'
    square: 'https://oss.insyent.today/6.jpg'
    alt: 'cover'
tags: ["游戏开发", "客户端", "UnrealEngine"]
theme: 'light'
featured: ture
---

# 概述

目前主流商业物理引擎

- PhysX
  - Unity3D、UE4

- Bullet
- Havok

# 物理元素

- Geometry 几何体
- 包围盒
  - AABB
  - OBB
  - DOP，K-DOP
- StaticMesh
  - 基本集合体
  - KDOPTree，Convex Mesh，Triangle Meshes
- SkeletalMesh
  - 采用基本几何体，配合约束作为骨骼模型的Physics Asset
- LandScape
  - 按照网格坐标，记录高度
- 在物理引擎中，碰撞会按照几何类型划分。在UE4中，会按照引擎中的对应，把实际应用的对象与物理实体联系起来

|              | SphereCapsulesBoxesPlanes | ConvexMeshes | TriangleMeshes | HeightFields |
| :----------: | :-----------------------: | :----------: | :------------: | :----------: |
| SkeletalMesh |            YES            |     YES      |       NO       |      NO      |
|  StaticMesh  |            YES            |     YES      |      YES       |      NO      |
|  LandScape   |            NO             |      NO      |      YES       |     YES      |

- 根据特性不同，对于引擎中使用的模型分别支持不同类型的碰撞几何体

- 物理材质
  - 摩擦力、弹力、密度

# 物理查询

主要是场景进行相关的碰撞检测，返回查询结果。

通过PhysX的接口实现

- `RayCast`（零大小的射线检测，即有向线段）
- `Sweeps`（非零大小的检测，即扫掠体）
- `Overlaps`（空间相交检测）

应用场景

- 命中判断
  - 有弹道则进行分帧分段的物理检测，如果是直线则是起点至终点的`RayCast`	
- 角色移动

查询流程

- 输入
  - 查询起点
  - 查询终点
  - 查询类型
- 过程
  - 遍历场景空间结构划分 AABBTree（二叉树）
  - 粗略找到一组可能的碰撞体
  - `Pre-filtering`（比如频道剔除）
  - 结构内部剔除（对复杂碰撞体损耗很高）
  - `Post-filtering`（某些结果是否被丢弃）
- 输出
  - 是否查到交互Shape
  - 位置
  - 法线

# 物理模拟

相当于把物体的移动托管给物理系统，不需要任何的接口调用，通过`PxScene::simulate`对场景进行模拟

基本流程

- Constrain生成
- 受力
- 更新位置和速度
- 检测碰撞
- 解算

开启物理模拟

- Damping
  - 线速度阻尼、角速度阻尼
- Constrains

力和冲量

- 力是持续效果，需要在Tick函数内持续增加
- 瞬时效果
