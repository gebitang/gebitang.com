+++
title = "算法练习LinkedList(四)--P21"
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



[link on JianShu](https://www.jianshu.com/p/563653ad8de2)

[21. Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) Easy

Merge two sorted linked lists and return it as a new list. The new list should be made by splicing together the nodes of the first two lists.

>     Example:
>
>     Input: 1->2->4, 1->3->4
>     Output: 1->1->2->3->4->4

这是个Easy级别的，有前两次的经验，一次做对，但执行效率不高

>Runtime: 1 ms, faster than 23.66% of Java online submissions for Merge Two Sorted Lists.
>Memory Usage: 39.5 MB, less than 16.84% of Java online submissions for Merge Two Sorted Lists.

```
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {

        if(l1==null) {
            return l2;
        }
        if(l2==null) {
            return l1;
        }

        ListNode n = new ListNode(0);
        ListNode temp = n;
        int j , k;
        while (l1 != null || l2 != null) {

            if(l1 ==null) {
                temp.next = new ListNode(l2.val);
                temp = temp.next;
                l2 = l2.next;
                continue;
            }

            if(l2 ==null) {
                temp.next = new ListNode(l1.val);
                temp = temp.next;
                l1 = l1.next;
                continue;
            }

            j= l1.val;
            k = l2.val;

            if(j==k) {
                temp.next = new ListNode(j);
                temp = temp.next;

                temp.next = new ListNode(k);
                temp = temp.next;

                l1 = l1.next;
                l2 = l2.next;

            }else if(j > k) {
                temp.next = new ListNode(k);
                temp = temp.next;
                l2 = l2.next;
            }else {
                temp.next = new ListNode(j);
                temp = temp.next;
                l1 = l1.next;
            }

        }

        return n.next;
    }
```

学习一下[讨论区](https://leetcode.com/problems/merge-two-sorted-lists/discuss/?currentPage=1&orderBy=most_votes&query=)的高效方法—— 

几点改进的地方：
1. 循环处理为 且的关系，（关键点）
2. 结束循环后再分别判断两个链表的情况
3. 直接使用节点本身进行挂载，不需要新建（那为什么我采用新建的反而占用内存更少？新建的只包含一个节点。验证了一下使用新建也只是降低了0.1MB的内存占用） 
4. 不需要特殊处理相等的条件；


```
/**
     Runtime: 0 ms, faster than 100.00% of Java online submissions for Merge Two Sorted Lists.
     Memory Usage: 40.7 MB, less than 11.78% of Java online submissions for Merge Two Sorted Lists
     */
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode n = new ListNode(0);
        ListNode temp = n;
        while (l1 != null && l2 != null) {

            if(l1.val >= l2.val) {
                temp.next = l2;
                temp = temp.next;
                l2 = l2.next;
            }else {
                temp.next = l1;
                temp = temp.next;
                l1 = l1.next;
            }
        }

        if(l1 != null) {
            temp.next = l1;
        }

        if(l2 != null) {
            temp.next = l2;
        }
        return n.next;
    }
```
