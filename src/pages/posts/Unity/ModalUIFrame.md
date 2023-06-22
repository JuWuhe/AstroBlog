---
layout: '../../../layouts/MarkdownPost.astro'
title: Unity | 模态UI框架
pubDate: 2022-09-13
description: ''
author: 'Insyent居无何'
cover:
    url: 'https://oss.insyent.today/1.jpg'
    square: 'https://oss.insyent.today/1.jpg'
    alt: 'cover'
tags: ["游戏开发", "客户端", "Unity"]
theme: 'dark'
featured: ture
---


- 模态：一种禁用父屏幕但保持可见的模式，用户必须与模态交互才能返回主屏幕
- 完整含注释代码 + 示例 在[Github](https://github.com/JuWuhe/ModalUIFrame)上

# 概念图

![|inline](/images/ModalUIFrame.png)

# 代码详解

## BasePanel.cs

职责：所有UIPanel的抽象父类

基本属性如下

```csharp
public abstract class BasePanel
{
	public UIType UIType { get; private set; }
    public UITool UITool { get; private set; }
    protected Func<BasePanel,GameObject> createPanelAction;
    protected Action exitAction;
}
```

- UIManager 根据各种情况使用 PanelManager 管理各个具体的面板，对于这个接口我们做如下设计
  - BasePanel 的构造函数固定接收一个 UIType 类型变量，而具体面板类则根据需求构造
    - 因此我们可以在具体面板类中保存一个 static UIType 成员用于 base(UIType)
      *注：BasePanel 的 UIType 不可以声明为 static ，因为派生类与基类共享静态成员*
  - PanelManager 的接口接收一个 BasePanel 类型变量，在模块内部完成该面板的生成与成员的初始化，具体的下文再说

```csharp
    protected BasePanel(UIType uiType)
    {
        UIType = uiType;
    }

    public void Init(UITool uiTool,Func<BasePanel,GameObject> createPanel,Action exit)
    {
        UITool = uiTool;
        createPanelAction = createPanel;
        exitAction = exit;
    }
```

- 对于**模态**的 UIPanel 来讲，大致会处于四种状态：进入、暂停、继续、退出，于是很自然定义出四个虚方法
  - 这四个方法一定是通过 PanelManager 管理调用的
  - 其中 OnEnter 和 OnExit将各接收一个 Action，分别是进入完成后回调与退出完成后回调
    - 这个设计是考虑到面板在有进入动效和退出动效时（比如淡入淡出）
      销毁Object、取消模态状态 等上层 PanelManager 对该面板的管理行为的触发时机

```csharp
    public virtual void OnEnter(Action onEnterFinished = null)
    {
    }    

	public virtual void OnPause()
    {
    }

	public virtual void OnResume()
    {
    }

    public virtual void OnExit(Action onExitFinished = null)
    {
    }
```

## UIType.cs

职责：保管面板的属性和生成面板的必要数据

- 生成面板的必要数据
  - 这里给出一个简单直接的方法，在 UIController 模块中通过 `Resources.Load` 生成，于是必要的数据就是 Prefabs 的 路径

```csharp
    public string Path { get; private set; }
    public UIType(string path)
    {
        Path = path;
    }
```

- 面板的属性
  - 如上文中提到的，面板在有进入动效和退出动效时（比如淡入淡出）
    除了 PanelManager 管理的需要回调，面板自身也会有相应的在 
    **退出完成后** **销毁执行前**需要处理的逻辑

```csharp
public Action OnDestroy;
```

-  还有一些属性可以根据需求添加，比如该面板以哪种动效进入和退出等等
  - 如上文中提到的派生的具体面板类中使用 static 的 UIType 成员，那么这些属性使用枚举表示会更好操作
    这也是为什么这个类仍然被命名为 UIType 而不是 UIInfo

## UIController.cs

职责：创建和销毁面板的Object

- 必要的数据成员
  - 存储面板的数据结构

```csharp
private readonly Dictionary<UIType, GameObject> dicUI;
```

-  该 UIController 创建的所有面板的 Canvas

```csharp
private readonly GameObject parentCanvas;
```

- 必要的方法
  - 构造函数

```csharp
    public UIController(GameObject canvas)
    {
        dicUI = new Dictionary<UIType, GameObject>();
        parentCanvas = canvas;
    }
```

- 获取/创建一个 UI 面板

```csharp
public GameObject GetSingleUI(UIType type)
{
    if (!parentCanvas)
    {
        return null;
    }
    var ui = Object.Instantiate(Resources.Load<GameObject>(type.Path),parentCanvas.transform);
    dicUI.Add(type,ui);
    return ui;
}
```

- 销毁一个 UI 面板

```csharp
    public void DestroyUI(UIType type)
    {
        if (!dicUI.ContainsKey(type))
        {
            return;
        }
        Object.DestroyImmediate(dicUI[type]);
        dicUI.Remove(type);
        type.OnDestroy?.Invoke();
    }
```

## PanelManager.cs

职责：管理所有的面板

- UI 在屏幕上的 堆叠 显然是 栈 式结构，最后弹出的 UI 必然是最先退出的（模态的情况下），于是有

```csharp
private readonly Stack<BasePanel> stackPanel;
public bool HasPanelInStack => stackPanel.Count > 0;
```

- 同时PanelManager将 构造并独有 UIController 组件，用于管理面板

```csharp
    private readonly UIController uiController;
    public PanelManager(GameObject canvas)
    {
        stackPanel = new Stack<BasePanel>();
        uiController = new UIController(canvas);
    }
```

- 以及必备的栈操作方法
  - 在面板初始化时除了将 UITool 构造传入，同时给出了一个 Push 方法和 Pop 方法
    - 这样设计是给面板提供
      - 一个构造并唤起出该面板 子面板 的方法，实现多层级的面板关系
      - 一个主动回调上层模块退出方法的机制，通常用在面板的退出按钮上


```csharp
public GameObject Push(BasePanel nextPanel)
{
    if (HasPanelInStack)
    {
        stackPanel.Peek().OnPause();
    }

    stackPanel.Push(nextPanel);
    GameObject targetPanel = uiController.GetSingleUI(nextPanel.UIType);

    nextPanel.Init(new UITool(targetPanel),Push,Pop);
    nextPanel.OnEnter(() =>
    {
        
    });

    return targetPanel;
}

public void Pop()
{
    if (HasPanelInStack)
    {
        var currPanel = stackPanel.Peek();
        currPanel.OnExit(() =>
        {
            uiController.DestroyUI(currPanel.UIType);
        });
        stackPanel.Pop();
    }
    if (HasPanelInStack && !isClean)
    {
        stackPanel.Peek().OnResume();
    }
}

private bool isClean;
public void Clean()
{        
    isClean = true;
    while (HasPanelInStack)
    {
        Pop(true);
    }
    isClean = false;
}
```

## UIManager.cs

职责：外部沟通UI系统的唯一途径，也就是UI系统的Shell，构造并独有PanelManager组件

```csharp
public class UIManager : MonoBehaviour
{
    private PanelManager PanelManager { get; set; }

    private void Awake()
    {
        PanelManager = new PanelManager(panelCanvas);
    }
}
```

## UITool.cs

职责：获取面板及其子物体的Component

- 拥有该 UITool 的面板 GameObject

```csharp
    private readonly GameObject activePanel;

    public UITool(GameObject panel)
    {
        activePanel = panel;
    }
```

- 获取或添加一个组件

```csharp
    public T GetOrAddComponent<T>() where T : Component
    {
        if (!activePanel.TryGetComponent(out T component))
        {
            component = activePanel.AddComponent<T>();
        }

        return component;
    }
```

- 查找子对象

```csharp
public GameObject FindChildGameObject(string name)
{
    var trans = activePanel.GetComponentsInChildren<Transform>();
    return (from item in trans where item.name == name select item.gameObject).FirstOrDefault();
}
```

- 子对象获取或添加一个组件

```csharp
public T GetOrAddComponentInChildren<T>(string name) where T : Component
{
    var child = FindChildGameObject(name);
    if (!child)
    {
        return null;
    }
    if (!child.TryGetComponent(out T component))
    {
        component = child.AddComponent<T>();
    }

    return component;

}
```
