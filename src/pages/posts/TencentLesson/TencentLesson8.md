---
layout: '../../../layouts/MarkdownPost.astro'
title: 腾讯游戏学院 | 渲染基础
pubDate: 2021-10-17
description: 'UE的Animation'
author: 'Insyent居无何'
cover:
    url: 'https://oss.insyent.today/8.jpg'
    square: 'https://oss.insyent.today/8.jpg'
    alt: 'cover'
tags: ["游戏开发", "客户端", "UnrealEngine"]
theme: 'light'
featured: ture
---

# 渲染基础概念介绍

## 按算法划分

- 光栅化
  - 投影
  - 速度快、硬件处理方便
- 光线追踪
  - 光线求交、采样
  - 质量更好、全局光照

## GPU编程语言 (Shader Language)

​	OpenGL、Vulkan、DX、Metal

## VS 顶点着色器

Vertex Shader

- 空间坐标系转换(矩阵运算)
  - 模型空间->世界空间->视图空间->投影空间->NDC空间->屏幕空间
- 顶点属性(顶点色、法向、切向等)计算

## PS 像素着色器

Pixel Shader

​	针对每个像素进行单独的着色计算，可使用的信息包括经过光栅化插值得到的顶点属性、相邻像素梯度信息等

## 编写GPU程序的过程

- 模型数据定义
- CPU侧定义数据解析方式（管线状态等）
- GPU侧
  - VS
  - PS

## 术语

- Draw Call
  - 一次数据准备、Shader设置输出到Render Target的过程
  - 从硬件层看就是一次完整的Pipeline（VS->光栅化->PS）循环
- Render Pass
  - 一组渲染状态相似（Shader、Blend Mode等）的Draw Call集合
  - 减少频繁状态转换的消耗

# UE4渲染基础

## Render Pipeline

不是一成不变的，要根据游戏的需求（渲染质量、性能等）**自定义**最合适的管线

## 材质系统

### 材质定义

反应物体与光交互的表面属性(Diffuse、Specular、Normal)的数据集(Texture)和算法

### PBR（Physically Based Rendering）

基于物理的渲染，利用真实世界的物理原理，通过各种数学方法推导并简化渲染方程，最终渲染出极具真实感画面的技术

### 材质编辑器

可视化Shader编辑工具

### 材质节点

- BaseColor：基础颜色
  - 接受RGB值
  - Const Color
  - Texture
- Metallic：金属度
  - 取值范围[0.0,1.0]（[非金属，纯金属]）
  - 中间值往往用于表现破败的金属
- Specular：光泽度
  - 取值范围[0.0,1.0]
  - 反射高光
- Roughness：粗糙度
  - 取值范围[0.0,1.0]
  - 值越小越光滑
- Anisotropy：各向异性
  - 取值范围[0.0,1.0]
  - 用作头发
- Emissive Color：自发光
  - BaseCikir是基础属性，会受到光照强度影响
  - EmissiveColor是物体自身能量发射，不受光照强弱影响
- Normal：法向
  - 低建模精度表现出高精度的效果
- Tangent：切向
  - 发丝、天使环
- World Position Offset：世界空间位置偏移
- Opacity：透明度
  - 需要设置对应的Blend Mode(非Opaque/Masked)
- Opacity Mask：透明度遮罩
  - 透明度小于指定值(Opacity Mask Clip Value)的像素会被Clip掉
  - 需要设置对应的Blend Mode(Masked)

### 材质属性

- 材质域
  - Surface：最常用，Static Mesh、Skeletal Mesh、Landscape等
  - Deferred Decals：喷漆
  - Light Function：灯光投射效果
  - PostProcess：后处理
  - User Interface：UMG、Slate UI
- Blend Mode
  - 非透明：Opaque、Masked(植被、头发等)
  - 透明：Translucent、Additive、Modulated
- 着色模型

### 材质实例

- 母材质
  - 模板
- 材质实例
  - 母材质实例化

## 后处理

正常渲染结束后，对最终图像的后期加工

### 操作方法

- 将Post Process Volume 拖入场景
- 选中PPV，在Details面板中选择需要激活的后处理效果

## 粒子特效

- 激活Niagara插件
- 从模板创建Niagara System
  - Render 设置粒子使用的材质
  - Emitter Update 设置发射器每帧更新的方式
  - Particle Spawn 设置粒子初始生成时的属性（Lifetime、Size、Rotation、Velocity）

## 自定义Shader

- Custom Node

## 蓝图调用材质

- 应用
  - 捏脸
  - 枪口火焰
- 使用
  - 创建参数化材质实例
  - 添加UI
  - 将Actor的PrimitiveComponent作为参数传给UI蓝图
  - 按钮事件创建动态材质实例(Create Dynamic Material Instance)，调用Set Texture Parameter Value改变材质贴图

# 图形调试工具

以Renderdoc为例

- 下载安装
- 激活Renderdoc插件
- 修改配置文件，打开
  - r.Shaders.Optimize = 0
  - r.Shaders.KeepDebugInfo = 1
- 运行
- 输入Renderdoc的命令
