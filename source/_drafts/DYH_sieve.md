---
title: 杜教筛小结
date: 2017-05-23 20:01:43
tags:
    - Number Theory
---

### 引入

&emsp;在一系列数论函数问题中，常常要快速地计算一些积性函数比如$\mu$,$\varphi$等函数的前缀和。 用线性筛来做自然不必说，那有没有低于线性的做法呢？
&emsp;答案是肯定的。

<!--more-->

&emsp;一般来说，对于积性函数$f$, 当 $\sum _ {d|n} f(d) $ 能够快速计算时， 用杜教筛就能够快速地处理出函数$f$的前缀和。

### 举个栗子

&emsp;对于$\mu$, $\sum _ {d|n} \mu(d) = [n=1]$ ：
    $$ 
    \mu(n) = [n=1] - \sum_{d|n, d \neq n} \mu(d) \\
    \\
    \begin{align}
        \sum_{i=1}^{n} \mu(i) 
        &= 1 + \sum_{i=2}^{n} \mu(i) \\
        &= 1 - \sum_{i=2}^{n} \sum_{d|i, d \neq i} \mu(d) \\
        &= 1 - \sum_{k=2}^{n} \sum_{d=1}^{\lfloor \frac{n}{k} \rfloor} \mu(d) 
    \end{align}
    $$
P.S. 最后一步中的$k$，枚举$ \frac{i}{d} $的值。

&emsp;那么如果记 $S _ n = \sum _ {i=1}^{n} \mu(i) $, 则$S _ n = 1 - \sum _ {k=2}^{n} S _ {\lfloor \frac{n}{k} \rfloor}$，原式转化为可递归的形式。
&emsp;类似的, $\varphi$也可以这么算，如果你问我复杂度怎么证，我只能说无可奉告。

### 看一道题

&emsp;求：
    $$ \sum_{i=1}^{n} \sum_{j=1}^{n} lcm(i, j) $$

&emsp;首先按照惯用套路化简：
    $$
    \begin{align}
        \sum_{i=1}^{n} \sum_{j=1}^{n} lcm(i, j) 
        &= \sum_{i=1}^{n} \sum_{j=1}^{n} \frac{ij}{gcd(i, j)} \\
        &= \sum_{d=1}^{n} d \sum_{i=1}^{\lfloor \frac{n}{d} \rfloor} i \sum_{j=1}^{\lfloor \frac{n}{d} \rfloor} j[gcd(i, j) = 1] \\
        &= \sum_{d=1}^{n} d (1 + 2\sum_{i=2}^{\lfloor \frac{n}{d} \rfloor} i \times \frac{i\varphi(i)}{2}) \\
        &= \sum_{d=1}^{n} d \sum_{i=1}^{\lfloor \frac{n}{d} \rfloor} i^2 \varphi(i) 
    \end{align}
    $$
P.S.
&emsp;&emsp;倒数第二步中强制使 $i > j$, 然后特判 $i = j = 1$ 。
&emsp;&emsp;同时其中用到了一个重要的结论: $\sum _ {i=1}^{n} i[gcd(i, n) = 1] = \frac {\varphi(n) \times n + [n=1]}{2}$ 

&emsp;然后我们需要快速求$S _ n = \sum _ {i=1}^{n} \varphi(i) i^2$

&emsp;类似$\mu$:
    $$ 
    \sum_{d|n} \varphi(d) = n
    \\
    \begin{align}
        S_n  
        &= \sum_{i=1}^{n} (i - \sum_{d|i, d \neq i} \varphi(d)) i^2 \\
        &= \sum_{i=1}^{n} i^3 - \sum_{i=1}^{n} i^2 \sum_{d|n, d \neq i} \varphi(d) \\
        &= \sum_{i=1}^{n} i^3 - \sum_{k=2}^{n} \sum_{d=1}^{\lfloor \frac{n}{k} \rfloor} \varphi(d) d^2 k^2 \\
        &= \sum_{i=1}^{n} i^3 - \sum_{k=2}^{n} k^2 \sum_{d=1}^{\lfloor \frac{n}{k} \rfloor} \varphi(d) d^2 \\
        &= \sum_{i=1}^{n} i^3 - \sum_{k=2}^{n} k^2 S_{\lfloor \frac{n}{k} \rfloor}
    \end{align}
    $$

