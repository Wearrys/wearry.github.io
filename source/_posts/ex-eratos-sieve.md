---
title: 拓展埃氏筛法小结
date: 2018-08-21 19:42:41
tags:
    - Number Theory
---

学完洲阁筛之后忘得差不多了, 不过现在已经可以将洲阁筛扔进垃圾桶了...

<!--more-->

### Description

求:

$$S(N) = \sum_{i=1}^{N} F(i)$$ 

其中 $F$ 满足在 $i$ 是质数幂的情况下比较好算并且是一个积性函数.

### Conversion

首先将所有的 $i$ 按照最大质因子的次数是否大于一分为两类计算答案.

对于第一类的数, 我们将最大质因子除去, 剩下的部分一定只包含不超过 $\sqrt{N}$ 的质因子, 于是可以暴搜所有剩下的部分, 同时这个过程中一定会遍历所有第二类的数. 接下来只需计算 $s(i)$ 表示 $[1, i]$ 中所有质数的 $F$ 的和.

因为 $p \le \sqrt{N}$, 所以有用的状态只有 $O(\sqrt{N})$ 个.
枚举 $[1, \sqrt{N}]$ 中的所有质数 $p$, 考虑每次筛去最小质因子为 $p$ 的数的贡献.
那么对于每个 $p$ , 都会使得一个满足 $i \ge p^2$ 的 $s(i)$ 减去 $s(\lfloor \frac{i}{p} \rfloor) - s(p-1)$. 

这样子的复杂度大约为 $O\left (\frac{N^{\frac{3}{4}}}{\log{N}}\right )$

### Code

以 $\varphi$ 函数为例 (实际效率似乎不输杜教筛?).

```cpp
namespace Sieve {

    ll n;
    ll s0[N + 5], S0[N + 5];
    ll s1[N + 5], S1[N + 5];
    int m, prime[N + 5], pcnt;

    inline int calc(ll res, int lst, ll mul) {
        ll temp = (res > m ? S1[n / res] : s1[res]) - s1[prime[lst]];
        ll ans = temp * mul % mo;

        for(int i = lst + 1; i <= pcnt; ++i) {
            int p = prime[i];
            if((ll) p * p > res) break;
            for(ll j = p, nmul = mul * (p-1) % mo; res >= j * p; j *= p) {
                ans = (ans + calc(res / j, i, nmul)) % mo;
                nmul = nmul * p % mo;
                ans = (ans + nmul) % mo;
            }
        }
        return ans;
    }

    int solve(ll _n) {
        n = _n;
        if(n <= 1) return n;

        pcnt = 0;
        m = (int) sqrt(n + 0.5);

        for(int i = 1; i <= m; ++i) {
            s0[i] = i - 1;
            s1[i] = (ll) (2 + i) * (i - 1) / 2 % mo;

            ll t = (n / i) % mo;
            S0[i] = t - 1;
            S1[i] = (ll) (2 + t) * (t - 1) / 2 % mo;
        }

        for(int i = 2; i <= m; ++i) {
            if(s0[i] == s0[i-1]) continue;
            prime[++ pcnt] = i;

            const ll t0 = s0[i-1], t1 = s1[i-1], p = (ll) i * i;
            const ll lim = std::min((ll) m, n / p), x = m / i, y = n / i;

            for(int j = 1; j <= x; ++j) {
                S0[j] = (S0[j] - (S0[j * i] - t0)) % mo;
                S1[j] = (S1[j] - (S1[j * i] - t1) * i) % mo;
            }
            for(int j = x+1; j <= lim; ++j) {
                S0[j] = (S0[j] - (s0[y / j] - t0)) % mo;
                S1[j] = (S1[j] - (s1[y / j] - t1) * i) % mo;
            }
            for(int j = m; j >= p; --j) {
                s0[j] = (s0[j] - (s0[j / i] - t0)) % mo;
                s1[j] = (s1[j] - (s1[j / i] - t1) * i) % mo;
            }
        }
        for(int i = 1; i <= m; ++i) {
            s1[i] = (s1[i] - s0[i] + mo) % mo;
            S1[i] = (S1[i] - S0[i] + mo) % mo; 
        }
        return (1 + calc(n, 0, 1)) % mo;
    }
}
```
