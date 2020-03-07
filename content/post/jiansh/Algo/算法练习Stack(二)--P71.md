+++
title = "算法练习Stack(二)--P71"
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



[link on JianShu](https://www.jianshu.com/p/f5a38317dcda)

[栈stack](https://leetcode.com/tag/stack/)  
[71. simplify path](https://leetcode.com/problems/simplify-path/) Medium

Given an **absolute path** for a file (Unix-style), simplify it. Or in other words, convert it to the **canonical path**.

In a UNIX-style file system, a period `.` refers to the current directory. Furthermore, a double period `..` moves the directory up a level. For more information, see: [Absolute path vs relative path in Linux/Unix](https://www.linuxnix.com/abslute-path-vs-relative-path-in-linuxunix/)

Note that the returned canonical path must always begin with a slash `/`, and there must be only a single slash `/` between two directory names. The last directory name (if it exists) **must not** end with a trailing `/`. Also, the canonical path must be the **shortest** string representing the absolute path.

Example 1:
>Input: "/home/"
Output: "/home"

Explanation: Note that there is no trailing slash after the last directory name.

Example 2:
>Input: "/../"
Output: "/"

Explanation: Going one level up from the root directory is a no-op, as the root level is the highest level you can go.

Example 3:
>Input: "/home//foo/"
Output: "/home/foo"

Explanation: In the canonical path, multiple consecutive slashes are replaced by a single one.

Example 4:
>Input: "/a/./b/../../c/"
Output: "/c"

Example 5:
>Input: "/a/../../b/../c//.//"
Output: "/c"

Example 6:
>Input: "/a//b////c/d//././/.."
Output: "/a/b/c"

---

既然是stack相关的题目，肯定要用到stack。栈里存的应该是什么？如果存？如何取？

- 存的应该是最终的目录名称，
- 如果找到最终的目录名称：关键
- 最后从栈里取出来，生成完整的路径

关键点：  
1. 利用`/`对字符串进行分割：得到待处理的最小单位  
2. 处理三种特殊场景
    - 遇到 `..`时，需要出栈
    - 忽略`a/b///c`分割后长度为0的单位
    - 忽略 `.`表示当前路径

```
/*
Runtime: 6 ms, faster than 38.10% of Java online submissions for Simplify Path.
Memory Usage: 36.5 MB, less than 100.00% of Java online submissions for Simplify Path.
*/
public String simplifyPath(String path) {
    Stack<String> stack = new Stack<>();
    //以 / 分割字符串是关键。这样就构造出来了要处理的最小单位
    for(String cur: path.split("/")){
        // 特殊处理：需要出栈
        if(cur.equals("..")) {
            if(!stack.empty()) stack.pop();
        }
        // 两个条件
        // 1) length属性可以过滤重复的//字符串
        // 2) 要忽略的条件 .
        else if(cur.length()>0 && !cur.equals(".")) {
            stack.push(cur);
        }
    }
    return "/"+String.join("/",stack);
}
```

