---
layout: post
title: Linux系统优化与调试之五 - Cache Miss的性能影响
categories: 系统优化
tags: [系统优化]
---

> 现代计算机体系结构，存储有很多层次，各个层次硬件访问延迟存在数量级上的差异，越高的性能，往往意味着更高的成本和更小的容量。因为不同层级之间的性能差距一般是指数级别, 如何有效利用高性能的存储介质，对应用程序的性能影响很大。

![cpu_cache.jpg](http://ygjs-static-hz.oss-cn-beijing.aliyuncs.com/images/2018-2-8/cpu_cache.jpg)

------

下面两段程序的时间复杂度完全一致, 但是在我的执行环境上，前者耗时0.864s, 而后者耗时仅为0.111s。

这是因为前者循环中a, b内存不连续，所以每次都要分别从内存中加载a，b到cache, 导致cache miss; 相反的，后者a, b每次都会同时被加载到cache, 所以性能高很多。

用`perf stat -e cache-misses ./a.out`可以看出，前者cache-miss数量117,056， 而后者仅有26,274。

```c
#define NUM 393216

int main(){
    float a[NUM],b[NUM];
    int i;
    for(i=0;i<1000;i++)
        add(a,b,NUM);
}

int add(int *a,int *b,int num){
    int i=0;
    for(i=0;i<num;i++){
        *a=*a+*b;
        a++;
        b++;
    }
}
```

```c
#define NUM 39216
typedef struct{
    float a;
    float b;
}Array;

int main(){
    Array myarray[NUM];
    int j=0;
    for(j=0;j<1000;j++)
        add(myarray,NUM);
}

int add(Array *myarray,int num){
    int i=0;
    for(i=0;i<num;i++){
        myarray->a=myarray->a+myarray->b;
        myarray++;
    }
}

```
