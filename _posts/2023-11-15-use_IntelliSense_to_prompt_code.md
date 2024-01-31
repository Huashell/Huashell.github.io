---
layout: post
title: "内核开发时使用IntelliSense进行代码提示"
date:   2023-11-20
tags: [geek]
comments: true
author: ddd
toc: true
---


<!-- more -->



​	IntelliSense是各种代码编辑功能的通用术语，包括：代码完成、参数信息、快速信息和成员列表等。

​	这是vscode关于IntelliSense介绍的官方文档<http://code.visualstudio.com/docs/editor/intellisense>

​	不过这篇文档大多是在介绍IntelliSense的使用方法，并没有过多地介绍IntelliSense如何进行配置。大多数时候，IntelliSense都能很好地工作。但当我试图调用linux内核库中的一个<sched.h>时，IntelliSense指向了/usr/include/linux/sched.h 。这个库文件只有一些用于用户使用进程调度相关的系统调用时所用到的宏，而我需要的是完整的系统库，定义了task_struct的内核用调度库。虽然正常使用时，我会告诉编译器如何找到这个库文件的所在地，但是我仍然十分需要IntelliSense带来的优越的代码提示和成员列表，以免我每次都得通过在内核源码仔细查找才敢调用函数并填入参数。



```json
{
  "configurations": [
    {
      "name": "linux-gcc-x64",
      "includePath": [
        "${workspaceFolder}/**"
      ],
      "compilerPath": "/usr/bin/gcc",
      "cStandard": "${default}",
      "cppStandard": "${default}",
      "intelliSenseMode": "linux-gcc-x64",
      "compilerArgs": [
        ""
      ]
    }
  ],
  "version": 4
}
```

​	这是vscode为我默认生成的c_cpp_properties.json,里面指定了intelliSenseMode为linux-gcc-x64，这也是intellisense为linux平台设置的默认mode。我在includePath选项中填入了"${workspaceFolder}/**"，根据网上的资料得知，intellisense搜寻库路径的顺序为

```text
查找项目内的头文件：首先，IntelliSense 会查找项目内所有包含路径下的头文件，包括在 C/C++ 配置文件 c_cpp_properties.json 中配置的路径，以及通过 #include 包含的头文件。
查找系统头文件：如果在项目内找不到所需的头文件，IntelliSense 会继续查找系统的标准头文件路径，通常是 /usr/include 或 /usr/local/include。
查找其他路径：如果在项目内和系统路径中都找不到所需的头文件，IntelliSense 可能会查找其他路径，例如通过 -I 选项指定的路径
```

​	理论上来说，如果我在include中指定内核sched.c的路径，它应该是能够找到的，即

```json
"includePath": [
        "${workspaceFolder}/**",
        "/usr/src/linux-headers-5.15.0-78-generic/include/**",
      ]
```

​	其中5.15.0-78是我的内核版本号，而我所需的sched.c正是在该目录下的<linux/sched.c>。然而遗憾的是，intellisense仍然为我指向了/usr/include/linux/sched.c。/usr/include是gcc的内置扫描路径，在intellisense中，这个目录的优先级似乎高于我在c_cpp_properties.json中指定的includePath。而我也无法改变gcc的内置路径。所以我干脆注解掉了 "compilerPath": "/usr/bin/gcc"。让人惊讶的是，这次intellisense指向了我想要的头文件。