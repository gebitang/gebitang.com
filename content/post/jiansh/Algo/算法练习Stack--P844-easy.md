+++
title = "算法练习Stack--P844-easy"
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



[link on JianShu](https://www.jianshu.com/p/dc1fa553754a)

[栈stack](https://leetcode.com/tag/stack/)    
[844. Backspace String Compare](https://leetcode.com/problems/backspace-string-compare/) Easy

Given two strings S and T, return if they are equal when both are typed into empty text editors. # means a backspace character.

Example 1:
>Input: S = "ab#c", T = "ad#c"
>Output: true
>Explanation: Both S and T become "ac".

Note:
>1 <= S.length <= 200
1 <= T.length <= 200
S and T only contain lowercase letters and '#' characters.

这个比较好理解了，对比打字输出的内容是否一致。 使用`#`代表退格键`Backspace`。

使用stack还是比较好理解，按照正常的思路就可以解出来。但写代码还是出现了bug，不够仔细。

变量两个String，将每个String最后剩下的内容存在stack中，然后对比两个stack即可。 注意边界条件。

Runtime: 2 ms, faster than 50.07% of Java online submissions for Backspace String Compare.
Memory Usage: 34.4 MB, less than 100.00% of Java online submissions for Backspace String Compare.
```
public boolean backspaceCompare(String S, String T) {

    Stack<Character> s = new Stack<>(), t = new Stack<>();
    for (int i = 0; i < S.length(); i++) {
        char c = S.charAt(i);
        if(c == '#') {
            if(!s.isEmpty())
                s.pop();
        }else {
            s.push(c);
        }
    }

    for (int i = 0; i < T.length(); i++) {
        char c = T.charAt(i);
        if(c == '#') {
            if(!t.isEmpty())
                t.pop();
        }else {
            t.push(c);
        }
    }

    if(s.isEmpty() && t.isEmpty()) {
        return true;
    }

    if(s.size() != t.size()) {
        return false;
    }else {
        while (!s.isEmpty() && !t.isEmpty() && s.peek() == t.peek()) {
            s.pop();
            t.pop();
        }
        return s.isEmpty() && t.isEmpty();
    }

}
```
