---
title: OI 环境配置指南
published: 2022-05-20
description: 设置 VSCode 算法竞赛环境
image: 'https://s2.loli.net/2023/08/17/tJR6olpYaNW1Am8.jpg'
tags: [C++, Algorithms]
category: 'C++'
draft: false
---

## 零、序言

随着时代的发展，`CCF` 终于摒弃了使用多年的 `Noi Linux v1`，转而使用基于 `Ubuntu 20.04 LTS` 的 `Noi Linux v2`。其中的 C++ 编译器由 `g++ 4.8.4` 更新为 `g++ 9.3.0`，其默认语言标准也由 C++98 转为 C++14，这意味着 C++11 以上的优良新特性可以在竞赛时使用，大大方便了我们代码的编写。

然而，很多老师仍然推荐使用老旧的 `Dev-C++` 编写竞赛代码。`Dev-C++` 作为算法竞赛编辑器，其最大优势便是支持单文件编译及便捷调试。但其自带的 C++ 编译器仍使用 `gcc 4.9.2`，仅支持 C++11 的部分特性并且完全不支持 C++14 标准；其代码编辑界面也简陋至极，一些新字体竟然不支持 Ligature（如 JetBrains Mono，Fira Code 等）；作为一款 IDE，其代码补全功能仅为半残且经常出现 bug。可见 `Dev-C++` 已经完全落后于时代 （ps，国人重新开发的 [小熊猫 Dev-C++](https://royqh.net/devcpp/) 是一个高效的算法竞赛 IDE，但其仍不够稳定故本文未提及）

还有一些同学在编写竞赛代码时，文件经常乱命名、乱堆桌面，想第二天调试的时候不知道哪个文件对应哪道题（笔者真的看到学习 OI 的同学出现这种情况）

本文将从新款 C++ 编译器、代码编辑器的安装和使用方面对高效且整洁的 OI 环境配置进行简要阐述（基于 Windows 10）。成品如下：

![成品](https://s2.loli.net/2022/05/07/HFzqdg5yNAlJY4w.png)

请注意：

* 确保你的系统盘拥有 **1 GB** 以上空间
* 安装过程中请确保**网络畅通**

## 一、C++ 编译器的安装

在 Windows 平台下，主流 C/C++ 编译器分为 `MSVC` 系列与 `MinGW` 系列。`MSVC` 系列往往需要和微软的 `Visual Studio` 捆绑安装，体积巨大，使用复杂，适合工程开发却不适合竞赛使用。故网上大多配置环境的教程多为 `MinGW` 系列编译器，即 `gcc` 或 `clang` 工具链。此套工具链与 `Linux` 平台工具链一致，且体积小、易于使用，多为轻度 C/C++ 使用者的最爱。本文也使用 `MinGW` 系列编译器进行安装配置。

`MinGW` 系列编译器分为 `MinGW` 与 `MinGW w64`。其中 `MinGW` 仅支持 32 位开发，不支持 64 位，并且更新慢，故逐渐被淘汰。而 `MinGW w64` 支持 64 位开发，故受到欢迎。[Jmeubank](https://github.com/jmeubank) 又基于 `MinGW w64` 对其进行优化，并发行了 `TDM GCC`，易于安装和配置。本文采用 `TDM GCC` 进行配置。

首先，进入 `TDM GCC` 的下载网站：[https://jmeubank.github.io/tdm-gcc/download/](https://jmeubank.github.io/tdm-gcc/download/)，点击左端的 `tdm64-gcc-*.*.*.exe` 下载 32/64 位的 `TDM GCC`（一定要下载 `MinGW w64 based），如图。

::github{repo="jmeubank/tdm-gcc"}

![TDM GCC 下载](https://s2.loli.net/2022/05/07/YiWHFGTLygmhRMk.png)

(ps，GitHub 在国内访问缓慢，可访问 [ghproxy.com](https://ghproxy.com) 加速下载文件)

安装步骤见图（**安装路径不要有中文或特殊字符**）：

![1](https://s2.loli.net/2022/05/07/9S8NQ6CAmyPT5E2.png)

![2](https://s2.loli.net/2022/05/07/vwc47Z1MJEp2Ie6.png)

![3](https://s2.loli.net/2022/05/07/ZW2xLEnpSNAGMm9.png)

![4](https://s2.loli.net/2022/05/08/6qiZ4ogvce9TXbl.png)

安装完成后关闭窗口，打开 `cmd` 后分别输入 `gcc -v` 和 `g++ -v` 后，出现类似以下输出

![cmd 输出图片](https://s2.loli.net/2022/05/08/j2wyL5xuQgfK6k9.png)

则代表安装成功，否则请重启电脑重试。

## 二、Visual Studio Code 的安装与配置

安装好了编译器，下一步就是安装代码编辑工具了。`Dev-C++` 已经被淘汰，如今知名的 C/C++ IDE 有微软家的 `Visual Studio`、JetBrains 家的 `CLion` 、开源的 `CodeBlocks` 等等。但是这些 IDE 大多都是设计用来工程开发的，不适合竞赛代码编写（例如这三款 IDE 不能对单文件使用内置调试器），且空间占用都不小。于是我们便转向使用更加轻量化的代码编辑器，如 `Visual Studio Code`（不是 `Visual Studio`！简称为 `VSCode`，以下皆用此名）、`Sublime Text`、`(Neo)Vim`、`Emacs`、`Atom` 等。这些编辑器的共同优势便是轻量化、可扩展性强、跨平台（三款 PC 平台主流操作系统均支持）。`(Neo)Vim` 和 `Emacs` 虽打开速度快、使用效率远高于其他编辑器（因为几乎是纯键盘操作），但是配置复杂，需要会使用命令行和内置命令，对于 OIer 不够友好。 Stack Overflow 上著名的问题之一："How to exit the Vim editor?" 它已经有接近两百万的访问量。这就说明了这一点。`Atom` 尽管编辑界面非常美观，但其问题则是反应迟缓和内存占用过高。`Sublime Text` 则非免费开源，尽管速度快，但其插件质量普遍低下。于是我们选择了 `VSCode`，其运行时内存占用日常仅有不到 500MB，我正在用 `VSCode` 写这篇文章时运行内存占用仅有不到 200MB，如图。

![VSCode 运行时内存占用](https://s2.loli.net/2022/05/08/IGiPdkCcHtuQY2V.png)

以下是 [PYPL](https://pypl.github.io/IDE.html) 的 IDE Usage 统计，榜一和榜二的均为大型 IDE，可见 `VSCode` 的强悍！

![IDE 对比](https://s2.loli.net/2022/05/08/4vkZDCYldyMBtuI.png)

### 1. 安装

废话少说，我们打开 `VSCode` 的官网，[https://code.visualstudio.com/](https://code.visualstudio.com/)，点击中间大大的 *Download for Windows* 进行下载。

![VSCode下载](https://s2.loli.net/2022/05/08/yOMhC9LnNc3EtGa.png)

打开安装包，一路下一步，但是建议在这里勾选除“创建桌面快捷方式”（可选）以外其余所有选项。

![](https://s2.loli.net/2022/05/08/acuwp8NbKUgeBZj.png)

安装完成后就可以打开 `VSCode` 了。

### 2. 配置

#### A. 设置中文界面

刚打开的 `VSCode` 显示界面为英文，可以安装中文语言包来切换成中文。

![](https://s2.loli.net/2022/05/08/4RsxHYkhXLIeTSG.png)

安装并重启后，即可看到中文界面。

#### B. 创建竞赛代码文件夹

这时先别急着探索 `VSCode`。先创建一个专门用来写竞赛代码的空文件夹（例如 `E:\CPCode_test'`，建议路径中不要含有中文或特殊字符），并使用 `VSCode` 打开该文件夹。如图：

![打开文件夹](https://s2.loli.net/2022/05/08/AZivFMQdTLqG49b.png)

这时有同学要问了，为什么要单独创建一个文件夹呢？一是为了方便代码的管理和存放，二是因为 `VSCode` 的配置大多是基于文件夹（workspace）的，创建单独的竞赛代码文件夹可以方便地管理竞赛代码的 `VSCode` 配置，也可以方便多环境的切换（比如说换用另外一台电脑进行编写代码，仅需将原电脑竞赛代码文件夹配置复制到新电脑中，即可进行无缝切换）做到 `VSCode` 的真正定制化和便于使用。

#### C. 安装插件

插件系统是 `VSCode` 的核心之一。点击侧边栏的第五个插件图标就可以搜索并安装插件了。

这里建议安装的几个插件分别是：

 * C/C++：**必须安装**，用于支持 C++ 语言（代码高亮提示、函数补全、调试等），**注意不是 Extensions Pack**，Pack 里面的大多数插件在竞赛代码编辑时无用且占用内存、拖慢运行速度。
 * Error Lens：可选，方便于代码编辑时查看语法错误（避免出现 CE）
 * Code Runner：一键编译运行 C++ 程序
 * Competitive Programming Helper(cph)：用于便捷在本地进行样例测试并对代码进行管理，**强烈推荐，后面会讲解如何配置**
 * TabOut：觉得填写完函数参数或数组下标后要按右箭头退出括号麻烦？我也一样！这个插件使得我们仅需按 `Tab` 键即可退出当前括号，非常方便！

除此之外，你还可以额外安装一个主题来使 `VSCode` 更加美观，例如 One Dark Pro 或 GitHub Theme：

![One Dark Pro](https://s2.loli.net/2022/05/08/9mscNDCrgLFJAo4.png)

![GitHub Dark Default](https://s2.loli.net/2022/05/08/DCtpQsA9WjB6qfm.png)

这里还给大家推荐一个文件图标主题：Material Icon Theme，设计感十足。

![Material Icon Theme](https://s2.loli.net/2022/05/08/VH5Ukuo79q2ngNQ.png)

#### D. 配置文件与调试方法

安装好插件后，你一定兴高采烈地跑去新建文件码好了一个 `Hello World`，然而有的同学却发现出现如图所示报错：

![报错](https://s2.loli.net/2022/05/08/S6BKQ5ZPVkufl8q.png)

如果出现此种报错，可以在竞赛代码文件夹下新建 `.vscode` 文件夹（点不能丢！），再在 `.vscode` 文件夹里新建 `c_cpp_properties.json`，填写以下内容：

```json
{
    "configurations": [
        {
            "name": "windows-gcc-x64",
            "includePath": [
                "${workspaceFolder}/**"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"
            ],
            // "windowsSdkVersion": "10.0.19041.0",
            "compilerPath": "D:/TDM-GCC-64/bin/g++.exe", // 此处双引号内的 D:/TDM-GCC-64 改为刚才安装编译器的路径，后面的 /bin/g++.exe 不动
            "cStandard": "c11",
            "cppStandard": "c++14",
            "intelliSenseMode": "${default}",
            "compilerArgs": []
        }
    ],
    "version": 4
}
```

若此文件报错，则一定是 `compilerPath` 字段填写路径错误，再检查一下吧（请用正斜杠而非反斜杠）

填写完成后回到刚才编写的 `.cpp` 文件内，发现不报错了，且代码提示也可用了。

编写完成后，你一定想要尝试运行一下这个程序。安装过 `Code Runner` 后可以右击代码编辑窗口，点击 `Run Code`，等待后即可看到运行结果。

但是，如果你写的测试程序是 `A+B Problem` 则 `Code Runner` 的运行就不管用了，需要配置 `VSCode` 的内置运行。

点击侧边栏图标第四个，即运行与调试，再点击中间的 `运行和调试` 框，选择 `GDB/LLDB`，如图：

![配置调试1](https://s2.loli.net/2022/05/08/ZvdcbNRAMaIB2pS.png)

再点击 `g++.exe 生成活动文件` 便可编译运行（在弹出的终端窗口中点击右侧的 `终端` 一栏即可输入并查看输出）

每次需要运行代码时都要点击这么多次，觉得麻烦？没关系，在 `.vscode` 文件夹下创建 `launch.json`，填入以下内容：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) 启动",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "gdb",
            "miDebuggerPath": "D:/TDM-GCC-64/bin/gdb.exe",   // 将 D:/TDM-GCC-64 改为你的安装路径 
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                },
                {
                    "description":  "将反汇编风格设置为 Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```

此时回到你的 `.cpp` 文件，按 `Ctrl+F5` 即可一键运行，按 `F5` 即可一键调试。

#### E. 设置内微调

这是笔者本人使用得出的一些方便编写竞赛代码的小技巧

点击左下角的设置图标，点击设置，将 `Font Size` 改为 13。

Fira Code 字体是一款专门为代码编写而生的字体，非常好看且易于辨识，可以去 [https://github.com/tonsky/FiraCode](https://github.com/tonsky/FiraCode) 下载 Fira Code 字体，解压后右键安装。安装完重启 `VSCode`。

进入设置，将 `Font Family` 改为 `Fira Code`。

![字体设置](https://s2.loli.net/2022/05/08/T7hUJOqWtnAxSXo.png)

在设置中搜索 `brackets`，点击左侧的 `C/C++`，将右侧的 `Autocomplete Add Parentheses` 打上钩，这样可以在出现函数自动补全时加上括号，非常便捷。

![补全括号](https://s2.loli.net/2022/05/08/oVYl1v7ZUQJwfKz.png)

#### F. 总结

至此，你的 `VSCode` 已经配置完成，可以进行一些简单的代码编辑了。

## 三、Competitive Companion 浏览器插件的安装及使用

`Competitive Companion` 是一个算法竞赛刷题辅助插件，其作用是从刷题网站上爬取样例数据并通过发送请求将数据传送给代码编辑器，大大有益于批量测试样例数据，提升刷题效率。

打开你的浏览器（以 `Chrome` 演示），打开 [https://chrome.google.com/webstore/detail/competitive-companion/cjnmckjndlpiamhfimnnjmnckgghkjbl](https://chrome.google.com/webstore/detail/competitive-companion/cjnmckjndlpiamhfimnnjmnckgghkjbl) 这个链接并点击安装到 `Chrome` 来安装 `Competitive Companion` 插件。

如果无法打开该链接（不会魔法），在这里 [https://www.crx4.com/5705.html](https://www.crx4.com/5705.html) 下载 `.crx` 文件后离线安装。（离线安装 `Chrome` 插件可参考 [https://huajiakeji.com/utilities/2019-01/1791.html](https://huajiakeji.com/utilities/2019-01/1791.html)

安装完成后，将插件图标固定，如图所示：

![固定图标](https://s2.loli.net/2022/05/08/mAHW6xzGQ9INkUK.png)

再次打开 `VSCode`，并在 `VSCode` 中打开你的竞赛代码文件夹。打开 `VSCode` 的设置，并点击设置右上角的文件图标，打开配置文件，如图：

![打开配置文件的图标](https://s2.loli.net/2022/05/08/OHcC7KDVe3nYfdS.png)

在配置文件中的大括号内加入如下几行：

```json
    "cph.general.defaultLanguage": "cpp",
    "cph.general.defaultLanguageTemplateFileLocation": "E:\\Code\\template.cpp",    // 这是你的模板文件路径
    "cph.general.firstTime": false,
    "cph.language.cpp.SubmissionCompiler": "GNU G++14 6.4.0",   // 使用 C++14 标准
    "cph.language.cpp.Args": "-std=c++14 -O2",    // -O2
```

保存后关闭配置文件。打开 OJ 的一道题后点击 `Competitive Companion` 图标即可开始刷题。如图所示：

![GIF 动图](https://s2.loli.net/2022/05/08/SsgP4UZF7kMOydI.gif)

## 四、总结

至此，所有的 OI 环境配置完全结束。这篇文章仅仅介绍了环境配置的九牛一毛，如果需要频繁更换电脑刷题，还可以使用 Git 发布到代码仓库进行同步和维护配置文件。OI 之路道阻且长，选用一款好的工具至关重要。希望看到这里的你能够在 OI 中如虎添翼，成绩更上一层楼。