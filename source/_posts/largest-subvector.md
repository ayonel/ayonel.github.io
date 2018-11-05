---
title: 计算最大连续子向量
url: 237.html
id: 237
tags:
  算法与数据结构
date: 2016-12-21 11:40:03
---

##### 问题描述：
输入具有n个浮点数的向量x，输出向量x中任何连续子向量中的最大和。例如：
如果x={1,2,3}，则output=6
如果x={1,-2,3}，则output=3
如果x={31,-41,59,26,-53,58,97,-93,-23,84},则output=187 
当所有输入为负数时，最大连续子向量为空向量，输出为0 
##### 问题来源：
《编程珠玑 第2版》第8章 对于该问题，平方算法书中给出了两种，都比较直观。
 **作者提出了一种O(nlogn)的算法**，主要利用分治和递归。将原向量从中间分开，分为左向量和右向量。递归寻找左向量，右向量，以及以分开点为起点的向左最大向量+向右最大向量。 书中也给出了算法。如下：
```
#include <stdlib.h>
int x[10] = {31,-41,59,26,-53,58,97,-93,-23,84};
float max(float x, float y) {
    return x >= y? x : y;
}

float max3(float x, float y, float z) {
    int larger = max(x, y);
    return max(larger, z);
}
//核心算法
float maxsum3(int l, int u) {
    if (l > u)
        return 0;
    if (l == u)
        return max(0, x[1]);
    int m = (l + u) / 2;
    //找出以m为起点的向左的最大子向量
    float lmax  = 0;
    float sum = 0;
    for (int i = m; i >= 1; i--) {
        sum += x[i];
        lmax = max(lmax, sum);
    }
    //找出以m为起点的向右的最大子向量
    float rmax = 0;
    sum = 0;
    for(int i = m + 1; i < u; i++) {
        sum += x[i];
        rmax = max(rmax, sum);
    }
    //lmax+rmax 就是经过m的最大子向量
    return max3(lmax+rmax, maxsum3(l, m), maxsum3(m+1, u));
}
int main(){
    printf("%f", maxsum3_l(0,9));
}
```

**书中又给出了一个线性扫描算法**，时间复杂度为O(n),程序的关键就是对maxendinghere变量，在每一次for循环中对其赋值之前，其是结束位置为i-1的最大子向量的和。赋值后其实结束位置为i的最大子向量的和；若加上x\[i\]之后结果依然为正，则该赋值语句将maxendinghere增大x\[i\];若加上x\[i\]之后结果为负，则该赋值语句将maxendinghere置为0（因为结束为止为i的最大子向量现在为空向量） 代码如下：

```
#include <stdlib.h>
int x[10] = {31,-41,59,26,-53,58,97,-93,-23,84};
float maxsum4(int n){
    float maxsofar = 0;
    float maxendinghere = 0;
    for (int i = 0; i < n; ++i) {
        maxendinghere = max(maxendinghere+x[i], 0);
        maxsofar = max(maxsofar, maxendinghere);
    }
    return maxsofar;
}

int main(){
    printf("%f", maxsum4(10));

}
```
**可以证明不会存在快于O(n)的算法**
在课后题中作者提出改进原来O(nlogn)的递归算法，使其在最坏情况下的复杂度为O(n),主要核心思想是从m向左以及向右找最大子向量时，要记下最大子向量的结束位置，下一轮递归寻找时，直接从上一次记下的结束位置开始，分别向左和向右找。代码如下：

```
include <stdlib.h>
int x[10] = {31,-41,59,26,-53,58,97,-93,-23,84};
float max(float x, float y) {
    return x >= y? x : y;
}

float max3(float x, float y, float z) {
    int larger = max(x, y);
    return max(larger, z);
}
float maxsum3_l(int l, int u) {
    if (l > u)
        return 0;
    if (l == u)
        return max(0, x[l]);
    int m = (l + u) / 2;

    float lmax  = 0;
    float sum = 0;
    int last_l = m;  // 向左找时最大子向量的结束位置
    for (int i = m; i >= 0; i--) {
        sum += x[i];
        if (lmax<=sum) {
            lmax = sum;
            last_l--;
        }
    }

    int last_r = m+1; //向右找时最大子向量的结束位置
    float rmax = 0;
    sum = 0;
    for(int i = m + 1; i <= u; i++) {
        sum += x[i];
        if (rmax<sum) {
            rmax = sum;
            last_r++;
        }
    }
    return max3(lmax+rmax, maxsum3_l(l, last_l-1), maxsum3_l(last_r, u)); //直接从结束位置向左及向右找
}
int main(){
    printf("%f", maxsum3_l(0,9));
}
```
**上述程序的输出应该均为187.000000**