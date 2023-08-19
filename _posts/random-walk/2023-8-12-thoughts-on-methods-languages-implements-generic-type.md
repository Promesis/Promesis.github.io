---
related: false # not related to any other posts.
title: "对不同语言实现泛型的不同方式的理解和批判"
category: "Random Walk"
---

泛型是现代编程语言中提升编程体验（这是显然的）和代码重用程度的重要方式。泛型本身是一种哲学，指的是对不同类型的操作手段可以相同和“跨类型”。泛型本身也没有特定的实现方式。比如，C也可以实现泛型：

```c
// 这只是一个示例。可能有其他方式可以省去size参数。
void swap(void *lhs, void *rhs, size_t const size)
{
    void *buffer = malloc(size);
    *buffer = *lhs;
    *lhs = *rhs;
    *rhs = *buffer;

    free(buffer); 
}

int main(void)
{
    int a = 1, b = 2;
    assert(a == 1 && b == 2);
    float c = 1.0f, d = 2.0f;
    assert(c == 1.0f && d == 2.0f);
    //     ^~~~~~~~~    ^~~~~~~~~   <----没有修改变量，所以直接比较，不使用epsilon。
    swap(&a, &b, sizeof(int));
    assert(a == 2 && b == 1);
    swap(&c, &d, sizeof(float));
    assert(c == 2.0f && d == 1.0f);

    return EXIT_SUCCESS;
}
```

但是，这样的泛型容易出错。一旦我传入不同的类型（即使它们相同大小），代码的行为是未定义的。

Python本身也没有泛型，这种语言的泛型是通过特定的库实现的（`typing`，多么垃圾的机制啊）：

```python
T = TypeVar("T", int, str)
class GenericClass(object, Generic[T]):
    # ...
```

关于以上的根本不常用的方法，暂且不提。

接下来，开始介绍两大实现泛型的方法。很明显，C++两个都支持（因为C++支持的编程范式多）：

- 类型擦除（C#, Java等，当然C++可以）
- 代码生成（C++）

首先是类型擦除。实际上这是一种概括，指的是将类型的特性“擦除”掉，也就是消去。Java和C#中，由于任何类型都可以转化为Object（或C#中的`object`关键字），那么通过转换成C#和Java中的“根”类型即可实现类型擦除，通过类型向下转换便可以转换成原来的类型。（听说）早期Java没泛型的时候便是用这种方式实现伪泛型的。前面所说的类型向下转换，很明显需要进行一次类型识别，类似于C++中的RTTI，很明显，频繁的使用这些会大大降低性能。所以基本没有在不损失性能的方式下使用泛型的方法（如果使用语言内置的泛型）（就算有，我本身也不是Java程序员，欢迎在评论中讨论，我还是会写Java和C#代码的）

这是类型擦除的例子：

```java
public class Stack<T>
{
    private class Node
    {
        public Node(T x, Node down = null)
        {
            this.x = x;
            this.down = down;
        }
        public T x;
        public Node down;
    }

    private Node top;
    public Stack(T init)
    {
        top = new Node(init, null);
    }

    public void push(T x)
    {
        Node new_top = new Node(x, top);
        top = new_top;
    }

    public T pop()
    {
        if (top == null)
            throw new Exception("stack is empty");
        Node removed_top = top;
        top = top.down;
        removed_top.down = null;
        return removed_top.x;
    }

    public boolean empty()
    {
        return top == null;
    }
}
```

在字节码中，这段代码只会产生一次，显然使用了同一段代码应对多种类型。C#同。

我不是很喜欢为自己必须使用的特性付出额外的代价，特别是这个特性本身可以没有代价。

C++则可以实现无损失泛型。这是一个例子，实现了基于Lambda的自定义RAII：

```cpp
template <typename BeforeFunctionType, typename AfterFunctionType>
    requires requires(BeforeFunctionType beforeFunc, AfterFunctionType afterFunc)
    {
        {beforeFunc()} noexcept;
        {afterFunc()} noexcept;
    }
class Reminder
{
    private:
        AfterFunctionType after;
    public:
        inline Reminder(BeforeFunctionType before, AfterFunctionType after) noexcept :
            after {std::move(after)}
        {before();}

        inline ~Reminder(void)
        {after();}


        Reminder(Reminder const&) = delete;
        Reminder &operator=(Reminder const&) = delete;

        Reminder(Reminder &&) = default;
        Reminder &operator=(Reminder &&) = default;

        Reminder(void) = delete;
};
```

注意此时构造函数和析构函数是`inline`的，所以真正的所谓“损失”，只有声明`after`变量所占用的内存和初始化所占用的时间。

C++11还有变参模板，让你写得出类型安全的多参数函数。C标准库（同C++头文件`<cstdio>`）有`std::printf`函数想必大家都非常熟悉，其接受一个字符串作为格式描述符，以及接下来的变参：`<cstdarg>`，这个函数的原型如下：

```c
int printf(char const* restrict format, ...);
```

如果你用不同类型的实参填入在格式描述符中的param，会有未定义行为：

```c
int main(int argc, char const**argv)
{
    float flt = 3.0f;
    printf("%d\n", flt);
    //      ^~          <--------- oops!
    return EXIT_SUCCESS;
}
```

而且不仅有类型安全问题，还有性能问题。每一次调用`std::printf`都会扫描一遍`va_list`的参数，（我自己的测试发现）对自定义类型的传参也有问题。

但有了变参模板，“变参”的实现变得类型安全，而且所有东西都是编译期完成的。`std::thread`的构造函数便是一个例子。`std::make_tuple`是另一个例子。

```cpp
template <typename ...Types>
constexpr std::tuple<VTypes...> make_tuple(Types &&...args); // C++14起
```
注意：

> （摘自cppreference）对于每个 Types... 中的 Ti ， Vtypes... 中的对应类型 Vi 为 std::decay<Ti>::type ，除非应用 std::decay 对某些类型 X 导致 std::reference_wrapper<X> ，该情况下推导的类型为 X&。

C++使用代码生成。你可以在标准库中找到大量使用模板的代码。可以说，模板便是C++标准库的基石。我并没有特指STL，而是整个标准库。读者可以自己去阅览FSF的标准库实现或者任何一个（LLVM Compiler Infrastructure，MSVC等），可以发现这句话的正确性。

泛型编程到元编程，是现代编程范式的新发展。无可置疑的是，C++的基于代码生成（模板）的泛型编程是最灵活最强大，性能的，当然也是最晦涩难懂的。

但是，C++20的概念（`concept`s）出来之后，这个情况被大大改善。C++20的Ranges库可以说完全基于概念，没有概念就没有现在这么好用的Ranges库。

很多人会说，这种代码生成会增大程序二进制体积，减慢编译速度。对于程序二进制体积，现在的电脑在这方面早就已经没有任何问题。对于编译速度，C++20随着模块和概念的出现，不仅让模板更加好用，而且编译速度也在**复杂度级别上**变快（从$O(N \cdot{} M)$变为$O(N + M)$了啊啊啊啊啊）。