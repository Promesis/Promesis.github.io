---
related: false # not related to any other posts.
title: "多个对象轮流参与消息传递的另一种方式 - 责任链"
category: "C-C++"
---

有些时候，我们并没有很大的空间去让多个对象在单个事件上部署，而且若是某个对象的成员函数，我们又要调用`std::bind`，或者使用Lambda。更重要的理由是，源代码级别的包含导致的可能是更紧密的耦合，而这是万万不可以的。

如果有一种将单个事件类型在一个类型层次结构“簇”中进行处理的方法就好了。

这就是“责任链”。

在执行特定操作时，若要使多个对象参与进来，**可以使用**责任链。

责任链常常用于事件处理。许多GUI都被设计成一系列事件和对应的响应。例如，用户点击File菜单，又在Edit菜单上浏览，最终选择了Undo项，就会发生`Undo`事件。

当事件发生时，会以某种方式与程序通信，接着，程序采取适当的行动。

就用GUI的一个简单例子讲解。

```cpp
export module design_patterns.chain_of_responsibility_pattern;

export template <typename Event>
class EventHandler
{
    public:
        // 等会讲为什么要用裸指针
        explicit EventHandler(Handler *next);
        virtual void handle(Event event);
    private:
        // NOTICE
        Handler *next_handler {nullptr};
};
```

```cpp
module design_patterns.chain_of_responsibility_pattern;

template <typename Event>
EventHandler::EventHandler(Handler *next) :
    next_handler {next}
{ }

template <typename Event>
void EventHandler::handle(Event event)
{
    if (next_handler)
        next_handler->handle(event);
}
```

接下来建立层次结构，可以处理消息。

```cpp
enum class Event : std::uint32_t
{
    application_handlable_event,
    window_handlable_event,
    button_handlable_event,
};

class Application : public EventHandler<Event>
{
    public:
        using EventHandler<Event>::EventHandler;

        void handle(Event event) override
        {
            std::cout << "Application::handle" << std::endl;
            if (event == Event::application_handlable_event)
                std::cout << "\tHandling application_handlable_event" << std::endl;
            else
                // 别忘了调用基类的handle方法 !!!
                EventHandler::handle(event);
        }
};

class Window : public EventHandler<Event>
{
    public:
        using EventHandler<Event>::EventHandler;

        void handle(Event event) override
        {
            std::cout << "Window::handle" << std::endl;
            if (event == Event::window_handlable_event)
                std::cout << "\tHandling window_handlable_event" << std::endl;
            else
                // 别忘了调用基类的handle方法 !!!
                EventHandler::handle(event);
        }
};

class Button : public EventHandler<Event>
{
    public:
        using EventHandler<Event>::EventHandler;

        void handle(Event event) override
        {
            std::cout << "Button::handle" << std::endl;
            if (event == Event::button_handlable_event)
                std::cout << "\tHandling button_handlable_event" << std::endl;
            else
                // 别忘了调用基类的handle方法 !!!
                EventHandler::handle(event);
        }
};

// 未完，接下面的代码框
```

接下来使用**正确**的方式调用它（等一下我会讲为什么要用正确的方式调用它）：

```cpp
// 接上面的代码框

int main(void)
{
    // 这就是为什么要在构造函数中使用裸指针   
    Application application {nullptr};  // ^   <-- 终点
    Window window {&application};       // |
    Button button {&window};            // |
                                        // 责任“链”，知道为什么要叫这个名字了吗？
    

    // 来试一下

    button.handle(Event::button_handlable_event);
    button.handle(Event::window_handlable_event);
    button.handle(Event::application_handlable_event);
    
    window.handle(Event::button_handlable_event);
    window.handle(Event::window_handlable_event);
    window.handle(Event::application_handlable_event);
    
    application.handle(Event::button_handlable_event);
    application.handle(Event::window_handlable_event);
    application.handle(Event::application_handlable_event);
    
    return EXIT_SUCCESS;
}
```

结果很好预测。（为什么我不显示编译和运行结果？因为我**最爱~的Clang编译器**对modules支持还不完善啊啊啊啊啊）

很明显，**必须有一些机制，将事件分派到正确的对象上**。三个对象的`handle`调用并没有被三个对象完全处理：后面两个对象都忽略了什么（由读者探索，很明显的）。

而且要很小心：

- 第一是，**一定要调用基类的`handle`方法**，不然事件会**丢失**。
- 更重要的是，**如果使用不当，会导致无限循环**。

~~待编辑，加入对上面两点的解释。~~（2023/8/24 更新）

首先讲为什么要调用基类的`handle`方法。这很显然。基类的`handle`方法使得派生类在不能处理事件时可以转移给另一个注册的事件。这是一个链状结构，不能“掉链子”。

然后讲为什么可能会出现无限循环。假如我故意对上面的调用代码这么改：

```cpp
int main(void)
{
    // oops! using lazy initialization
    Button button;
    Window window {&button};
    button = Button {&window};

    /*
                +----> [window] -----+
                |                    |
                +----- [button] <----+
    */

    button.handle(Event::application_handlable_event);
    return EXIT_SUCCESS;
}
```

两个对象都不能处理`Event::application_handlable_event`，互相推责任，导致责任链重环。更可惜的是，再智能的就算是`clang-format`都不能分析出任何问题。如果你有一个责任链没有“串联”好，会漏掉事件类型，而你的“串联”方式又是通过易错的责任环，就很容易导致无限循环。