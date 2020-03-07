+++
title = "算法练习Stack(七)--P496"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Algo"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/98aa8d2bfdf0)

[栈stack](https://leetcode.com/tag/stack/)    
[496. Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/) Easy

You are given two arrays (**without duplicates**) `nums1` and `nums2` where `nums1’s` elements are subset of `nums2`. Find all the next greater numbers for `nums1's` elements in the corresponding places of `nums2`.

The Next Greater Number of a number `x` in `nums1` is the first greater number to its right in `nums2`. If it does not exist, output -1 for this number.

Example 1:

>Input: nums1 = [4,1,2], nums2 = [1,3,4,2].
Output: [-1,3,-1]

Explanation:
>    For number 4 in the first array, you cannot find the next greater number for it in the second array, so output -1.
    For number 1 in the first array, the next greater number for it in the second array is 3.
    For number 2 in the first array, there is no next greater number for it in the second array, so output -1.

Example 2:

>Input: nums1 = [2,4], nums2 = [1,2,3,4].
Output: [3,-1]

Explanation:
>    For number 2 in the first array, the next greater number for it in the second array is 3.
    For number 4 in the first array, there is no next greater number for it in the second array, so output -1.

Note:  
1. All elements in nums1 and nums2 are unique.
2. The length of both nums1 and nums2 would not exceed 1000.

---

一开始理解错了。 是说第一个数组中的对应的值：首先找到这个值在第二个数组中的位置（不是这个值在第一个数组中的位置 相等的 第二个数组中的位置）假设 第一个数组的第二个位置是3； 要找的是第二个数组中3的位置，而不是第二个数组的第二个位置。

暴-这里好像有个不那么敞亮的表达-力-的问题-破bl不能使用-解有效，速度比较慢。而且完全没有用到Stack数据结构——
Runtime: 11 ms, faster than 5.88% of Java online submissions for Next Greater Element I.  
Memory Usage: 37.2 MB, less than 100.00% of Java online submissions for Next Greater Element I.
```
public int[] nextGreaterElement(int[] nums1, int[] nums2) {
        int[] f = new int[nums1.length];

        for (int i = 0; i < nums1.length; i++) {
            int t = nums1[i];

            boolean u = false, found = false;
            for (int j = 0; j < nums2.length; j++) {
                int n = nums2[j];
                if(n == t) {
                    u = true;
                }else if(u && n > t) {
                    f[i] = n;
                    found = true;
                    break;
                }
            }
            if(!found) {
                f[i] = -1;
            }
        }
        return f;
    }
```


