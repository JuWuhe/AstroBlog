---
layout: '../../layouts/MarkdownPost.astro'
title: 腾讯游戏学院 | 网络同步
pubDate: 2021-10-17
description: 'UE的Animation'
author: 'Insyent居无何'
cover:
    url: 'https://oss.insyent.today/7.jpg'
    square: 'https://oss.insyent.today/7.jpg'
    alt: 'cover'
tags: ["游戏开发", "客户端", "UnrealEngine"]
theme: 'light'
featured: ture
---

# 计算机网络基础

- Socket抽象层
  - TCP
  - UDP
- TCP/IP
  - 三次握手协议
- UDP
  - 没有Listen、Accept
  - 客户端没有Connect流程
- **Unreal使用纯UDP协议**
  - TCP协议的可靠性无法定制
    - 所有数据都是可靠的，而游戏中很多数据允许丢失
    - 为了保证数据的时序性，牺牲了时效性，一旦丢包则会阻塞后面数据发送
  - TCP与UDP混合使用？
    - 对于 *可靠性*  要求很高的数据，通过 *TCP* 传输
    - 对于 *时效性*  要求很高的数据，通过 *UDP* 传输
    - 增加了设计的复杂度
    - TCP与UDP都是基于IP协议，在底层会相互干扰
  - 纯UDP协议的好处
    - 可以定制丢包时的处理逻辑
    - 可以同时兼顾可靠性数据和时效性数据
- P2P连接
  - 点对点
- C/S架构
  - 客户端之间的通信必须通过服务器

# 数据同步基础

- 帧同步
  - MOBA、RTS
  - 只同步操作，大部分游戏逻辑在客户端上实现，服务器只负责广播和验证操作
  - 一致性好、可重播
- 状态同步
  - FPS
  - 客户端上传操作到服务器，服务器收到后计算游戏行为的结果，然后广播的方式下发游戏中各种状态，客户端收到状态后再根据状态显示内容
  - 不严谨、对网络延迟要求不高

| 状态同步VS帧同步 | 帧同步 | 状态同步 |
| :--------------: | :----: | :------: |
|      安全性      |   低   |    高    |
|     开发效率     |   高   |    低    |
|     网速要求     |   高   |    低    |
|     流量消耗     |   低   |    高    |
|    打击感表现    |  较好  |   较差   |
|    反外挂能力    |   弱   |    强    |
|   断线重连速度   |  较长  |   较短   |
|  重播实现难易度  |   易   |    难    |

- RPC 远程过程调用
  - 本地调用远端提供的函数/方法
  - 问题
    - Call ID映射
    - 序列化和反序列化（传递参数）
    - 网络传输
- 对象序列化
  - 本地对象实例在另一端也能直接使用
  - 属性同步
    - 服务器更改，同步到其他所有客户端，解决数据冲突和安全性

# UE4的网络同步

## Unreal网络模式

- NM_Standalone
- NM_DedicatedServer
- NM_ListenServer
- NM_Client

## Actor的复制

- 属性更新
- RPC

## 权威/主控/模拟

- ClientA 
  - 主控A 模拟B 

- ClientB 
  - 主控B 模拟A

- Server端 
  - 权威A 权威B

**Actor role/remote role **

## C++复制

```C++
class ENGINE_API AActor:public UObject
{
	UPROPERTY(replicated)
    AActor* Owner;
};

class ENGINE_API AActor:public UObject
{
    UPROPERTY(ReplicatedUsing = "function name");
    AActor* Owner;
    UFUNCTION()
    void ReplicatedFunction();
};

void AActor::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    ...
    DOREPLIFETIME(AActor,Owner);
}
```

## 属性复制条件

|          条件           | 效果                                                         | 使用场合                                                     |
| :---------------------: | :----------------------------------------------------------- | :----------------------------------------------------------- |
|    COND_InitialOnly     | 该属性仅在初始数据时尝试发送                                 | 只需要在初始化时同步，其他时候不需要同步（如玩家名字）       |
|     COND_OwnerOnly      | 该属性仅发送至actor的所有者                                  | 同步给主控端（比如玩家背包模拟端不需要，只有主控端需要）     |
|     COND_SkipOwner      | 该属性将发送至除所有者之外的每个连接                         |                                                              |
|   COND_SimulatedOnly    | 该属性仅发送至模拟actor                                      | 仅模拟端需要（如开火同步，主控端已经处理过了）               |
|   COND_AutonomousOnly   | 该属性仅发送给主控actor                                      | 仅主控Actor，比如玩家角色、PlayerController等                |
| COND_SimulatedOrPhysics | 该属性将发送至模拟或bRepPhysics actor                        |                                                              |
|   COND_InitialOrOwner   | 该属性将发送初始数据包，或者发送至actor所有者                |                                                              |
|       COND_Custom       | 该属性没有特定条件，但需要通过`SetCustomIsActiveOverride`得到开启/关闭能力 | 需要自定义控制复制开关（比如玩家濒死状态才需要同步的数据等） |

## RPC蓝图

- Multicast广播
- Run on Server
- Run on owing Client

添加自定义事件，选中事件节点，在details面板中修改Replicates属性

## 服务器调用RPC所有权

| Actor所有权  | 未复制         | NetMulticast               | Server         | Client                    |
| ------------ | -------------- | -------------------------- | -------------- | ------------------------- |
| Client-owned | 在服务器上运行 | 在服务器和所有客户端上运行 | 在服务器上运行 | 在actor的所属客户端上运行 |
| Server-owned | 在服务器上运行 | 在服务器和所有客户端上运行 | 在服务器上运行 | 在服务器上运行            |
| Unowned      | 在服务器上运行 | 在服务器和所有客户端上运行 | 在服务器上运行 | 在服务器上运行            |

## 客户端调用RPC所有权

| Actor所有权                 | 未复制                   | NetMulticast             | Server         | Client                   |
| --------------------------- | ------------------------ | ------------------------ | -------------- | ------------------------ |
| Owned by invoking client    | 在执行调用的客户端上运行 | 在执行调用的客户端上运行 | 在服务器上运行 | 在执行调用的客户端上运行 |
| Owned by a different client | 在执行调用的客户端上运行 | 在执行调用的客户端上运行 | 丢弃           | 在执行调用的客户端上运行 |
| Server-owned                | 在执行调用的客户端上运行 | 在执行调用的客户端上运行 | 丢弃           | 在执行调用的客户端上运行 |
| Unowned                     | 在执行调用的客户端上运行 | 在执行调用的客户端上运行 | 丢弃           | 在执行调用的客户端上运行 |

## 开火效果优化

问题：Server端开火客户端能看到，客户端开火Server端看不到

解决：添加两个CustomEvent，一个设定Server端运行RPC（ServerFire），一个广播RPC（MultiOnFire）

# 网络优化监测工具

Network Profiler
