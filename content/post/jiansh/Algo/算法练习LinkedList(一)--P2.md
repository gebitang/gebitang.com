+++
title = "算法练习LinkedList(一)--P2"
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



[link on JianShu](https://www.jianshu.com/p/14927be86814)

已废弃，参考--[算法练习LinkedList(三): P2、P19](https://www.jianshu.com/p/5c384936995c)

---

[linked-list](https://leetcode.com/tag/linked-list/)

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
``

Runtime: 7 ms, faster than 5.04% of Java online submissions for Add Two Numbers.
Memory Usage: 39.7 MB, less than 99.69% of Java online submissions for Add Two Numbers.
