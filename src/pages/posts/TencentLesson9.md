---
layout: '../../layouts/MarkdownPost.astro'
title: 腾讯游戏学院 | AI技术
pubDate: 2021-10-17
description: 'UE的Animation'
author: 'Insyent居无何'
cover:
    url: 'https://oss.insyent.today/1.jpg'
    square: 'https://oss.insyent.today/1.jpg'
    alt: 'cover'
tags: ["游戏开发", "客户端", "UnrealEngine"]
theme: 'light'
featured: ture
---

# AI框架简介

## AI分层结构体系

- 感知层：视觉、听觉、记忆信息、环境信息等
- 决策层：决策后的命令下达到命令层
  - Finite State Machines 有限状态机
    - 基本节点是状态，包含了一系列运行在该状态的行为以及离开这个状态的条件
    - 数据表示是图，状态可以任意跳转
  - Hierarchical FSM 层次状态机
    - 基本概念与有限状态机一样
    - 可以将一些状态节点归结成一个超级状态（Super-States），共享一些状态跳转逻辑（Generalized Transitions）
  - Behaviour Tree 行为树
    - 高度模块化状态，去掉状态中的跳转逻辑，使得状态变成一个行为
    - 行为之间的跳转通过父节点（Composite）的类型来决定
    - 数据表示是树，无需关注节点之间的跳转关系，节点在树中的层级以及先后顺序决定执行顺序
    - 拓展方便，通过增加控制节点的类型，可以达到复用行为的目的
  - 机器学习
    - 不需要现成的训练样本
    - 智能体与环境的交互中收集相应的样本（状态、动作、奖赏）进行试错学习
    - 从而不断地改善自身策略来获取最大的累积奖赏
- 命令层：将决策细化成行为指令
- 行为层：移动、开火、捡物品等
- 运动层：寻路系统、动画系统、物理系统等
  - 导航系统：根据地形生成导航网格，根据导航网格动态生成一条有效路径、A*算法
  - 动画系统：根据角色当前状态播放合适动画、多个动画序列融合、支持Inverted kinematics，根据约束条件计算匹配动作
  - 物理系统：场景碰撞查询功能、模拟一些物理效果

# UE4行为树介绍

## 节点介绍

- Root
  - 行为树起始执行节点
  - 每个行为树只有一个Root节点
  - 绑定黑板
  - 不能附加Decorators和Services节点
- Composite：定义一个分支的根以及该分支如何被执行的基本规则
  - Sequence
    - 从左到右执行子节点
    - 其中一个子节点失败时，序列节点也停止执行
    - 如果有子节点失败，那么序列就会失败
    - 如果所有子节点成功运行，则序列节点成功
  - Selector
    - 从左到右执行子节点
    - 其中一个子节点执行成功时，选择器节点停止执行
    - 如果一个子节点成功运行，则选择器运行成功
    - 如果所有子节点运行失败，则选择器运行失败
  - Simple Parallel
    - 节点允许一个主任务节点沿整个的行为树执行
    - 主任务完成后，结束模式（Finish Mode）中的设置会指示该节点是应该立即结束同时中止次要树，还是推迟结束知道次要树完成
  - Service
    - 节点通常附加至复合（Composite）或任务（Task）节点，只要其分支被执行，它们就会以定义的频率执行
    - 这些节点常用于检查和更新黑板
  - Decorator
    - 附加到复合（Composite）或任务（Task）节点，并定义树中的分支，甚至单个节点是否可以执行
    - 这些节点常用作节点是否可以执行的附加条件
  - Task
    - 行为树的叶子，是可执行的操作，没有输出链接

## 数据存储和传递

- Blackboard

  - AI的大脑，以Key-Value形式存储数据

  - 供行为树节点之间共享数据，以及决策使用

  - 可以供单个AI使用，也可以团队共享数据

  - 优点
    - 能够在节点之间共享中间数据
    - 将行为树执行需要的数据集中到一起，方便管理和编辑

- 节点实例内存

  ```c++
  struct FBTFocusMemory
  {
      //节点运行时临时数据结构
      ......
  };
  
  class AIMODULE_API UBTService_DefaultFocus : public UBTService_BlackboardBase
  {
      ......
  protected:
      //临时数据结构大小声明
      virtual uint16 GetInstanceMemorySize() const override { return sizeof(FBTFocusMemory); }
      //临时数据结构首地址 uint8* NodeMemory
  }
  ```

  

## 场景查询系统 EQS

- 用于收集场景相关数据
- 可以使用 生成器（用户可自定义），通过各种用户定义的测试就这些数据进行测试，返回符合所有测试的最佳结果

------

- 在编辑器设置中启用EQS

- 创建并编辑EQS
- 行为树中运行EQS `Run EQS Query`

# 参考资料

https://docs.unrealengine.com/en-US/index.html

https://www.gamasutra.com/

参考书籍

- AI GAME PROGRAMMING WISDOM 1~4
- GAME AI PRO 1~3

