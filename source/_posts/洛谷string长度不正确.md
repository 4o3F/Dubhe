---
title: 洛谷string长度不正确
tags:
  - OI
  - luogu
  - memorandum
cover: /img/archives/luogu_string_length_problem.jpg
abbrlink: 24181
date: 2022-10-13 00:00:00
---
我这废物今天做**P1320**时，提交上去全部WA,放到洛谷的IDE里跑了下后发现长度比预期要多了1
输出ASCII码，发现最后一位多了一位`13`，正常情况下，`getline`会读取到`\n`停止，但是此时结束符号变为了`\r\n`因此导致错误
解决代码很简单如下
```c++
s.erase(std::remove(s.begin(), s.end(), '\r' ), s.end());
s.erase(std::remove(s.begin(), s.end(), '\n' ), s.end());
```
当然也可以直接丢弃最后一个字符