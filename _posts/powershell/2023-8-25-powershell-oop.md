---
related: false # not related to any other posts.
title: "论PowerShell中的OO思想体现"
category: "O&M"
---

PowerShell作为我认为最性感的shell，其操作手感是我认为最顺手的。在PowerShell Core发布后，对它的印象更加深刻：

- 所有输出都用表格显示，简直不能再好看了
- 预测性Intellisense（你有没有想过，敲代码时可以获得像IDE一样的补全体验——一个预测列表在你输入的下面提供补全——而且还是内置的，不需要任何插件！）
- 完整且统一的模块支持（PSGallery下载速度说的过去，而且生态统一——一个`Install-Module`就可以安装模块）
- **面向对象的编程模式**！！！
- 根本不用记忆任何指令——完整的不依赖于其他工具的（在线）帮助系统，而且命名格式统一——谓语宾语结构，虽然可能很长，但是，有Intellisense啊！！
- 完整的集成.NET函数的调用（你甚至可以像在C#中调用`System.Math.Sin()`一样，在PowerShell中调用`[System.Math]::Sin()`函数！！！）
- 完美的数学运算（`bc`去死吧）

尤其是其极其富有特色的面向对象编程模式。你可以访问一个函数返回的对象的属性：

```powershell
$path = (Get-Location).Path;
Write-Output "Get-Location returns a object contains a attribute name Path.";
Write-Output "It is: $path";
```

还可以调用.NET（就是C#）中的函数：

```powershell
$epsilon = 0.01;
$sinList = @();
# 同System.Math.PI
for ($i = 0; $i -lt [System.Math]::PI; $i += $epsilon)
{
    # 同System.Math.Sin()
    $sinList += [System.Math]::Sin($i);
}

Write-Output $sinList;
```

输出：

```
0
0.00999983333416666
0.0199986666933331
0.0299955002024957
0.0399893341866342
0.0499791692706783
0.0599640064794446
0.0699428473375328
0.0799146939691727
   ... snip ...
0.0315873984364761
0.0215909757261186
0.0115923939361809
0.00159265291650992
```

再想想，微软的命名一向很谦逊，自己公司的名字，又微，又软，还有很多，比如说，微软式中文。

就这个PowerShell，还有Power Toys，微软的命名非常有信心。

再看看函数吧。别人家bash，函数的参数根本没有规范，直接`$1``$2``$3``$xxx`了事，不同人写的脚本，命名的法则也不一致，完全就是灾难：

```bash

# 一种命名法，Pascal
function UpAltInst {
    # 直接用数字传参
    sudo update-alternatives --install /usr/bin$1 $1 /usr/bin/$2
}

# *nix脚本的完全无法猜测意图的简写命名法，类似于grep，sed，awk
function cfc {
    # 意图完全由内部代码表达
    sed -i "s#https\?://mirror.msys2.org/#https://mirrors.bfsu.edu.cn/msys2/#g" /etc/pacman.d/mirrorlist*
}
```
而PowerShell呢：

```powershell
# 统一的谓语-对象表达形式
function Install-Alternatives
{
    param (
        $AlternativeName,
        $RealReference
    );

    sudo update-alternatives --install /usr/bin/$AlternativeName $AlternativeName /usr/bin/$RealReference;
}

function Edit-PacmanDistFileContent
{
    param (
        # empty
    );
    # sed这么强大的工具还是要用的
    sed -i "s#https\?://mirror.msys2.org/#https://mirrors.bfsu.edu.cn/msys2/#g" /etc/pacman.d/mirrorlist*
}
```

它甚至有自己的谓语列表：

<img src="https://i.postimg.cc/Kvdt33Rz/image.png" alt="Microsoft Verbs list" width=790 height=699>

<img src="https://i.postimg.cc/RZmC6vQf/image.png" alt="MVL - Detail" width=800 height=887>

甚至，内置列表式补全（不需要安装任何插件）：

<img src="https://i.postimg.cc/cHFtTrhV/image.png" alt="PSReadline exampl." width=1048 height=279>

就连出错提示都跟Clang一样细心地把原来的代码显示出来：

<img src="https://i.postimg.cc/DyTdZbDc/image.png" alt="Error show" width=1038 height=294>

赶紧试一试享受吧。

PowerShell完全是跨平台的哦，Linux上，macOS上都可以用，免费**开源**。

可以看看<https://github.com/PowerShell>。