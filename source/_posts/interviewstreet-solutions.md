---
layout: post
title: "InterviewStreet solutions"
date: 2012-08-16 00:10
comments: true
categories: algorithm
---

#### Kingdom Connectivity

大意就是计算一个图中从A到B一共有多少条路径，如果包含环就返回infinit。思路是首先找出从A到B所有可能经过的点，这里可以从A和B分别dfs，找出可达点的交集。然后再对这些点做拓扑排序，这个过程中记录到每个点的路径数。有环的情况也可以在时候排除掉。

#### String Reduction

一个字符串只有a，b，c三种字符，相邻的不同字符可以转变成另一个字符，问最后字符串最短的长度。这题是规律题，数学题证明一直不是我的强项，所以就大概领会下吧。首先要发现如果只有一种字符，字符串的长度就无法缩小了。其余情况最后字符串长度只能是1或为2。长度为2的情况最后字符个数只能为1，1，1，逆着推回去，可以发现三种字符串个数同时为偶数或者奇数。长度为1的最后状态是1，1，0，同理往回推。

#### Binomial Coefficients

`log_p(C)`表示排列组合C中p的个数，`b_p(n)`表示n以p为base表示时每一位数字之和,则
```
	log_p(C) = log_p(n!) - log_p(k!) - log_p((n-k))
	log_p(n!) = sum(floor(n/p^i)) = (n - b_p(n)) / (p -1)
	log_p(C) = (b_p(k) + b_p(n-k) - b_p(n)) / (p-1)
```
若p base下k加上n-k没有进位，则`b_p(k)+b_p(n-k)-b_p(n)`等于0，则C中不存在p。若有进位，低位减p，高位加1，C中至少存在一个p。
没有进位的k满足每一位小于等于n中对应位的值
```
	n = a0 + a1*p + ... + ai*p^i
	ans = n + 1 - mul(ai+1)
```

<!-- more -->
#### Meeting Point
可移动到相邻的八个格子中，即求Chebyshev距离。
```
  d = max( |x1-x2|, |y1-y2| )
    = ( |x1+y1-x2-y2| + |x1-y1-(x2+y2)| ) / 2
```
所以`point(x, y)`间的Chebyshev距离可以转化为求`point(x+y, x-y)`间的Manhattan距离。Manhattan距离比较好求，可以对点集在x轴和y轴分别排序求出，然后枚举每个点X轴和Y轴距离之和，求出最小值即可。
```
  ans = min(lx[xi]+rx[xi]+ly[yi]+ry[yi])
```

#### Matrix
需要隔离K个machine，必须删除K-1条边。把边按时间从大到小排序，依次选择，用并查集维护连通子图和子图中的machine。若边加入后，存在两个machine连通，则不选择此条边。

#### Problem Solving
即求有向无环图的最小路径覆盖数 = 定点数 - 扩点后二分图最大匹配数。扩点方法为二分图左边右边各有所有一个点集V,V'，若有从点i到点j的边，则添加从i到j'的边。


代码略, 更新于2012.09.18