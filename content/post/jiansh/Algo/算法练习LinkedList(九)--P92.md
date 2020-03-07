+++
title = "算法练习LinkedList(九)--P92"
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



[link on JianShu](https://www.jianshu.com/p/8550b5adf9b6)

[92. Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/)
 
Reverse a linked list from position m to n. Do it in one-pass.

Note: 1 ≤ m ≤ n ≤ length of list.

Example:

>Input: 1->2->3->4->5->NULL, m = 2, n = 4  
>Output: 1->4->3->2->5->NULL

可以先复习一下翻转链表的实现。链表操作两个基本场景——
- 将节点挂在头部
- 将节点挂在尾部

翻转链表则是典型的不断地向新的链表头部挂载新的节点。

```
private ListNode reverseNode(ListNode node) {
    ListNode prev = null;
    ListNode current = node;
    while (current != null) {
        ListNode temp = current.next; //保存临时变量
        current.next = prev; // 翻转操作。向量表头结点增加
        prev = current; // 移动头结点位置
        current = temp;
    }
    return prev;
}
```

只翻转一部分，需要多记录两个位置：
- 开始翻转的位置；  
-  翻转结束的位置；  

有几个特殊情况需要处理：  
1. m和n相等的情况，一开始没有考虑这个场景，导致提交的算法出现死循环`Memory Limit Exceeded`  
2. n是否是最后一个节点

Runtime: 0 ms, faster than 100.00% of Java online submissions for Reverse Linked List II.  
Memory Usage: 34.5 MB, less than 100.00% of Java online submissions for Reverse Linked List II.  
```
public ListNode reverseBetween(ListNode head, int m, int n) {

    //特殊场景，不需要处理
    if(m  == n) {
        return head;
    }

    ListNode dummy = new ListNode(0);
    dummy.next = head;

    ListNode g = dummy;
    ListNode reverse = null, tail = null, temp;
    int size = 0;
    while (head != null) {
        size++;
        if( m <= size && size <= n) {
            temp = head.next;
            head.next = reverse;
            reverse = head;
            head = temp;

            if(size == m ) {
                //翻转之后的尾结点
                tail = reverse;
            }else if(size == n) {
                break;
            }
        }else {
            g = g.next; //记录开始翻转的位置
            head = head.next;
        }
    }
    //先连接上翻转后的节点
    g.next = reverse;

    //n等于节点长度的情况。
    if(head == null) {
        return dummy.next;
    }

    // 还剩下有未翻转的节点，挂载到新节点上
    if(tail != null) {
        tail.next = head;
    }

    return dummy.next;
}
```
