---
related: false # not related to any other posts.
title: "pImpl - 向用户代码隐藏实现的绝佳手段"
category: "C/C++"
---

**注意，从2023/8/19日后的所有代码，都将完全使用C++20 modules编写，一般情况下不再使用头文件的语法/句法编写。**

如标题所言，pImpl，中文名为“指向实现的指针”，是向用户代码隐藏实现的绝佳手段，为大多数OOP纯粹主义者所热爱。

其具体原理很简单，C++编译器需要在类的定义中（注意，不是前向声明）知道类的具体大小，因此需要提供（或由编译器推断出）下列内容，编译器才可以接受：

- 类的数据成员；
- 类的虚函数有多少（由方法定义中的虚函数可推断出）；
- 类的动态类型推断标签（由`dynamic_cast<>()`使用。若没有虚函数，则不需要此项）；

因为不管是什么类型，其指针大小已知（这是显然的），所以可以**在类的定义中，声明**嵌套结构/类（即所谓的“实现”），以及指向“实现”结构/类的指针，并**在实现代码中，定义**嵌套结构/类的数据成员和私有成员函数，从而向用户代码提供整洁的接口，隐藏所有具体实现。

请看此例。这是我最喜欢举的代码例子——这是一个多线程的同步字符流，提供类似于`std::osyncstream`（C++20）的功能。


```cpp 
// File name: logger.cppm
export module logger_interface;
import <string>;

export class Logger
{
    public:
        Logger(void);

        Logger(Logger const&) = delete;
        Logger &operator=(Logger const&) = delete;

        void send(std::string);
        virtual ~Logger(void) = default; // Implementation自身可以保证其不变量
    private:
        // ------ NOTICE ------
        struct Implementation;
        std::unique_ptr<Implementation> implementation;
};
```

```cpp
// File name: logger.cpp
// NOTICE the whole file!
module logger_interface;

import <thread>;
import <mutex>;
import <condition_variable>;
import <chrono>;

struct Logger::Implementation
{
    std::thread internal_thread;
    
    bool exit_flag;
    std::mutex exit_flag_mutex;
    
    std::queue<
        std::string,
        std::list<std::string>
        >   message_queue;
    std::condition_variable message_queue_cv;
    std::mutex message_queue_mutex;

    Implementation(void);
    ~Implementation(void);
    void listen(void);
    void addMessage(std::string);
    void stop(void);
    bool listeningProcessShouldStop(void);
    void getAndSendMessage(std::unique_lock<std::mutex>);
};

Logger::Implementation::Implementation(void) :
    internal_thread {&Logger::Implementation::listen, this},
    exit_flag {},
    exit_flag_mutex {},
    message_queue {},
    message_queue_cv {},
    message_queue_mutex {}
{ }

Logger::Implementation::~Implementation(void)
{
    implementation->stop();
}

Logger::Logger(void) :
    implementation {std::make_unique<Implementation>()}
{ }

void Logger::send(std::string message)
{
    implementation->addMessage(message);
}

void Logger::Implementation::listen(void)
{
    while(true)
    {
        if (listeningProcessShouldStop())
            break;
        std::unique_lock lk {message_queue_mutex};
        bool cv_stat = message_queue_cv.wait_for(
            lk, 
            std::chrono::milliseconds(100),
            [this]{return !message_queue.empty();}
        );
        if (!cv_stat)
            continue;
        else
            getAndSendMessage(std::move(lk));
    }
}

```

具体实现略去吧。懒得写了，主要看的是接口的简洁性。现在去掉实现代码框，只显示之前的接口代码，你可以观察一下它到底有多整洁，用户完全无法更改和hack。

更重要的一点是，你可以在修改实现的时候，避免所有代码的整体重新编译，给客户代码减轻负担，自己团队也很好维护。