---
related: false # not related to any other posts.
title: "对教科书中垃圾代码的批判"
category: "随笔"
---

# 对教科书中垃圾代码的批判

整洁的代码让人感到有“代码感”，整齐划一，简直没有办法再整洁了，而垃圾的代码让人头痛，单单看着就头痛，更别说作为一个高中生，竟然还要看着教科书和练习中的垃圾代码写题。哎。

人们都说Python代码简洁，“甚至写代码还要拿着一把尺子”，但人们似乎完全忽视了其他语言可以完全比Python整洁，所以我们可以得到一个结论：

> 代码的整洁只取决于编写者本身，而和语言一点关系都没有。

Python限制缩进，必须让你写出这样的代码：

```python
if something:
    doSomething()
```

但若你是个不擦屁股的流氓，你也可以滥用这种所谓“优势”：

```python
def someJunkFunction():
    while some_thing:
        if something:
            doThing()
            doOtherThing()
            if some_other_thing:
                doSomething()
                doSomeOtherThing()
            else:
                doSomeChecks()
                if some_other_other_thing:
                    doSomeOtherOtherThing()
                else:
                    doSomeThhhhhhing()
                    doSomeThhhhhhhing()
                doSomeOtherChecks()
                if some_check_failed:
                    doSomeOtherOtherChecks()
                    doSomeThiiiing()
                while some_predicate:
                    doSomeThiiiiiing()
            doOOOOOOOTHERTHING()
        doSomeTHing()
        for some_iterator in some_list:
            something()
            someOtherThing()
            if something:
                doThing()
                doOtherThing()
                if some_other_thing:
                    doSomething()
                    doSomeOtherThing()
                else:
                    doSomeChecks()
                    if some_other_other_thing:
                        doSomeOtherOtherThing()
                    else:
                        doSomeThhhhhhing()
                        doSomeThhhhhhhing()
                    doSomeOtherChecks()
                    if some_check_failed:
                        doSomeOtherOtherChecks()
                        doSomeThiiiing()
                    while some_predicate:
                        doSomeThiiiiiing()
                doOOOOOOOTHERTHING()
    for some_thing in some_other_list:
        if something:
            doThing()
            doOtherThing()
            if some_other_thing:
                doSomething()
                doSomeOtherThing()
            else:
                doSomeChecks()
                if some_other_other_thing:
                    doSomeOtherOtherThing()
                else:
                    doSomeThhhhhhing()
                    doSomeThhhhhhhing()
                doSomeOtherChecks()
                if some_check_failed:
                    doSomeOtherOtherChecks()
                    doSomeThiiiing()
                while some_predicate:
                    doSomeThiiiiiing()
            doOOOOOOOTHERTHING()
        doSomeTHing()
        for some_iterator in some_list:
            something()
            someOtherThing()
            if something:
                doThing()
                doOtherThing()
                if some_other_thing:
                    doSomething()
                    doSomeOtherThing()
                else:
                    doSomeChecks()
                    if some_other_other_thing:
                        doSomeOtherOtherThing()
                    else:
                        doSomeThhhhhhing()
                        doSomeThhhhhhhing()
                    doSomeOtherChecks()
                    if some_check_failed:
                        doSomeOtherOtherChecks()
                        doSomeThiiiing()
                    while some_predicate:
                        doSomeThiiiiiing()
                doOOOOOOOTHERTHING()
```

特别是再教科书和练习中，你没有那么大一页纸，但那些没有素养的技术老师们永远把代码写在寥寥几个函数中，但凡有稍微拆分一下代码的结构就谢天谢地了，于是，我们在书中看起来像这样：


```python
def someJunkFunction():
    while some_thing:
        if something:
            doThing()
            doOtherThing()
            if some_other_thing:
                doSomething()
```

```python
                doSomeOtherThing()
            else:
                doSomeChecks()
                if some_other_other_thing:
                    doSomeOtherOtherThing()
                else:
                    doSomeThhhhhhing()
                    doSomeThhhhhhhing()
```

```python
                doSomeOtherChecks()
                if some_check_failed:
                    doSomeOtherOtherChecks()
                    doSomeThiiiing()
                while some_predicate:
                    doSomeThiiiiiing()
            doOOOOOOOTHERTHING()
```

