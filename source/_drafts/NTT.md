---
title: Fast Number-Theoretic Transformation
date: 2017-02-26
tags: Mathematics
---

### Preface
&emsp;&emsp;在前面的博客[Fast Fourier Transformation](https://wearrys.github.io/2017/02/17/FFT/)中，已经简单介绍了FFT的原理及其实现，但是FFT并不是完美的，由于涉及到复数的运算，FFT在实际使用过程中存在比较大的精度误差。如果我们对整数进行运算要求取模时，就可以用FNT来进行处理。

### Preparatory knowledge

#### 原根
&emsp;&emsp;在FFT的实现中，进行蝴蝶操作的关键就是利用了单位复数根的性质$w_n^{\frac{n}{2}} = -1$以及每个单位复根的n次方为1等，那么在模意义下的乘法中，是否有类似的东西呢？
答案是肯定的：[原根](https://zh.wikipedia.org/wiki/%E5%8E%9F%E6%A0%B9)
&emsp;&emsp;利用原根的相关性质，我们发现可以像FFT一样的进行蝴蝶操作了，关于蝴蝶操作的具体过程请参见有关FFT的讲稿。

<!--more-->
### Algorithm
&emsp;&emsp;说了这么多，其实大家会发现FNT的基本流程和FFT几乎是完全一样的，只是两者的适用范围有些许的差异而已。当然，如果模数不是那么的贴心比较坑的话，还需要更多的小技巧来处理。
&emsp;&emsp;然而我不会。。。

### Code
&emsp;&emsp;同样的FNT也有递归和迭代的两种实现方式，由于在FFT中都介绍过，因此这里只给出更优的迭代版本。

``` cpp
void init() {
    N = 1;
    while(N <= 2*k) N <<= 1, ++bit;

    for(int i = 0; i < N; ++i) 
        rev[i] = ((rev[i>>1]>>1) | ((i&1) << (bit-1)));

    w[0] = w[N] = 1;
    int wn = Pow(3, (mod-1)/N);
    for(int i = 1; i < N; ++i) w[i] = 1LL*w[i-1]*wn%mod;
}

void DFT(int *a, int N, int type) {
    for(int i = 0; i < N; i++) 
        if(i < rev[i]) swap(a[i], a[rev[i]]);

    for(int l = 2; l <= N; l <<= 1) {
        for(int i = 0; i < N; i += l) {
            for(int j = 0; j < (l>>1); j++) {
                int u = a[i+j]; 
                int v = 1LL * a[i+j+(l>>1)]*(type>0 ? w[N/l*j]:w[N-N/l*j]) % mod;

                a[i + j] = (u + v) % mod;
                a[i + j + (l>>1)] = (u - v + mod) % mod;
            }
        }
    }
    if(!~type) {
        int invn = Pow(N, mod-2);
        for(int i = 0; i < N; i++) a[i] = 1LL*invn*a[i] % mod;
    }
}
```
