---
title: "VScode代码风格设置"
tags: ["C/C++" ]
date: "2018-09-05"
---
这一段时间用VScode，各方面都挺好用的。但是，有一点让我难受，格式化代码时，花括号的自动换行，没法忍啊！  
<!--more-->
em... 个人习惯花括号一般与function or class 名字同行，类似这个：
```cpp
void function(){
    ...
}
```

但是，vscode默认的formater，会将代码格式化成下面这样：
```cpp
void function()
{
    ...   
}
```

哇！ 看着就难受啊！！！ 

#### 解决方法
内置的formater是解决不了这个问题的， 首先需要安装一个插件： Clang-Format  
然后在setting文件中加入如下设置:
```cpp
 "C_Cpp.clang_format_fallbackStyle": "{BasedOnStyle: Google, IndentWidth: 4, ColumnLimit: 0 , AlignConsecutiveAssignments: true}",
```
讲解一下： BasedOnStyle原本是"Visual Studio", 改成 "Google“ 括号问题就解决了； ColumnLimit的限制也是有用的，google style可能会将代码拆到下一行。  
最后加了一个 "AlignConsecutiveAssignments: true",可以让多行的等式，以等号对齐：
```cpp
int aaaa = 12;
int b    = 23;
int ccc  = 23;
```

更多的配置参考 https://clang.llvm.org/docs/ClangFormatStyleOptions.html ，打造适合自己的风格配置~
