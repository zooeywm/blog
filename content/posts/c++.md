+++
title = 'C++'
date = 2024-08-15T15:03:19+08:00
lastUpdate = 2024-08-15T15:03:19+08:00
summary = "C++ 学习"
tags = ["C++"]
+++

# 零碎知识

## #define

#define 的作用：用于定义宏，创建符号常量、简化代码的重复部分，或进行简单的文本替换，例如：
```C++
#define PI 3.1415926
#define MAX(x,y) ((x) > (y) ? (x) : (y))
```

此外， C++ 还可以使用 const inline 模板等来代替 #define
例如：
```C++
const double PI = 3.1415926
template<typename T>
inline T max(T x, T y) {
    return (x > y) ? x : y;
}
```
尽管如此，#define 仍然在某些场景中有其应用，比如条件编译（#ifdef、#ifndef）等