```python
        doSomeTHing()
        for some_iterator in some_list:
            something()
            someOtherThing()
            if something:
                doThing()
```

```python
                doOtherThing()
                if some_other_thing:
                    doSomething()
                    doSomeOtherThing()
                else:
                    doSomeChecks()
                    if some_other_other_thing:
                        doSomeOtherOtherThing()
                    else:
                        doSomeThhhhhhing()

```

```python
                        doSomeThhhhhhhing()
                    doSomeOtherChecks()
                    if some_check_failed:
                        doSomeOtherOtherChecks()
                        doSomeThiiiing()
                    while some_predicate:
                        doSomeThiiiiiing()
```

```python
                doOOOOOOOTHERTHING()
    for some_thing in some_other_list:
        if something:
            doThing()
            doOtherThing()
            if some_other_thing:
                doSomething()
                doSomeOtherThing()
            else:
                doSomeChecks()
```

```python
                if some_other_other_thing:
                    doSomeOtherOtherThing()
                else:
                    doSomeThhhhhhing()
                    doSomeThhhhhhhing()
                doSomeOtherChecks()
                if some_check_failed:
                    doSomeOtherOtherChecks()
```

```python
                    doSomeThiiiing()
                while some_predicate:
                    doSomeThiiiiiing()
            doOOOOOOOTHERTHING()
        doSomeTHing()
        for some_iterator in some_list:
            something()
```

```python
            someOtherThing()
            if something:
                doThing()
                doOtherThing()
                if some_other_thing:
                    doSomething()
                    doSomeOtherThing()
                else:
```

```python
                    doSomeChecks()
                    if some_other_other_thing:
                        doSomeOtherOtherThing()
                    else:
                        doSomeThhhhhhing()
                        doSomeThhhhhhhing()
                    doSomeOtherChecks()
                    if some_check_failed:
                        doSomeOtherOtherChecks()
                        doSomeThiiiing()
                    while some_predicate:
                        doSomeThiiiiiing()
                doOOOOOOOTHERTHING()
```

这还是老师有些良心，愿意为你眼前的代码添加长名字的情况。随着时间的推移，我们领到的试卷总算是将代码分块了，但永远都会违反SRP，OCP，LSP，ISP，DIP等基本设计原则，对象到处随意操控，而且名字！！！名字！！！名字！！！照样用单字符的名字！！！根本没有改正！！！写长名字你会死啊？你是没钱装IDE吗？？？

最终，今年，我们看到的是诸如此类的代码，没有任何改善（从53 2024 A版上面摘来）：

```python
s="1a20b3"
s=s[::-1]
t,sum=0,0
for ch in s:
    if "0" <= ch and ch <= 9:
        t=t*10+int(ch)
    else:
        sum+=t
        t=0
print(sum)
```

还有（二叉查找树，同样摘自53）：

```python
class BTNode:
    def __init__(self,data=None,left=None,right=None):
        self.data=data
        self.left=left
        self.right=right

class SBTree: # 这个名字取的太傻逼了，跟技术老师们的水平很相配
    def __init__(self,root=None):
        self.root=root
    def insert(self,root,data):
        if self.root is None:
            self.root=BTNode(data)
        elif data<root.data:
            if root.left is None:
                _____(1)_____
            else:
                self.insert(root.left,data)
        else:
            if root.right is None:
                root.right=BTNode(data)
            else:
                _____(2)_____
    def inorder(self,root):
        if root is None:
            return _____(3)_____
        print(root.data,end=",")
    def search(self,root,key):
        if root is None or root.data==key:
            return root
        elif key<root.data:
            return _____(5)_____
        else:
            return _____(6)_____

if __name__=="__main__":
    a=[3,5,2,6,4,1]
    print(a)
    sbt=SBTree()
    for i in a:
        sbt.insert(sbt.root,i)
    sbt.inorder(sbt.root)
    print()
    k=int(input())
    p=sbt.search(sbt.root,k)
    if p is not None:
        print(k,p.data)
    else:
        print(k,p)
```

我起草过Dimensium Coding Standard，我是不是整洁代码的实行者一目了然。身为深陷整洁代码中的开发者，对这种代码及其仇恨。渴望教育部能发通知下令整改代码规范和整洁要求，让技术考试和练习更考研素养，而非阅读理解。