# 理解均摊分析

# 理解平摊分析

[[MIT算法导论第13集] 平摊分析，表的扩增，势能方法_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1A4411a7pf?spm_id_from=333.337.search-card.all.click)

## 哈希表的均摊分析

Hash Table 在足够长时，搜索时间会很短。

提出dynamic tables：当 Hash Table 元素过多的时候，grow it。这意味着要copy所有的旧元素到新空间之中。

**此时插入的最高代价为O(n)，但并不是每次都是O(n)。**

因此时间复杂度为：

$$
\sum_{i=1}^n c_i = n+\sum_{j=0}^{[\log(n-1)]}2^j\leq 3n
$$

均摊则为：

$$
\frac1n\sum_{i=1}^n c_i=O(1)
$$

<aside>
⛱️ 对于很多数据结构而言，只需要证明某次复杂的操作复杂度小于O(n)。

</aside>

## 三种均摊分析方法

1. 聚集方法（就是直接算出n步的总复杂度）。
2. 记账法。
3. 势能法。

后两种方法能更精确地表达每个操作的平摊复杂度是多少。

## 记账法

对每个操作$c_i$进行收费（虚构的平摊代价），如果支付的费用不够，则余额来支付。且不能向未来借款。

$$
\sum_{i=1}^n c_i\leq\sum_{i=1}^n \hat{c_i}
$$

对于动态hash table，假设**每个操作有3元**：

- 直接插入时：
    - 插入支出1元。
    - 存储2元。
- 需要翻倍时：
    - 移动旧元素从银行取出n元。
    - 插入新元素支出1元。
    - 储存2元。

![Untitled](http://brucemarkdown.top/images/%E7%90%86%E8%A7%A3%E5%B9%B3%E6%91%8A%E5%88%86%E6%9E%90%207a26c/Untitled.png)

如此过程，**可以持续（显然）**，说明每个操作平摊复杂度上界为3元。

## 势能方法

实际上银行账户就是我们要分析的势能。

从数据结构$D_0$开始，$i$操作将$D_{i-1}$转化为$D_i$。定义$\Phi$为数据结构到实数的映射，使得$\Phi(D_0)=0$，且$\Phi(D_i)>=0$。

定义平摊代价为$\hat{c_i}$，

$$
\hat{c_i}=c_i+\Phi(D_i)-\Phi(D_{i-1})=c_i+\Delta\Phi_i
$$

若$\Delta\Phi_i\geq0$，那么说明操作$i$可以存储一些功到银行。

否则，则说明操作$i$需要取出一些功。

因此，只要找到适合的势函数（任意符合条件$\Phi(D_0)=0$，且$\Phi(D_i)>=0$的函数），那么就可以得到均摊复杂度的**其中一个**上界。

$$
\begin{aligned}\frac1n\sum_{i=1}^n\hat{c_i}&=\frac1n\sum_{i=1}^n(c_i+\Phi(D_i)-\Phi(D_{i-1}))\\ &=\frac1n\sum_{i=1}^nc_i+\frac1n\Phi(D_i) \\ & \geq\frac1n\sum_{i=1}^nc_i\end{aligned}
$$

实际上势能方法，只是用代数来更精确地靠近上界。但是原理跟记账法其实是相同的。

