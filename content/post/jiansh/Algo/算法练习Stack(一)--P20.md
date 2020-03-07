+++
title = "算法练习Stack(一)--P20"
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



[link on JianShu](https://www.jianshu.com/p/9fb3903fd0db)

[栈stack](https://leetcode.com/tag/stack/)

[valid parentheses ](https://leetcode.com/problems/valid-parentheses/)

Given a string containing just the characters '(', ')', '{', '}', '[' and ']', determine if the input string is valid.

An input string is valid if:
>Open brackets must be closed by the same type of brackets.
Open brackets must be closed in the correct order.
Note that an empty string is also considered valid.

先从最简单的开始练习:) 

基本操作：遇到开始的括号，如` (, {, [`进行压栈操作；遇到结束的括号，判断栈顶是否是匹配的括号`), }, ]`，如果不是或栈为空，则说明无效。最后判断栈是否为空即可。

利用了stack的特性，使用栈顶进行匹配动作。匹配就弹出，匹配就弹出……

```
public boolean isValid(String s) {

    Stack<Character> stack = new Stack<>();

    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        switch (c) {
            case '(':
                stack.push('(');
                break;
            case '{':
                stack.push('{');
                break;
            case '[':
                stack.push('[');
                break;
            case ')':
                if (stack.size() == 0 || stack.pop() != '(')
                    return false;
                break;
            case '}':
                if (stack.size() == 0 || stack.pop() != '{')
                    return false;
                break;
            case ']':
                if (stack.size() == 0 || stack.pop() != '[')
                    return false;
                break;
        }

    }

    return stack.isEmpty();
}
```

看了这里的[答案](https://leetcode.com/problems/valid-parentheses/discuss/9178/Short-java-solution)，代码逻辑优化，算法是一样的。但的确优美了一些——

```
public boolean isValidSimpler(String s) {
    Stack<Character> stack = new Stack<Character>();
    for (char c : s.toCharArray()) {
        if (c == '(')
            stack.push(')');
        else if (c == '{')
            stack.push('}');
        else if (c == '[')
            stack.push(']');
        else if (stack.isEmpty() || stack.pop() != c)
            return false;
    }
    return stack.isEmpty();
}
```
