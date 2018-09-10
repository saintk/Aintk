---
title: "OpenMP并行计算"
tags: ["C/C++" ]
date: "2018-07-21"
---

最近在写工程的时候，涉及到了一些cpu并行计算的部分，简单的了解一下如何使用openmp进行cpu并行计算.

<!--more-->

## 一、什么是OpenMP？
OpenMP 即 Open MultiProcessing. 它并不是一个简单的函数库，而是一个诸多编译器支持的框架，在编译时加入 -fopenmp 即可直接使用。 
OpenMP提供了对并行算法的高层的抽象描述，通过在code中加入 pargma 来表明自己的意图，告知编译器将这部分程序进行并行化，并在必要之处加入同步互斥和通信。  
但这也限制了它不适合复杂的线程间同步和互斥的场景。  

## 二、使用
基本用法： 
```cpp
#include <iostream>
#include <omp.h>

int main()
{
#pragma omp parallel for
    for (char i = 'a'; i <= 'z'; i++)
        std::cout << i << std::endl;

    return 0;
}
```
使用openmp对for循环的任务进行切分，若电脑有4核，则for循环会并行的切分成4部分，分别计算。  

就一句话，是不是很简单？  
但里面也会遇到一些问题，比如各线程的数据依赖、对公共数据进行修改等等，都有可能出现问题。 这些问题在设计时要认真考量。  

