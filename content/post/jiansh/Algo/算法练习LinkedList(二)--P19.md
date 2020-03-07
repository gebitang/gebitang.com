+++
title = "算法练习LinkedList(二)--P19"
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



[link on JianShu](https://www.jianshu.com/p/48acb3e748d3)

已废弃，参考--[算法练习LinkedList(三): P2、P19](https://www.jianshu.com/p/5c384936995c)

---

[19. Remove Nth Node From End of List ](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)

Given a linked list, remove the n-th node from the end of list and return its head.

**Example:**
```
Given linked list: 1->2->3->4->5, and n = 2.
After removing the second node from the end, the linked list becomes 1->2->3->5.
```

**Note:** Given `n` will always be valid.
**Follow up:** Could you do this in one pass?

笨办法：从头走到尾，找到第N位；再重新……

直接上手写到这一步就懵了—— 如何进行链表节点的摘取还是不清楚。
```
public ListNode removeNthFromEnd(ListNode head, int n) {

        if(head == null ||  n <= 0) return head;

        int i = 0;
        ListNode node = head;
        while (node != null) {
            i++;
            node = node.next;
        }

        int s = i - n;

        node = head;
        if(i == 0) {
            return node.next;
        }else {
            ListNode f = node;
            int step = 0;
        }

        return  null;

    }
```

看了网上的[实现](https://leetcode.com/problems/remove-nth-node-from-end-of-list/discuss/8804/Simple-Java-solution-in-one-pass)

1. 先固定头部：创建一个新的头节点，让头结点的下一个节点指向传入的节点
2. 将链表复制多份，操作多个备份，只“读”不“写”，所以不会对原始的链表产生“破坏”。
3. 实际对应链表的操作只有一步：跳过需要处理的节点。
4. 找到头部节点的下一个节点。

只循环一次的逻辑：需要两个“指针”同时跑——所有类似的一次循环都需要用到所谓的“快慢指针”。

关键点：先让一个备份跑n步。然后再同时跑两个备份，先跑的走到头之后，就是要找的位置。

明天来试试自己实现这个算法。

