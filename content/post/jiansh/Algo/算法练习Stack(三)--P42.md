+++
title = "算法练习Stack(三)--P42"
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



[link on JianShu](https://www.jianshu.com/p/d631db1f823e)

[栈stack](https://leetcode.com/tag/stack/)  
[42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) Hard

Given n non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it is able to trap after raining.

![trapped water](https://upload-images.jianshu.io/upload_images/3296949-31e227fe71d71a8b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

The above elevation map is represented by array [0,1,0,2,1,0,1,3,2,1,2,1]. In this case, 6 units of rain water (blue section) are being trapped. Thanks Marcos for contributing this image!

Example:

>Input: [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6

---

Hard模式果然不同凡响啊，这是什么鬼。完全没有思路好嘛。只能先去看看别人是怎么搞出来的了。

你说这些第一个搞出来不同算法的人是怎么想出来“这么干就可以”的。

别说还得用Stack来解决问题了。先看看别人是怎么处理这个问题的——

```
public int trap(int[] height) {
        if (height == null || height.length < 2) return 0;

        Stack<Integer> stack = new Stack<>();
        int water = 0, i = 0;
        while (i < height.length) {
            if (stack.isEmpty() || height[i] <= height[stack.peek()]) {
                stack.push(i++);
            } else {
                int pre = stack.pop();
                if (!stack.isEmpty()) {
                    // find the smaller height between the two sides
                    int minHeight = Math.min(height[stack.peek()], height[i]);
                    // calculate the area
                    water += (minHeight - height[pre]) * (i - stack.peek() - 1);
                }
            }
        }
        return water;
    }
```
