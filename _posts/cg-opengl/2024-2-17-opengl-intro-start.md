---
related: false # not related to any other posts.
title: "OpenGL 学习笔记 - 序曲"
category: 
  - "OpenGL"
  - "Computer Graphics"
toc: true
---

接下来，我会时不时地更新关于计算机图形学的内容，全部发布在`Computer Graphics`分类之下。

让我们开始OpenGL的第一个程序——不是画三角形，而是画一个点。

# 第一个OpenGL程序

## OpenGL管线

屏幕上有那么多的像素，想象一下，如果我们用CPU来依次绘制屏幕上的每一个像素，那么每一帧将会像PPT一样卡顿。这就是为什么GPU和CPU都是一台电脑的重要组成部分。看似两者都可以进行数学运算，实际上两者的效率非常不一样，GPU的运算能力**远远高于**CPU。一个GPU的浮点计算能力可以达到**6.45TFLOPS**（以Geforce RTX 2060为例），而一个CPU的浮点计算能力极弱，以 Intel Cascade Lake 架构的 Xeon Platinum 8280 为例，该 CPU 的理论峰值双精度浮点性能为 2.4192 TFLOPS。而这已经是最顶尖的CPU级别了。

那么问题来了，屏幕上的那么多像素，我该如何编写程序让GPU帮我绘制呢？这就是图形API的用途。OpenGL、DirectX等都是图形API，它们都定义了一套**图形渲染管线**（Graphics Pipeline）。图形渲染管线可以被理解为一个流水线，它由一系列的阶段组成，每个阶段负责不同的任务。

