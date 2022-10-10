---
title: 'std::sort排序导致内存错误'
tags:
  - OI
  - C++
cover: /img/archives/sortmemoryerror.jpg
abbrlink: 39408
date: 2022-10-09 00:00:00
---
实例代码如下
```C++
struct Hero {
    std::string Name;
    int Moe;
    int Strength;
    int Mind;
};

bool compareMoe(const Hero &hero1, const Hero &hero2) {
    if (hero1.Moe == hero2.Moe) {
        return !hero1.Name.compare(hero2.Name);
    }
    return hero1.Moe > hero2.Moe;
}

int main() {
    // 一片输入
    std::sort(heros.begin(), heros.end(), compareMoe);
    // 一堆输出
}
```

看起来似乎没什么问题，但是运行起来会core dump，原因如下
参照[CPP Reference](https://cplusplus.com/reference/algorithm/sort/)
一个sort函数需要满足以下性质
+ 反自反性: 也即compare(x, x)必须是false
+ 非对称性: 也即如果compare(x, y)和compare(y, x)的结果必然相反
+ 可传递性: 也即如果compare(x, y)为true，compare(y, z)为true，那么compare(x, z)必然为true
上文代码不符合第一条，因而会把内存写坏