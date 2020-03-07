+++
title = "算法练习Stack--P1021-easy"
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



[link on JianShu](https://www.jianshu.com/p/13676a9683c0)

[栈stack](https://leetcode.com/tag/stack/)    
[1021. Remove Outermost Parentheses](https://leetcode.com/problems/remove-outermost-parentheses/)


A valid parentheses string is either empty (""), "(" + A + ")", or A + B, where A and B are valid parentheses strings, and + represents string concatenation.  For example, "", "()", "(())()", and "(()(()))" are all valid parentheses strings.

A valid parentheses string S is primitive if it is nonempty, and there does not exist a way to split it into S = A+B, with A and B nonempty valid parentheses strings.

Given a valid parentheses string S, consider its primitive decomposition: S = P_1 + P_2 + ... + P_k, where P_i are primitive valid parentheses strings.

Return S after removing the outermost parentheses of every primitive string in the primitive decomposition of S.


Example 1:

>Input: "(()())(())"
Output: "()()()"
Explanation: 
The input string is "(()())(())", with primitive decomposition "(()())" + "(())".
After removing outer parentheses of each part, this is "()()" + "()" = "()()()".

Example 2:
>Input: "(()())(())(()(()))"
Output: "()()()()(())"
Explanation: 
The input string is "(()())(())(()(()))", with primitive decomposition "(()())" + "(())" + "(()(()))".
After removing outer parentheses of each part, this is "()()" + "()" + "()(())" = "()()()()(())".

Example 3:

>Input: "()()"
>Output: ""
Explanation: 
The input string is "()()", with primitive decomposition "()" + "()".
After removing outer parentheses of each part, this is "" + "" = "".

将最外层的括号删除，意味着只保留“最外层括号内的内容”。使用这个思路，需要判断什么时候出现了“最外层括号”。

顺序遍历，当左括号的个数与右括号的个数相同时，意味着可以“收集”括号中的内容了。然后开始下一个“括号内”内容。

所以需要记录三个变量：左括号个数、右括号个数、需要收集的内容的开始位置。

Runtime: 3 ms, faster than 69.58% of Java online submissions for Remove Outermost Parentheses.
Memory Usage: 37 MB, less than 94.81% of Java online submissions for Remove Outermost Parentheses.
```
public String removeOuterParentheses(String S) {

    StringBuilder sb = new StringBuilder();
    int open=0, close=0, start=0;
    for(int i=0; i<S.length(); i++) {
        if(S.charAt(i) == '(') {
            open++;
        } else if(S.charAt(i) == ')') {
            close++;
        }
        if(open==close) {
            sb.append(S.substring(start+1, i));
            start=i+1;
        }
    }
    return sb.toString();
}
```