![OpenGL管线](https://i.postimg.cc/w3KRG5j0/opengl-pipe.png)

OpenGL管线有**固定函数阶段**和**可编程阶段**。固定函数阶段由OpenGL自身控制，不可更改。可编程阶段由开发者控制，开发者可以更改这些阶段，以实现不同的效果。可编程阶段由以下**着色器**组成：

- **顶点着色器**（Vertex Shader）
- **细分曲面控制着色器**（Tessellation Control Shader）
- **细分曲面评估着色器**（Tessellation Evaluation Shader）
- **几何着色器**（Geometry Shader）
- **片段着色器**（Fragment Shader）
- **计算着色器**（Compute Shader）

那么什么是着色器呢？着色器是运行在GPU核心上的小型程序，它们可以接受输入数据，并生成输出数据。着色器之间的输入与输出可以由一定的方式连接，就像形成了一个管道一般这就是为什么OpenGL管线被称为管线（pipeline）的原因。

OpenGL应用程序必须配备顶点着色器和片段着色器，而在OpenGL中，着色器使用GLSL（OpenGL Shading Language）编写。GLSL是一种类似于C语言的编程语言，它被设计为能够在GPU上运行。GLSL的语法与C语言类似，但也有所不同。稍后我们将会编写顶点着色器，并通过一定的方式加载到程序中，传送到显存上，等到绘制图像（一个像素）的时候，GPU会运行它们。

OpenGL绘制图像时必须有一个窗口吧？这个窗口，如果通过Windows API等方式创建会非常麻烦（不是一点点麻烦！），所以我们使用OpenGL的一个库GLFW来创建窗口。GLFW是一个跨平台的库，它提供了一系列的函数，用于创建和管理窗口，接受设备（如键盘、鼠标、手柄等）的输入，总之所有本来需要我们干的杂事都可以交给GLFW，我们只需要专注于我们的OpenGL。

> ### GLAD，与手动加载OpenGL函数说再见
> OpenGL的函数是通过*动态链接库*的方式调用的，所以按理来说我们应该手动加载所有我们用到的OpenGL函数，也就是说，我们（本来）应该这么做（以`glUseProgram`为例）：
> ```cpp
>   void *p = (void *)wglGetProcAddress("glUseProgram");
>   if(p == 0 ||
>       (p == (void*)0x1) || (p == (void*)0x2) || (p == (void*)0x3) ||
>       (p == (void*)-1) )
>   {
>       HMODULE module = LoadLibraryA("opengl32.dll");
>       p = (void *)GetProcAddress(module, "glUseProgram");
>   }
>   // 定义函数原型
>   typedef void (APIENTRYP PFNGLUSEPROGRAMPROC) (GLuint program);
>   // 定义函数指针
>   PFNGLUSEPROGRAMPROC glUseProgram;
>   // 初始化函数指针
>   glUseProgram = (PFNGLUSEPROGRAMPROC)p;
> ```
> 刚刚的代码，你看懂了吗？是个人都看不懂。而且这种手动获取函数地址的行为不仅没有可移植性（我们的这个示例仅在Windows平台上适用），而且极其容易出错。就连[OpenGL官方](https://www.khronos.org/opengl/wiki/Load_OpenGL_Functions)都非常不建议这种做法：
> > Loading OpenGL Functions is an important task for initializing OpenGL after creating an OpenGL context. You are **strongly advised to use an OpenGL Loading Library instead of a manual process**.
> 
> 因此，我们使用GLAD来获取OpenGL函数的地址。GLAD是一个开源的库，它能够自动获取OpenGL函数的地址，并将其存储在全局变量中，我们只需要调用GLAD的函数即可。

## 杂项准备

关于GLAD，GLFW等内容我全部放在一个代码块中。接下来我们的代码将会在下面的代码块中展示的代码里展开。我们使用OpenGL 4.5。

```cpp
#include <glad/gl.h>
#include <GLFW/glfw3.h>
#include <string>
#include <iostream>

std::string readFile(std::string file_name);

int main() 
{
    GLFWwindow* window;
 
    glfwSetErrorCallback(error_callback);
 
    if (!glfwInit())
        return EXIT_FAILURE;
 
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 2);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 0);
 
    window = glfwCreateWindow(640, 480, "example", NULL, NULL);
    if (!window)
    {
        glfwTerminate();
        return EXIT_FAILURE;
    }

    glfwMakeContextCurrent(window);
    gladLoadGL(glfwGetProcAddress);

    // 渲染循环
    while (!glfwWindowShouldClose(window))
    {
        // 渲染循环代码
        glfwSwapBuffers(window);
        glfwPollEvents();
    }
}

```

## 着色器代码加载器

为了加载着色器的GLSL代码，我们使用`readFile`函数。下面给出实现。

```cpp
std::string readFile(std::string file_name)
{
    std::ifstream file(file_name.c_str());
    if (!file.is_open())
    {
        std::cerr << "Could not open file " << file_name << std::endl;
        return "";
    }
    return std::string {
            std::istreambuf_iterator<char> {file},
            std::istreambuf_iterator<char> {};
        };
}

```

这段代码使用了C++标准库中的fstream文件流库和istreambuf_iterator。首先，通过fstream类的ifstream对象file打开文件，并使用std::istreambuf_iterator将文件内容迭代器绑定到file。然后，使用std::string的构造函数将迭代器范围内的字符创建一个新的std::string对象。最后，使用std::istreambuf_iterator的析构函数释放迭代器。

## 顶点着色器代码

```glsl
#version 450 core

void main()
{
    gl_Position = vec4(0.0, 0.0, 0.0, 1.0);
}
```

任何一个GLSL着色器代码都需要一个版本声明，告诉编译器代码所使用的GLSL版本。在例子中，我们使用的是4.50版本，并使用*核心模式*。

> ### 核心模式和立即渲染模式
>
> 刚刚的代码中，我们使用了`core`关键字。这个关键字告诉编译器使用**核心模式**。在核心模式下，编译器将只使用GLSL语言规范中OpenGL核心模式定义的特性。在很久以前的OpenGL程序中，使用立即渲染模式（Immediate mode，也就是固定渲染管线），这个模式下绘制图形很方便。OpenGL的大多数功能都被库隐藏起来，开发者很少有控制OpenGL如何进行计算的自由。而开发者迫切希望能有更多的灵活性。随着时间推移，规范越来越灵活，开发者对绘图细节有了更多的掌控。立即渲染模式确实容易使用和理解，但是效率太低。因此从OpenGL3.2开始，规范文档开始废弃立即渲染模式，并鼓励开发者在OpenGL的核心模式(Core-profile)下进行开发，这个分支的规范完全移除了旧的特性。
>
> 当使用OpenGL的核心模式时，OpenGL迫使我们使用现代的函数。当我们试图使用一个已废弃的函数时，OpenGL会抛出一个错误并终止绘图。现代函数的优势是更高的灵活性和效率，然而也更难于学习。立即渲染模式从OpenGL实际运作中抽象掉了很多细节，因此它在易于学习的同时，也很难让人去把握OpenGL具体是如何运作的。现代函数要求使用者真正理解OpenGL和图形编程，它有一些难度，然而提供了更多的灵活性，更高的效率，更重要的是可以更深入的理解图形编程。
>
> 使用`compatibility`关键字可以告诉编译器使用立即渲染模式（也称*兼容模式*）。

然后，我们声明了主函数`main`，这是每一个着色器函数的入口点，并且它没有参数。在主函数中，我们设置`gl_Position`变量，这个变量是顶点着色器输出的一部分，它定义了顶点在屏幕上的位置。  

> ### 输出变量，与内置变量
> 我们使用了`gl_Position`变量，它是一个输出变量。输出变量是顶点着色器向片段着色器传递数据的一种方式。本来根据GLSL的语法，我们应当这么声明`gl_Position`：
>
> ```glsl
> out vec4 gl_Position;
> ```
>
> 然而，`gl_Position`是一个GLSL预定义的变量，它代表的是当前顶点在屏幕上的位置。我们不需要声明它，只需要使用它即可。
> 在OpenGL的参考卡片中，我们可以看到内置的变量列表：
> 
> [![1f2c965ca18ab3498f820cbed3f224af.png](https://s1.imagehub.cc/images/2024/02/17/1f2c965ca18ab3498f820cbed3f224af.png)](https://www.imagehub.cc/image/1V5wKh)

我们的输出是`gl_Position`变量，在截图中可以看到，它是一个`vec4`类型的变量。`vec4`代表的是一个包含4个分量的4维向量。之后我会讲到向量的分量，重组（swizzling）和重组的顺序。

这篇博客尚未写完。敬请期待。