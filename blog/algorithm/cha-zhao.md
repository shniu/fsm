---
description: 查找相对而言较为简单，不外乎顺序查找、二分查找、哈希表查找和二叉排序树查找。熟练掌握二分查找是必须的。
---

# 查找

### [二分查找](https://time.geekbang.org/column/article/42520)

> 二分查找针对的是一个有序的数据集合，查找思想有点类似分治思想。每次都通过跟区间的中间元素对比，将待查找的区间缩小为之前的一半，直到找到要查找的元素，或者区间被缩小为 0。

二分查找的局限：

1. 二分查找依赖的是顺序表结构，简单点说就是数组。
2. 二分查找针对的是有序数据。
3. 数据量太小不适合二分查找。
4. 数据量太大也不适合二分查找。因为二分查找依赖数组，需要连续的内存空间，如果数据太大，很难放下

必须掌握的几种二分查找的变形：

* 最常规的 二分查找 算法实现 （两种实现方式：[while loop](https://github.com/shniu/java-eco/blob/master/eco-algorithm/src/main/java/io/github/shniu/algorithm/search/WhileLoopBinarySearch.java) 和 [递归](https://github.com/shniu/java-eco/blob/master/eco-algorithm/src/main/java/io/github/shniu/algorithm/search/RecursiveBinarySearch.java)实现\)
* 找不包含重复元素的有序数组旋转后的最小值
* 找包含重复元素的有序数组旋转后的最小值
* 有序数组中查找第一个等于给定值的位置（有序数组可能有重复元素）
* 有序数组中查找最后一个等于给定值的位置（有序数组可能有重复元素）
* 有序数组中查找第一个大于等于给定值的位置（有序数组可能有重复元素）
* 有序数组中查找最后一个小于等于给定值的位置（有序数组可能有重复元素）

### 哈希表查找

--

### 二叉排序树

--

### 题目

* 假设我们有 1000 万个整数数据，每个数据占 8 个字节，如何设计数据结构和算法，快速判断某个整数是否出现在这 1000 万数据中？ 我们希望这个功能不要占用太多的内存空间，最多不要超过 100MB，你会怎么做呢？（思路：1000万数据，每个数据8个字节，大约80MB，放在内存是可以的，然后排序后，使用二分查找；但是使用哈希查找和二叉树查找，有可能是不满足内存要求的）
* 如何编程实现“求一个数的平方根”？要求精确到小数点后 6 位。
* 给定一个最多包含 40 亿个随机排列的 32 位整数的顺序文件，找出一个不在文件中的 32 位整数。在具有足够内存的情况下，如何解决该问题？如果有几个外部的 临时 文件可用，但是仅有几百字节的内存，又该如何解决该问题？

