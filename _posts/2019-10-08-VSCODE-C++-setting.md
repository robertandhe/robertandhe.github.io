---
layout: post
title: "VSCODE C++ 配置"
subtitle: 'vscode'
date:       2019-10-08 12:00:00
author: "mudux"
# header-img: "img/post-bg-2015.jpg"
header-style: text
catalog: false
tags:
  - technique
---


>reference:https://www.zhihu.com/question/30315894/answer/154979413  
-thx to [谭九鼎](https://www.zhihu.com/people/tan-jiu-ding/activities)

# 安装扩展
我安装的扩展为
- C/C++ Tools
- Coder Runner
- Markdown All in One
- Markdown Preview Github Styling
- One Dark Pro主题  

# 安装编译器
- [LLVM Download Page](http://releases.llvm.org/download.html) 在此页面下载Clang。选 Pre-Built Binaries 中的 Windows (64-bit)，不需要下.sig文件
- [MinGW-w64- for 32 and 64 bit Windows](https://sourceforge.net/projects/mingw-w64/files/)选最新版本中的x86_64-posix-seh
*安装clang* 添加环境变量时，选 Add LLVM to the system PATH for all users, 我的路径为C:\LLVM
*安装MigGW-w64*解压压缩包，复制所有文件到clang文件夹 

# 验证
运行cmd，输入clang-v 或 gcc-v显示各自版本

# 配置json文件
创建工作目录，在工作目录下创建.vscode文件夹，json配置文件放在.vscode下。
经验证仅需配置两个即可  
## 1. launch.json
```c++
// https://github.com/Microsoft/vscode-cpptools/blob/master/launch.md
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch", // 配置名称，将会在启动配置的下拉菜单中显示
            "type": "cppdbg", // 配置类型，cppdbg对应cpptools提供的调试功能；可以认为此处只能是cppdbg
            "request": "launch", // 请求配置类型，可以为launch（启动）或attach（附加）
            "program": "${fileDirname}/${fileBasenameNoExtension}.exe", // 将要进行调试的程序的路径
            "args": [], // 程序调试时传递给程序的命令行参数，一般设为空即可
            "stopAtEntry": false, // 设为true时程序将暂停在程序入口处，相当于在main上打断点
            "cwd": "${workspaceFolder}", // 调试程序时的工作目录，此为工作区文件夹；改成${fileDirname}可变为文件所在目录
            "environment": [], // 环境变量
            "externalConsole": true, // 为true时使用单独的cmd窗口，与其它IDE一致；18年10月后设为false可调用VSC内置终端
            "internalConsoleOptions": "neverOpen", // 如果不设为neverOpen，调试时会跳到“调试控制台”选项卡，你应该不需要对gdb手动输命令吧？
            "MIMode": "gdb", // 指定连接的调试器，可以为gdb或lldb。但我没试过lldb
            "miDebuggerPath": "gdb.exe", // 调试器路径，Windows下后缀不能省略，Linux下则不要
            "setupCommands": [
                { // 模板自带，好像可以更好地显示STL容器的内容，具体作用自行Google
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": false
                }
            ],
            "preLaunchTask": "Compile" // 调试会话开始前执行的任务，一般为编译程序。与tasks.json的label相对应
        }
    ]
}
```

## 2. tasks.json
```c++
// https://code.visualstudio.com/docs/editor/tasks
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Compile", // 任务名称，与launch.json的preLaunchTask相对应
            "command": "clang++", // 要使用的编译器，C++用clang++；如果编译失败，改成gcc或g++试试，还有问题那就是你自己的代码有错误
            "args": [
                "${file}",
                "-o", // 指定输出文件名，不加该参数则默认输出a.exe，Linux下默认a.out
                "${fileDirname}/${fileBasenameNoExtension}.exe",
                "-g", // 生成和调试有关的信息
                "-Wall", // 开启额外警告
                "-static-libgcc", // 静态链接libgcc，一般都会加上
                // "--target=x86_64-w64-mingw", // clang的默认target为msvc，不加这一条就会找不到头文件；用gcc或者Linux则掉这一条
                "--target=x86_64-w64-mingw32", // clang的默认target为msvc，不加这一条就会找不到头文件；用gcc或者Linux则掉这一条
                "-fno-limit-debug-info",
                // "-exec -enable-pretty-printing",
                //"-std=c17", // C++最新标准为c++17，或根据自己的需要进行修改
            ], // 编译命令参数
            "type": "process", // process是vsc把预定义变量和转义解析后直接全部传给command；shell相当于先打开shell再输入命令，所以args还会经过shell再解析一遍
            "group": {
                "kind": "build",
                "isDefault": true // 不为true时ctrl shift B就要手动选择了
            },
            "presentation": {
                "echo": true,
                "reveal": "always", // 执行任务时是否跳转到终端面板，可以为always，silent，never。具体参见VSC的文档
                "focus": false, // 设为true后可以使执行task时焦点聚集在终端，但对编译C/C++来说，设为true没有意义
                "panel": "shared" // 不同的文件的编译信息共享一个终端面板
            },
            // "problemMatcher":"$gcc" // 此选项可以捕捉编译时终端里的报错信息；本文用的是clang，开了可能会出现双重报错信息；只用cpptools可以考虑启用
        }
    ]
}
```
实测``-fno-limit-debug-info``必须要有，否则调试时无法显示字符串。