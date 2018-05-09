---
layout: post
title:  Dual-Pivot QuickSort
date:   2018-05-09 11:04:00 +0800
categories: Algorithm
tag: Algorithm
author: Michael Wang
source_id: 180509110400
---

* content
{:toc}
### 0.引言
>&emsp;&emsp;[LeetCode P169](https://leetcode.com/problems/majority-element/description/)中，题意可以转化为输出所给数组排序后第$\lfloor n/2 \rfloor$个元素。
所以使用快排+二分的方法进行查找（即只要找到排序后数组的第$\lfloor n/2 \rfloor$个元素，就不再往下排序）。但是使用（JDK 1.8）Arrays.sort()方法对数组全部排序也会比我的实现方式快好多。所以查看了Arrays.sort()的源码。
发现其并非使用传统经典的快速排序，而是使用Dual-Pivot QuickSort（双轴快速排序）。

### 1.Java中的快排
<br/>&emsp;&emsp;在Java1.7之前的快排只是普通的快排，跟我们今天要研究的快排不一样，性能也差了许多，但其中对快排所做的各种优化我们依然是可以学习的。
<br/>&emsp;&emsp;Java1.7的快排是一种双轴快排，顾名思义：双轴快排是基于两个轴来进行比较，跟普通的选择一个点来作为轴点的快排是有很大的区别的。算法是由俄罗斯人Vladimir Yaroslavskiy在2009年研究出来的，并在2011年发布在了Java1.7。<br/>

### 2.Dual-Pivot QuickSort算法

#### 算法步骤
1.对于很小的数组（长度小于47），会使用插入排序。<br/>
2.选择两个点P1,P2作为轴心，比如我们可以使用第一个元素和最后一个元素。<br/>
3.P1必须比P2要小，否则将这两个元素交换，现在将整个数组分为四部分：<br/>
（1）第一部分：比P1小的元素。<br/>
（2）第二部分：比P1大但是比P2小的元素。<br/>
（3）第三部分：比P2大的元素。<br/>
（4）第四部分：尚未比较的部分。<br/>
在开始比较前，除了轴点，其余元素几乎都在第四部分，直到比较完之后第四部分没有元素。<br/>
4.从第四部分选出一个元素a[K]，与两个轴心比较，然后放到第一二三部分中的一个。<br/>
5.移动L，K，G指向。<br/>
6.重复 4 5 步，直到第四部分没有元素。<br/>
7.将P1与第一部分的最后一个元素交换。将P2与第三部分的第一个元素交换。<br/>
8.递归的将第一二三部分排序。<br/>
<center><img src="{{ '/styles/images/blog_images/20180509_00.png' | prepend: site.baseurl }}" alt="Pic of DualPivotQuickSort" width="70%" height="70%"/><br><font color="gray" size="1">注：图片来自Vladimir Yaroslavskiy的论文。</font></center>

#### 算法伪码
```java
dual_pivot_quicksort(A,left,right) // sort A[left..right]
    if right−left ≥ 1
        // Take outermost elements as pivots (replace by sampling)
        p := min {A[left],A[right]}
        q := max{A[left],A[right]}
        ℓ := left +1; g := right −1; k := ℓ
        while k ≤ g
            if A[k] < p
                 Swap A[k] and A[ℓ]; ℓ := ℓ+1
            else if A[k] ≥ q
                while A[g] > q and k < g
                    g := g −1
                end while
                Swap A[k] and A[g]; g := g −1
                if A[k] < p
                    Swap A[k] and A[ℓ]; ℓ := ℓ+1
                end if
            end if
            k := k +1
        end while
        ℓ := ℓ−1; g := g +1
        A[left] := A[ℓ]; A[ℓ] := p // p to final position
        A[right] := A[g]; A[g] := q // q to final position
        dual_pivot_quicksort(A, left , ℓ−1)
        dual_pivot_quicksort(A, ℓ+1,g −1)
        dual_pivot_quicksort(A,g +1,right)
    end if

```

看起来主要的区别就是经典快排递归的时候把输入数组分两段，而Dual-Pivot则分三段，就这么简单，那为什么这就快了呢？
其实如果按照元素比较次数来比较的话，Dual-Pivot快排元素比较次数其实比经典快排要多:
> 1.5697nlnn VS 1.7043nlnn
理论跟实际情况不符合的时候，如果实际情况没有错，那么就是理论错了。

### 3.CPU与内存
&emsp;&emsp;要理解上面的问题，先介绍点背景知识。我们平常很少考虑过```CPU的速度```，```内存的速度```，```CPU和内存速度是否匹配```的问题。

其实它们是不匹配的。

距统计在过去的25年里面，CPU的速度平均每年增长46%, 而内存的带宽每年只增长37%，那么经过25年的这种不均衡发展，它们之间的差距已经蛮大了。

假如这种不均衡持续持续发展，有一天CPU速度再增长也不会让程序变得更快，因为CPU始终在等待内存传输数据，这就是传说中内存墙(Memory Wall)。

有点像木桶理论：木桶的容量是由最短的那块板决定的，所以你CPU再快，如果内存带宽不够，那也没用。

25年前Dual-Pivot快排可能真的比经典快排要慢，但是25年之后虽然算法还是以前的那个算法，但是计算机已经不是以前的计算机了。在现在的计算机里面Dual-Pivot算法更快！


### 4.扫描元素个数
那么既然光比较元素比较次数这种计算排序算法复杂度的方法已经无法客观的反映算法优劣了，那么应该如何来评价一个算法呢？作者提出了一个叫做```扫描元素个数```的算法。

在这种新的算法里面，我们把对于数组里面一个元素的访问: ```array[i]```称为一次```扫描```。但是对于同一个下标，并且对应的值也不变得话，即使访问多次我们也只算一次。而且我们不管这个访问到底是读还是写。

其实这个所谓的```扫描元素个数```反应的是CPU与内存之间的数据流量的大小。

因为内存比较慢，统计CPU与内存之间的数据流量的大小也就把这个比较慢的内存的因素考虑进去了，因此也就比```元素比较次数```更能体现算法在当下计算机里面的性能指标。

在这种新的算法下面经典快排和Dual-Pivot快排的```扫描元素个数```分别为:

> 1.5697nlnn VS 1.4035nlnn
也就是说经典快排确实进行了更多的元素扫描动作，因此也就比较慢。在这种新的算法下面，Dual-Pivot快排比经典快排t节省了12%的元素扫描，从实验来看节省了10%的时间。

## 参考
[新的快速排序算法: 《Dual-Pivot QuickSort》阅读笔记](https://www.jianshu.com/p/2c6f79e8ce6e)
[Java源码解析-DualPivotQuicksort](https://blog.csdn.net/xjyzxx/article/details/18465661)