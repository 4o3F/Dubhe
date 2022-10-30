---
title: WSL和JetBrain IDE的一些坑
tags:
  - WSL
  - JetBrains
  - IDE
cover: /img/archives/wsl.jpg
abbrlink: 30331
date: 2022-10-30 00:00:00
---

### CLion配合WSL中的OpenCV时会找不到头文件

在`CMakeLists.txt`中的`add_executable(xxx)`后面加上

```C++
find_package( OpenCV REQUIRED )
include_directories(${OpenCV_INCLUDE_DIRS})
target_link_libraries( xxx ${OpenCV_LIBS} )
```
即可