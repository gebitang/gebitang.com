+++
title = "算法练习LinkedList(五)--P61"
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



[link on JianShu](https://www.jianshu.com/p/06c00c9a2ca4)

[Rotate List](https://leetcode.com/problems/rotate-list/) Medium

Given a linked list, rotate the list to the right by k places, where `k` is non-negative.

Example 1:
>Input: 1->2->3->4->5->NULL, k = 2  
>Output: 4->5->1->2->3->NULL  

Explanation:  
>rotate 1 steps to the right: 5->1->2->3->4->NULL  
>rotate 2 steps to the right: 4->5->1->2->3->NULL  

Example 2:

>Input: 0->1->2->NULL, k = 4  
>Output: 2->0->1->NULL  

Explanation:  
>rotate 1 steps to the right: 2->0->1->NULL  
>rotate 2 steps to the right: 1->2->0->NULL  
>rotate 3 steps to the right: 0->1->2->NULL  
>rotate 4 steps to the right: 2->0->1->NULL  

对比后可知，k有可能大于整个链表的长度，需要判断出来实际上需要翻转的位置。 所以必须要获取链表的长度。
获取长度之后，要翻转的位置为 S 个位置（int s = k % size）。

自己写的时候，总是不好控制要移动到哪个位置。——目前看，链表题都需要先手动添加一个 dummy节点，然后将head节点挂在这个dummy节点的下一个位置。

然后操作时，都直接操作dummy这个新链表。这样确保操作的最终目标都是移动之后的 target.next节点。

```
public ListNode rotateRight(ListNode head, int k) {

        if (head==null||head.next==null) return head;

        ListNode dummy=new ListNode(0);
        dummy.next=head;

        ListNode fast=dummy,slow=dummy;

        int size= 0; //链表长度
        while (fast.next != null) {
            fast = fast.next;
            size++;
        }

        int s = k % size; //移动次数

        if(s == 0) return dummy.next; //不需要移动

        for (int i = 0; i < (size - s); i++) { //要移动的位数
            slow=slow.next;
        }

        //关键的翻转操作
        fast.next=dummy.next; //最后一个节点的next需要指向最初的头结点
        dummy.next=slow.next; // slow.next是目前新的头结点
        slow.next=null; //将slow.next节点至为null。完成翻转动作

        return dummy.next;

    }
```
