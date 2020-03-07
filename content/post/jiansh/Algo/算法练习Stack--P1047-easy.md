+++
title = "算法练习Stack--P1047-easy"
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



[link on JianShu](https://www.jianshu.com/p/2f56dfe13fa8)

[栈stack](https://leetcode.com/tag/stack/)    
[1047. Remove All Adjacent Duplicates In String](https://leetcode.com/problems/remove-all-adjacent-duplicates-in-string/)

Given a string S of lowercase letters, a duplicate removal consists of choosing two adjacent and equal letters, and removing them.

We repeatedly make duplicate removals on S until we no longer can.

Return the final string after all such duplicate removals have been made.  It is guaranteed the answer is unique.

 

Example 1:

>Input: "abbaca"
Output: "ca"
Explanation: 
For example, in "abbaca" we could remove "bb" since the letters are adjacent and equal, and this is the only possible move.  The result of this move is that the string is "aaca", of which only "aa" is possible, so the final string is "ca".
 
Note:

- 1 <= S.length <= 20000
- S consists only of English lowercase letters.

纯暴力操作，只关注重复（相连的两个字母即可）。

Runtime: 399 ms, faster than 10.31% of Java online submissions for Remove All Adjacent Duplicates In String.
Memory Usage: 41.1 MB, less than 100.00% of Java online submissions for Remove All Adjacent Duplicates In String.
```
public String removeDuplicates(String S) {
    Stack<Character> stack = new Stack<>();
    for (int i = 0; i < S.length(); i++) {
        char c = S.charAt(i);

        if(stack.isEmpty()) {
            stack.push(c);
        }
        else if (stack.peek() == c) {
            stack.pop();
        }else {
            stack.push(c);
        }
    }
    String str = "";
    while (!stack.isEmpty()) {
        str = stack.pop() + str;
    }

    return str;
}
```
