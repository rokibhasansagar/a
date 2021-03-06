---
title: NPC, NP完全
date: 2018-09-26
---
# NPC, NP完全
NP完全或NP完备（NP-Complete，缩写为NP-C或NPC）。NPC应该是最不可能被化简为P（多项式时间可决定）的决定性问题的集合。若任何NPC问题得到多项式时间的解法，那此解法就可应用在所有NP问题上。
定义：
1. 它是NP问题且
2. 其他属于NP的问题都可归约成它, 解决了它就解决了所有NP

## 例子: 子集合加总问题
1. 给予一个有限数量的整数集合，找出任何一个此集合的非空子集且此子集内整数和为零. 
2. 目前能找到的解是遍历每一个集合: C(1, n)+C(2,n)+...+C(n,n)=2^n

### Pascal's triangle, 杨逃三角
杨辉三角形第 n 层（顶层称第 0 层，第 1 行，正好对应于二项式 (a+b)^{n} 展开的系数C(k, n) = C(k-1, n-1) + C(k, n-1)

## Reference
- [NPC]

[NPC]: https://zh.wikipedia.org/wiki/NP%E5%AE%8C%E5%85%A8