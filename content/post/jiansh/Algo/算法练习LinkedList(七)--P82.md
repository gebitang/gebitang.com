+++
title = "算法练习LinkedList(七)--P82"
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



[link on JianShu](https://www.jianshu.com/p/e3830bfc76e7)

[Remove Duplicates from Sorted List 2](https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/) Medium

Given a sorted linked list, delete all nodes that have duplicate numbers, leaving only distinct numbers from the original list.

Example 1:

>Input: 1->2->3->3->4->4->5
>Output: 1->2->5

Example 2:

>Input: 1->1->1->2->3
>Output: 2->3

昨天做得有点郁闷了。‘精准原子’操作的套路还是没有掌握，写半天没完成效果。按照O(n2)的思路实现就有点暴力了。

- 两个list，一个保存重复的节点；一个保存所有的节点
- 重新构建链表：循环包含所有节点的list，去除在重复节点list中的节点

这个暴力算法结果：
Runtime: 3 ms, faster than 6.87% of Java online submissions for Remove Duplicates from Sorted List II.
Memory Usage: 36.3 MB, less than 100.00% of Java online submissions for Remove Duplicates from Sorted List II.

```
public ListNode deleteDuplicatesForce(ListNode head) {
    ListNode dummy=new ListNode(0);

    List<Integer> all = new ArrayList<>();
    List<Integer> dup = new ArrayList<>();
    while (head != null) {
        int v = head.val;
        if(!all.contains(v)) {
            all.add(v);
        }else {
            dup.add(v);
        }
        head = head.next;
    }

    ListNode n = dummy;
    for(Integer i : all) {
        if(!dup.contains(i)) {
            n.next = new ListNode(i);
            n = n.next;
        }
    }

    return dummy.next;
}
```
所谓‘精准原子’操作，是指：

- 遍历所有节点——大前提
- **循环**判断当前节点是否是重复节点，找到不重复的第一个节点
- 此时的当前节点为需要的一个**有效**节点，挂载到新的链表上
- 挂载操作是关键点：分两种情况(有重复节点、没有重复节点)

Runtime: 1 ms, faster than 79.85% of Java online submissions for Remove Duplicates from Sorted List II.  
Memory Usage: 37.8 MB, less than 32.56% of Java online submissions for Remove Duplicates from Sorted List II.  
```
public ListNode deleteDuplicates(ListNode head) {

    ListNode dummy=new ListNode(0);
    dummy.next=head; //挂上head，生成一个新链表

    ListNode pre=dummy; // 操作节点，保留dummy作为返回值

    while(head!=null){
        //**循环**判断当前节点是否是重复节点，找到不重复的第一个节点head.nex
        while(head.next!=null&&head.val==head.next.val){
            head=head.next;
        }

        // 如何相同，说明不是重复节点，正常循环到下一个节点
        if(pre.next==head){
            pre=pre.next;
        }
        // 有重复节点， head.next为“不重复的第一个节点”
        else{
            pre.next=head.next;
        }

        //循环条件
        head=head.next;
    }
    return dummy.next;
}
```