&emsp;至此已经可以用杜教筛来处理了，时间复杂度坑待填。。。

### Code

```cpp
#include<bits/stdc++.h>
using namespace std;

typedef long long LL;
typedef pair<int, int> pii;

const LL mod = 1e9 + 7;
const LL maxn = 5e6 + 10;
const int oo = 0x3f3f3f3f;

template <typename T> bool chkmax(T& a, T b) { return a < b ? a = b, 1 : 0; }
template <typename T> bool chkmin(T& a, T b) { return a > b ? a = b, 1 : 0; }

#define fst first
#define snd second
#define debug(x) cerr << #x <<":" << (x) << endl
#define REP(i, a, b) for(int i = (a), i##end = (b); i < i##end; ++i)
#define DREP(i, a, b) for(int i = (a)-1, i##bgn = (b); i >= i##bgn; --i)

template<typename T> T read() {
	T n = 0, f = 1;
	char ch = getchar();
	for( ;!isdigit(ch); ch = getchar()) if(ch == '-') f = -1;
	for( ; isdigit(ch); ch = getchar()) n = n * 10 + ch - 48;
	return n * f;
}

bool isprime[maxn];
LL prime[maxn/10], pcnt, phi[maxn];

void sieve() {
    memset(isprime, 1, sizeof isprime);

    phi[1] = 1;
    for(int i = 2; i < maxn; i++) {
        if(isprime[i]) {
            phi[i] = i-1;
            prime[pcnt++] = i;
        }
        for(int j = 0; j < pcnt && 1LL*i*prime[j] < maxn; j++) {
            isprime[i * prime[j]] = 0;
            if(i % prime[j] == 0) {
                phi[i * prime[j]] = phi[i] * prime[j];
                break;
            }
            phi[i * prime[j]] = phi[i] * phi[prime[j]];
        }
    }
    for(int i = 1; i < maxn; i++) phi[i] = (phi[i-1] + phi[i] * i % mod * i % mod) % mod;
}

LL fpm(LL base, LL exp) {
    LL ans = 1;
    base %= mod;
    for(; exp > 0; exp >>= 1, (base *= base) %= mod) 
        if(exp & 1) 
            (ans *= base) %= mod;
    return ans;
}

const LL inv6 = fpm(6, mod-2), inv2 = fpm(2, mod-2);

LL calc(LL n) { n %= mod; return n * (n+1) % mod * inv2 % mod; }
LL calc2(LL n) { n %= mod; return n * (n+1) % mod * (2*n+1) % mod * inv6 % mod; }
LL calc3(LL n) { return fpm(calc(n), 2); } 

map<LL, LL> mp;
LL f(LL n) {
    if(n < maxn) return phi[n];
    if(mp.count(n)) return mp[n];
    
    LL ans = calc3(n);
    for(LL i = 2, lst; i <= n; i = lst + 1) {
        lst = n / (n/i);
        (ans -= (calc2(lst) - calc2(i-1)) * f(n/i) % mod) %= mod;
    }
    return mp[n] = ans;
}

LL n;
int main() {
#ifndef ONLINE_JUDGE
    freopen("data.txt","r", stdin);
    freopen("ans.txt","w", stdout);
#endif

    sieve();
    n = read<LL>();

    LL ans = 0;
    for(LL i = 1, lst; i <= n; i = lst + 1) {
        lst = n / (n/i);
        (ans += (calc(lst) - calc(i-1)) * f(n/i) % mod) %= mod;
    }
    ans = (ans % mod + mod) % mod;
    printf("%lld\n", ans);

    return 0;
}
```
