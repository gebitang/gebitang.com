+++
title = "Linked list problems solution"
description = "链表类型的算法题"
tags = [
    "algo",
    "list",
    "linked-list"
]
date = "2019-10-21"
topics = [
    "tech",
    "algo"
]
draft = true
toc = true
+++

[linked-list](https://leetcode.com/tag/linked-list/)

## Problem 2 Add Two Numbers

[2. Add Two Numbers Medium](https://leetcode.com/problems/add-two-numbers/)

You are given two non-empty linked lists representing two non-negative integers. The digits are stored in **reverse order** and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

Example:
```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```

1 - 读取两个链表构造为顺序的list，分别得到342,464
2 - 获取和
3 - 重新构造新的ListNode

```
public class ListNode{
        int val;
        ListNode next;
        ListNode(int x) {
            this.val = x;
        }
    }

public ListNode addTwoNumbers(ListNode l1, ListNode l2) {

        String i1 = readToNumber(l1);
        String i2 = readToNumber(l2);

        String s = addTwoStr(i1, i2);

        ListNode node = null;
        ListNode t = null;
        for (int i = s.length() - 1; i >=0 ; i--) {
            int v = Integer.parseInt(String.valueOf(s.charAt(i)));
            ListNode temp = new ListNode(v);
            if(node != null) {
                node.next = temp;
                node = node.next;
            }else {
                node = temp;
                t = temp;
            }
        }

        return t;
    }

    private String readToNumber(ListNode node) {
        String s = "";
        while (node != null) {
            s = node.val + s;
            node = node.next;
        }

        return s;
    }

    private static String addTwoStr(String s, String y) {
        int max = Math.max(s.length(), y.length());
        int t = 0;
        String v = "";
        for (int i = 0; i < max ; i++) {
            int s1 = i > s.length() -1 ? 0 : Integer.parseInt(String.valueOf(s.charAt(s.length() - 1 -i)));
            int y1 = i > y.length() -1 ? 0 : Integer.parseInt(String.valueOf(y.charAt(y.length() - 1 -i)));
            int c =(s1+y1+ t) % 10 ;
            t = s1+y1+t >= 10 ? 1 : 0 ;
            v = c + v;
        }

        if(t > 0) {
            v = 1 + v;
        }
        return v;
    }

```

第2步需要处理为String，否则可能越界。问题装换为数字格式的两个数组进行位置求和。但看起来我这个算法好复杂啊- -|| 

检查先练习再说吧:( 

晚上也算做出来了——这个“数字字符串”的“求和”有点“特殊化”了

```
private static String addTwoStr(String s, String y) {
        int max = Math.max(s.length(), y.length());
        int t = 0;
        String v = "";
        for (int i = 0; i < max ; i++) {
            int s1 = i > s.length() -1 ? 0 : Integer.parseInt(String.valueOf(s.charAt(s.length() - 1 -i)));
            int y1 = i > y.length() -1 ? 0 : Integer.parseInt(String.valueOf(y.charAt(y.length() - 1 -i)));
            int c =(s1+y1+ t) % 10 ;
            t = s1+y1+t >= 10 ? 1 : 0 ;
            v = c + v;
        }

        if(t > 0) {
            v = 1 + v;
        }
        return v;
    }
```

Runtime: 7 ms, faster than 5.04% of Java online submissions for Add Two Numbers.
Memory Usage: 39.7 MB, less than 99.69% of Java online submissions for Add Two Numbers.

[更高效的解决方法](https://leetcode.com/problems/add-two-numbers/discuss/1059/My-accepted-Java-solution)

### Refer

```
/**
Runtime: 1 ms, faster than 99.99% of Java online submissions for Add Two Numbers.
Memory Usage: 44.8 MB, less than 85.58% of Java online submissions for Add Two Numbers.
*/
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode prev = new ListNode(0);
        ListNode head = prev;
        int carry = 0;
        while (l1 != null || l2 != null || carry != 0) {
            
            ListNode cur = l1!=null?l1:(l2!=null?l2:(new ListNode(0)));
            
            
            int sum = ((l1 == null) ? 0 : l1.val) + ((l2 == null) ? 0 : l2.val) + carry;
            cur.val = sum % 10;
            carry = sum / 10;
            
            //挂上当前节点
            prev.next = cur;
            // 移动指针到下一个位置，为下一次挂节点做准备
            prev = prev.next;
            
            l1 = (l1 == null) ? l1 : l1.next;
            l2 = (l2 == null) ? l2 : l2.next;
        }
        return head.next;
    }
```

看完别人的解法，感觉我对题目理解的有问题。**reverse order**实际上正好满足正确的加法计算规则。`[2->4->5]  +  [3 -> 6 -> 9]` 对应于 542+964，计算时正好是按照顺序从第一位2、3开始计算加法

--- 

## Problem 19 Remove Nth Node From End of List

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

Runtime: 0 ms, faster than 100.00% of Java online submissions for Remove Nth Node From End of List.
Memory Usage: 35.1 MB, less than 100.00% of Java online submissions for Remove Nth Node From End of List.


看了网上的[实现](https://leetcode.com/problems/remove-nth-node-from-end-of-list/discuss/8804/Simple-Java-solution-in-one-pass)

1. 先固定头部：创建一个新的头节点，让头结点的下一个节点指向传入的节点  
2. 将链表复制多份，操作多个备份，只“读”不“写”，所以不会对原始的链表产生“破坏”。  
3. 实际对应链表的操作只有一步：跳过需要处理的节点。  
4. 找到头部节点的下一个节点。  

只循环一次的逻辑：需要两个“指针”同时跑——所有类似的一次循环都需要用到所谓的“快慢指针”。

关键点：先让一个备份跑n步。然后再同时跑两个备份，先跑的走到头之后，就是要找的位置。

明天来试试自己实现这个算法。


---

我以为我理解了这个算法的关键，但在自己的实现上执行时，还是没写出来应该怎么计算位置。参考上面的实现—— 

- 要走到要摘除的节点的前一个节点就停止。（否则：走到要拆除的节点位置时，时不知道前一个节点是什么的）
- 需要先创建一个节点保存原始的位置：最后返回时返回next节点。——为什么这样操作？如果不使用占位节点如何实现？还不清楚:(

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
        if(s == 0) {
            return node.next;
        }else {
            //新建一个节点
            ListNode f = new ListNode(0);
            ListNode temp = f;
            temp.next = node;
            while (s > 0) {
               temp = temp.next;
                s--;
            }
            //temp.next 为要移除的节点
            temp.next = temp.next.next;
            return f.next;

        }

    }
```

---

## Problem 21: Merge Two Sorted Lists

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

## Problem 61: Rotate List

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

## Problem 83: Remove Duplicates from Sorted List

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