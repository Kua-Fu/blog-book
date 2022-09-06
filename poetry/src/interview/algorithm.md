# 算法

## 参考

> [面试最常考的 100 道算法题分类整理](https://www.jianshu.com/p/b71cb1ab66bf)

> [leetcode](https://leetcode.cn/)

## 一、数组

## [1.1 合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/)

给你两个按 非递减顺序 排列的整数数组 nums1 和 nums2，另有两个整数 m 和 n ，分别表示 nums1 和 nums2 中的元素数目。

请你 合并 nums2 到 nums1 中，使合并后的数组同样按 非递减顺序 排列。

注意：最终，合并后数组不应由函数返回，而是存储在数组 nums1 中。为了应对这种情况，nums1 的初始长度为 m + n，其中前 m 个元素表示应合并的元素，后 n 个元素为 0 ，应忽略。nums2 的长度为 n 。


### 1.1.1 方法一：直接合并后排序

```go

func merge(nums1 []int, m int, nums2 []int, _ int) {
    copy(nums1[m:], nums2)
    sort.Ints(nums1)
}

```

复杂度分析

时间复杂度：O((m+n)\log(m+n))O((m+n)log(m+n))。
排序序列长度为 m+nm+n，套用快速排序的时间复杂度即可，平均情况为 O((m+n)\log(m+n))O((m+n)log(m+n))。

空间复杂度：O(\log(m+n))O(log(m+n))。
排序序列长度为 m+nm+n，套用快速排序的空间复杂度即可，平均情况为 O(\log(m+n))O(log(m+n))。




