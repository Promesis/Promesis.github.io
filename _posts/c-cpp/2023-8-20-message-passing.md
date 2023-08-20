---
related: false # not related to any other posts.
title: "论C++中的消息传递 - 责任链"
category: "C/C++"
---

消息传递是一个非常有用的程序设计技术。利用消息传递，你可以编写出灵活的GUI代码，与GUI中种种时间打交道；你可以编写出模组，当实体被攻击时做出反击玩家的操作；总之，消息传递的重要性，不言而喻。

很可惜C++中暂无语言内置的消息传递机制，但是像C#中类似的代码你可以这么写（摘自Christian Nagel, Professional C# and .NET 2021 Edition）：

```csharp
using System;

public class CarInfoEventArgs : EventArgs
{
    public CarInfoEventArgs(string car) => Car = car;
    public string Car { get; }
}


public class CarDealer
{
    public event EventHandler<CarInfoEventArgs>? NewCarCreated;
    public void CreateANewCar(string car)
    {
        Console.WriteLine($"CarDealer, new car {car}");
        RaiseNewCarCreated(car);
    }

    private void RaiseNewCarCreated(string car) =>
        NewCarCreated?.Invoke(this, new CarInfoEventArgs(car));
}
```

这是买汽车的人：

```csharp
public record Consumer(string Name)
{
    public void NewCarIsHere(object? sender, CarInfoEventArgs e) =>
        Console.WriteLine($"{Name}: car {e.Car} is new here");
}
```

调用代码（顶级语句）：

```csharp
CarDealer dealer = new();
Consumer sebastian = new("sebastian");
// 注册事件
dealer.NewCarInfo += sebastian.NewCarIsHere;
dealer.NewCar("Ford Transit");
```

输出：

```
CarDealer, new car Ford Transit
sebastian: car Ford Transit is new here
```


方便吧？使用多播委托，可以轻松实现多个对象的广播，即，消息传递。

调用所有注册方法的顺序不一定相同。

但是C++没有呀？没关系，我们有“责任链模式”。

**（待编辑）**