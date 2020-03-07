+++
title = "算法练习LinkedList(八)--P86"
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



[link on JianShu](https://www.jianshu.com/p/6e8a45afed24)

[86. Partition List](https://leetcode.com/problems/partition-list/) Medium

Given a linked list and a value `x`, partition it such that all nodes less than `x` come before nodes greater than or equal to x.

You should preserve the original relative order of the nodes in each of the two partitions.

Example:

>Input: head = 1->4->3->2->5->2, x = 3
>Output: 1->2->2->4->3->5

梳理一下思路，应该很快可以写出来这样一个架子：
- 将来两个链表，分别存储分开之后的两个链表
- 遍历给定的链表，所有小于x的都放在前面的链表里；否则则加入到后面的链表

```
public ListNode partition(ListNode head, int x) {

    ListNode before = new ListNode(0);
    ListNode after = new ListNode(0);

    ListNode bh = before, ah = after;
    while (head != null) {
        int v = head.val;
        if(v < x) {
            //TODO
        }else {
            //TODO
        }
        head = head.next;
    }
    return null;
}
```

到这里之后，如果操作链表就越写越糊涂了:( 

链表操作：**挂上节点；然后移动到链表尾部**。

Runtime: 0 ms, faster than 100.00% of Java online submissions for Partition List.
Memory Usage: 36 MB, less than 100.00% of Java online submissions for Partition List.
```
public ListNode partition(ListNode head, int x) {

    ListNode before = new ListNode(0);
    ListNode after = new ListNode(0);

    ListNode bh = before, ah = after;
    while (head != null) {
        int v = head.val;
        if(v < x) {
            bh.next = head; //添加到前一个链表里
            bh = bh.next; //移动到前一个链表的尾部
        }else {
            ah.next = head; //添加到后一个链表里
            ah = ah.next; //移动到后一个链表的尾部
        }
        head = head.next;
    }

    ah.next = null; //确保断开后一个链表； 避免ah.next是一个小于x的节点时发生循环
    bh.next = after.next; //连接两个链表：将前一个链表的尾部挂到后面的链表上

    return before.next;
}
```

