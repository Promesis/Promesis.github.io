---
related: false # not related to any other posts.
title: "`moment-code`仓库的CI全解"
category: "O&M"
---

**最爱PowerShell啦！！！最性感的Shell软件！！！**


先展示[`moment-code`]的代码层次结构：

1. Languages
2. Topics
3. Projects


我使用PowerShell Core 7对[`moment-code`]仓库的构建工作进行自动化。使用根目录的PowerShell脚本`build-all.ps1`可以调用每一个language下的自定义构建脚本：

```powershell
function Build-All
{
    param ();

    $language_dir_objects = Get-LanguageDirObjects;
    $language_dirs = $language_dir_objects.Name;

    $built_language_dirs_count = 0;
    foreach ($language_dir in $language_dirs)
    {
        $built_language_dirs_count++;
        Build-LanguageDir $language_dir $built_language_dirs_count $language_dir_objects.Length;

    }
}

function Get-LanguageDirObjects
{
    return (Get-ChildItem -Attributes "directory" | Select-Object -Property "Name");
}

function Build-LanguageDir
{
    param (
        $language_dir,
        $built_language_dirs,
        $total_language_dirs
    );

    Write-Output "---- Building language `"$language_dir`" ($built_language_dirs / $total_language_dirs)";

    $root = (Get-Location).Path;
    if (Test-LanguageBuildingScript $language_dir)
    {
        Set-Location $language_dir;
        ./build.ps1;
        Set-Location $root;
    }
    else 
    {
        Write-Error -Message "Directory `"$language_dir`" doesn't contain build script";
    }

    Set-Location $root;
}


function Test-LanguageBuildingScript
{
    param (
        $language_dir
    );
    $root = (Get-Location).Path;
    Set-Location $language_dir;
    $result = Test-Path -Path "build.ps1";
    Set-Location $root;

    return $result;
}
Build-All;
```

然后在每一个Language下调用`/[lang]/build.ps1`。`build.ps1`可以调用每一个topic下的`build-topic.ps1`，并在缺失`build-topic.ps1`时发送错误，跳过文件夹。这是C#的`build-topic.ps1`：

```powershell
# build.ps1
function Test-TopicBuildingScript
{
    param (
        $topic_dir
    );
    $root = (Get-Location).Path;
    Set-Location $topic_dir;
    $result = Test-Path -Path "build-topic.ps1";
    Set-Location $root;

    return $result;
}


$subdirectory_objects = Get-ChildItem -Attributes "directory" | Select-Object -Property "Name"
$subdirectories = $subdirectory_objects.Name;
$count = 1;
foreach ($topic in $subdirectories)
{
    Write-Output "---- Building topic `"$item`" ($count / $($subdirectory_objects.Length)) ...";
    $root = (Get-Location).Path;
    if (Test-TopicBuildingScript $topic)
    {
        Set-Location $topic;
        ./build-topic.ps1;
        Set-Location $root;
    }
    else 
    {
        Write-Error -Message "Error: topic `"$topic`" doesn't contain building script. Skipping...";
    }

    $count++;
}
```

一个topic下可以包含多个item，说白了就是单个项目，所以对具有统一构建系统（哦~好爽~）的.NET项目来说，如果不需要自己提供构建脚本，那么直接`dotnet build`就可以了。但是注意：为了节省GitHub Actions的时间，并避免一些莫名其妙的bug，我把`dotnet build`变成了先`dotnet restore`后`dotnet msbuild`，并去除logo：`--nologo`，这样在GitHub Actions上的日志可以很好看（单击截图可以直接前往log页面）：

<img src="https://i.postimg.cc/zGq9sqKD/image.png" alt="GitHub Action Log" width=800 height=819>

那么，`build-topic.ps1`在这里（仅指.NET项目的`build-topic-ps1`）：

```powershell
function Test-ItemBuildingScript
{
    param (
        $item_dir
    );
    $root = (Get-Location).Path;
    Set-Location $item_dir;
    $result = Test-Path -Path "build-item.ps1";
    Set-Location $root;

    return $result;
}

function Test-ProjectFileOrSolutionFile
{
    param (
        $item_dir
    );
    $root = (Get-Location).Path;
    Set-Location $item_dir;
    $result = (Test-Path -Path "*.csproj") -or (Test-Path -Path "*.sln");
    Set-Location $root;

    return $result;
}


$subdirectory_objects = Get-ChildItem -Attributes "directory" | Select-Object -Property "Name"
$subdirectories = $subdirectory_objects.Name;
$count = 1;
foreach ($item in $subdirectories)
{
    Write-Output "---- Building item `"$item`" ($count / $($subdirectory_objects.Length)) ...";
    $root = (Get-Location).Path;
    if (Test-ItemBuildingScript $item)
    {
        Set-Location $item;
        .\build-item.ps1;
        Set-Location $root;
    }
    elseif (Test-ProjectFileOrSolutionFile $item)
    {
        Set-Location $item;
        dotnet restore --nologo;
        dotnet msbuild --nologo;
        Set-Location $root;
    }
    else 
    {
        Write-Error -Message "Error: item `"$item`" contains neither project / solution file nor custom building script. Skipping...";
    }

    $count++;
}
```


最后，使用对应的`ci.yml`来配置GitHub Actions，大功告成：

```yaml
name: Continuous Intergration

on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]

jobs:
  "Build-CSharp":

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Set up .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 6.0.x
    - name: Build all using PowerShell scripts
      run: pwsh build-all.ps1
```

这样我就可以在每一次提交时运行CI过程（点击图片可以直接查看所有Actions）：

<img src="https://i.postimg.cc/xjMMXzFC/image.png" alt="GitHub Actions Screenshot" width=1280 height=499>

甚至每一次提交PR都会自动运行状态检查。这是还没有完成检查的界面：

<img src="https://i.postimg.cc/mDhVxgb9/image.png" alt="GitHub Actions runs when submitting PR - not completed" width=1153 height=521>

这是检查完成后的：


<img src="https://i.postimg.cc/Y94NnD3f/image.png" alt="GitHub Actions runs when submitting PR - not completed" width=1131 height=518>


[`moment-code`]: https://github.com/Promesis/moment-code