---
title: LeetCode上的二分查找
date: 2020-06-17 20:32
tags: 
categories: 算法
thumbnail: /thumbnail/matteo-catanese-621793-unsplash.jpg
---

将 `LeetCode` 上二分查找相关的几道题目 AC 了，这里做个总结。

<!-- more -->

### 经典二分查找

> LeetCode - 705
>
> Given a **sorted** (in ascending order) integer array `nums` of `n` elements and a `target` value, write a function to search `target` in `nums`. If `target` exists, then return its index, otherwise return `-1`.

题意比较好理解，给定一个已经排序好的数组，并给定一个 `target` 要求找到 `target` 在数组中的对应下标，若没找到，则返回 `-1`。

`Kotlin` 解法如下：

```kotlin
class Solution {
    fun search(nums: IntArray, target: Int): Int {
        var left = 0
        var right = nums.size - 1

        while (left <= right){
            val mid = (left + right) / 2
            if (nums[mid] == target) return mid

            if (nums[mid] > target){
                right = mid -1
            } else {
                left = mid + 1
            }
        }

        return -1
    }
}
```

需要注意的点有：

* `right` 初始值为 `nums.size - 1` ，这是为了与下面的 `left <= right` 呼应。

* 中位数的值 `> target`时，即说明 `target`的值可能在下标`left（Inclusive）` 和 `mid（Exclusive）`之间，所以要将查找范围向左收缩，即`right = mid -1`.
* 中位数的值 `< target`时，即说明 `target`的值可能在下标`mid（Exclusive`和 `right（Inclusive）`之间，所以要将查找范围向右收缩，即`left = mid +1`.
* 前面说之所以用可能是因为 `target`的值可能根本不在当前的查找区间内，试想`target`值大于或者小于数组中的所有值的情况。

### 二分查找变种，返回最佳插入位置

> LeetCode - 35
>
> Given a sorted array and a target value, return the index if the target is found. If not, return the index where it would be if it were inserted in order.
>
> You may assume no duplicates in the array.

解法如下：

```kotlin
class Solution {
    fun searchInsert(nums: IntArray, target: Int): Int {
        var left = 0
        var right = nums.size - 1

        while (left <= right){
            val mid = (left + right) / 2
            if (nums[mid] == target) return mid

            if (nums[mid] > target){
                right = mid -1
            } else {
                left = mid + 1
            }
        }

        return left
    }
}
```

解法和经典查找几乎一模一样，唯一不同的是如果没查找到，返回`left`的值就行了。为什么返回`left`就可以了呢？其实可以这样考虑，`while`循环的条件是 `left<=right`，而如果一直没有查找到，则进行最后一次循时必有`left = right = mid`，此时，对`num[mid]`和`tartget`的值的三种情况进行分析:

* `nums[mid] == target`,则表示查找到了，直接返回`mid`即可。
* `nums[mid] > target`，很容易得出`target`插入位置应该为`mid `，而恰好循环结束后，`left = mid`，即可以返回`left`。
* `nums[mid]` < `target`，很容易得出`target` 的插入位置应该为`mid +1`， 恰好循环结束后，`left = mid +1 `,即可以返回`left`。
* 综上，我们只需要返回`left`的值就行了。

### 进阶，找到最左边和最右边的下标

> LeetCode - 34
> Given an array of integers `nums` sorted in ascending order, find the starting and ending position of a given `target` value.
>
> Your algorithm's runtime complexity must be in the order of *O*(log *n*).
>
> If the target is not found in the array, return `[-1, -1]`.

对于前面两道题，我们都预设了数组中有且仅有一个数字与`tartget`相同，这道题却不一样，他考虑到了，数组中有多个值和`target`相同的情况，并需要求出最左边和最右边的`index`。这道题有两种思路，第一种是按照经典方法找到和`tartget`对应的`index`之后，向左右两边继续搜索；另一种则是，找到`target`后对应的`index`之后，继续收缩边界，并在最后进行判断取值。第一种思路比较好理解，但是第二种方法维持了二分查找的连贯性。我们来看看采取第二种思路的解法。

解法如下：

```kotlin
class Solution {
    fun searchRange(nums: IntArray, target: Int): IntArray {
        val result = intArrayOf(0,0)
        result[0] = findLeft(nums,target)
        result[1] = findRight(nums,target)
        return result
    }

    private fun findLeft(nums: IntArray, target: Int): Int {
        var left = 0
        var right = nums.size - 1

        while (left <= right){
            val mid = (left + right) / 2
            if (nums[mid] >= target){
                right = mid - 1
            } else {
                left = mid + 1
            }
        }
      
        val candidate = right + 1
        if (candidate > nums.size -1 ) return  -1
        return if (nums[candidate] == target) candidate else -1
    }

    private fun findRight(nums: IntArray, target: Int): Int {
        var left = 0
        var right = nums.size - 1

        while (left <= right){
            val mid = (left + right) /2
            if (nums[mid] <= target){
                left = mid +1
            } else{
                right = mid -1
            }
        }

        val candidate = left - 1
        if (candidate < 0) return  -1
        return if (nums[candidate] == target) candidate else -1
    }

}
```

* 先分析查找最左边`index`值的情况。假设到某个时刻，`mid`对应的值等于`target`，因为我们要找到最左边的`index`，所以我们继续将查找范围向左收缩，即`right = mid -1`，那么结束搜索后，`right`对应的值必定小于`target`,而`right+1`所对应的`index`的值是最接近`target`，并`>=target`的值的。所以我们判断一下`right+1`的对应的值就行了，如果相等，说明找到了最左边的值；不相等，说明，没有查找到对应的值。但是，某种情况比较特殊，就是整个数组的值都小于`target`，此时搜索结束后，`right+1`越界了，所以我们需要考虑这种边界情况。
* 查找最右边的`index`值同理。假设到某个时刻，`mid`对应的值等于`target`，因为我们需要找到最右边的`index`，所以我们需要继续将范围向右收缩，即`left = mid + 1`,到最后，`left`的值必定大于`target`,而`left-1`所对应的值是最接近`tartget`并 `<= tartget`的，同理进行判断就行了。此时也要注意所有元素都大于`target`的情况，循环结束后`left = 0`，因此也要先进行边界判断。

### 结语

以上就是`LeetCode`上关于二分查找的几个典型的题，这里差不多就总结完了。

