---
title: Calc
date: 2018-08-16 10:32:35
tags:
    - Number Theory
    - Complexity Analysis
---


### Description

> 给定 $N$, 求有多少组 $a$, $b$ 满足:
$1 \le a < b \le N$
$a + b \mid a \times b$

$N \le 2 ^ {31} - 1$

<!--more-->

### Analysis

考虑转化条件二:
令 $d = \gcd(a, b),\,\, a = a'd,\,\, b = b'd$. 那么条件二等价于:

$$a'd + b'd \mid a'b'd^2 \Rightarrow a' + b' \mid a'b'd \Rightarrow a' + b' \mid d$$

令 $d = t(a' + b')$, 则:

$$b = b'd = b'(a' + b')t$$

考虑枚举 $p = b',\,\, q = (a' + b')$, 计算 $t$ 的数量:

$$ Ans = \sum_{p=2}^{\sqrt{N}}\sum_{q=p+1}^{2p-1} [\gcd(p, q) = 1] \left\lfloor \frac{N}{pq} \right\rfloor $$

#### Algorithm 1

定义 $L_i$ 为 $i$ 的质因子个数.

首先枚举 $p$, 考虑有哪些 $q$ 能对答案产生贡献, 每次用 $O(L_p)$ 的时间判断 $p, q$ 是否互质.

复杂度为 $O(\sqrt{N} \sum_{i=2}^{\sqrt{N}} L_i) = O(N \log\log \sqrt{N})$

#### Algorithm 2

枚举完 $p$ 之后, $T = \lfloor \frac{N}{p} \rfloor$ 的值就确定了.

只需要考虑 $\lfloor \frac{T}{q} \rfloor$ 的值即可, 考虑对这样的 $q$ 进行分块, 每一块内部利用 $2^{L_p}$ 的容斥计算与 $p$ 互质的 $q$ 的个数.

复杂度近似为 $O(\sqrt{N} \sum_{i=2}^{\sqrt{N}} 2^{L_i})$

### Complexity Analysis

分析实际运行的情况, 发现理论复杂度更高的算法二表现优于算法一, 原因是 $\lfloor \frac{T}{q} \rfloor$ 的取值很少.

考虑综合两种算法的长处, 在 $q$ 比较小的时候, $\lfloor \frac{T}{q} \rfloor$ 的取值比较多, 我们使用算法一, $q$ 比较大的时候使用算法二.

假设这个分界点为 $k$, 复杂度:

$$f(k) = (k - p) L + (\left\lfloor \frac{T}{k} \right\rfloor + \left\lfloor \frac{T}{2p} \right\rfloor)2^L$$

只考虑其中与 $k$ 相关的部分:

$$f(k) = kL + \frac{T2^L}{k}$$

根据基本不等式, 使得 $f$ 最优的 $k = \sqrt{\frac{T2^L}{L}}$, 此时 $f(k) = 2\sqrt{T2^LL}$.

然后算一下复杂度, 发现在 $N = 2^{31}-1$ 时, 复杂度大约为 $8 \times 10 ^ 7$.
