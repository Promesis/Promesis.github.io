---
related: false # not related to any other posts.
title: "论C++中的消息传递 - 观察者模式"
category: "C-C++"
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
dealer.CreateANewCar("Ford Transit");
```

输出：

```
CarDealer, new car Ford Transit
sebastian: car Ford Transit is new here
```


方便吧？使用多播委托，可以轻松实现多个对象的广播，即，消息传递。

调用所有注册方法的顺序不一定相同。

但是C++没有呀？没关系，我们有“观察者模式”。

模块接口：

```cpp
export module design_patterns.observer_pattern;
import <functional>;
import <map>;

export template <typename ...Arguments>
class Event 
{
    public:
        using Handle = std::uint32_t;
        using Observer = std::function<void(Arguments...)>;

        virtual ~Event(void) = default;

        [[nodiscard]]
        Handle operator+=(Observer observer);

        [[nodiscard]]
        Handle add(Observer observer);

        Event &operator-=(Handle handle);

        Event &remove(Handle handle);


        void raise(Arguments ...arguments);

        /**
         * @brief Raise the event. As same as raise() mem fn.
         */
        void operator()(Arguments ...arguments);

    private:
        std::map<Handle, Observer> observers;
        Handle observer_counter {0};
};
```

实现模块：

```cpp
module design_patterns.observer_pattern;

import <utility>;

template <typename ...Arguments>
Event::Handle Event::operator+=(Event::Observer observer)
{
    return add(std::forward<Event::Observer>(observer));
}

template <typename ...Arguments>
Event::Handle Event::add(Event::Observer observer)
{
    auto current_counter {++observer_counter};
    observers[current_counter] = observer;
    return current_counter;
}

template <typename ...Arguments>
Event &Event::operator-=(Event::Handle handle)
{
    return remove(handle);
}

template <typename ...Arguments>
Event &Event::remove(Event::Handle handle)
{
    observers.erase(handle);
    return *this;
}

template <typename ...Arguments>
void Event::raise(Arguments ...arguments)
{
    for (auto &observer : observers)
        (observer.second)(arguments...);
}

template <typename ...Arguments>
void Event::operator()(Arguments ...arguments)
{
    raise(arguments...);
}
```

好啦！现在，跟之前C#中一样的，我们也可以写出类似的代码（导入模块什么的，省去了）：

车交易者的模块接口文件：

```cpp
export class CarDealer
{
    public:
        void createNewCar(std::string car_name);
        Event<std::string> &getEvent(void);
    private:
        Event<std::string> new_car_created_event;
};
```

对应的模块实现文件：

```cpp
void CarDealer::createNewCar(std::string car_name)
{
    std::cout << std::format("CarDealer, new car {}", car_name) << std::endl;
    new_car_created_event.raise(car_name);
}

Event<std::string> &getEvent(void)
{
    return new_car_created_event;
}
```

接下来是买车的人。

模块接口文件：

```cpp
export class Consumer
{
    public:
        Consumer(std::string full_name);
        void newCarIsHere(std::string car_name);
    private:
        std::string full_name;
};
```

对应的模块实现文件：

```cpp
Consumer::Consumer(std::string full_name) :
    full_name {full_name}
{ }

void Consumer::newCarIsHere(std::string car_name)
{
    std::cout << std:format("{}: car {} is new here", full_name, car_name) << std::endl; 
}
```

请注意看接下来的调用代码：

```cpp
int main(void)
{
    CarDeal dealer;
    Consumer sebastian {"sebastian"};

    dealer.getEvent() += std::bind(&Consumer::newCarIsHere, &sebastian, std::placeholders::_1);
    dealer.createNewCar("Ford Transit");

    return EXIT_SUCCESS;
}
```

输出类似。

注意，在对调用的`dealer.getEvent()`成员函数的返回值进行注册时，我们必须使用`std::bind`将一个生成的新函数提供给`Event::add() / Event::operator+=()`函数注册。其原因为C#的委托可以自动绑定到对应的对象，而C++内部使用对象指针`this`来跟踪调用母对象，因此需要将对象提供给`std::bind()`的第一个参数，而将第二个参数（不考虑对象指针，就是`car_name`参数）需要使用`std::placeholders::_1`来绑定。

> 有关对象指针的有关内容，请参阅《深度探索C++对象模型》，Stanley B. Lippman著。

实际上，C#中的`event`关键字就是一个语言内置的观察者模式，只不过通过多播委托和自动对象绑定，可以实现很大的简化。

最终完成于8/23日，此时玄关门口正下大暴雨。