---
related: false # not related to any other posts.
title: "使用C#再次实现观察者模式"
category: "C#"
---

之前使用C#和C++实现过观察者模式，本质上C#的实现利用的是`System.EventHandler`类，其基于多播委托实现了语言内置的观察者模式。

实现起来很简单。支持注册事件的类如下：

```csharp
public class ObservableSubject
{
    public class EventArgument
    {
        public EventArgument(string concreteArgument) =>
            ConcreteArgument = concreteArgument;
        public string Car { get; }
    }

    public event EventHandler<EventArgument>? ObservableEvent;

    public void RaiseObservers(EventArgument argument) =>
        ObservableEvent?.Invoke(this, argument);
}
```

内部的事件是公开的，所以外部类可以注册，调用代码我觉得不需要专门写了吧...