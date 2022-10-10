---
title: 在cin后cin.get不生效问题解析
tags:
  - C++
  - ACM
cover: /img/archives/cinget.jpg
abbrlink: 1222
date: 2022-09-09 21:57:30
---
> 来源: [https://stackoverflow.com/questions/45201034/why-does-the-program-skip-cin-get-after-a-cin](https://stackoverflow.com/questions/45201034/why-does-the-program-skip-cin-get-after-a-cin)

在
`cin >> n;`  
后，输入缓存区中其实还有一个`\n`因此在下一行  
`cin.get(str,n);`  
的时候，第一个读取到的就是`\n`因此会直接结束，解决方法是在`cin`后`cin.get`前，加上一行  
```C++
std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
```