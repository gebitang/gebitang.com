+++
title = "算法练习LinkedList(十一)--P109"
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



[link on JianShu](https://www.jianshu.com/p/4781e504ed19)

[109. Convert Sorted List to Binary Search Tree](https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree/) 

Given a singly linked list where elements are sorted in ascending order, convert it to a height balanced BST.

For this problem, a height-balanced binary tree is defined as a binary tree in which the depth of the two subtrees of every node never differ by more than 1.

**Example:**  
Given the sorted linked list: [-10,-3,0,5,9], One possible answer is: [0,-3,9,-10,null,5], which represents the following height balanced BST:
```
      0
     / \
   -3   9
   /   /
 -10  5
```
看示例，看起来关键是“每个节点 的左右两个子树的高度差的绝对值不超过 1。”是不是平衡二叉树不重要？看到二叉树这种东西就心虚，先存个[链接](https://blog.csdn.net/xiaohui987987/article/details/81347463)

这样似乎有个思路：快慢指针遍历链表，获取到中间位置。分别翻转前后两个链表，然后按照示例的方式构造二叉树？


参考了[这里](https://leetcode.com/problems/convert-sorted-list-to-binary-search-tree/discuss/35476/Share-my-JAVA-solution-1ms-very-short-and-concise./229025)的实现，验证我的思路是(一半)正确的。 但我自己实现了半天也没出来- -||，还是个半成品。

Runtime: 1 ms, faster than 94.08% of Java online submissions for Convert Sorted List to Binary Search Tree.
Memory Usage: 38.3 MB, less than 100.00% of Java online submissions for Convert Sorted List to Binary Search Tree.
```
public TreeNode sortedListToBST(ListNode head) {
    if (head == null) {

        return null;
    }
    if (head.next == null) {
        return new TreeNode(head.val);
    }
    ListNode slow = head, pre = null, fast = head;
    //快慢指针走到中间：走到中间的判断条件——判断fast && fast.next
    while (fast != null && fast.next != null) {
        pre = slow;
        slow = slow.next;
        fast = fast.next.next;
    }
    //截断左节点
    pre.next = null;
    //第一次循环后，找到树的root节点位置
    TreeNode n = new TreeNode(slow.val);
    n.left = sortedListToBST(head); //传递上半区的链表，挂载到左节点上
    n.right = sortedListToBST(slow.next); //开始挂右节点
    return n;
}
```

