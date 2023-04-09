---
layout: '../../layouts/MarkdownPost.astro'
title: 腾讯游戏学院 | 游戏模式
pubDate: 2021-10-17
description: 'UE的游戏模式'
author: 'Insyent居无何'
cover:
    url: 'https://oss.insyent.today/7.jpg'
    square: 'https://oss.insyent.today/7.jpg'
    alt: 'cover'
tags: ["游戏开发", "客户端", "UnrealEngine"]
theme: 'light'
featured: ture
---

# 目录

- UE4 GamePlay 关键元素
  - 规则与状态保持
  - 生命体
  - 必不可少的其他元素
- 网络同步
  - 值同步
  - RPC
- 联机加入对局
  - 连接对局流程
  - 角色控制
  - 角色和游戏流程的关联
  - 多人联机支持

# UE4 GamePlay 关键元素

- 游戏世界的规则与状态保持
  - GameMode，GameState，PlayerState
- 游戏世界中的生命体
  - Actor，Pawn，Character，"Controller"
- 其他必不可少的元素
  - Movement，Camera

## GameMode

​	制定和检测胜利规则，并支配世界内的“元素”，通过"GameState"记录游戏世界的关键状态，使用"PlayerState"保存玩家状态。通过 GameState 和 PlayerState 持久化游戏世界的数据。为对局恢复进行，为世界的重建提供支持

### 关键方法

- InitGame：在所有Actor激活(执行 PreInitializeComponents )之前调用 BeginPlay
- **PreLogin**：登陆准入操作，过滤非法玩家
- PostLogin
- HandleStartingNewPlayer：修改新玩家身上发生的事件
  - RestartPlayer：生成玩家pawn
    - SpawnDefaultPawnAtTransform
- **Logout**：离开游戏或被摧毁时调用（断线重连）

### 灵活配置

- WordSetting
- URL作为启动参数，指定GameMode
  - UE4Editor.exe/Game/Maps/MyMap?game=MyGameMode -game
- 配置默认的GameMode
  - DeafultEngine.ini文件的/Script/Engine.WorldSettings/ 部分中设置地图的前缀（和 URL法的别名）。这些前缀设置所有拥有特定前缀的地图的默认GameMode

## GameState

​	包含要复制到游戏中的每个客户端的信息，简而言之，它表示每个联网玩家的“游戏状态"。它通常包含有关游戏分数、比赛是否已开始和基于世界场景玩家人数要生成的AI数量的信息，以及其他特定于游戏的信息。

​	对于多人游戏，每个玩家的机器上都有一个游戏状态实例，而服务器的实例为权威实例。

### 关键方法

- **GetServerWorldTimeSeconds：GetTimeSeconds**的服务器版本，保持客户端和服务器上时间的同步
- PlayerArray：存储了所有玩家的APlayerState，方便遍历和获取玩家数据信息

## GameInstance

存在于整个游戏的生命周期，不随着地图的切换而销毁，非常适合非业务逻辑的全局管理操作

> Unreal另提供：submodule

## Camera

### 空间位置

- Transform
  - location
  - rotation

### 相机

- fov
- 景深
- camera apperance effect

# 网络同步

## 网络角色

- autonomous
- simulate

## RPC

### 两种设置

- Reliable
- Unreliable

### 类型

- Multicast

- RuN on Server client --> server
- Run on OwingClient：角色+背包（发光） server->client specific show

# 联机加入对局

[多人游戏框架理解](https://stonelzp.github.io/ue4-multiplay-framework/)

## 多人联机支持

- SteamCMD
