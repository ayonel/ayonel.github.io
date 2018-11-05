---
title: 寻找变位词
url: 243.html
id: 243
tags:
  - 算法与数据结构
date: 2016-12-27 23:23:16
---

##### 题目描述：
> 如果两个字符串中的字符一样，出现次数也一样，只是出现的顺序的顺序不一样，则认为这两个字符串是兄弟字符串，或称变位词。例如"bad"和"adb"即为兄弟字符串。现提供一个字符串，请问如何在字典中迅速找到它的兄弟字符串。例如待查找的字符串是"apple",字典是{"appl","paal","aplep","bb","leapp","hello"}，则输出的单词应为："aplep", "leapp"。

##### 题目来源：
《编程之法》、《编程珠玑》
原理比较简单，但是要完整地实现起来，需要很多基础的小算法，如排序，搜索等。应该，用这个题顺便来练习一下基本算法，还是挺有意义的。
##### 解题思路：
首先用快排对整个字典排序，排序的比较标准是每个单词内部先排好序之后的签名，比如"apple"--->"aelpp","dddap"--->"adddp";再用排序之后的字典二分搜索目标单词，找到签名一样的单词，则分别向上，向下找到同样签名的单词，并输出；
##### 代码： 
```C
/**
 * 如果两个字符串中的字符一样，出现次数也一样，只是出现的顺序的顺序不一样，则认为这两个字符串是兄弟字符串。
 * 例如"bad"和"adb"即为兄弟字符串。现提供一个字符串，请问如何在字典中迅速找到它的兄弟字符串
 *
 * 例如待查找的字符串是"apple",字典是{"appl","paal","aplep","bb","leapp","hello"}
 * 则输出的单词应为："aplep", "leapp"
 *
 * 首先用快排对整个字典排序，排序的比较标准是每个单词内部先排好序之后的签名，比如"apple"--->"aelpp","dddap"--->"adddp";
 * 再用排序之后的字典二分搜索目标单词，找到签名一样的单词，则分别向上，向下找同样签名的单词，并输出；
 **/
#include <stdio.h>
#include <string.h>
#define MAX_LEN 20 // 假定每个单词的最大字符数是20
#define WORD_COUNT 6
char set[][WORD_COUNT] = {"appl","paal","aplep","bb","leapp","hello"};


// 通过对单词插入排序，获取签名，不要直接传入原数组中的单词，应该传入单词的副本，反正覆盖原数组
char* sign(char *result) {
    int j = 0;
    int i = 1;
    char tmp = ' ';
    //插入排序
    while (result[i] != '\0') {
        tmp = result[i];
        for(j = i; j > 0 && result[j-1] > tmp; j--) {
            result[j] = result[j-1];
        }
        result[j] = tmp;
        i++;
    }
    return result;
}

//比较两个单词的签名大小
int compare_by_sign(char *a, char *b) {
    char sort_a[MAX_LEN];
    char sort_b[MAX_LEN];
    strcpy(sort_a, a);
    strcpy(sort_b, b);

    return strcmp(sign(sort_a), sign(sort_b));
}

//比较两个单词大小
int compare(char *a, char *b) {
    return strcmp(a, b);
}

//交换原数组中下标为i和j的单词位置
void swap(int i, int j){
    if (i == j)
        return;
    char tmp[MAX_LEN];
    strcpy(tmp, set[i]);
    strcpy(set[i], set[j]);
    strcpy(set[j], tmp);
}

// 对原数组按照签名大小，进行快排
void qsort(int l, int u) {
    if (l > u)
        return;
    int m = l;

    for(int i = l+1; i <= u; i++) {
        if(compare_by_sign(set[i], set[l]) < 0){
            swap(i,++m);
        }
    }
    swap(m,l);

    qsort(l, m-1);
    qsort(m+1, u);
}

//在排序好的数组中二分查找签名相同的单词，并输出
void bsearch(int l, int u, char *word_sign, int *is_found) {
    int mid = (u - l) / 2;
    if (l > u) {
        return;
    }
    char mid_sign[MAX_LEN];
    strcpy(mid_sign, set[mid]);
    sign(mid_sign);
    int com_result = compare(mid_sign, word_sign);
    if (com_result > 0)
        bsearch(l, mid-1, word_sign, is_found);
    if (com_result < 0)
        bsearch(mid+1, u, word_sign, is_found);
    if (com_result == 0){
        //找到了，开始向下以及向上寻找相同签名的词
        int flag = mid;
        printf("found brother word:\n%s\n",set[mid]);
        *is_found = 1;
        mid--;
        flag++;
        //向上找
        while(mid > 0){
            if(compare(sign(set[mid]), word_sign) == 0)
                printf("%s\n",set[mid]);
            else
                break;
            mid--;
        }
        //向下找
        while(flag < WORD_COUNT) {
            if(compare(sign(set[flag]), word_sign) == 0)
                printf("%s\n",set[flag]);
            else
                break;
            flag++;
        }
    }
}

int main() {
    qsort(0, WORD_COUNT-1); //对原数组排序
    char des[] = "apple"; //待查找单词
    char *word_sign;
    word_sign = sign(des); //待查找单词的签名
    printf("sign is: %s\n", word_sign);

    int is_found = 0; //标记是否找到
    bsearch(0, WORD_COUNT, sign(des), &is_found); //二分查找
    if (is_found == 0) { //未找到
        printf("%s\n", "can not found brother word!!");
    }
}
```
输出为：
```bash
sign is: aelpp
found brother word:
leapp
aelpp
```

