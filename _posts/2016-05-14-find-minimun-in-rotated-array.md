---
layout:     post
title:      "Leetcode刷题"
subtitle:   "Find Minimum in Rotated Sorted Array"
date:       2016-05-14 15:00:00
author:     "sethbrin"
header-img: "img/post-bg-2016.jpg"
tags:
    - Leetcode
    - binary search
---

# 题目

> Suppose a sorted array is rotated at some pivot unknown to you beforehand.
>
> (i.e., 0 1 2 4 5 6 7 might become 4 5 6 7 0 1 2).
>
> Find the minimum element.
>
> You may assume no duplicate exists in the array.

首先最简单的方法是直接遍历数组，O(n)时间复杂度，但是显然没有完全利用题目条件，
题目提到原数组是一个排好序的数组，只是经过几次旋转平移得到。我们尝试用二分法求。

显然数组是先增加后减少，其中转折点就是我们所求的最小值。

1. 如果nums[mid] > nums[end], 那么显然转则点在mid右侧.
2. 如果nums[mid] < nums[end]，那么抓则点在mid左侧,或则是mid值。

代码如下：
{% highlight cpp %}
class Solution {
public:
    int findMin(vector<int>& nums) {
        // binary search

        int start = 0;
        int end = nums.size() - 1;
        
        while (start < end) {
            int mid = (start + end) / 2;
            if (nums[mid] > nums[end]) {
                start = mid + 1;
            } else {
                end = mid;
            }
            
        }
        return nums[start];
    }
};
{% endhighlight %}

# 变种
> The array may contain duplicates.

如果数组中允许重复，则考虑:
1. nums[mid] = nums[end], 此时无法判定转则点在左侧还是右侧，所以可以去掉end，数组中最小值仍然存在,所以代码：

{% highlight cpp %}
class Solution {
public:
    int findMin(vector<int>& nums) {
        // binary search
        int start = 0;
        int end = nums.size() - 1;
        
        while (start < end) {
            int mid = (start + end) / 2;
            if (nums[mid] > nums[end]) {
                start = mid + 1;
            } else if(nums[mid] == nums[end]) {
                end --;
            } else {
                end = mid;
            }
            
        }
        return nums[start];
    }
};
{% endhighlight %}

