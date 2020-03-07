+++
title = "算法练习LinkedList(六)--P83"
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



[link on JianShu](https://www.jianshu.com/p/a59afd2f3f5c)

[Remove Duplicates from Sorted List](https://leetcode.com/problems/remove-duplicates-from-sorted-list/) Easy

Given a sorted linked list, delete all duplicates such that each element appear only once.

Example 1:

> Input: 1->1->2  
> Output: 1->2

Example 2:

> Input: 1->1->2->3->3  
> Output: 1->2->3

直接的实现对于最后一个节点的处理不够精细，调试后完成了算法。

Runtime: 2 ms, faster than 6.29% of Java online submissions for Remove Duplicates from Sorted List.  
Memory Usage: 36.9 MB, less than 100.00% of Java online submissions for Remove Duplicates from Sorted List.

```
public ListNode deleteDuplicates(ListNode head) {

    if(head == null || head.next == null) return head;

    List<Integer> unique = new ArrayList<>();

    ListNode dummy = new ListNode(0);
    dummy.next = head;

    ListNode t = dummy;
    while (head != null) {
        int v = head.val;
        if(!unique.contains(v)) {
            unique.add(v);
            t.next = head;
            t = t.next;
            head = head.next;
        }else {
            head = head.next;
            if(head == null) {
                t.next = null; //最后一个节点断开
                break;
            }
        }
    }
    return dummy.next;
}
```

还是题目意图理解的有偏差，` Sorted List`意味着链表中不可能有重复(重复2)这种结构的节点`1->2->2->4->2`。这种情况下，可以不依赖外部数据结构，直接操作链表即可——

```
public ListNode deleteDuplicates(ListNode n) {
    ListNode head = n;
    while (n != null) {
        if(n.next == null) {
            break;
        }
        if(n.val == n.next.val) {
            n.next = n.next.next;
        }else {
            n = n.next;
        }
    }
    return head;
}
```
